---
title: Android Status Bar and Navigation Bar Control
date: 2021-05-26 10:43
tags:
     - Android 系统开发
     - SystemUI
---

本次修改基于 Android 9.0。

## What?

公司 Android 系统定制，需要提供控制「状态栏」和「导航栏」的 API 。包括显示/隐藏状态栏和导航栏；屏蔽/开启状态栏下拉菜单；导航栏 back/home/recent 键的屏蔽/开启。

## Why?

这几个 API 是 Android 系统定制开发的基本需求，客户一般都会要求提供。因为像 POS、自助收银机、学习机、广告机、视频话机等终端，都需要做全屏、禁止退出这些功能。

## How?

### 状态栏和导航栏显隐控制

1. 初始化 visibility 和注册广播

   framework/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/StatusBar.java

   ```java
   protected void makeStatusBarView(){
       ...
   	if (MULTIUSER_DEBUG) {
       	mNotificationPanelDebugText = mNotificationPanel.findViewById(R.id.header_debug_info);	
       	mNotificationPanelDebugText.setVisibility(View.VISIBLE);
   	}
   
   	// ++ set statusbar visibility
   	boolean isShowStatusBar = 
       StatusBarMgr.instance().isStatusBarVisible();
   	if (mStatusBarWindow != null) {
       	mStatusBarWindow.setVisibility(isShowStatusBar ? View.VISIBLE : View.GONE);
   	}
   
   	// ++ set navigationbar visibility
   	boolean isShowNavBar = NavBarMgr.instance().isNavBarVisible();
   	try {
       	boolean showNav = isShowNavBar && 
   mWindowManagerService.hasNavigationBar();
       	if (DEBUG)
           	Log.v(TAG, "hasNavigationBar=" + showNav);
       	if (showNav) {
           	createNavigationBar();
       	}
   	} catch (RemoteException ex) {
       	// no window manager? good luck with that
   	}
       ...
   	// receive broadcasts
       IntentFilter filter = new IntentFilter();
       filter.addAction(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
       filter.addAction(Intent.ACTION_SCREEN_OFF);
       filter.addAction(DevicePolicyManager.ACTION_SHOW_DEVICE_MONITORING_DIALOG);
       // ++ add statusbar and navigationbar visibility change action.
       filter.addAction(StatusBarMgr.ACTION_STATUSBAR_VISIBILITY);
       filter.addAction(NavBarMgr.ACTION_NAVBAR_VISIBILITY);
       context.registerReceiverAsUser(mBroadcastReceiver, UserHandle.ALL, filter, null, null);
   	...
   }
   ```

2. 添加广播控制 visibility

   framework/base/packages/SystemUI/src/com/android/systemui/statusbar/phone/StatusBar.java

   ```java
   private final BroadcastReceiver mBroadcastReceiver = new BroadcastReceiver() {
   	@Override
   	public void onReceive(Context context, Intent intent) {
   		...
   		else if (DevicePolicyManager.ACTION_SHOW_DEVICE_MONITORING_DIALOG.equals(action)) {
   			mQSPanel.showDeviceMonitoringDialog();
   		} else if (StatusBarMgr.ACTION_STATUSBAR_VISIBILITY.equals(action)) {
               // ++ deal status bar visibility
   			String statusBarVisibility = intent.getStringExtra(StatusBarMgr.ACTION_KEY_STATUSBAR_VISIBILITY);
    			if (statusBarVisibility == null) {
   				return;
   			}
                boolean ret = "show".equals(statusBarVisibility);
                StatusBarMgr.instance().setStatusBarVisibility(ret);
                if (mStatusBarWindow != null) {
                    mStatusBarWindow.setVisibility(ret ? View.VISIBLE : View.GONE);
                }
   		} else if (NavBarMgr.ACTION_NAVBAR_VISIBILITY.equals(action)) {
               // ++ deal navigation bar visibility
            	Log.d(TAG, "nav action");
               String navBarVisibility = intent.getStringExtra(NavBarMgr.ACTION_KEY_NAVBAR_VISIBILITY);
               if (navBarVisibility == null) {
               	Log.e(TAG, "nav key string null");
               		return;
               }
               if ("show".equals(navBarVisibility)) {
               	Log.d(TAG, "nav show");
               	NavBarMgr.instance().setNavBarVisibility(true);
               	if (mNavigationBarView == null) {
               		 createNavigationBar();
   					return;
                   }
                	mNavigationBarView.setVisibility(View.VISIBLE);
   			} else if ("dismiss".equals(navBarVisibility)) {
                	Log.d(TAG, "nav dismiss");
                	NavBarMgr.instance().setNavBarVisibility(false);
                	if (mNavigationBarView == null) {
                		return;
                   }
                   WindowManagerGlobal.getInstance().removeView(mNavigationBarView.getRootView(), true);
                   mNavigationBarView = null;
                   // just change navigation bar icons' visibility, doesn't remove the bar.
                   // mNavigationBarView.setVisibility(View.GONE);
   			}
           ...
       }
   }
   ```

