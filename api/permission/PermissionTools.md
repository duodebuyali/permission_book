# 基于RxPermissions的权限申请工具类
首先我们需要明确工具类的用途：
* 判断单个权限是否已经被授权
* 申请单个或多个权限且返回申请的状态

其次我们需要确认我们的工具类的使用范围：
* 是否可以在`Activity`或`Fragment`使用
* 是否可以在其他组件中使用

最后根据这些这些条件我们来进行封装

## 单个权限是否被授予
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

## 单个权限的申请
申请权限，我们一般需要知道以下几种情况：
* 权限是否已被授予
* 权限未授予时，是否弹出了系统的权限申请框

这里我们需要一个对象[PermissionState](PermissionState.md)来描述这些属性。

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

## 多个权限的申请
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
