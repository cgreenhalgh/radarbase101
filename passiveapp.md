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
