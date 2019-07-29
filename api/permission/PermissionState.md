# 描述Android权限的状态
包含Android权限的一些状态和状态的获取以及设置方法。
其中设置方法权限为Default(防止可以在工具类之外设置)，获取方法权限为public。

## 属性

|   字段    |   类型   |  说明  |
|:--------:|:--------:|:--------:|
| name    |  String |   权限的名字,使用系统定义的字符串,</br>参考{@link android.Manifest.permission#CAMERA}  |
| explain   |    String   |     申请权限失败时，弹框给用户的说明    |
| isGranted     | boolean |   权限是否已经获取  |
| needRationale     | boolean |     是否权限申请失败并且用户关闭了申请权限时的提示      |

## 代码如下
```java
/**
 * @Description:用来描述对应权限的状态
 * @Author: hekang
 * @CreateDate: 2019/6/26 16:09
 */
public class PermissionState implements Parcelable {

    /**
     * 权限的名字,使用系统定义的字符串
     * 参考{@link android.Manifest.permission#CAMERA}
     */
    private String name;

    /**
     * 申请权限失败时，弹框给用户的说明
     */
    private String explain;

    /**
     * 权限是否已经获取
     */
    private boolean isGranted;

    /**
     * 权限申请失败并且用户关闭了申请权限时的提示
     */
    private boolean needRationale;

    public PermissionState(String name) {
        this(name, "");
    }

    public PermissionState(String name, String explain) {
        this.name = name;
        this.explain = explain;
    }

    public String getName() {
        return name;
    }

    void setName(String name) {
        this.name = name;
    }

    public String getExplain() {
        return explain;
    }

    void setExplain(String explain) {
        this.explain = explain;
    }

    public boolean isGranted() {
        return isGranted;
    }

    void setGranted(boolean granted) {
        isGranted = granted;
    }

    public boolean isNeedRationale() {
        return needRationale;
    }

    void setNeedRationale(boolean needRationale) {
        this.needRationale = needRationale;
    }

    @Override
    public String toString() {
        return "PermissionState{" +
                "name='" + name + '\'' +
                ", explain='" + explain + '\'' +
                ", isGranted=" + isGranted +
                ", needRationale=" + needRationale +
                '}';
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.name);
        dest.writeString(this.explain);
        dest.writeByte(this.isGranted ? (byte) 1 : (byte) 0);
        dest.writeByte(this.needRationale ? (byte) 1 : (byte) 0);
    }

    protected PermissionState(Parcel in) {
        this.name = in.readString();
        this.explain = in.readString();
        this.isGranted = in.readByte() != 0;
        this.needRationale = in.readByte() != 0;
    }

    public static final Parcelable.Creator<PermissionState> CREATOR = new Parcelable.Creator<PermissionState>() {
        @Override
        public PermissionState createFromParcel(Parcel source) {
            return new PermissionState(source);
        }

        @Override
        public PermissionState[] newArray(int size) {
            return new PermissionState[size];
        }
    };
}

```