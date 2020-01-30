# GrapheneOS_fork

These instructions are specific to the Pixel 3a (sargo) because reasons.

## Repo setup

mkdir -p workspace
cd workspace

repo init -u https://github.com/svcraig/platform_manifest.git -b 10

reposynclite

## Build Kernel

cd kernel/google/crosshatch

git submodule sync

git submodule update --init

./build.sh bonito

cd ../../..

## Build env

source script/envsetup.sh

choosecombo release aosp_sargo user

## Reproducible builds (untested)

See: https://grapheneos.org/build#reproducible-builds

## Extracting vendor files

-b flag is the build you want to use, e.g. QQ1A.200105.002

vendor/android-prepare-vendor/execute-all.sh -d sargo -b QQ1A.200105.002 -o vendor/android-prepare-vendor

mkdir -p vendor/google_devices

rm -rf vendor/google_devices/sargo

mv vendor/android-prepare-vendor/sargo/QQ1A.200105.002/vendor/google_devices/* vendor/google_devices/

## Building

(For production, wipe the out directory first)

make target-files-package -j20

## Generate release keys

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

## Generate signed release build

script/release.sh sargo

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
