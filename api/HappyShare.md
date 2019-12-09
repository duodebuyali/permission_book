## AOS为什么需要运行时权限申请
由于**Google**在**Android-6.0[^1]**之后对app使用权限作了进一步限制，部分权限需要进行运行时申请，所以无可避免的我们的项目中都
会需要增加**运行时权限[^2]**的逻辑处理。

Android系统提供了对应的api给开发者来实现该逻辑，市面上已经存在基于此套逻辑封装的权限框架如：[RxPermissions](https://github.com/tbruyelle/RxPermissions)、[AndPermission](https://github.com/yanzhenjie/AndPermission)、
[EasyPermissions](https://github.com/googlesamples/easypermissions)和[PermissionsDispatcher](https://github.com/permissions-dispatcher/PermissionsDispatcher)等。如果我们从零开始来实现整套逻辑，势必会耗费我们大量的时间和精力，所以这里
我们选取**RxPermissions**，以申请 **摄像头权限** (Manifest.permission.CAMERA) 来对它的用法做一个简单说明，且在**RxPermissions**进一步进行封装，以更方便和更适用于目前的项目。

## RxPermissions申请摄像头权限
使用原始RxPermissions申请摄像头权限
### 第一步
在**AndroidManifest**中填写你要申请的权限
```java
  <uses-permission android:name="android.permission.CAMERA" />
```

### 第二步
在项目中依赖**RxPermissions**，在项目的`build.gradle`中实现依赖
```gradle
implementation 'com.github.tbruyelle:rxpermissions:0.10.2'
```

### 第三步
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

## 进一步封装的RxPermissions
进过上面的申请**摄像头权限**例子，我们已经可以简单知道**RxPermissions**的用法，但是还存在一些问题：
 * 无法在非Activity和Fragment中去使用
 * 无法一次就获取到申请的权限的各个状态(是否已被授权、是否申请时已经不再弹窗系统申请框等)

所以基于这些已知的问题，我们进行进一步的封装。

### 封装申请权限申请的状态
我们需要一个对象(PermissionState)，用来描述申请权限时的一些状态。
|   字段    |   类型   |  说明  |
|:--------:|:--------:|:--------:|
| name    |  String |   权限的名字,使用系统定义的字符串,</br>参考{@link android.Manifest.permission#CAMERA}  |
| explain   |    String   |     申请权限失败时，弹框给用户的说明    |
| isGranted     | boolean |   权限是否已经获取  |
| needRationale     | boolean |     是否权限申请失败并且用户关闭了申请权限时的提示      |

### 封装权限申请的调用
我们希望做到，申请权限可以在任何地方调用，可以申请一个或多个权限。

#### 单个权限是否被授予
一般来说该方法主要用来提示权限未被授予，所以直接返回状态即可。
```java
    /**
     * 判断单个权限是否已经授予
     * 不需要传递FragmentActivity对象，使得方法可以在任何地方调用
     */
    public static boolean rxGranted(String permission) {
        return rxGranted((FragmentActivity) AppManager.getInstance().currentActivity(), permission);
    }

    /**
     * 判断单个权限是否已经授予
     *
     * @param activity   RxPermissions需要用来实例化的FragmentActivity对象
     * @param permission 需要申请的权限，参考{@link android.Manifest.permission#CAMERA}
     */
    public static boolean rxGranted(FragmentActivity activity, String permission) {
        RxPermissions rxPermissions = new RxPermissions(activity);
        return rxPermissions.isGranted(permission);
    }
```

#### 单个权限的申请
申请权限，我们一般需要知道以下几种情况：
* 权限是否已被授予
* 权限未授予时，是否弹出了系统的权限申请框

```java

    /**
     * 运行时权限申请
     *
     * @param activity   RxPermissions需要用来实例化的FragmentActivity对象
     * @param permission 需要申请的权限数据，参考{@link android.Manifest.permission#CAMERA}
     */
    public static Observable<PermissionState> rxPermission(FragmentActivity activity, String permission) {
        PermissionState permissionState = new PermissionState(permission);
        return rxPermission(activity, permissionState);
    }

    /**
     * 运行时权限申请
     * 第一次请求权限时ActivityCompat.shouldShowRequestPermissionRationale=false;
     * 第一次请求权限被禁止，但未选择【不再提醒】ActivityCompat.shouldShowRequestPermissionRationale=true;
     * 允许某权限后ActivityCompat.shouldShowRequestPermissionRationale=false;
     * 禁止权限，并选中【禁止后不再询问】ActivityCompat.shouldShowRequestPermissionRationale=false；
     */
    @SuppressLint("CheckResult")
    public static Observable<PermissionState> rxPermission(FragmentActivity activity, final PermissionState permission) {
        RxPermissions rxPermissions = new RxPermissions(activity);
        return rxPermissions.request(permission.getName())
                .flatMap(aBoolean -> {
                    permission.setGranted(aBoolean);
                    if (!aBoolean) {
//                        return rxPermissions.shouldShowRequestPermissionRationale(activity, permission.getName());
                        return Observable.just(ActivityCompat.shouldShowRequestPermissionRationale(activity, permission.getName()));
                    } else {
                        return Observable.just(true);
                    }
                }).flatMap(it -> {
                    permission.setNeedRationale(!it);
                    return Observable.just(permission);
                });
    }
```

#### 多个权限的申请
多个权限的申请依赖于单个权限申请实现，且外部调用时，直接调用该方法即可。

> 只返回权限申请的最终结果

```java

    /**
     * 多个运行时权限申请，只会返回最终所有权限是否已经授予的状态
     *
     * @param activity    RxPermissions需要用来实例化的FragmentActivity对象
     * @param permissions 需要申请的权限数据数组，参考{@link android.Manifest.permission#CAMERA}
     */
    public static Observable<Boolean> rxPermissions(FragmentActivity activity, String... permissions) {
        RxPermissions rxPermissions = new RxPermissions(activity);
        return rxPermissions.request(permissions);
    }
```

> 返回每个权限申请的状态对象

```java

    /**
     * 多个运行时权限申请，每个权限都会返回对应的PermissionState对象
     */
    public static Observable<PermissionState> rxPermissionList(String... permissions) {
        return rxPermissionList((FragmentActivity) AppManager.getInstance().currentActivity(), permissions);
    }

    /**
     * 多个运行时权限申请，每个权限都会返回对应的PermissionState对象
     *
     * @param activity    RxPermissions需要用来实例化的FragmentActivity对象
     * @param permissions 需要申请的权限数据数组，参考{@link android.Manifest.permission#CAMERA}
     */
    public static Observable<PermissionState> rxPermissionList(FragmentActivity activity, String... permissions) {
        return Observable.fromArray(permissions)
                .flatMap((Function<String, ObservableSource<PermissionState>>) s -> rxPermission(activity, s));
    }
```


[^1]国产ROM部分手机在6.0以下也存在运行时权限申请，这里暂不讨论。

[^2]Runtime权限：运行时权限，是指在app运行过程中，赋予app的权限。这个过程中，会显示明显的权限授予界面，让用户决定是否授予权限。如果app的targetSdkVersion是22（Lollipop MR1）及以下，dangerous权限是安装时权限，否则dangerous权限是运行时权限。
   如果一个app的targetSdkVersion是23（或者23以上），那么该app所申请的所有dangerous权限都是运行时权限。如果运行在Android 6.0的环境中，该app在运行时必须主动申请这些dangerous权限（调用requestPermissions()），否则该app没有获取到dangerous权限。
