# Android原生申请摄像头权限
这里使用Android原生[api](http://androidxref.com/6.0.0_r1/xref/developers/build/prebuilts/gradle/RuntimePermissions/)，进行**摄像头权限**的申请

## 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

## 第二步
在我们需要在打开相机的位置添加申请权限的代码，如：
```java
 public void requestCamera() {
        // checkSelfPermission 判断是否已经申请了此权限
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED) {
            //申请对应权限
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CAMERA}, 1);
        } else {
            //do something
        }
    }
```

## 第三步
在onRequestPermissionsResult回调方法中去进行处理。代码如下:
```java
@Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == 1) {//code需要是之前申请权限时使用的值
            for (int i = 0; i < permissions.length; i++) {
                if (grantResults[i] == PackageManager.PERMISSION_GRANTED) {
                    //申请成功,do something
                } else {
                    //申请失败，判断是否被禁止了弹窗提示
                    if (!ActivityCompat.shouldShowRequestPermissionRationale(this,
                            Manifest.permission.CAMERA)) {
                        //弹出设置框，跳转系统对应的权限设置

                    }
                }
            }
        }
    }
```