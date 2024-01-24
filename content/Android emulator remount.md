---
|-
By now emulator is freezed Then you can try this
$ adb root $ adb shell avbctl disable-verification  <
---
 this will not freeze the emulator
$ adb reboot
When the emulator restarted Try to remount will be no issue

adb root
adb remount 
adb push will be able to write on system directory