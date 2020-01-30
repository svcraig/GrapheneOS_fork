# GrapheneOS_fork

These instructions are specific to the Pixel 3a (sargo) because reasons. Credit to Daniel M. of GrapheneOS and Dan V. of RattlesnakeOS for various repos, patches, investigation, and documentation.

These directions are intended for someone familiar with Linux, AOSP, building Android apps, etc.

## Repo setup

Note that this repo links to a custom fork of F-Droid's privileged extension, where I've hardcoded in my release keys to simply the build process. Either remove this if you're not interested, modify it manually after sync, or refork it yourself and change accordingly

```
mkdir -p workspace
cd workspace
repo init -u https://github.com/svcraig/platform_manifest.git -b 10
reposynclite
```

## Build Kernel

```
cd kernel/google/crosshatch
git submodule sync
git submodule update --init
./build.sh bonito
cd ../../..
```

## Build F-Droid

### Download the SDK

```
cd /somewhere/outside/the/workspace
mkdir sdk
cd sdk
wget https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip -O sdk-tools.zip
unzip sdk-tools.zip
yes | ./tools/bin/sdkmanager --licenses
./tools/android update sdk -u --use-sdk-wrapper
```

Workaround for licnese issue with f-droid using older sdk (didn't investigate further)

`yes | ./tools/bin/sdkmanager "build-tools;27.0.3" "platforms;android-27"`

### Check out client and build

Note that this fork of fdroidclient adds support for an extra system permission as well as unreleased support for API29.

``
cd /somewhere/outside/the/workspace
git clone git@gitlab.com:svcraig/fdroidclient.git
cd fdroidclient
git checkout
SDK_DIR=/path/to/sdk
echo "sdk.dir=$SDK_DIR" > local.properties
echo "sdk.dir=$SDK_DIR" > app/local.properties
./gradlew assembleRelease

cp -f app/build/outputs/apk/full/release/app-full-release-unsigned.apk /path/to/workspace/packages/apps/F-Droid/F-Droid.apk

cd /path/to/workspace
```

## Setup build env

```
source script/envsetup.sh
choosecombo release aosp_sargo user
```

## Reproducible builds (untested)

See: https://grapheneos.org/build#reproducible-builds

## Extracting vendor files

The -b flag needs to match the build the repos from the manifest are based on

```
vendor/android-prepare-vendor/execute-all.sh -d sargo -b QQ1A.200105.002 -o vendor/android-prepare-vendor
mkdir -p vendor/google_devices
rm -rf vendor/google_devices/sargo
mv vendor/android-prepare-vendor/sargo/QQ1A.200105.002/vendor/google_devices/* vendor/google_devices/
```

## Building

(For production, wipe the out directory first)

`make target-files-package -j20`

## Generate release keys

```
mkdir -p keys/sargo
cd keys/sargo
cd keys/crosshatch
../../development/tools/make_key releasekey '/CN=svcOS/'
../../development/tools/make_key platform '/CN=svcOS/'
../../development/tools/make_key shared '/CN=svcOS/'
../../development/tools/make_key media '/CN=svcOS/'
../../development/tools/make_key networkstack '/CN=svcOS/'
openssl genrsa -out avb.pem 2048
../../external/avb/avbtool extract_public_key --key avb.pem --output avb_pkmd.bin
cd ../..

make -j20 brillo_update_payload
```

## Generate signed release build

`script/release.sh sargo`

## Install

- Enable OEM unlock (in Settings)
- Unlock bootloader (fastboot flashing unlock)
- Unzip factory image, run ./flash-all.sh
- Device won't auto reboot after flash
- Lock bootloader (flash script loads the AVB key) (fastboot flashing lock)
- Disable OEM unlocking

## Remove

- Enable OEM unlock (in Settings)
- Unlock bootloader (fastboot flashing unlock)
- Erase AVB key (fastboot erase avb_custom_key)