### 状态栏下拉控制

- framework\base\packages\SystemUI\src\com\android\statusbar\phone\PhoneStatusBarView.java

  ```java
  ...
  @override
  public boolean onTouchEvent(MotionEvent event) {
      // ++ 拦截状态栏下拉菜单
      if(!StatusBarMgr.instance().isDropMenuEnable()) {
          return false;
      }
      ...
  }
  ...
  ```

### 导航栏  back/home/recent 键控制

1. back/home  键

   framework\base\services\core\java\com\android\server\policy\PhoneWindowManager.java

   ```java
   ...
   /** {@inheritDoc} */
   @Override
   public int interceptKeyBeforeQueueing(KeyEvent event, int policyFlags) {
       ...
       // Handle special keys.
       switch (keycode) {
       	case KeyEvent.KEYCODE_BACK
               // ++ 拦截 back 键
               if(!NavBarMgr.instance().isBackEnable()) {
                   result = 0;
                   break;
               }
               ...  
       }
       ...
   }
   ...
   /** {@inheritDoc} */
   @Override
   public long interceptKeyBeforeDispatching(WindowState win, KeyEvent event, int policyFlags) {
       ...
       if(keycode == KeyEvent.KEYCODE_HOME) {
           // ++ 拦截 home 键
           if(!NavBarMgr.instance().isHomeEnable()) {
               return -1;
           }
           ...
       }
       ...
   }
   
   ```

   

2. recent 键

   framework\base\packages\SystemUI\src\com\android\statusbar\phone\NavigationBarFragment.java

   ```java
   ...
       private boolean onRecentsTouch(View v, MotionEvent event) {
       	// ++ 拦截 recent 键
       	if(!NavBarMgr.instance().isRecentEnable()) {
               return false;
           }
       	...
   	}
   	
   	private void onRecentsClick(View v) {
           // ++ 拦截 recent 键
           if(!NavBarMgr.instance().isRecentEnable()) {
               return;
           }
       }
   	...
        private boolean onLongPressRecents() {
           // ++ 拦截 recent 键
           if(!NavBarMgr.instance().isRecentEnable()) {
               return false;
           }
       }
   ...
   ```



### Framework 新增 API

添加状态栏、导航栏管理的 API 到 framework。

- frameworks\base\core\java\android\huoergai\PropertyMgr.java

```java
package com.huoergai.api;

import android.os.SystemProperties;

import java.util.ArrayList;
import java.util.List;

/**
 * framework/base/core/java/android/huoergai/PropertyMgr.java
 */
public class PropertyMgr {
 
    public static void setProp(String key, String value) {
        SystemProperties.set(key, value);
    }

    public static String getProp(String key, String def) {
        return SystemProperties.get(key, def);
    }

}
```

