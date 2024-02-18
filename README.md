1. ОПРЕДЕЛЕНИЕ АЛГОРИТМА С НАИЛУЧШИМ СЖАТИЕМ

[root@zfs ~]# lsblk        #выводим список дисков, которые есть в виртуальной машине
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk 
`-sda1   8:1    0   40G  0 part /
sdb      8:16   0  512M  0 disk 
sdc      8:32   0  512M  0 disk 
sdd      8:48   0  512M  0 disk 
sde      8:64   0  512M  0 disk 
sdf      8:80   0  512M  0 disk 
sdg      8:96   0  512M  0 disk 
sdh      8:112  0  512M  0 disk 
sdi      8:128  0  512M  0 disk 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zpool create otus1 mirror /dev/sdb /dev/sdc       #создаем RAID1 пул из 2х дисков
[root@zfs ~]# zpool create otus2 mirror /dev/sdd /dev/sde 	#создаем RAID1 пул из 2х дисков
[root@zfs ~]# zpool create otus3 mirror /dev/sdf /dev/sdg 	#создаем RAID1 пул из 2х дисков
[root@zfs ~]# zpool create otus4 mirror /dev/sdh /dev/sdi 	#создаем RAID1 пул из 2х дисков
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zpool list 	#выводим информацию о пулах
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M   106K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs set compression=lzjb otus1		#добавляем алгоритм сжатия в FS
[root@zfs ~]# zfs set compression=lz4 otus2		#добавляем алгоритм сжатия в FS	
[root@zfs ~]# zfs set compression=gzip-9 otus3		#добавляем алгоритм сжатия в FS
[root@zfs ~]# zfs set compression=zle otus4		#добавляем алгоритм сжатия в FS
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get all | grep compression	#выводим информацию о методах компрессии FS
otus1  compression           lzjb                   local
otus2  compression           lz4                    local
otus3  compression           gzip-9                 local
otus4  compression           zle                    local
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# #качаем одинаковый файл во все пулы
[root@zfs ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
--2024-02-13 10:53:23--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus1/pg2600.converter.log'

100%[==================================================================================================================================================================================================>] 41,016,061   753KB/s   in 57s    

2024-02-13 10:54:21 (704 KB/s) - '/otus1/pg2600.converter.log' saved [41016061/41016061]

--2024-02-13 10:54:21--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus2/pg2600.converter.log'

100%[==================================================================================================================================================================================================>] 41,016,061  2.49MB/s   in 27s    

2024-02-13 10:54:49 (1.47 MB/s) - '/otus2/pg2600.converter.log' saved [41016061/41016061]

--2024-02-13 10:54:49--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus3/pg2600.converter.log'

100%[==================================================================================================================================================================================================>] 41,016,061   545KB/s   in 26s    

2024-02-13 10:55:16 (1.49 MB/s) - '/otus3/pg2600.converter.log' saved [41016061/41016061]

--2024-02-13 10:55:16--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41016061 (39M) [text/plain]
Saving to: '/otus4/pg2600.converter.log'

100%[==================================================================================================================================================================================================>] 41,016,061   550KB/s   in 64s    

2024-02-13 10:56:21 (627 KB/s) - '/otus4/pg2600.converter.log' saved [41016061/41016061]

[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# ls -l /otus* 	#проверяем, что файл скачался
/otus1:
total 22068
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus2:
total 17994
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus3:
total 10959
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log

/otus4:
total 40091
-rw-r--r--. 1 root root 41016061 Feb  2 08:53 pg2600.converter.log
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.7M   330M     21.6M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.3M   313M     39.2M  /otus4
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get all | grep compressratio | grep -v ref #проверяем степень сжатия, лучше всего сжатие в пуле otus3 методом gzip3
otus1  compressratio         1.81x                  -
otus2  compressratio         2.22x                  -
otus3  compressratio         3.65x                  -
otus4  compressratio         1.00x                  -
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 





2. ОПРЕДЕЛЕНИЕ НАСТРОЕК ПУЛА

При скачивании архива, как указано в методичке, он скачивается битым, поэтому скачала его в машине-хосте, потом через шареную вагрант папку перекинула в гостевую.

liliya@metallica:~/HW/HW8$ vagrant ssh
[vagrant@zfs ~]$ 
[vagrant@zfs ~]$ sudo -i
[root@zfs ~]# ls -la /vagrant/
total 4217800
drwxrwxr-x.  2 vagrant vagrant       185 Feb 13 14:47 .
dr-xr-xr-x. 18 root    root          255 Feb 13 14:49 ..
-rw-rw-r--.  1 vagrant vagrant      2960 Feb 13 10:40 Vagrantfile
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata1.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata2.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata3.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata4.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata5.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata6.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata7.vdi
-rw-------.  1 vagrant vagrant 538968064 Feb 13 14:47 sata8.vdi
-rw-rw-rw-.  1 vagrant vagrant   7275140 Feb 13 14:33 zfs_task1.tar.gz
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# cp /vagrant/zfs_task1.tar.gz 
cp: missing destination file operand after '/vagrant/zfs_task1.tar.gz'
Try 'cp --help' for more information.
[root@zfs ~]# cp /vagrant/zfs_task1.tar.gz ~/
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# ls -la
total 7144
dr-xr-x---.  2 root root     161 Feb 13 14:56 .
dr-xr-xr-x. 18 root root     255 Feb 13 14:49 ..
-rw-r--r--.  1 root root      18 Dec 29  2013 .bash_logout
-rw-r--r--.  1 root root     176 Dec 29  2013 .bash_profile
-rw-r--r--.  1 root root     176 Dec 29  2013 .bashrc
-rw-r--r--.  1 root root     100 Dec 29  2013 .cshrc
-rw-r--r--.  1 root root     129 Dec 29  2013 .tcshrc
-rw-------.  1 root root    5570 Apr 30  2020 anaconda-ks.cfg
-rw-------.  1 root root    5300 Apr 30  2020 original-ks.cfg
-rw-r--r--.  1 root root 7275140 Feb 13 14:56 zfs_task1.tar.gz
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# tar -xzvf zfs_task1.tar.gz 
zpoolexport/
zpoolexport/filea
zpoolexport/fileb
[root@zfs ~]# zpool import -d zpoolexport/ 	#проверяем, можем ли импортировать каталог в пул
   pool: otus
     id: 6554193320433390805
  state: ONLINE
 action: The pool can be imported using its name or numeric identifier.
 config:

        otus                         ONLINE
          mirror-0                   ONLINE
            /root/zpoolexport/filea  ONLINE
            /root/zpoolexport/fileb  ONLINE
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zpool import -d zpoolexport/ otus		#делаем импорт данного каталога в пул
[root@zfs ~]# 
[root@zfs ~]# zpool status #проверяем информацию о импортированного пула
  pool: otus
 state: ONLINE
  scan: none requested
config:

        NAME                         STATE     READ WRITE CKSUM
        otus                         ONLINE       0     0     0
          mirror-0                   ONLINE       0     0     0
            /root/zpoolexport/filea  ONLINE       0     0     0
            /root/zpoolexport/fileb  ONLINE       0     0     0

errors: No known data errors
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get all otus		#определяем настройки
NAME  PROPERTY              VALUE                  SOURCE
otus  type                  filesystem             -
otus  creation              Fri May 15  4:00 2020  -
otus  used                  2.04M                  -
otus  available             350M                   -
otus  referenced            24K                    -
otus  compressratio         1.00x                  -
otus  mounted               yes                    -
otus  quota                 none                   default
otus  reservation           none                   default
otus  recordsize            128K                   local
otus  mountpoint            /otus                  default
otus  sharenfs              off                    default
otus  checksum              sha256                 local
otus  compression           zle                    local
otus  atime                 on                     default
otus  devices               on                     default
otus  exec                  on                     default
otus  setuid                on                     default
otus  readonly              off                    default
otus  zoned                 off                    default
otus  snapdir               hidden                 default
otus  aclinherit            restricted             default
otus  createtxg             1                      -
otus  canmount              on                     default
otus  xattr                 on                     default
otus  copies                1                      default
otus  version               5                      -
otus  utf8only              off                    -
otus  normalization         none                   -
otus  casesensitivity       sensitive              -
otus  vscan                 off                    default
otus  nbmand                off                    default
otus  sharesmb              off                    default
otus  refquota              none                   default
otus  refreservation        none                   default
otus  guid                  14592242904030363272   -
otus  primarycache          all                    default
otus  secondarycache        all                    default
otus  usedbysnapshots       0B                     -
otus  usedbydataset         24K                    -
otus  usedbychildren        2.01M                  -
otus  usedbyrefreservation  0B                     -
otus  logbias               latency                default
otus  objsetid              54                     -
otus  dedup                 off                    default
otus  mlslabel              none                   default
otus  sync                  standard               default
otus  dnodesize             legacy                 default
otus  refcompressratio      1.00x                  -
otus  written               24K                    -
otus  logicalused           1020K                  -
otus  logicalreferenced     12K                    -
otus  volmode               default                default
otus  filesystem_limit      none                   default
otus  snapshot_limit        none                   default
otus  filesystem_count      none                   default
otus  snapshot_count        none                   default
otus  snapdev               hidden                 default
otus  acltype               off                    default
otus  context               none                   default
otus  fscontext             none                   default
otus  defcontext            none                   default
otus  rootcontext           none                   default
otus  relatime              off                    default
otus  redundant_metadata    all                    default
otus  overlay               off                    default
otus  encryption            off                    default
otus  keylocation           none                   default
otus  keyformat             none                   default
otus  pbkdf2iters           0                      default
otus  special_small_blocks  0                      default
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get available otus	#определяем размер
NAME  PROPERTY   VALUE  SOURCE
otus  available  350M   -
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get readonly otus	#определяем тип FS
NAME  PROPERTY  VALUE   SOURCE
otus  readonly  off     default
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get recordsize otus		#определяем значение recordsize
NAME  PROPERTY    VALUE    SOURCE
otus  recordsize  128K     local
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get compression otus		#определяем тип сжатия
NAME  PROPERTY     VALUE     SOURCE
otus  compression  zle       local
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# zfs get checksum otus	#определяем тип контрольной суммы
NAME  PROPERTY  VALUE      SOURCE
otus  checksum  sha256     local
[root@zfs ~]# 


3. РАБОТА СО СНАПШОТОМ, ПОИСК СООБЩЕНИЯ ОТ ПРЕПОДАВАТЕЛЯ

[root@zfs ~]# zfs receive otus/test@today < otus_task2.file 	#восстанавливаем файловую систему из снапшота
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# find /otus/test -name "secret_message"		#ищем в каталоге /otus/test файл с именем “secret_message”
/otus/test/task1/file_mess/secret_message
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# 
[root@zfs ~]# cat /otus/test/task1/file_mess/secret_message	#смотрим содержимое файла
https://otus.ru/lessons/linux-hl/

