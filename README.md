# CloseWindow
android桌面息屏小工具
长按Android桌面调用此工具

## 相关code
继承DeviceAdminReceiver的空实现
```
open class AdminReceiver: DeviceAdminReceiver()
```
清单文件里注册
```
        <receiver
            android:name=".custom.AdminReceiver"
            android:permission="android.permission.BIND_DEVICE_ADMIN">
        <meta-data
            android:name="android.app.device_admin"
            android:resource="@xml/admin_manager" />
        <intent-filter>
            <action android:name="android.app.action.DEVICE_ADMIN_ENABLED" />
        </intent-filter>
        </receiver>
```
xml目录下添加resource文件 admin_manager
```
<?xml version="1.0" encoding="utf-8"?>
<device-admin xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-policies>
        <!--锁屏 -->
        <force-lock />
        <!--限制密码类型-->
        <!--        <limit-password />-->
        <!-- 监控屏幕解锁尝试次数 -->
        <watch-login />
        <!-- 重置密码-->
        <!--        <reset-password />-->
        <!--恢复出厂设置-->
        <!--        <wipe-data />-->
        <!-- 设置锁定屏幕密码的有效期 -->
        <!--        <expire-password />-->
        <!-- 设置存储设备加密 -->
        <!--        <encrypted-storage />-->
        <!-- 停用相机 -->
        <!--        <disable-camera />-->
    </uses-policies>
</device-admin>
```
开始使用
```
            val systemService = getSystemService(DEVICE_POLICY_SERVICE) as DevicePolicyManager
            if (requestLockAdmins(systemService, ComponentName(this, AdminReceiver::class.java))) {
                systemService.lockNow();
            }
```
相关方法,下边主要是跳转到系统页面获取权限的。至于回调自己处理下
```
    private fun requestLockAdmins(systemService: DevicePolicyManager?, adminReceiver: ComponentName): Boolean {
        if (systemService == null) return false
        //检查是否已经获取设备管理权限
        val active = systemService.isAdminActive(adminReceiver)
        if (!active) {
            //打开DevicePolicyManager管理器，授权页面
            val intent = Intent()
            //授权页面Action -->  DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN
            intent.setAction(DevicePolicyManager.ACTION_ADD_DEVICE_ADMIN)
            //设置DEVICE_ADMIN，告诉系统申请管理者权限的Component/DeviceAdminReceiver
            intent.putExtra(DevicePolicyManager.EXTRA_DEVICE_ADMIN, adminReceiver)
            //设置 提示语--可不添加
            intent.putExtra(DevicePolicyManager.EXTRA_ADD_EXPLANATION, "DevicePolicyManager涉及的管理权限,一次性激活!")
            startActivityForResult(intent, 222)//回调自己处理下
        }
        return active
    }
```