- frameworks\base\core\java\android\huoergai\StatusBarMgr.java

  ```java
  package com.huoergai.api;
  
  /**
   * @author huoergai
   */
  public class StatusBarMgr {
      private static StatusBarMgr INSTANCE;
      private static final String SYS_PROP_STATUS_BAR_DROP_MENU = "persist.sysui.drop.menu";
      private static final String SYS_PROP_STATUS_BAR_VISIBILITY = "persist.stsbar.visibility";
  
      public static final String ACTION_STATUSBAR_VISIBILITY = "huoergai.intent.action.STATUSBAR_VISIBILITY";
      public static final String ACTION_KEY_STATUSBAR_VISIBILITY = "statusbar_visibility";
  
      private StatusBarMgr() {
      }
  
      public static StatusBarMgr instance() {
          if (INSTANCE == null) {
              synchronized (StatusBarMgr.class) {
                  if (INSTANCE == null) {
                      INSTANCE = new StatusBarMgr();
                  }
              }
          }
          return INSTANCE;
      }
  
      public void setStatusBarVisibility(boolean visible) {
          PropertyMgr.setProp(SYS_PROP_STATUS_BAR_VISIBILITY, visible ? "show" : "dismiss");
      }
  
      public boolean isStatusBarVisible() {
          return "show".equals(PropertyMgr.getProp(SYS_PROP_STATUS_BAR_VISIBILITY, "show"));
      }
  
      public void setDropMenu(boolean enable) {
          PropertyMgr.setProp(SYS_PROP_STATUS_BAR_DROP_MENU, enable ? "on" : "off");
      }
  
      public boolean isDropMenuEnable() {
          return "on".equals(PropertyMgr.getProp(SYS_PROP_STATUS_BAR_DROP_MENU, "on"));
      }
  }
  
  ```

- frameworks\base\core\java\android\huoergai\NavBarMgr.java

  ```java
  package com.huoergai.api;
  
  import android.util.Log;
  
  /**
   * huoergai 2020/05/21 14:19
   * 导航栏管理类
   * framework/base/core/java/android/huoergai/NavBarMgr.java
   */
  public class NavBarMgr {
      private static final String TAG = "NavBarMgr";
  
      private static final String SYS_PROP_NAV_KEY_VISIBILITY = "persist.sysui.nav.visibility";
      private static final String SYS_PROP_NAV_KEY_BACK = "persist.sysui.nav.back";
      private static final String SYS_PROP_NAV_KEY_HOME = "persist.sysui.nav.home";
      private static final String SYS_PROP_NAV_KEY_RECENT = "persist.sysui.nav.recent";
  
      public static final String ACTION_NAVBAR_VISIBILITY = "huoergai.intent.action.NAVBAR_VISIBILITY";
      public static final String ACTION_KEY_NAVBAR_VISIBILITY = "navbar_visibility";
  
      private NavBarMgr() {
      }
  
      private static NavBarMgr INSTANCE;
  
      public static NavBarMgr instance() {
          if (INSTANCE == null) {
              synchronized (NavBarMgr.class) {
                  if (INSTANCE == null) {
                      INSTANCE = new NavBarMgr();
                  }
              }
          }
          return INSTANCE;
      }
  
      public void setNavBarVisibility(boolean visible) {
          Log.d(TAG, "set nav:" + visible);
          PropertyMgr.setProp(SYS_PROP_NAV_KEY_VISIBILITY, visible ? "show" : "dismiss");
      }
  
      public boolean isNavBarVisible() {
          return "show".equals(PropertyMgr.getProp(SYS_PROP_NAV_KEY_VISIBILITY, "show"));
      }
  
      public boolean isBackEnable() {
          String backStatus = PropertyMgr.getProp(SYS_PROP_NAV_KEY_BACK, "on");
          // Log.d(TAG, "back:" + backStatus);
          return "on".equals(backStatus);
      }
  
      public void setBackStatus(boolean enable) {
          // Log.d(TAG, "set back:" + enable);
          PropertyMgr.setProp(SYS_PROP_NAV_KEY_BACK, enable ? "on" : "off");
      }
  
      public boolean isHomeEnable() {
          String homeStatus = PropertyMgr.getProp(SYS_PROP_NAV_KEY_HOME, "on");
          // Log.d(TAG, "home:" + homeStatus);
          return "on".equals(homeStatus);
      }
  
      public void setHomeStatus(boolean enable) {
          // Log.d(TAG, "set home:" + enable);
          PropertyMgr.setProp(SYS_PROP_NAV_KEY_HOME, enable ? "on" : "off");
      }
  
      public boolean isRecentEnable() {
          String recentStatus = PropertyMgr.getProp(SYS_PROP_NAV_KEY_RECENT, "on");
          // Log.d(TAG, "recent:" + recentStatus);
          return "on".equals(recentStatus);
      }
  
      public void setRecentStatus(boolean enable) {
          // Log.d(TAG, "set recent:" + enable);
          PropertyMgr.setProp(SYS_PROP_NAV_KEY_RECENT, enable ? "on" : "off");
      }
  
  }
  ```

