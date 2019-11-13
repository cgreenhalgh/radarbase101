# RADAR-BASE Passive App

i.e. the app that stream data from (local) bluetooth devices. 
See

- [wiki](https://radar-base.atlassian.net/wiki/spaces/RAD/pages/38010905/Passive+Remote+Monitoring+App)
- [source](https://github.com/RADAR-base/radar-prmt-android)

## Build

Built with gradle from 
[source](https://github.com/RADAR-base/radar-prmt-android).
See 
[README](https://github.com/RADAR-base/radar-prmt-android/blob/master/README.md)
for instructions.

Uses gradle.
[build.gradle](https://github.com/RADAR-base/radar-prmt-android/blob/master/build.gradle)
says `gradleVersion '5.6.2'`.

### Windows

Android Studio (3.5.2) ... ?

cygwin, 
```
git clone https://github.com/RADAR-base/radar-prmt-android.git
```

- Open project -> licenses not accepted for SDK build tools 29.0.2 & SDK Platform 28 
- (button) SDK Manager
- select Android SDK, SDK Platforms, API level 28 (Android 9.0 (Pie))
- ...and SDK Tools, Android SDK Build-Tools & Android SDK Platform-Tools
- Apply/OK (or maybe just apply would have been OK) --> accept license(s)

```
14:20	Gradle sync failed: Could not resolve all dependencies for configuration ':app:debugRuntimeClasspath'.
			Could not determine artifacts for org.radarbase:radar-android-faros:1.0.0-alpha1: Skipped due to earlier error
			Consult IDE log for more details (Help | Show Log) (9 m 12 s 799 ms)
```

remove all 'faros' refs (find).

See [empatica plugin](https://github.com/RADAR-base/radar-commons-android/tree/master/plugins/radar-android-empatica)
```
First, request an Empatica Connect developer account from Empatica's Developer Area. Download the Empatica Android SDK there. Copy downloaded AAR empalink-2.2.aar from the Empatica Android SDK package to the libs directory of your application.
```

make `app/libs/` in project and copy `empalink-2.2.aar` into it.

`Sync project with gradle files`

Set-up firebase - see 
[README](https://github.com/RADAR-base/radar-prmt-android#setup-firebase-remote-configuration)
and 
[android setup](https://firebase.google.com/docs/android/setup)

- sign in to firebase 
- create project ('RADAR passive app`)
- no google analytics ?!
- add android app, try their package name for now `org.radarcns.detail`
- download google-services.json
- copy to the app folder
- (note the SDK is already configured in the various build.gradle files)

Note, fallback config is in `app/src/main/res/xml/remote_config_defaults.xml`.
Change at least:
- `https://radar-cns-platform.rosalind.kcl.ac.uk/` -> `http://localhost:8080/`
- in `management_portal_url`, `kafka_rest_proxy_url`, `schema_registry_url`, 
`oauth2_authorize_url`, `oauth2_token_url`
- for dev set `unsafe_kafka_connection` to `true`

to set up config in the firebase console see 
[docs](https://firebase.google.com/docs/remote-config/use-config-android):
- in firebase console, your app,
- `Remote Config`

maybe, in `app/src/main/res/values/strings.xml` change `app_name`.

[see](https://stackoverflow.com/questions/51389533/program-type-already-present-android-support-v4-media-mediabrowsercompatcustom)
try adding
```
android.useAndroidX=true
android.enableJetifier=true
```
to `gradle.properties`

Run? (with an Android 9/Pie virtual Pixel 2)

Hmm, has to be HTTPS for server, even if self-signed...

## running

Note - see Source Type setting `Dynamic Source Registration`.

With `Dynamic Source Registration`
Pair subject with app -> creates source type ANDROID_PHONE_1.0.0

Without `Dynamic Source Registration`...
Make a `ANDROID_PHONE_1.0.0`, add to subject before registering pRMT => source row.

Either way, logging locally but fails upload...
```
...
2019-11-13 09:22:07.587 1067-1157/org.radarcns.detail I/pRMT:g: Broadcast config changed based on base URL https://128.243.22.74
2019-11-13 09:22:07.591 1067-1157/org.radarcns.detail I/pRMT:a: RADAR configuration changed: RadarConfiguration:
      start_at_boot: true
      radar_base_url: https://128.243.22.74
      kafka_rest_proxy_url: https://128.243.22.74/kafka/
      unsafe_kafka_connection: true
      kafka_clean_rate: 3600
      kafka_upload_rate: 60
      device_services_to_connect: .phone.PhoneSensorProvider .application.ApplicationServiceProvider .weather.WeatherApiProvider .phone.PhoneLocationProvider .phone.PhoneBluetoothProvider .phone.PhoneContactListProvider .phone.PhoneUsageProvider
      oauth2_token_url: https://128.243.22.74/managementportal/oauth/token
      management_portal_url: https://128.243.22.74/managementportal/
      schema_registry_url: https://128.243.22.74/schema/
      kafka_records_send_limit: 1000
      send_only_with_wifi: true
      sender_connection_timeout: 20
      ui_refresh_rate_millis: 250
      oauth2_authorize_url: https://128.243.22.74/managementportal/oauth/authorize
      privacy_policy: http://info.thehyve.nl/radar-cns-privacy-policy
      oauth2_redirect_url: org.radarcns.detail://oauth2/redirect
      oauth2_client_id: pRMT
...
2019-11-13 09:22:08.251 1067-1157/org.radarcns.detail I/pRMT:d: Requesting subject 143a246a-8c6e-47d9-9b95-2e7b76a98a79 with parseHeaders Authorization: Bearer 
...
2019-11-13 09:22:08.403 1067-1157/org.radarcns.detail D/pRMT:c: Parsing source type {"id":1103,"producer":"ANDROID","model":"PHONE","catalogVersion":"1.0.0","sourceTypeScope":"PASSIVE","canRegisterDynamically":false,"name":"ANDROID_PHONE","description":null,"assessmentType":null,"appProvider":null,"sourceData":[{"id":1161,"sourceDataType":"ACCELEROMETER","sourceDataName":"ANDROID_PHONE_1.0.0_ACCELEROMETER","frequency":"5.0","unit":"G","processingState":"RAW","dataClass":null,"keySchema":"org.radarcns.kafka.ObservationKey","valueSchema":"org.radarcns.passive.phone.PhoneAcceleration","topic":"android_phone_acceleration","provider":"org.radarcns.phone.PhoneSensorProvider","enabled":true,"sourceType":{"id":1103,"model":"PHONE","producer":"ANDROID","catalogVersion":"1.0.0"}},
...
[all types registered with study??]
...
2019-11-13 09:22:08.438 1067-1157/org.radarcns.detail I/pRMT:c: Sources from Management Portal: [SourceMetadata{type=SourceType{id='1103', producer='ANDROID', model='PHONE', catalogVersion='1.0.0', dynamicRegistration=false}, sourceId='e020f34b-ca8c-493b-b328-2ee7dafb6d51', sourceName='phone1', expectedSourceName='phone1', attributes={}'}]
2019-11-13 09:22:08.439 1067-1157/org.radarcns.detail I/pRMT:e: Refreshed JWT
2019-11-13 09:22:08.439 1067-1157/org.radarcns.detail I/pRMT:AuthService: Log in succeeded.
...
....
org.radarcns.android.auth.portal.ManagementPortalClient.privacyPolicyUrl=http://info.thehyve.nl/radar-cns-privacy-policy}, 
    sourceMetadata=[SourceMetadata{type=SourceType{id='1103', producer='ANDROID', model='PHONE', catalogVersion='1.0.0', dynamicRegistration=false}, sourceId='e020f34b-ca8c-493b-b328-2ee7dafb6d51', sourceName='phone1', expectedSourceName='phone1', attributes={}'}], 
...
2019-11-13 09:27:05.490 1067-1186/org.radarcns.detail I/pRMT:j: Writing 151 records to file in topic android_phone_gyroscope
2019-11-13 09:27:05.499 1067-1186/org.radarcns.detail I/pRMT:j: Writing 151 records to file in topic android_phone_acceleration
2019-11-13 09:27:11.762 1067-1195/org.radarcns.detail I/pRMT:a: Sender reconnected
2019-11-13 09:27:12.944 1067-1195/org.radarcns.detail D/pRMT:b: Uploading topics [android_phone_acceleration, android_phone_step_count, android_phone_user_interaction, android_phone_usage_event, android_phone_battery_level, android_phone_light, android_phone_gyroscope, android_phone_magnetic_field]
2019-11-13 09:27:12.945 1067-1195/org.radarcns.detail D/pRMT:j: Trying to retrieve records from topic a<android_phone_acceleration>
2019-11-13 09:27:13.122 1067-1195/org.radarcns.detail W/pRMT:f: android_phone_acceleration has FAILED uploading
2019-11-13 09:27:13.122 1067-1195/org.radarcns.detail W/pRMT:a: Sender is disconnected
    java.io.IOException: o.a.d.d.o: FAILED to transmit message (HTTP status code 404):
        0x020200004865303230663334622D636138632D343933622D623332382D326565376461666236643531D00F386DE73BD1F272D7416DE73BD1F272D741000000003E357F3F35A6A93D3885EB41D1F272D74185EB41D1F272D741000000003E357F3F35A6A93D3852B84ED1F272D74152B84ED1F272D741000000003E357F3F35A6A93D381F855BD1F272D7411F855BD1F272D741000000003E357F3F35A6A93D38759368D1F272D741759368D1F272D741000000003E357F3F35A6A93D38B81E75D1F272D741B81E75D1F272D741000000003E357F3F35A6A93D38E7FB81D1F272D741E7FB81D1F272D741000000003E357F3F35A6A93D3879E98ED1F272D74179E98ED1F272D741000000003E357F3F35A6A93D3846B69BD1F272D74146B69BD1F272D741000000003E357F3F35A6A93D387593A8D1F272D7417593A8D1F272D741000000003E357F3F35A6A93D38DF4FB5D1F272D741DF4FB5D1F272D741000000003E357F3F35A6A93D380E2DC2D1F272D7410E2DC2D1F272D741000000003E357F3F35A6A93D38DBF9CED1F272D741DBF9CED1F272D741000000003E357F3F35A6A93D383108DCD1F272D7413108DCD1F272D741000000003E357F3F35A6A93D3860E5E8D1F272D74160E5E8D1F272D741000000003E357F3F35A6A93D
        {"error":"not_found","error_description":"Topic android_phone_acceleration not present in Kafka."}
        at o.a.d.d.l.a(RestTopicSender.java:106)
        at o.a.a.k.b.a(KafkaDataSubmitter.kt:261)
        at o.a.a.k.b.b(KafkaDataSubmitter.kt:206)
        at o.a.a.k.b.a(KafkaDataSubmitter.kt:44)
        at o.a.a.k.b$g.a(KafkaDataSubmitter.kt:113)
        at o.a.a.k.b$g.a(KafkaDataSubmitter.kt:44)
        at org.radarbase.android.util.SafeHandler.f(SafeHandler.kt:70)
        at org.radarbase.android.util.SafeHandler.a(SafeHandler.kt:11)
        at org.radarbase.android.util.SafeHandler$i.a(SafeHandler.kt:126)
        at org.radarbase.android.util.SafeHandler$i.a(SafeHandler.kt:11)
        at org.radarbase.android.util.SafeHandler.f(SafeHandler.kt:70)
        at org.radarbase.android.util.SafeHandler.a(SafeHandler.kt:11)
        at org.radarbase.android.util.SafeHandler$f.run(SafeHandler.kt:109)
        at android.os.Handler.handleCallback(Handler.java:873)
        at android.os.Handler.dispatchMessage(Handler.java:99)
        at android.os.Looper.loop(Looper.java:193)
        at android.os.HandlerThread.run(HandlerThread.java:65)
     Caused by: o.a.d.d.o: FAILED to transmit message (HTTP status code 404):
        0x020200004865303230663334622D636138632D343933622D623332382D326565376461666236643531D00F386DE73BD1F272D7416DE73BD1F272D741000000003E357F3F35A6A93D3885EB41D1F272D74185EB41D1F272D741000000003E357F3F35A6A93D3852B84ED1F272D74152B84ED1F272D741000000003E357F3F35A6A93D381F855BD1F272D7411F855BD1F272D741000000003E357F3F35A6A93D38759368D1F272D741759368D1F272D741000000003E357F3F35A6A93D38B81E75D1F272D741B81E75D1F272D741000000003E357F3F35A6A93D38E7FB81D1F272D741E7FB81D1F272D741000000003E357F3F35A6A93D3879E98ED1F272D74179E98ED1F272D741000000003E357F3F35A6A93D3846B69BD1F272D74146B69BD1F272D741000000003E357F3F35A6A93D387593A8D1F272D7417593A8D1F272D741000000003E357F3F35A6A93D38DF4FB5D1F272D741DF4FB5D1F272D741000000003E357F3F35A6A93D380E2DC2D1F272D7410E2DC2D1F272D741000000003E357F3F35A6A93D38DBF9CED1F272D741DBF9CED1F272D741000000003E357F3F35A6A93D383108DCD1F272D7413108DCD1F272D741000000003E357F3F35A6A93D3860E5E8D1F272D74160E5E8D1F272D741000000003E357F3F35A6A93D
        {"error":"not_found","error_description":"Topic android_phone_acceleration not present in Kafka."}
        at o.a.d.d.l.a(RestTopicSender.java:99)
         ... 16 more
```

nginx config??
