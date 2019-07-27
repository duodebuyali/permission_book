# AndPermission申请摄像头权限
这里使用[AndPermission](https://github.com/yanzhenjie/AndPermission)，进行**摄像头权限**的申请

## 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

## 第二步
在项目中依赖**AndPermission**，在项目的`build.gradle`中实现依赖
```gradle
implementation 'com.yanzhenjie:permission:2.0.0-rc4'
```

## 第三步
在我们需要在打开相机的位置添加申请权限的代码，如：
```java
 private void requestPermission() {
        AndPermission.with(this)
                .permission(Manifest.permission.CAMERA)
                // 准备方法，和 okhttp 的拦截器一样，在请求权限之前先运行改方法，已经拥有权限不会触发该方法
                .rationale((context, permissions, executor) -> {
                    // 此处可以选择显示提示弹窗
                    executor.execute();
                })
                // 用户给权限了
                .onGranted(permissions -> //do something)
                // 用户拒绝权限，包括不再显示权限弹窗也在此列
                .onDenied(permissions -> {
                    // 判断用户是不是不再显示权限弹窗了，若不再显示的话进入权限设置页
                    if (AndPermission.hasAlwaysDeniedPermission(MainActivity.this, permissions)) {
                        // 打开权限设置页
                        AndPermission.permissionSetting(MainActivity.this).execute();
                        return;
                    }
                })
                .start();
    }
```