### 向外提供的 SDK



- StatusBarUtil.kt

  ```kotlin
  package com.huoergai.sdk

  import android.content.Context
  import android.content.Intent
  import com.huoergai.api.StatusBarMgr

  object StatusBarUtil {

    @JvmStatic
    fun isShowStatusBar(): Boolean {
        return StatusBarMgr.instance().isStatusBarVisible
    }

    @JvmStatic
    fun setStatusBarVisibility(ctx: Context, visible: Boolean) {
        val intent = Intent(StatusBarMgr.ACTION_STATUSBAR_VISIBILITY)
        if (visible) {
            intent.putExtra(StatusBarMgr.ACTION_KEY_STATUSBAR_VISIBILITY, "show")
        } else {
            intent.putExtra(StatusBarMgr.ACTION_KEY_STATUSBAR_VISIBILITY, "dismiss")
        }
        ctx.sendBroadcast(intent)
    }

    @JvmStatic
    fun disableDropMenu() {
        StatusBarMgr.instance().setDropMenu(false)
    }

    @JvmStatic
    fun enableDropMenu() {
        StatusBarMgr.instance().setDropMenu(true)
    }

    @JvmStatic
    fun isDropMenuEnable(): Boolean {
        return StatusBarMgr.instance().isDropMenuEnable
    }
  }
  ```
  
- NavBarUtil.kt

  ```kotlin
  package com.huoergai.sdk

  import android.content.Context
  import android.content.Intent
  import com.huoergai.api.NavBarMgr

  object NavBarUtil {

    @JvmStatic
    fun isShowNavBar(): Boolean {
        return NavBarMgr.instance().isNavBarVisible
    }

    @JvmStatic
    fun setNavBarVisibility(ctx: Context, visible: Boolean) {
        val intent = Intent(NavBarMgr.ACTION_NAVBAR_VISIBILITY)
        if (visible) {
            intent.putExtra(NavBarMgr.ACTION_KEY_NAVBAR_VISIBILITY, "show")
        } else {
            intent.putExtra(NavBarMgr.ACTION_KEY_NAVBAR_VISIBILITY, "dismiss")
        }
        ctx.sendBroadcast(intent)
    }

    /**
     * 屏蔽 back 键
     */
    @JvmStatic
    fun disableBack() {
        NavBarMgr.instance().setBackStatus(false)
    }

    /**
     * 开启 back 键
     */
    @JvmStatic
    fun enableBack() {
        NavBarMgr.instance().setBackStatus(true)
    }

    /**
     * 屏蔽 home 键
     */
    @JvmStatic
    fun disableHome() {
        NavBarMgr.instance().setHomeStatus(false)
    }

    /**
     * 开启 home 键
     */
    @JvmStatic
    fun enableHome() {
        NavBarMgr.instance().setHomeStatus(true)
    }

    /**
     * 屏蔽 recent 键
     */
    @JvmStatic
    fun disableRecent() {
        NavBarMgr.instance().setRecentStatus(false)
    }

    /**
     * 开启 recent 键
     */
    @JvmStatic
    fun enableRecent() {
        NavBarMgr.instance().setRecentStatus(true)
    }
  }
  ```