# Unable to add window android.view.ViewRootImpl$W@deb9492 -- permission denied for window

需要添加悬浮显示权限

```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
```

动态权限请求

```
if (!Settings.canDrawOverlays(context)) {
    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + context.getPackageName()));
    context.startActivity(intent);
}
```