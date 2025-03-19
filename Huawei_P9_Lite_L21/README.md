# Instruction For Huawei_P9_Lite_L21
---
### 1. Get the key codes that need to be changed:
```shell
busybox script -q -c 'getevent /dev/input/event0' /dev/null </dev/null
```
When you press the volume up button, the phone will send two lines of code, the first of which is useful (1 73 1). When you release the key, the same code (1 73 0). In case of volume down (1 72 1 and 1 72 0).
<pre>
<b>output:</b>
0001 0073 00000001
0000 0000 00000000
0001 0073 00000001
0000 0000 00000000
0001 0072 00000001
0000 0000 00000000
0001 0072 00000001
0000 0000 00000000
</pre>

### 2. Next, find the files in the following path "/sys/class/leds/..." You need to find 3 files: 1 of them is responsible for the screen brightness (it reads the display status from it), 2 for turning on the flashlight and 3 in which the maximum brightness value is set (strangely enough, instead of 255, I had 8, this value is needed for the correct operation of the expression "expr 8 - $led_torch"):
```shell
cat /sys/class/leds/lcd_backight0/brightness # output: 0
```
```shell
cat /sys/class/leds/torch/brightness         # output: 0
```
```shell
ls /sys/class/leds/torch/
```
<pre>
<b>output:</b>
brighness device flash_led_fault flash_thermal_protect max_brightness power subsystem trigger uevent
</pre>
```shell
cat /sys/class/leds/torch/max_brightness
```
<pre>
<b>output:</b>
8
</pre>

Check that the flashlight works with the following commands:
```shell
echo "8" > /sys/class/leds/torch/brightness
```
```shell
echo "0" > /sys/class/leds/torch/brightness
```

### 3. First, edit the script for your device, changing all paths, files, and values ​​if they differ:
```shell
#!/system/bin/sh

(su -c 'flag=0
busybox script -q -c "getevent /dev/input/event0" /dev/null </dev/null | while read code1 code2 code3
do
led_screen=$(cat /sys/class/leds/lcd_backlight0/brightness)
led_torch=$(cat /sys/class/leds/torch/brightness)
if [[ "$led_screen" -eq "0" ]]
then
if [[ "$code2" -eq "0072" && "$code3" -eq "00000001" ]]
then
flag=1
elif [[ "$code2" -eq "0073" && "$code3" -eq "00000001" && "$flag" -eq "1" ]]
then
led_torch=$(expr 8 - $led_torch)
echo "$led_torch" > /sys/class/leds/torch/brightness
elif [[ "$code2" -eq "0072" && "$code3" -eq "00000000" ]]
then
flag=0
fi
fi
done') &
```

### 4. Then throw the script into the folder "/data/adb/post-fs-data.d/" in case there is Magisk and set the rights (don't forget to remount the system for recording):
```shell
mount -o rw,remount /
```
```shell 
su
```
```shell 
cp /storage/emulated/0/torch /data/adb/post-fs-data.d/
```
```shell 
chmod 755 /data/adb/post-fs-data.d/torch
```
```shell
mount -o ro,remount /
```
### 5. Reboot and test.
