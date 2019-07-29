# AndPermission申请摄像头权限
这里使用[EasyPermissions GITHUB](https://github.com/googlesamples/easypermissions)，进行**摄像头权限**的申请

## 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

## 第二步
在项目中依赖**EasyPermissions**，在项目的`build.gradle`中实现依赖
```gradle
implementation 'pub.devrel:easypermissions:2.0.0'
```

## 第三步
在我们需要在打开相机的位置添加申请权限的代码，如：
```java
//参数为请求权限的code
    @AfterPermissionGranted(1)
    private void methodRequiresPermission() {
        if (EasyPermissions.hasPermissions(this, Manifest.permission.CAMERA)) {
            // Already have permission, do the thing
            // ...
        } else {
            // Do not have permissions, request them now
            //这个方法是用户在拒绝权限之后，再次申请权限，才会弹出自定义的dialog，详情可以查看下源码 shouldShowRequestPermissionRationale()方法
            PermissionRequest request = new PermissionRequest.Builder(SettingActivity.this, 1, Manifest.permission.CAMERA)
                    .setRationale(R.string.camera)
                    .setPositiveButtonText(R.string.yes)
                    .setNegativeButtonText(R.string.no)
                    .build();
            EasyPermissions.requestPermissions(request);
        }
    }
```

## 第四步
在onRequestPermissionsResult回调方法中去进行处理。代码如下:
```java
//接受系统权限的处理，这里交给EasyPermissions来处理，回调到 EasyPermissions.PermissionCallbacks接口
      @Override
      public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
            EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults,this)//注意这个this，内部对实现该方法进行了查询，所以没有this的话，回调结果的方法不生效
       }
```

## 第五步
实现 EasyPermissions.RationaleCallbacks,重写以下两个方法 (弹出自定义dialog时，用户点击接受和拒绝按钮)
```java
    @Override
    public void onRationaleDenied(requestCode: Int) {
     //如果用户点击永远禁止，这个时候就需要跳到系统设置页面去手动打开了
     if (EasyPermissions.somePermissionPermanentlyDenied(this, perms)) {
         AppSettingsDialog.Builder(this).build().show()
      }
        
    }
    
    @Override
    public void onRationaleAccepted(requestCode: Int) {
         //do something
    }
   
```