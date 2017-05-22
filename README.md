### ラズパイでResilioを動かす

外付けフォーマット　（ここはWindowsでやったほうがいい？ntfsフォーマットを利用した。）

その後　⇒　ラズパイに挿して、マウント設定

```
dmesg
sudo fdisk -l
sudo blkid などを駆使してマウントする情報を確保
```

マウントするディスク情報の取得
```
/dev/sda1: LABEL="SyncDrive" UUID="42C0B480C0B47BAB" TYPE="ntfs" PARTUUID="3701b122-01"
```

ディスクはhomeディレクトリのSyncDriveにマウントさせる。


sudo /etc/fstabに書き込む（一番下のUUIDが書き込んだ状態）

```
pi@raspberrypi:~ $ cat /etc/fstab
proc            /proc           proc    defaults          0       0
PARTUUID=3b732aa2-01  /boot           vfat    defaults          0       2
PARTUUID=3b732aa2-02  /               ext4    defaults,noatime  0       1
UUID=42C0B480C0B47BAB   /home/SyncDrive ntfs    defaults,nofail 0       0


# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that

```

sudo reboot して df -hで確認。

```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/root       14848752 3840444  10346780  28% /
devtmpfs          469532       0    469532   0% /dev
tmpfs             473864       0    473864   0% /dev/shm
tmpfs             473864    6404    467460   2% /run
tmpfs               5120       4      5116   1% /run/lock
tmpfs             473864       0    473864   0% /sys/fs/cgroup
/dev/mmcblk0p1     41322   20763     20560  51% /boot
/dev/sda1      125032444  110028 124922416   1% /home/SyncDrive
tmpfs              94776       0     94776   0% /run/user/1000
```

### プログラムをゲット
```
mkdir Resilio
cd Resilio
wget https://download-cdn.resilio.com/stable/linux-arm/resilio-sync_arm.tar.gz
tar xvf resilio-sync_arm.tar.gz
mkdir archive
mv resilio-sync_arm.tar.gz archive
```

起動する
```
./rslsync
```

ここで自動起動もできるようにしておく

```
sudo editor /etc/init.d/btsync
```

```
#! /bin/sh
# /etc/init.d/rslsync
#

# Carry out specific functions when asked to by the system
case "$1" in
start)
    /home/pi/Resilio/rslsync
    ;;
stop)
    killall rslsync
    ;;
*)
    echo "Usage: /etc/init.d/rslsync {start|stop}"
    exit 1
    ;;
esac

exit 0
                                                 
```

で、Resilioフォルダ内のrslsyncに実行権限を与える。

```
chmod 755 rslsync
```
