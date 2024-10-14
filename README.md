# Helloworld Service

A binding example for AGL

## Pre-requisites

Please follow [this guide](https://docs.automotivelinux.org/en/master/#3_Developer_Guides/2_Application_Framework_Binder/2_How_to_write_a_binding/)
to add the AGL-Master repository to your distribution.\
In order to load these files into the current shell script, use the following command :

* **Debian/Ubuntu**

```bash
source /etc/profile.d/agl-app-framework-binder.sh
```

### CMake module

Then install the cmake module using your distribution package manager.

* **Debian/Ubuntu**

```bash
sudo apt-get install agl-cmake-apps-module-bin
```

* **openSUSE**

```bash
sudo zypper install agl-cmake-apps-module
```

* **Fedora**

```bash
sudo dnf install agl-cmake-apps-module
```

### Build dependency

Because the helloworld-service binding uses json, the following package has to be installed.

* **Debian/Ubuntu**

```bash
sudo apt-get install libjson-c-dev agl-libafb-helpers-dev
```

* **openSUSE**

```bash
sudo zypper install libjson-c-devel agl-libafb-helpers-devel
```

* **Fedora**

```bash
sudo dnf install libjson-c-devel
```

## Setup

```bash
git clone https://gerrit.automotivelinux.org/gerrit/apps/agl-service-helloworld
cd agl-service-helloworld
```

## Build  for AGL

```bash
#setup your build environnement
. /xdt/sdk/environment-setup-aarch64-agl-linux
#build your application
./autobuild/agl/autobuild package
```

## Build for 'native' Linux distribution (Fedora, openSUSE, Debian, Ubuntu, ...)

```bash
./autobuild/linux/autobuild package
```

## Deploy

### Deploy on AGL

```bash
export YOUR_BOARD_IP=192.168.1.X
export APP_NAME=agl-service-helloworld
scp build/${APP_NAME}.wgt root@${YOUR_BOARD_IP}:/tmp
ssh root@${YOUR_BOARD_IP} afm-util install /tmp/${APP_NAME}.wgt
APP_VERSION=$(ssh root@${YOUR_BOARD_IP} afm-util list | grep ${APP_NAME}@ | cut -d"\"" -f4| cut -d"@" -f2)
ssh root@${YOUR_BOARD_IP} afm-util start ${APP_NAME}@${APP_VERSION}
```

## TEST

### TEST on AGL

```bash
export YOUR_BOARD_IP=192.168.1.X
export PORT=8000
ssh root@${YOUR_BOARD_IP} afb-daemon --ws-client=unix:/run/platform/apis/ws/helloworld --port=${PORT} --token='x' -v

#On an other terminal
ssh root@${YOUR_BOARD_IP} afb-client-demo -H 127.0.0.1:${PORT}/api?token=x helloworld ping true
#or
curl http://${YOUR_BOARD_IP}:${PORT}/api/helloworld/ping?token=x
#For a nice display
curl http://${YOUR_BOARD_IP}:${PORT}/api/helloworld/ping?token=x 2>/dev/null | python -m json.tool
```

### Native Linux

For native build you need to have tools **afb-daemon**.  
You can build it by yourself [app-framework-binder][app-framework-binder], or use binary package from OBS:[https://en.opensuse.org/LinuxAutomotive](https://en.opensuse.org/LinuxAutomotive)

```bash
export PORT=1234
cd build
afb-daemon --port=${PORT}  --ldpaths=package --workdir=. --roothttp=../htdocs --token= --verbose

xdg-open http://localhost:${PORT}/

#Command line test
curl http://localhost:${PORT}/api/helloworld/ping
#For a nice display
curl http://localhost:${PORT}/api/helloworld/ping 2>/dev/null | python -m json.tool
```

## Activate authentication security

To test auth just switch the line:

```diff
 static const struct afb_verb_v2 verbs[]= {
   /*Without security*/
-  { .verb = "ping"     , .session = AFB_SESSION_NONE, .callback = pingSample  , .auth = NULL},
+  /*{ .verb = "ping"     , .session = AFB_SESSION_NONE, .callback = pingSample  , .auth = NULL},*/
   /*With security "urn:AGL:permission:monitor:public:get"*/
-  /*{ .verb = "ping"     , .session = AFB_SESSION_NONE, .callback = pingSample  , .auth = &_afb_auths_v2_monitor[1]},*/
+  { .verb = "ping"     , .session = AFB_SESSION_NONE, .callback = pingSample  , .auth = &_afb_auths_v2_monitor[1]},
   {NULL}
 };
```

And rebuild your application

[app-framework-binder]:https://gerrit.automotivelinux.org/gerrit/#/admin/projects/src/app-framework-binder
