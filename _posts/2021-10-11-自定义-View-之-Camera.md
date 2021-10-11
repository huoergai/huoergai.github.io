---
title: 自定义 View 之 Camera
date: 2021-09-30 08:50
tags:
    - 自定义 View

---

> 此处特指 android.graphics 包下的 Camera

Camera 可以协助实现 3D 效果；因为除了可以做 x y 方向的变换，还可以做 z 轴方向的变化。

##  基础概念

### 3D 坐标系

> Camera 使用的 3 维**左手坐标系**；左手手臂指向 x 轴正方向，四指弯曲指向 y 轴正方向，大拇指指向 z 轴正方向。

**2D 和 3D 坐标是通过 Matrix 关联起来的，两者 y 轴方向不同**

| 坐标系   | 2D 坐标系 | 3D 坐标系    |
| -------- | --------- | ------------ |
| 原点位置 | 左上角    | 左上角       |
| x 轴方向 | 右        | 右           |
| y 轴方向 | 下        | 上           |
| z 轴方向 | N         | 垂直屏幕向内 |

### 三维投影

> **三维投影**：将三维空间中的点映射到二维平面上。

三维投影的两种方式：

- 正交投影：正视图、侧视图、俯视图
- 透视投影：从某个投射中心将物体投射到单一影面上所得到的图形。

### 摄像机

> 摄像机是模拟在 3D 空间中进行观察的眼睛。
>
> Android 上观察 View 的摄像机默认在屏幕左上角，与屏幕前方有一定距离的位置。
>
> 摄像机默认位置是 (0, 0, -576)，其中 -567 = -8 * 72，即是在距离屏幕 -8（-576 像素）的位置。
>
> Android 底层使用的 Skia 图像引擎，在 Skia 中，Camera 的位置单位是英寸，72 pixel/inch。

## Camera 方法

> android.graphics.Camera

### 基本方法

> save()/restore() 用于保存和恢复状态，一般成对使用。

```java
camera.save();
...
camera.restore();
```

### 常用方法

- getMatrix()

```java
// 计算当前状态下矩阵的状态，结果存入 matrix
void getMatrix (Matrix matrix)
```

- applyToCanvas()

```java
// 计算当前状态下对应的矩阵，并应用到指定的 Canvas。
void applyToCanvas (Canvas canvas)
```

### 平移方法

**沿 x 轴平移**

> 因为二维和三维坐标系的 x 轴坐标系同向，所以 Camera 和 Matrix 在 x 轴上平移是一致的。

```java
camera.translate(x, 0, 0);
matrix.postTranslate(x, 0);
```

**沿 y 轴平移**

> 因为二维和三维坐标系的 y 轴方向是相反的，所以 *camera.translate(0, -y, 0)* 与 *matrix.postTranslate(0, y)* 平移的方向和距离相同。

```java
camera.translate(0, -100, 0);
matrix.postTranslate(0, 100);
```

**沿 z 轴平移**

> 此时会以摄像机所在坐标 (x, y) 为中心进行缩放；当 View 与摄像机靠近时，View   变大，反之变小。

- 如果 View 中心和摄像机在同一直线上，平移时，View 只会呈现缩放效果。
- 如果 View 中心和摄像机不在同一直线上，平移时，View 除了呈现缩放效果，还会呈现位移效果。

### 旋转方法

> 旋转时 Camera 制作 3D 效果的核心。

```java
// 设置 View 同时绕 x 轴、y 轴和 z 轴旋转，可由下面单独设置的方法合成。
void rotate(float x, float y, float z);

// 设置 View 绕单个轴旋转
void rotateX(flaot deg);
void rotateY(float deg);
void rotateZ(flaot deg);
```

#### 旋转中心

> 旋转中心默认是坐标原点，即左上角的位置。
>
> 在 2D 中设置 旋转、缩放、错且等都能指定操作中心点，但在 3D 中没有。

**调节旋转中心点**

```kotlin
val matrix = Matrix()
camera.getMatrix(matrix)
// 调节中心点
matrix.preTranslate(-centerX, -centerY)
matrix.postTranslate(centerX, centerY)
```

#### 形变失真问题？

> 在低像素密度手机上，旋转效果正常，在高像素密度手机上，会显示夸张的效果； 表现为形变失真并且在中间因为形变过大导致无法显示。

**解决**

```kotlin
val scale = context.resources.displayMetrics.density

// 修复失真
val pos = FloatArray(9)
matrix.getValues(pos)
pos[6] /= scale
pos[7] /= scale
matrix.setValues(pos)
```

## 相机

**相机位置**

> 除了使用 translate、rotate 等来控制拍摄对象，也可以移动相机自身的位置，但这并不常用。

```java
// 设置相机的位置，默认（0，0，-8）
void setLocation(float x, float y, float z);

float getLocationX();
float getLocationY();
float getLocationZ();
```

物体之间的距离是相对的，让物体远离相机和让相机原理物体结果是一样的；设置相机位置的方法和平移的方法可以相互替代。

**要点**

- 相机和 View 的 z 轴距离不能为 0

  正如物体和相机重合时是无法拍摄的；

- 虚拟相机前后均可拍摄

  当 View 逐渐接近摄像机并越过时，仍能看到 View，View 大小与摄像机的绝对距离成反比；

- 摄像机右移等同于 View 左移

  View 的状态取决于其与摄像机的相对位置；因为单位不同，摄像机移动一个单位等于 View 移动72个像素；

```java
Camera camera1 = new Camera();
camera1.setLocation(1,0,-8);
Matrix matrix1 = new Matrix();
camera1.getMatrix(matrix1);
Log.d(TAG,"matrix1:" + matrix1.toShortString);

Camera camera2 = new Camera();
camera2.translate(-72,0,0);
Matrix matrix2 = new Matrix();
camera2.getMatrix(matrix2);
Log.d(TAG,"matrix2:" + matrix2.toShortString());
```