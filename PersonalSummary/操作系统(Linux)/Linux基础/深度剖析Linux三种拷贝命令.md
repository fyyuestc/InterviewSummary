[TOC]

# 1. Linux文件和目录

Linux文件是树形结构，inode是平坦结构，通过inode->i_mode字段，即S_ISREG、S_ISDIR两个宏判断是哪个类型。

* 普通文件：**inode**里面存储元数据，inode索引到block，block存储数据

* 目录文件：inode索引到block，block中存储许多dirent目录条目，即名字到inode number的映射表

  <div align="center">    
  <img src="https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNdiccaOa8HrBYevRaP1PxbA0sq6o0hBm92yFicAX2y3Tia5nvfSjOQ6HicV8xRczwSsBBkUY1gZOvE88g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" width = 500 height = 300 />
  </div>

目录文件的block区域如下：

<div align="center">    
<img src="https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNdiccaOa8HrBYevRaP1PxbA0VggkG4rAxt7Rom7p9fvwy0Iht406k1AtibicC75CGp5NeRuxvKF6xzqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" height = 500 />
</div>

内存的树形结构：

* dentry 绑定到唯一一个 inode 结构体；
* dentry 有父，子，兄弟的索引路径，有这个就足够在内存中构建一个树了，并且事实也确实如此；

```c
struct dentry {
   // ...
   struct dentry  *d_parent;   /* 父节点 */
   struct qstr     d_name;      // 名字
   struct inode   *d_inode;    // inode 结构体

   struct list_head d_child;     /* 兄弟节点 */
   struct list_head d_subdirs;   /* 子节点 */ 
};
```

------

# 2. ln命令

* 软链接：软链接文件是一个全新的文件，有独立的 inode，有自己的 block ，内容是一段 path 路径，这个路径直接指向源文件；

  <div align="center">    
  <img src="https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNdiccaOa8HrBYevRaP1PxbA0iaI3yL5GPG9DqiaPJ80mVxJoSesfv7XJPib7ibhPRwPO3JiaaqUZZZsickBw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" width = 500 height = 300 />
  </div>

* 硬链接：

  >硬链接文件其实并没有新建文件（也就是说，没有消耗 inode 和 文件所需的 block 块）；
  >
  >硬链接其实是修改了当前目录所在的目录文件，加了一个 dirent 而已，这个 dirent 用一个新的 name 名字指向原来的 inode number；
  >
  >由于新旧两个 dirent 都是指向同一个 inode，那么就导致了一个限制：**不能跨文件系统。因为，不同文件系统的 inode 管理都是独立的。**

<div align="center">    
<img src="https://mmbiz.qpic.cn/mmbiz_png/4UtmXsuLoNdiccaOa8HrBYevRaP1PxbA0N8hYhDUxKchFtnwDjNRepR5nSsA4vyLxwxr7jG8abvCAYGR8qXbD6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" height="500"/>
</div>

总结：**硬链接只增加了一个 dirent 项，只修改了目录文件而已。不涉及到 inode 数量的变化。新的 name 指向原来的 inode。**

---

# 3. mv命令

* 源文件和目标文件在同一文件系统下：

> mv 命令的核心操作是系统调用 rename ，rename 从内核实现来说只涉及到元数据的操作，只涉及到 dirent 的增删；
>
> **inode number 不变，inode 不变，不增不减，还是原来的 inode 结构体，所以数据完全没有拷贝。**

<br>

* 源文件和目标文件不在同一文件系统下：

>  系统调用 rename 的时候，如果**源**和**目的**不在同一文件系统时，会报告 EXDEV 的错误码，提示该调用不能跨文件系统。

**这个时候操作分成两步走，先 copy ，后 remove：**

1.  走不了 rename ，那么就退化成 copy ，也就是真正的拷贝。读取源文件，写入目标位置，生成一个全新的目标文件副本；

   > 这里调用的 copy_reg 的函数封装；
   >
   > ln，mv，cp 是在 coreutils 库里的命令，公用函数本身就是可以复用的；

2. 删除源文件，使用 rm 函数删除；

---

# 4. cp命令

[深度剖析 Linux cp 的秘密-博客](https://juejin.cn/post/6939328247922425869)



# 5. 总结

1. 目录文件是一种特殊的文件，可以理解成存储的是 dirent 列表。dirent 只是名字到 inode 的映射，这个是树形结构的基础；
2. 常说目录树在内存中确实是一个树的结构，每个节点由 dentry 结构体表示；
3. ln -s 创建软链接文件，软链接文件是一个独立的新文件，有一个新的 inode ，有新的 dentry，文件类型为 link，文件内容就是**一条指向源的路径**，所以**软链的创建可以无视文件系统，跨越山河；**
4. ln 默认创建硬连接，硬链接文件只在目录文件里添加了一个新 dirent 项 (新name:原inode)，文件 inode 还是和原文件同一个，**所以硬链接不能跨文件系统（因为不同的文件系统是独立的一套 inode 管理方式，不同的文件系统实例对 inode number 的解释各有不同）；**
5. ln 命令貌似创建出了新文件，但其实不然，ln 只跟元数据相关，涉及到 dirent  的变动，**不涉及到数据的拷贝**，起不到数据备份的目的；
6. mv 其实是调用 rename 调用，**在同一个文件系统中不涉及到数据拷贝，只涉及到元数据变更**（ dirent 的增删 ），所以速度也很快。但如果 mv 的源和目的**在不同的文件系统，那么就会退化成真正的 copy ，会涉及到数据拷贝**，这个时候速度相对慢一些，慢成什么样子？就跟 cp 命令一样；
7. cp 命令才是真正的数据拷贝命令，速度可能相对慢一些，但是 cp 命令有 --spare 可以优化拷贝速度，针对空洞和全 0 数据，可以跳过，从而**针对稀疏文件可以节省大量磁盘 IO**；



















