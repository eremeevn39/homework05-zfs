# homework05-zfs

1. [**Определить алгоритм с наилучшим сжатием**](#1-определить-алгоритм-с-наилучшим-сжатием)

1. [**Определить настройки pool’a**](#1-определить-алгоритм-с-наилучшим-сжатием)

1. [**Найти сообщение от преподавателей**](#3-найти-сообщение-от-преподавателей)


## 1. Определить алгоритм с наилучшим сжатием
Чтобы определить какие алгоритмы сжатия поддерживает **zfs (gzip gzip-N, zle lzjb, lz4)**, выполняем команду

```man zfs | grep compression```

и в выводе находим строку, в которой перечислены возможные алгоритмы сжатия:

```compression=on|off|gzip|gzip-N|lz4|lzjb|zle```

Далее создаем 4 датасета и на каждом применяем свой алгоритм сжатия:

```
# zpool create storage $PWD/pool/disk[1-6]
# zfs create storage/data-gzip
# zfs create storage/data-gzip-9
# zfs create storage/data-lz4
# zfs create storage/data-lzjb
# zfs create storage/data-zle
# zfs set compression=gzip storage/data-gzip
# zfs set compression=gzip-9 storage/data-gzip-9
# zfs set compression=lz4 storage/data-lz4
# zfs set compression=lzjb storage/data-lzjb
# zfs set compression=zle storage/data-zle
```
Скачиваем в корень каждого датасета файлик:

```
wget -O War_and_Peace.txt  http://www.gutenberg.org/files/2600/2600-0.txt
```
Сам файлик имеет размер **3,3 Mb**:
```
-rw-r--r--. 1 root root 3.3M Aug  6 14:10 War_and_Peace.txt
```
Смотрим, сколько он занимает места на каждом из датасетов:
```
# zfs list
NAME                  USED  AVAIL     REFER  MOUNTPOINT
storage              10.4M  2.68G     29.5K  /storage
storage/data-gzip    1.24M  2.68G     1.24M  /storage/data-gzip
storage/data-gzip-9  1.23M  2.68G     1.23M  /storage/data-gzip-9
storage/data-lz4     2.02M  2.68G     2.02M  /storage/data-lz4
storage/data-lzjb    2.41M  2.68G     2.41M  /storage/data-lzjb
storage/data-zle     3.23M  2.68G     3.23M  /storage/data-zle
```

А также посмотрим уровень сжатия у каждого из алгоритмов:
```
# zfs get compression,compressratio
NAME                 PROPERTY       VALUE     SOURCE
storage              compression    off       default
storage              compressratio  1.61x     -
storage/data-gzip    compression    gzip      local
storage/data-gzip    compressratio  2.67x     -
storage/data-gzip-9  compression    gzip-9    local
storage/data-gzip-9  compressratio  2.67x     -
storage/data-lz4     compression    lz4       local
storage/data-lz4     compressratio  1.62x     -
storage/data-lzjb    compression    lzjb      local
storage/data-lzjb    compressratio  1.36x     -
storage/data-zle     compression    zle       local
storage/data-zle     compressratio  1.01x     -
```

Мы видим, что самое эффективное сжатие у **gzip-9**, а на последнем месте - **zle**.

## 2. Определить настройки pool’a
Для начала скачаем и распакуем экспортированный пул:

```wget --no-check-certificate -r "https://docs.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg" -O zfs_task1.tar.gz```

```tar -xzvf zfs_task1.tar.gz```

После можем приступить к импорту пула и сразу посмотреть на результат:
```# zpool import -d zpoolexport/ otus```
```
# zpool status
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
```
Из вывода **zpool status** мы можем понять, что пул в зеркале.

Из вывода **zpool list** мы видим размер, равный **480М**.

Из вывода **zfs get recordsize** мы видим размер блоков обоих датасетов, равный **128К**. Датасет **otus/hometask2** наследует этот параметр от **otus**

Из вывода **zfs get compression** мы узнаем, что оба датасета используют сжатие по алгоритму **zle**. Датасет **otus/hometask2** наследует этот параметр от **otus**

Из вывода **zfs get checksum** мы узнаем, что чек сумма используется по алгоритму шифрования **sha256**. Датасет **otus/hometask2** наследует этот параметр от **otus**

```
# zpool list
NAME   SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
otus   480M  2.09M   478M        -         -     0%     0%  1.00x    ONLINE  -
```
```
# zfs list
NAME             USED  AVAIL     REFER  MOUNTPOINT
otus            2.04M   350M       24K  /otus
otus/hometask2  1.88M   350M     1.88M  /otus/hometask2
```
```
# zfs get recordsize
NAME            PROPERTY    VALUE    SOURCE
otus            recordsize  128K     local
otus/hometask2  recordsize  128K     inherited from otus
```
```
# zfs get compression
NAME            PROPERTY     VALUE     SOURCE
otus            compression  zle       local
otus/hometask2  compression  zle       inherited from otus
```
```
# zfs get checksum
NAME            PROPERTY  VALUE      SOURCE
otus            checksum  sha256     local
otus/hometask2  checksum  sha256     inherited from otus
```

## 3. Найти сообщение от преподавателей

Для начала скачаем файл и создадим пул (файлы для пула у нас уже есть в каталоге pool):

```
wget --no-check-certificate -r "https://docs.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG" -O otus_task2.file

# zpool create otus $PWD/pool/disk[1-6]
```
После этого можем восстановить бэкап из скаченного файла и посмотреть на результат.
```
# zfs receive otus/storage@task2 < otus_task2.file
# zfs list
NAME           USED  AVAIL     REFER  MOUNTPOINT
otus          3.97M  2.68G     25.5K  /otus
otus/storage  3.75M  2.68G     3.69M  /otus/storage
```
Ну и наконец найдем и откроем зашифрованное сообщение **secret_message**:
```
cat /otus/storage/task1/file_mess/secret_message
https://github.com/sindresorhus/awesome
```
:upside_down_face: :upside_down_face: :upside_down_face:
