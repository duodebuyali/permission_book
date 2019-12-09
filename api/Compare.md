# 对各个框架的简单说明
通过对`摄像头权限`的申请例子，我们已经了解各个框架的简单用法，现在我们列举一些各个框架明显的优缺点。

## 原生权限申请
基于**Android原生API**的运行时权限申请

> 优点
+ 不需要再进行另外的lib引入，减少项目的包大小

> 缺点
+ 针对每个功能的权限申请，都需要设置相应的REQUEST_CODE，且需要在`onRequestPermissionsResult`中进行判断
+ 只能在Activity和Fragment中使用
+ 需要自己判断运行环境的版本
+ 需要自己针对部分国产ROM在低于**Android 6.0**时的权限申请进行判断
+ 需要自己对用户禁止权限申请弹框进行判断

## RxPermissions权限申请
基于**RxPermissions**的运行时权限申请

> 优点
+ 开发者不用担心Android运行环境的版本，如果系统是**Android 6.0**之前的版本，RxPermissions返回的结果是，app请求的每个权限都被允许（granted）
+ 不需要为每个功能的权限申请，都需要设置相应的REQUEST_CODE
+ 不需要重写`onRequestPermissionsResult`,且在其中进行授权成功与否判断
+ 具备Rx（RxJava）的特性，例如可以使用链式操作，可以执行filter操作，transform操作，等等
+ 对用户禁止权限申请弹框做了实现

> 缺点
+ 需要引入依赖
+ 初始化需要传入Activity参数

## AndPermission权限申请
基于**AndPermission**的运行时权限申请

> 优点
+ 开发者不用担心Android运行环境的版本，如果系统是**Android 6.0**之前的版本(非特殊ROM)，AndPermission返回的结果是，app请求的每个权限都被允许（granted）
+ 不需要为每个功能的权限申请，都需要设置相应的REQUEST_CODE
+ 不需要重写`onRequestPermissionsResult`,且在其中进行授权成功与否判断
+ 在任何地方都可以直接调用
+ 针对部分国产ROM在低于**Android 6.0**时的权限申请进行了判断
+ 对权限申请时的自定义对话框作了实现
+ 对跳转app对应权限设置页面作了实现
+ 对用户禁止权限申请弹框做了实现

> 缺点
+ 需要引入依赖
+ 做了自定义对话框的实现，所以包体积相对其他会变大，且需要自定义对话框时需要满足对应的规则

## EasyPermissions权限申请
基于**EasyPermissions**的运行时权限申请

> 优点
+ 开发者不用担心Android运行环境的版本，如果系统是**Android 6.0**之前的版本(非特殊ROM)，EasyPermissions返回的结果是，app请求的每个权限都被允许（granted）
+ 将权限请求的逻辑从Activity或者Fragment剥离出来做了一个代理操作
+ 对用户禁止权限申请弹框做了实现

> 缺点
+ 需要引入依赖
+ 每个功能的权限申请，都需要设置相应的REQUEST_CODE
+ 需要重写`onRequestPermissionsResult`
+ 做了自定义对话框的实现，所以包体积相对其他会变大，且需要自定义对话框时需要满足对应的规则
+ 获取对应的情况需要实现相应的接口

## PermissionsDispatcher权限申请
基于**PermissionsDispatcher**的运行时权限申请

> 优点
+ 开发者不用担心Android运行环境的版本，如果系统是**Android 6.0**之前的版本(非特殊ROM)，PermissionsDispatcher返回的结果是，app请求的每个权限都被允许（granted）
+ 对用户禁止权限申请弹框做了实现
+ 使用注解对权限申请中的各个情况进行了隔离，让开发者简单明了的处理各种状况

> 缺点
+ 需要引入依赖
+ 每个功能的权限申请，都需要设置相应的注解
+ 需要重写`onRequestPermissionsResult`,里面写上生成类的`ActivityNamePermissionsDispatcher.onRequestPermissionsResult`，此方法需要编译之后才会产生
+ 需要调用权限的地方执行`ActivityNamePermissionsDispatcher.XXXWithCheck()`，此方法需要编译之后才会产生


# 总结
通过上述的归纳，不难发现，如果需要实现更多系统的兼容，我们首选应该是**AndPermission**，且其实现了权限提示框的集成和跳转
对应权限的设置页面的功能。但是由于目前国内大部分手机都已经属于**Android 6.0**以上，所以对这部分特殊ROM的判断可以暂时不占
太大的权重；其次由于RxJava的优越性，基本大部分项目都会基于其开发；最后考虑到权限框架就应该只处理权限相关的问题，而不需要
实现对话框的提示，最终选择了**`RxPermissions`**。

下面我们对**`RxPermissions`**进行进一步的封装，以让它更好的适用于我们的项目。
