# FAT16X 文件系统说明书



## 前言

「FAT16X 文件系统」是基于「FAT16 文件系统」的微调版本。主要区别在于：

- 对于「FAT16 文件系统」的「Reserved Region」中的一些操作系统相关的数据字段不再做严格规范要求
- 强制规定了一种较为通用的「FAT16 文件系统」中 `扇区 sector` 和 `簇 cluster` 的大小

旨在去除「FAT16 文件系统」的实现中可调整的变量，统一一种准确的存储结构。方便理解文件系统设计思想，能够更快的学习和实现。



## 介绍

「FAT16X 文件系统」中，存储设备会被划分为很多固定大小的存储单元，这些存储单元被称为簇（cluster），「FAT16X 文件系统」使用一个文件分配表（File Allocation table，既 FAT）来标识这些 clusters 的存储状态，文件分配表中每个条目大小为 `16-bit` 既 2个字节。

如果一个文件比较大，会存在多个 cluster 里，那么文件分配表中对应簇的条目里，会保存下一个簇的位置，形成一个簇链（cluster chain），来表示这个文件的数据存在哪些簇里。

因为每个条目能保存的最大簇编号为2的16次方，所以最多会有 65536 个 cluster。



### 基本结构

「FAT16X 文件系统」的结构包含以下几个区域

|               Region                |
| :---------------------------------: |
| Reserved Region (incl. Boot Sector) |
|     File Allocation Table (FAT)     |
|           Root Directory            |
|             Data Region             |

- Reserved Region (Boot Sector)

  启动扇区，保存文件系统的配置，存储设备的信息。

- File Allcation Table (FAT)

  文件分区表，标识每一个 cluster 的存储状态

- Root Directory

  根目录下的条目信息

- Data Region

  数据区，保存所有文件的数据，还有子目录的条目信息





## Reserved Region (Boot Sector)

![2021M5-fat16x介绍图-boot sector](fat16x-spec.assets/2021M5-fat16x%E4%BB%8B%E7%BB%8D%E5%9B%BE-boot%20sector.png)

**扇区（sector）**是一个物理设备的最小读写单元，我们假定我们使用的设备，一个扇区大小为 ***512 bytes***。而一个 cluster 会包含多个 sector，我们假定每个 cluster 包含 64 个 sectors。

Boot sector 是整个分区的第一个 sector，长度为 512 bytes，结构如图所示

| Offset | Size | Description |
| --- | --- | --- |
| 0000h | 3 bytes | **jump code** - 跳转到启动代码的位置，默认为`0xEB, 0x3C, 0x90` |
| 0003h | 8 bytes | **oem name** - 格式化系统名称，默认为 `mos-[4个字节以内的作者标识，不足补0]` |
| 000Bh | 2 bytes | **bytes per sector** - 每个 sector 字节数，默认为 `512 bytes` |
| 000Dh | 1 bytes | **sectors per cluster** - 每个 cluster 包含的 sector，默认为 `64` |
| 000Eh | 2 bytes | **reserved sectors** - 「保留区」使用几个 sector，因为保留扇区里总是保存着 Boot sector，默认为 `1` |
| 0010h | 1 bytes | **number of FAT copies** - 系统中使用的 FAT 副本有几份，默认为 `2`。FAT 副本的作用是为了防止其中一份 FAT 数据损坏。 |
| 0011h | 2 bytes | **number of possible root entries** - 根目录下支持的条目数量，我们定为 `63` 个 sectors，换算为条目数量为 `1008`。这 63 个 sectors 和 boot sector 正好组成一个完整的 cluster，而 FAT 区域是几个完整 cluster，这样，整个非数据区域，正好占满若干个 clusters。 |
| 0013h | 2 bytes | **small number of sectors** - 分区中保存的 sector 总数量，分区总大小小于32Mb时生效，我们不使用。默认为 `0` |
| 0015h | 1 bytes | **media descriptor** - 分区所使用的存储介质是什么，我们用的磁盘，默认为值 `0xF8` |
| 0016h | 2 bytes | **Sectors per FAT** - 每个 FAT 副本包含的 sector 数量，因为每个 FAT 包含 65536 个簇状态描述，每个簇状态描述 2 bytes，而每个 sector 使用 512 bytes，所以这里的默认值为 65536 * 2  / 512 = `256` |
| 0018h | 2 bytes | **Sectors per Track** - 存储介质属性，上每个磁道上的扇区数量，我们项目里不用，置为 `0` ， |
| 001Ah | 2 bytes | **Number of Heads** - 存储介质属性，有多少个磁头，我们项目里不用，置为 `0` |
| 001Ch | 4 bytes | **Hidden Sectors** - 如果一个存储介质被分区，如果当前分区前边还有分区，那么前面的分区占用的 sector 的数量。默认为 0 |
| 0020h | 4 bytes | **Large number of sectors** - 分区中保存的 sector 总数量，如果一个存储截止大小大于 32Mb时有效，默认值 4194240 |
| 0024h | 1 bytes | **Drive Number** - 驱动器编号，默认值 0 |
| 0025h | 1 bytes | **Reserved** - 系统特定字段，我们不用，默认 0 |
| 0026h | 1 bytes | **Extended Boot Signature** - 系统特定字段，我们不用，默认 0 |
| 0027h | 4 bytes | **Volume Serial Number**  - 系统特定字段，我们不用，默认 0 |
| 002Bh | 11 bytes | **Volume Labe** - 统特定字段，我们不用，默认 0 |
| 0036h | 8 bytes | **File System Type** - 文件系统的名称，默认为 `FAT16X` |
| 003Eh | 448 bytes | **Bootstrap code** - 系统启动代码，默认全设为 0 |
| 01FEh | 2 bytes | **Boot sector signature** - 签名字段，固定为 `0xAA55` |





