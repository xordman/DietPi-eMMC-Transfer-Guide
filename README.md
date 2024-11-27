
blkid - Дивимось що в нас зараз UUID
df -h  - дивимось які розділи в системі
root@31415926:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            188M     0  188M   0% /dev
tmpfs            50M  1.6M   48M   4% /run
/dev/mmcblk0p1  4.0G  2.2G  1.6G  59% /				- Ось це наша система
tmpfs           246M     0  246M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           1.0G     0  1.0G   0% /tmp
tmpfs            50M  8.0K   50M   1% /var/log


dietpi-drive_manager - Дивимось які зараз є диски та розділи, що змонтовано і що ні.
Тут нам треба вибрати наш внутрішній накопичувач і правильно його форматувати
/     : /dev/mmcblk0p1 | ext4 | Capacity: 3  - Розділ з системою
/mnt/ef229011-3276-47ca-b0ba-332a25d9: /dev/mmcblk2p1 | ext4 | Not mounted - щось інше, попередньо наш внутрішній диск, який і будемо форматувати. Вибираємо його:
далі 
 Format
 далі
  format mode
  Тут вибираємо прям весь диск  Drive     : /dev/mmcblk2  
	Далі<Ok>   
		Далі Partition Table : [GPT] Format with MBR or GPT partition table 
			Вибираємо  <MBR> 
		Далі Filesystem Type : [ext4] Select the new filesystem type
			вибираємо  0 : 		  | Default (Recommended)   
				Далі   <Ok>  
	Далі Format          : Start the format process with selected options
		 <Ok>  
	Ready to format:
  - /dev/mmcblk2
  - UUID=ef229011-3276-47ca-b0ba-332a25d9d847
  - Partition table: MBR
  - Filesystem type: ext4                                                     
 
 ALL DATA and PARTITIONS on this drive will be DELETED.                       
 Do you wish to continue?                                                     
 
                     <Ok>  


│ Please enter the desired mount point.                                        │
│  - Default and recommended = /mnt/142a0935-4721-4bfd-98d6-74476f529b87       │
│                                                                              │
│ NB: The path must start with /mnt/ and be unique. Spaces will be converted   │
│ automatically to underscores (_).                                            │
│                                                                              │
│ /mnt/142a0935-4721-4bfd-98d6-74476f529b87___________________________________ │
│                     <Ok>                         <Cancel>                    │
│                                                                              │

Тут мінямо на любе значеня, наприклад /mnt/disk1
і по закінченню повинно бути [  OK  ] Format completed  
тепер на диск ми можемо заходити через каталог /mnt/disk1

Кроки для запису образу і редагування файлів:
Завантаження і розпакування образу: Завантажуємо архів з образом і розпаковуємо його:
wget 'https://dietpi.com/downloads/images/DietPi_NanoPiNEOAir-ARMv7-Bookworm.img.xz' -O image.img.xz

так як архів в даному випадку .xz, треба встановити архіватор xz-utils:
apt update
apt install xz-utils	
unxz image.img.xz  # Розпаковуємо образ

Монтування образу для редагування: Використовуємо losetup, щоб створити пристрій блочного рівня, і монтуємо розділи для редагування:
modprobe loop
losetup -f  # Знаходимо вільний loop-пристрій
	/dev/loop0
losetup /dev/loop0 image.img
partprobe /dev/loop0  # Пробуджуємо розділи
mkdir -p /mnt/loop0p1 - можна подивитись правильну назву розділу ls /dev/loop0/
mkdir -p /mnt/loop0p2 - можна подивитись правильну назву розділу ls /dev/loop0/
Монтуємо розділи для редагування
mount /dev/loop0p1 /mnt/loop0p1
mount /dev/loop0p2 /mnt/loop0p2

Редагування конфігураційних файлів:
nano /mnt/loop0p2/dietpi.txt - також інші файли в цій діректорії треба преевірити і налаштувати під себе
зберігаємо і виходимо

sync
umount -f /mnt/loop0p1
umount -f /mnt/loop0p2
losetup -d /dev/loop0

Копіюємо
dd if=image.img of=/dev/mmcblk2 bs=1M status=progress conv=fsync

Після завершення запису образу, перезавантаж систему. Дістань флешку. Образ буде завантажуватися з внутрішньої пам'яті з усіма налаштованими параметрами.
