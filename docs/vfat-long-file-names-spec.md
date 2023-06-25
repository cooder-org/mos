# VFATX Long File Names (长文件名规格文档)



## 前言

「VFATX」是基于「VFAT」的微调版本。主要区别在于：

- 不强制使用unicode进行字符存储
- LFN Entey里去除了保留域以及checksum（详见LFN Entry一节）

## 介绍

VFATX本身不是一个文件系统，而是一种子文件系统，它可以放在FAT16X之上。VFATX系统是一种在FAT文件系统的目录结构中隐藏长文件名的方法，以达到前后兼容的目的。


## Coexistence with FAT16X
根据长文件名的长度，文件系统将在目录区创建一些无效的8.3条目，称为LFN (Long File Name) entries。这些LFN Entry将会按照逆序放置（也就是说，最后一个LFN Entry在最前面，第一个LFN entry在标准的directory entry的前面）。所以，当从上往下看时，目录区看起来是这样的：

| Entry No. | Without LFN Entries | With LFN Entries |  
| --- | --- | --- |
| ... | ... | ... |  
| n   | Normal 1 | Normal 1 |  
| n+1 | Normal 2 | LFN for Normal 2 - Part 3 |  
| n+2 | Normal 3 | LFN for Normal 2 - Part 2 |  
| n+3 | Normal 4 | LFN for Normal 2 - Part 1 |  
| n+4 | Normal 5 | Normal 2 |  
| n+5 | Normal 6 | Normal 3 |  
| ... | ... | ... |  



## Directory Entry

还记得FAT16X里的目录项结构吗？ 让我们来回忆一下：

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



### <a id="attr-byte" name="attr-byte">Attribute Byte - 文件属性</a>

|  7   |  6   |  5   |  4   |  3   |  2   |  1   |  0   |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
|  -   |  -   |  A   |  D   |  V   |  S   |  H   |  R   |

`R` - Read Only、 `H` - Hidden、 `S` - System、 `V` - Volume Name

**当以上4个标记位都被标记为1时，既代表这是一个LFN Entry。**



## LFN Entry
如前所述，LFN Entry被置于真正的Directory Entry之上，它们包含长文件名。第0-29个字节在第一个Entry中，第30-59个字节在第二个LFN Entry中，以此类推直到文件名结束。其中每个LFN Entry的结构如下:

| offset | size | description |
| --- | --- | --- |
| 00h | 1 bytes| Ordinal field |
| 01h | 10 bytes| part 1 |
| 0bh | 1 bytes| **Attribute Byte** - [文件属性](#attr-byte) |
| 0ch | 20 bytes| part 2 |

### Ordinal Field
Ordinal Field是用来告诉文件系统，在LFN字符串中，当前的LFN Entry是第几个。第一个LFN Entry 的值为01h。第二个LFN Entry的值为02h，以此类推。当一个LFN Entry是最后一个时，`Last LFN`位（第6位）被置为1。

| 7 | 6 | 5  4  3  2  1  0 |
| --- | --- | --- |
| Deleted LFN | Last LFN | LFN Number |


## 参考资料

VFAT Long File Names Specifications: http://www.maverick-os.dk/FileSystemFormats/VFAT_LongFileNames.html