## File Allocation Table

File Allocation Table 是一个链表结构，结构如下图所示：

![2021M5-fat16x介绍图-FAT structure](fat16x-spec.assets/2021M5-fat16x%E4%BB%8B%E7%BB%8D%E5%9B%BE-FAT%20structure.png)

每一个条目大小 2 bytes，表示了对应 cluster 的存储状态。

其中，第一个条目 `fat[0] = 0xFFF8`  ，值中的 'F8' 既 media descriptor；

第二个条目 `fat[1] = 0xFFFF`，除了标识第 2 个 cluster 被使用了以外，最高位 2 bit 还标识了当前分区的状态，不做赘述，默认取值 `0xFFFF`

每一个 NC***x*** （next cluster for **x**）标识 cluster[x] 的存储状态，这些条目的取值范围如下：

| Value         | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| 0000h         | **Free cluster** - 对应的 cluster 是空的，没有文件占用       |
| 0001h - 0002h | **Not allowed** - 不允许存在的值                             |
| 0003h - FFEFh | **Number of the next cluster** - 对应 cluster 只是一个文件的部分内容，后续内容在下一个 cluster，这个值就是下一个 cluster 的编码 |
| FFF7h         | **One or more bad sectors in cluster** - 对应 cluster 中有损坏扇区，不能使用 |
| FFF8h         | **End-of-file** - 对应 cluster 是文件的结尾，没有后续数据了  |





## Root Directory

Root Directory 区域保存了所有根目录下的文件/文件夹条目信息。根据 Boot Sector 中的配置，最多有 512 个。

这些条目被称为 directory entry structure，每个占用 32 bytes，结构如下：

![2021M5-fat16x介绍图-Directory entry Structure-原版](fat16x-spec.assets/2021M5-fat16x%E4%BB%8B%E7%BB%8D%E5%9B%BE-Directory%20entry%20Structure-%E5%8E%9F%E7%89%88.png)

| Offset | Size    | Description                                                  |
| ------ | ------- | ------------------------------------------------------------ |
| 00h    | 8 bytes | **Filename** - 文件名，ascii码表示，最多8个字符。A-Z, 0-1,  \#, $, %, &, ', (, ), -, @ |
| 08h    | 3 bytes | **Filename Extension** - 文件拓展名，ascii码表示，最多3个字符。A-Z, 0-1,  \#, $, %, &, ', (, ), -, @ |
| 0Bh    | 1 bytes | **Attribute Byte** - [文件属性](#attr-byte)                 |
| 0Ch    | 1 bytes | **Reserved for Windows NT** - 置为 0，无用途                 |
| 0Dh    | 1 bytes | **Creation** - 置为 0，暂不使用 |
| 0Eh    | 4 bytes | **Creation Time Stamp**  - 创建时间的秒级时间戳              |
| 12h    | 2 bytes | **Last Access Date Stamp**  - 上次访问日期，天级时间戳       |
| 14h    | 2 bytes | **Reserved for FAT32** - 置为 0，不使用                      |
| 16h    | 4 bytes | **Last Write Time Stamp**  - 最后写时间，秒级时间戳          |
| 1Ah    | 2 bytes | **Starting cluster** - 指向该文件/文件夹起始 cluster。如果是文件，cluster 里保存的是这个文件的第一部分数据；如果是文件夹，cluster 里保存改文件的子条目项 |
| 1Ch    | 4 bytes | **File size** - 如果是文件，表示文件字节数，最大 2 的 32 次方。如果不是文件，此值为 0 |



##### <a id="attr-byte" name="attr-byte">Attribute Byte - 文件属性</a>

|  7   |  6   |  5   |  4   |  3   |  2   |  1   |  0   |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|  -   |  -   |  A   |  D   |  V   |  S   |  H   |  R   |

###### R - Read Only

默认为 0。如果为 1，文件只读。

###### H - Hidden
默认为 0。如果为 1，表示隐藏。

###### S - System：

默认为 0。如果为 1，表示重要的系统文件。

###### V - Volume Name

默认为 0。用途自查。

###### D - Directory

0：文件，**Starting cluster** 字段指向的 cluster 是文件开始。1：文件夹，**Starting cluster** 字段指向的 cluster 保存的数据是该文件夹的子条目信息，这些字条目的结构也是 directory entry structure。

###### A - Achieve Flag

我们默认 0，不实现。



## 基本参数附录

1. 参数

|        参数         |    值     |
| :-----------------: | :-------: |
|     sector size     | 512 bytes |
| sectors per cluster |    64     |

2. 算式

   参考:  http://www.maverick-os.dk/FileSystemFormats/FAT16_FileSystem.html





## 参考资料

FAT16 File System Specifications: http://www.maverick-os.dk/FileSystemFormats/FAT16_FileSystem.html

FAT16 learn info: https://github.com/rweichler/FAT16


