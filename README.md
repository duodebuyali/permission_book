# 前言
由于**Google**在**Android-6.0**之后对app使用权限作了进一步限制，部分权限需要进行运行时申请，所以无可避免的我们的项目中都
会需要增加[运行时权限][1]的逻辑处理。

Android系统提供了对应的api给开发者来实现该逻辑，但是如果我们从零开始来实现整套逻辑，势必会耗费我们大量的时间和精力。且目
前已经存在很多成熟的运行时权限框架如：[RxPermissions](api/tools/RxPermissions.md)、[AndPermission](api/tools/AndPermission.md)、
[EasyPermissions](api/tools/EasyPermissions.md)和[PermissionsDispatcher](api/tools/PermissionsDispatcher.md)等。

这里我会以申请 **摄像头权限** (Manifest.permission.CAMERA) 来对这些列举的框架的用法和优点进行简单说明，且针对最终采取的
框架进行进一步的封装，以更方便和更适用于目前的项目。

[1]Runtime权限：运行时权限，是指在app运行过程中，赋予app的权限。这个过程中，会显示明显的权限授予界面，让用户决定是否授予权限。如果app的targetSdkVersion是22（Lollipop MR1）及以下，dangerous权限是安装时权限，否则dangerous权限是运行时权限。
   如果一个app的targetSdkVersion是23（或者23以上），那么该app所申请的所有dangerous权限都是运行时权限。如果运行在Android 6.0的环境中，该app在运行时必须主动申请这些dangerous权限（调用requestPermissions()），否则该app没有获取到dangerous权限。
