# AndroidManifest.xml
- 앱을 시작할 때 MainActivity로 가도록 하는 코드
```xml
<activity
    android:name=".MainActivity"
    android:exported="true" >
<intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
</activity>
```
