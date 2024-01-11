# Android明文通信报错

# CLEARTEXT communication

#### 解决方式

#### 1.可以将ip等信息保存在一个

##### config.xml中，读取这个值来获取IP信息

#### 2.可以配置安全性规则

**创建网络安全配置文件**：

在你的Android应用的 `res/xml/` 目录下创建一个名为 `network_security_config.xml` 的网络安全配置文件（如果该目录不存在，可以手动创建）。在这个文件中，你可以指定哪些域名或IP地址允许使用明文通信。

示例 `network_security_config.xml` 文件内容如下：

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

#### **在 AndroidManifest.xml 中引用网络安全配置**：

```
<application
    android:networkSecurityConfig="@xml/network_security_config">
```

