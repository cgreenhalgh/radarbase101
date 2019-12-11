# RADAR-BASE Active App

notes on the "active" i.e. questionnaire app.

- [wiki](https://radar-base.atlassian.net/wiki/spaces/RAD/pages/38043668/Active+Questionnaire+App+Overview)

- [Questionnaire App source](https://github.com/RADAR-base/RADAR-Questionnaire)

- [Questionnaire Definitions](https://github.com/RADAR-base/RADAR-REDCap-aRMT-Definitions)

- [Notification Schedule | Protocol](https://github.com/RADAR-base/RADAR-aRMT-protocols)

## Firebase

Is this current?

[wiki notes](https://radar-base.atlassian.net/wiki/spaces/RAD/pages/587464705/Firebase+Cloud+Messaging+Notifications+Server+XMPP)

[XMPP server](https://github.com/RADAR-base/fcmxmppserverv2)

## Build

NB need more than 10GB disk space in VM. 
[e.g.](https://technology.amis.nl/2017/01/30/ubuntu-vm-virtualbox-increase-size-disk-make-smaller-exports-distribution/)

See [app readme](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/README.md)

Requires node.js & yarn.

Having problems building so having a go at mirroring travis CI build...

## Travis CI-like build

From [.travis.yml](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/.travis.yml)
it is expecting node version 11 (current stable is 12.13.0), using nvm,
yarn version 1.12.3 (current stable is 1.19.1), 
android components android-28 (android 9.0?) & android-build-tools 28.0.3 & extra,
also uses gem.

[Travis](https://docs.travis-ci.com/user/reference/linux/)
says Ubuntu Xenial is current default linux, but 
[android](https://docs.travis-ci.com/user/languages/android/) says 
android only works with trusty. 
But the provided .travis.yml doesn't specify a distro...

[Travis language android](https://docs.travis-ci.com/user/languages/android/)
says following are also installed by default:
- tools
- platform-tools
- build-tools-25.0.2
- android-25
- extra-google-google_play_services
- extra-google-m2repository
- extra-android-m2repository

Also looks like mvn, ant, gradle will be available.

prep:
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```
from .travis:
```
nvm install 11
```
log out, back in (for nvm env).
```
curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version 1.12.3
export PATH="$HOME/.yarn/bin:$PATH"
```
Ruby for gem - nb needs at least ruby 2.4.0, so not default xenial ruby-full
```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
~/.rbenv/bin/rbenv init
```
logout/in (what version of ruby?? - current stable 2.6.5)
```
mkdir -p "$(rbenv root)"/plugins
git clone https://github.com/rbenv/ruby-build.git "$(rbenv root)"/plugins/ruby-build
sudo apt install -y git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
rbenv install 2.6.5
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
eval "$(rbenv init -)"
rbenv global 2.6.5
```
carry on...
```
gem install bundler --pre
```
Need source now...
```
git clone https://github.com/RADAR-base/RADAR-Questionnaire.git
cd RADAR-Questionnaire
```
.travis again:
```
bundle install
cd
yarn global add ionic cordova
```

Java...
```
sudo apt install -y openjdk-8-jdk-headless 
```

Android pre-reqs...

"The sdkmanager tool is provided in the Android SDK Tools package (25.2.3 and higher) and is located in android_sdk/tools/bin/" [docs](https://developer.android.com/studio/command-line/sdkmanager.html)

Download the "command line tools only", "Linux" from [here](https://developer.android.com/studio#downloads).
e.g. [url](https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip) 
```
cd
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
unzip sdk-tools-linux-4333796.zip
```
.travis expects android-28 (android 9.0?) & android-build-tools 28.0.3 & extras (?!); and defaults:
- build-tools-25.0.2
- android-25
- extra-google-google_play_services
- extra-google-m2repository
- extra-android-m2repository
```
tools/bin/sdkmanager "build-tools;28.0.3" "platforms;android-28" \
 "extras;google;google_play_services" "extras;google;m2repository" \
 "extras;android;m2repository"
```
(accept)
```
echo "export PATH=${PATH}:/home/vagrant/tools/bin" >> ~/.bashrc
```

from .travis:
```
cd RADAR-Questionnaire
mkdir www
#cp secret.ts src/assets/data/secret.ts
cordova prepare android
yarn install
```

### Secrets

Secrets?
- secret.ts
- google-services.json
- service-api.json
- radar-armt-release-key.keystore

For google-services.json see 
[firebase setup](https://github.com/RADAR-base/RADAR-Questionnaire#firebase)
i.e. create app, download google-services.json to top-level.

For secret.ts see
[other config](https://github.com/RADAR-base/RADAR-Questionnaire#other-config-options)
i.e. Copy src/assets/data/secret.ts.template to src/assets/data/secret.ts and fill in
value for `DefaultSourceProducerAndSecretExport` 'aRMT:<aRMT-secret>'

Note, value is set in the management portal, oauth clients. 
Ensure dynamic registration also set ('RADAR', 'aRMT-App', '1.4.3')

Note firebase can set
- `protocol_base_url` (default to value in defaultConfig)
- `oauth_client_secret`  (default to value in secret.ts)

in src/assets/data/defaultConfig.ts can set
- DefaultEndPoint 
- DefaultProtocolGithubRepo - use your own repo
- DefaultSchemaGithubRepo 
- FCMPluginProjectSenderId 

Firebase Cloud Messaging ... needs server-side support?!

### web build

from [readme](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/README.md),
```
cordova prepare
TERM=; ionic serve --address=0.0.0.0
```
NOTE: workaround for ionic thinking it is a windows machine if TERM=cygwin, i.e. i am
using a cygwin terminal to SSH to the VM, but still...
(otherwise `ionic info` doesn't find cordova cli or app-scripts etc)

(Note, ports 8100, 3572 & 53703)

### android build

NB needs gradle installed.
Download, e.g. [6.0.1](https://gradle.org/next-steps/?version=6.0.1&format=bin)
```
cd
sudo mkdir /opt/gradle
sudo unzip -d /opt/gradle gradle-6.0.1-bin.zip
echo 'export PATH=$PATH:/opt/gradle/gradle-6.0.1/bin' >> ~/.bashrc
```
(`gradle -v`)

From .travis - build
```
echo 'export ANDROID_HOME=~' >> ~/.bashrc
TERM=ansi; fastlane android build_debug
```
breaking on 
```
TERM=ansi;ionic cordova compile android --no-interactive --debug --device -- -- --cordovaNoFetch=false
```

### ...

[yarn.lock](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/yarn.lock)
suggests ionic 5.4.2, cordova-android 8.1.0, cordova-common 3.1.0.


### Android

"The sdkmanager tool is provided in the Android SDK Tools package (25.2.3 and higher) and is located in android_sdk/tools/bin/" [docs](https://developer.android.com/studio/command-line/sdkmanager.html)

Download the "command line tools only", "Linux" from [here](https://developer.android.com/studio#downloads).
e.g. [url](https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip) 

```
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip
sudo apt install -y unzip
unzip sdk-tools-linux-4333796.zip
```
-> tools (currently 26.1.1, sept. 2017?!)
```
tools/bin/sdkmanager --list
```

.travis expects android-28 (android 9.0?) & android-build-tools 28.0.3; try latest build tools, i.e.
```
tools/bin/sdkmanager "build-tools;29.0.2" "platforms;android-28"
```
(accept)
```
echo "export PATH=${PATH}:/home/vagrant/tools/bin" >> ~/.bashrc

## changes

Won't access an IP address - why does there have to be a short word at the end of the host name?
```
export const URLRegEx = '(https?://)?([\\da-z.-]+)\\.([a-z.]{2,6})[/\\w .-]*/?'
```
->
```
export const URLRegEx = '(https?://)?([\\da-z.-]+)(\\.([a-z.]{2,6}))?[/\\w .-]*/?'
```

Ignore SSL certificate error?

For browser, install certificate extracted from webserver container, 
/etc/letsencrypt/live/localhost/fullchain.pem, and install as trusted root.

For Android,
[see this](http://ivancevich.me/articles/ignoring-invalid-ssl-certificates-on-cordova-android-ios/)

but note, seems to actually be in `./platforms/android/CordovaLib/src/org/apache/cordova/engine/SystemWebViewClient.java`

Enabled by default in debug build. Otherwise edit.

## protocols

For project 'test1' attempts to fetch 
`https://api.github.com/repos/RADAR-Base/RADAR-aRMT-protocols/contents/test1/protocol.json?ref=master`

Clone `https://github.com/RADAR-Base/RADAR-aRMT-protocols` 
and change `DefaultProtocolGithubRepo` in `src/assets/data/defaultConfig.ts`
to your user/repo name.

E.g. copy `aRMT-TEST/protocol.json` to your new project directory.

## App problems

Won't upload results,
```
2VM1058:1 GET https://128.243.22.74/schema/subjects/questionnaire_app_event-value/versions/latest 404
...
(index):1 Access to XMLHttpRequest at 'https://128.243.22.74/schema/subjects/questionnaire_app_event-value/versions/latest' from origin 'http://128.243.22.74:8100' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```
and various other schema URLs

Add cors to schema in nginx, i.e. etc/webserver/nginx.conf
```
    location /schema/ {
      # at least for local dev armt app access ?!
      include cors.conf;
...
```

But also 404 for 
https://128.243.22.74/schema/subjects/questionnaire_app_event-key/versions/latest
https://128.243.22.74/schema/subjects/questionnaire_app_event-value/versions/latest

= missing topic!

Slightly oddly, this source type is specified in the 1.5.0 catalog version, but
not the 1.4.3 catalog version, and the latter is still the default for the 
questionnaire app master branch (2019-12-11).

some 401s initially:
POST `https://128.243.22.74/kafka/topics/questionnaire_completion_log` x4
https://128.243.22.74/kafka/topics/questionnaire_esm x1
https://128.243.22.74/kafka/topics/questionnaire_timezone x2

but now working... (schema issue in cached data?)

Sort of works on emulator, but have to restart/re-open, e.g. after enter token?

## Protcol definition

(note, browser app seems to deliver all instruments immediately)

In [protocol git repo](https://github.com/RADAR-Base/RADAR-aRMT-protocols)
in directory with name matching project name (possibly with ' ' -> '-')

Most questionnaires appear with avsc (schema type) `questionnaire`, 
although thincit appears at least once as `notification`.

E.g. see
[test1](https://github.com/cgreenhalgh/RADAR-aRMT-protocols/blob/master/test1/protocol.json)

### questionnaire definitions

Current questionnaires are in 
[this repo](https://github.com/RADAR-base/RADAR-REDCap-aRMT-Definitions/)
under `questionnaires/QNAME`.

These are (all?) auto-generated from REDCap definitions.

[spec](https://radar-base.github.io/RADAR-aRMT-protocols/)

E.g. see 
[test_text](https://github.com/cgreenhalgh/RADAR-aRMT-protocols/blob/master/questionnaires/test_text/test_text_armt.json)

Hmm.
Each questionnaire type maps to a different kafka topic, 'questionnaire_NAME'. 
These are listed in 
[the aRMT spec](https://github.com/RADAR-base/RADAR-Schemas/blob/master/specifications/active/aRMT-1.4.3.yml)
(currently version 1.4.3).

(And APP_EVENT aka questionnaire_app_event is also here, but not registered in my instance)
