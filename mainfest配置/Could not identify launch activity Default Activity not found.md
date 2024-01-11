# Could not identify launch activity: Default Activity not found

## Error while Launching activity

## Failed to launch an application on all devices

这个问题是因为在新建项目的时候，没有为程序配置入口

需要进行如下配置

标记activity为入口

```
		<intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
```



```
<application
    android:allowBackup="true"
    android:dataExtractionRules="@xml/data_extraction_rules"
    android:fullBackupContent="@xml/backup_rules"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/Theme.MyApplication"
    tools:targetApi="31">
    <activity
        android:launchMode="singleInstance"
        android:name=".MainActivity"
        android:exported="true"
        android:label="@string/title_activity_main"
        android:theme="@style/Theme.MyApplication"
        >
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
</application>
```