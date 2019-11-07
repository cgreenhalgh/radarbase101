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

From [.travis.yml](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/.travis.yml)
it is expecting node version 11 (current stable is 12.13.0), 
yarn version 1.12.3 (current stable is 1.19.1), 
android-28 (android 9.0?) & android-build-tools 28.0.3.

[yarn.lock](https://github.com/RADAR-base/RADAR-Questionnaire/blob/master/yarn.lock)
suggests ionic 5.4.2, cordova-android 8.1.0, cordova-common 3.1.0.

I'm on ubuntu bionic for now...

install nvm - latest as of 2019-11-07
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
source ~/.bashrc
```
install node - lts ?!
```
nvm install 12
```

[install yarn](https://yarnpkg.com/lang/en/docs/install/#debian-stable) - latest stable 1.19.1 
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update && sudo apt install --no-install-recommends yarn
```

```
git clone https://github.com/RADAR-base/RADAR-Questionnaire.git
cd RADAR-Questionnaire
```

back to radar instructions...
```
yarn global add ionic cordova
```
(installed ionic 5.4.5 & cordova 9.0.0)
and
```
echo "export PATH=\"$PATH:$(yarn global bin)\"" >> .bashrc
source ~/.bashrc
```

yarn
```
NB - maybe its omitted from git?!
```
mkdir www
```
```
cordova prepare
```

Hmm, not working (can't find ionic-app-scripts) - ionic info can't find npm - is that why?
```
ionic serve
```

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

back to RADAR...
```
ionic cordova platform add android
```
...