# RxPermissions申请摄像头权限
这里使用[RxPermissions GITHUB](https://github.com/tbruyelle/RxPermissions)，进行**摄像头权限**的申请

## 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

## 第二步
在项目中依赖**RxPermissions**，在项目的`build.gradle`中实现依赖
```gradle
implementation 'com.github.tbruyelle:rxpermissions:0.10.2'
```

## 第三步
在我们需要在打开相机的位置添加申请权限的代码，如：
```java
public void rxRequestCamera() {
        RxPermissions rxPermissions = new RxPermissions(this);
        rxPermissions.request(Manifest.permission.CAMERA).subscribe(new DisposableObserver<Boolean>() {
            @Override
            public void onNext(Boolean aBoolean) {
                if (aBoolean) {
                    // do something
                } else {
                    // 权限被拒绝
                    //申请失败，判断是否被禁止了弹窗提示
                    if (!ActivityCompat.shouldShowRequestPermissionRationale(SettingActivity.this,
                            Manifest.permission.CAMERA)) {
                        //弹出设置框，跳转系统对应的权限设置

                    }
                }
            }

            @Override
            public void onError(Throwable e) {
                // do something
            }

            @Override
            public void onComplete() {

            }
        });
    }
```