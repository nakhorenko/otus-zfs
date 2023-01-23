После создания виртуальной машины, приступаем к созданию пуллов из дисков (берем по два диска и собираем в пул like RAID 1
```
[root@zfs ~]# zpool create otus1 mirror /dev/sda /dev/sdb
[root@zfs ~]# zpool create otus2 mirror /dev/sdc /dev/sdd
[root@zfs ~]# zpool create otus3 mirror /dev/sde /dev/sdf
[root@zfs ~]# zpool create otus4 mirror /dev/sdg /dev/sdh# otus-zfs
```
```
[root@zfs ~]# zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus1   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -
```
И добавим 4 алгоритма сжатия данных для каждого из 4х пулов
```
[root@zfs ~]# zfs set compression=lzjb otus1
[root@zfs ~]# zfs set compression=lz4 otus2
[root@zfs ~]# zfs set compression=gzip-9 otus3
[root@zfs ~]# zfs set compression=zle otus4
```
Проверка:
```
[root@zfs ~]# zfs get all | grep compression
otus1  **compression**           lzjb                   local
otus2  **compression**           lz4                    local
otus3  **compression**           gzip-9                 local
otus4  **compression**           zle                    local
```
Всё ОК!

Качаем файл вв все четыре дисковых пула
```
[root@zfs ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```
Проверяем как сжались данные при скачивании одного и того же файла в разные пуллы с разными способами сжатия
```
[root@zfs ~]# zfs list
NAME    USED  AVAIL     REFER  MOUNTPOINT
otus1  21.6M   330M     21.5M  /otus1
otus2  17.7M   334M     17.6M  /otus2
otus3  10.8M   341M     10.7M  /otus3
otus4  39.1M   313M     39.0M  /otus4
```
Очевидно, что gzip на пуле otus3 работает лучше всего.

По заданию скачиваем файл и кладём его в архив
```
[root@zfs ~]# wget -O archive.tar.gz --no-check-certificate https://drive.google.com/u/0/uc?id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg&e
```
Распаковываем к себе на машину
```
[root@zfs ~]# tar -xzf archive.tar.gz 
[root@zfs ~]# ll
total 7124
-rw-------. 1 root root    5570 Apr 30  2020 anaconda-ks.cfg
-rw-r--r--. 1 root root 7275140 Jan 23 14:05 archive.tar.gz
-rw-------. 1 root root    5300 Apr 30  2020 original-ks.cfg
drwxr-xr-x. 2 root root      32 May 15  2020 **zpoolexport**
```

А потом импортируем этот пул к нам на машину
```
[root@zfs ~]# zpool import -d zpoolexport/ otus
```
Далее по заданию скачиваем файл
```
[root@zfs ~]# wget -O otus_task2.file --no-check-certificate https://drive.google.com/u/0/uc?id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG&e
xport=download
```
И восстанавливаем пул с FS из снапшота
```
[root@zfs ~]# zfs receive otus/test@today < otus_task2.file

[root@zfs ~]# zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            4.93M   347M       25K  /otus
otus/hometask2  1.88M   347M     1.88M  /otus/hometask2
otus/test       2.83M   347M     2.83M  /otus/test
otus1           21.6M   330M     21.5M  /otus1
otus2           17.7M   334M     17.6M  /otus2
otus3           10.8M   341M     10.7M  /otus3
otus4           39.1M   313M     39.0M  /otus4
```
Ищем секретное послание от преподавателя и смотрим что там спрятано :)
```
[root@zfs /]# cd otus
[root@zfs otus]# ll
total 4
drwxr-xr-x. 102 root root 102 May 15  2020 hometask2
drwxr-xr-x.   3 root root  11 May 15  2020 test
[root@zfs otus]# find /otus/test -name 'secret_message'
/otus/test/task1/file_mess/secret_message
```
```
[root@zfs otus]# cat /otus/test/task1/file_mess/secret_message
https://github.com/sindresorhus/awesome
```
