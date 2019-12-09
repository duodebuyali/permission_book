# RxPermissions申请摄像头权限
这里使用[PermissionsDispatcher GITHUB](https://github.com/permissions-dispatcher/PermissionsDispatcher)，进行**摄像头权限**的申请

## 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

## 第二步
在项目中依赖**PermissionsDispatcher**，在项目的`build.gradle`中实现依赖
```gradle
    implementation "org.permissionsdispatcher:permissionsdispatcher:${latest.version}"
    annotationProcessor "org.permissionsdispatcher:permissionsdispatcher-processor:${latest.version}"
```

## 第三步
在我们需要在打开相机的Activity实现如下的代码：
```java
@RuntimePermissions
public class MainActivity extends AppCompatActivity {

    @NeedsPermission(Manifest.permission.CAMERA)
    void showCamera() {
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.sample_content_fragment, CameraPreviewFragment.newInstance())
                .addToBackStack("camera")
                .commitAllowingStateLoss();
    }

    @OnShowRationale(Manifest.permission.CAMERA)
    void showRationaleForCamera(final PermissionRequest request) {
        new AlertDialog.Builder(this)
            .setMessage(R.string.permission_camera_rationale)
            .setPositiveButton(R.string.button_allow, (dialog, button) -> request.proceed())
            .setNegativeButton(R.string.button_deny, (dialog, button) -> request.cancel())
            .show();
    }

    @OnPermissionDenied(Manifest.permission.CAMERA)
    void showDeniedForCamera() {
        Toast.makeText(this, R.string.permission_camera_denied, Toast.LENGTH_SHORT).show();
    }

    @OnNeverAskAgain(Manifest.permission.CAMERA)
    void showNeverAskForCamera() {
        Toast.makeText(this, R.string.permission_camera_neverask, Toast.LENGTH_SHORT).show();
    }
}
```

## 第四步
在我们需要在该Activity实现如下的代码，以代理权限请求的返回状态：
```java
 @Override
 protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main);
     findViewById(R.id.button_camera).setOnClickListener(v -> {
       // NOTE: delegate the permission handling to generated method
       MainActivityPermissionsDispatcher.showCameraWithPermissionCheck(this);
     });
 }
 
 @Override
 public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
     super.onRequestPermissionsResult(requestCode, permissions, grantResults);
     // NOTE: delegate the permission handling to generated method
     MainActivityPermissionsDispatcher.onRequestPermissionsResult(this, requestCode, grantResults);
 }
```

## 使用须知:
被权限注解的方法不能用private修饰,详情看下图：
> ![introduction](../../res/drawable/permissions_dispatcher_introductions.png)