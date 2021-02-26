---
title: "A block layer introduction part 1: the bio layer (译)"
date: 2021-02-26 17:45:45
updte: 2021-02-26 17:45:45
categories:
- kernel
tags:
- linux
- block layer
---

Neil 是 linux 内核 raid 模块最初的维护者，他卸任后承诺会写一些文章来讲解相关的内容。
<!--more-->

https://lwn.net/Articles/736534/

**October 25, 2017** This article was contributed by Neil Brown

像 LInux 这样的操作系统的一项核心价值就是提供对具体设备的抽象接口。虽然最初对字符设备和块设备的抽象已经被许多其他的东西补充，例如“网络设备”和“位图显示”，但是它们并没有丢失重要性。尤其是块设备接口仍然是管理持久性存储的核心，即使持久性存储一直在发展，这种核心地位也很可能会持续一段时间。这两篇文章的目标事解码其中的一些角色。



术语 **block layer** 通常用来讨论内核中实现供应用程序和文件系统用来访问各种存储设备的接口的那些部分。究竟这一层具体事那些代码其实没有定论。最简单的回答是 block 子目录下的所有代码。这些代码可以被看做两层而不仅仅是一层，它们联系紧密但是完全不同。我知道这两个子层没有公认的名称，我把他们叫做 **bio layer** 和 **request layer** 。本文的剩余部分带我们进入前者，而后者我们留给下一篇。



# 块层之上

在深入 bio layer 之前，先了解 linux 中块层之上的部分是很有用的。见下图，这里的”上“表示靠近用户空间而远离硬件——它涵盖了可能会用到块层及之下的部分提供服务的客户端。

![neil-blocklayer](https://static.lwn.net/images/2017/neil-blocklayer.png)

对块设备的访问通常通过 /dev 下的具体设备来进行，一个具体的设备会映射成内核中的一个 S_IFBLK  inode。这些 inode 的作用有点像符号链接，因为它们不直接表示块设备而是简单的包含一对主次设备号（major:minor）作为指针指向块设备。在这种inode内部有 `i_bdev` 字段，i_bdev 内部链向 `struct block_device `，block_device 结构体实际代表一个目标设备。block_device 中还有一个 inode：`block_device->bd_inode`，这个 inode 与块设备 I/O 关系更密切。前文 /dev 下的那个 inode 只是一个指针。



bd_inode 扮演的主要角色是提供 `page cache`。当块设备文件以 非`O_DIRECT`标志打开时，与它 inode 相关联的 page_cache 就会被用来缓冲读取（包括预读(readahead)） 以及缓冲写入（通常时推迟写操作直到常规回写进程下刷它们）。 而使用 O_DIRECT 标志时读写都会直接下发给块设备。类似的，当文件系统挂载一个块设备时，来自文件系统的读写通常会直接下发到设备，不过有些文件系统（尤其是ext*家族）会访问 page_cache (传统上这种情况称作 buffer cache) 来管理一些文件系统的数据。



另一个和块设备很相关的 open() 标志位是 `O_EXCL`（译注：exclusive 的缩写）。块设备有一个简单的查询-锁定机制来保证它只有一个持有者(holder)。持有者是在激活块设备时记录的（例如调用  blkdev_get() 之类的内核函数）。如果一个另外的 holder 已经持有了这个设备那么再去打开它就会失败。文件系统安装设备时通常设置这个标志来保证独占访问。当一个应用程序以 O_EXCL 标志打开一个块设备，这会导致新创建的结构体 struct file 被用来当作持有者，如果一个文件系统已经从这个设备上挂载那么打开将失败。如果成功打开，只要设备保持打开，就会阻止后续的挂载尝试。使用O_EXCL不会阻止没有O_EXCL的情况下打开块设备，因此，它不会完全阻止并发写入-只是使应用程序可以轻松测试块设备是否正在使用。



无论以何种方式访问块设备，主要接口都涉及发送读取或写入请求或各种其他请求（例如丢弃），并最终获得答复。该接口由 bio 层提供。



## BIO 层

linux 中所有的块设备都以结构体 [struct gendisk](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/genhd.h?h=v4.13#n171) 来表示，正如其名一个通用的磁盘。这个结构体并不包含大量的信息和服务，而是作为上面的文件系统接口和下面的底层接口之间的链接。gendisk 之上是一个或多个`struct block_device`，这个我们前文介绍过。当一个 gendisk 有分区表时，它可以被多个 block_device 关联。其中会有一个 block_device  表示整个 gendisk，其余的表示 gendisk 中的分区。



bio 层得名于结构体 [`struct bio`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/blk_types.h?h=v4.14-rc1#n46) ，该结构体运输读写请求以及其他控制请求，从 block_device 经过 gendisk 最后到达设备。一个 bio 标识一个目标设备，目标设备上线性地址空间上的偏移，请求（通常是读写），大小，以及数据存在的内存空间。在Linux 4.14之前，目标设备将通过指向struct block_device的指针在bio中进行标识。之后，bio 使用指向 struct gendisk 的指针以及一个分区号，该分区号可以由 bio_set_dev（）设置。考虑到gendisk 结构的核心作用，这种使用更为自然。



bio 构造完成后，可通过调用 [`generic_make_request()`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/block/blk-core.c?h=v4.13#n2114)  或 [`submit_bio()`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/block/blk-core.c?h=v4.13#n2228).将 bio 传递给 bio 层。这里通常不等待请求完成，而只是将其排队等待后续处理。generic_make_request（）可以在短时间内阻塞，比如等待内存可用。考虑这种行为的一种有用方法是，它可能等待先前的请求完成（例如，在队列中腾出空间），而不是等待新的请求完成。如果在 bi_opf 字段中设置了 `REQ_NOWAIT`  标志，再没有足够空间的情况下 generic_make_request() 就不在等待了相反应该设置 bio 的状态为 `BLK_STS_AGAIN` 或者可能是`BLK_STS_NOTSUPP` 来结束 bio。在撰写本文时，此功能尚未正确或一致地实现。



bio layer 和 request layer 之间的接口需要设备调用 blk_queue_make_request() 并且传递一个 make_request_fn() 来注册。generic_make_request() 将为bio中标识的设备调用该函数。此函数必须安排一些事情，以便在bio描述的I / O请求完成时，将bi_status字段设置为指示成功或失败，并调用bio_endio（），而bio_endio（）依次将调用 bio 中的bi_end_io（）函数。



除了已经描述的对 bio 请求的简单处理之外，bio 层的两个最有趣的功能是避免递归和队列阻塞。



##  避免递归 Recursion avoidance 



使用虚拟块设备（例如“ md”（用于软件RAID）和“ dm”（例如，由LVM2使用））很有可能会导致生成一堆块设备，每个块设备都会修改bio并将其发送到堆栈中的下一个设备。一个简单的实现将导致大量的设备堆栈，导致过多使用内核的调用堆栈。在遥远的过去（在Linux 2.6.22之前），这有时会引起问题，特别是当bio由已经使用大量堆栈的文件系统提交时。



替代这种递归的方法是，generic_make_request() 检测何时它被递归调用并且不把 bio 发送给下一层虚拟设备，而是将 bio 压入内部的队列（当前进程的 current->bio_list ，参见[`struct task_struct`](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/sched.h?h=v4.13#n519) ）并且只在父辈 bio 提交完成后才提交。由于通常不预期 generic_make_request() 会等待 bio 完成，所以即使不立即处理 bio 也是安全的。



这个递归避免机制通常完美生效，但是有时候会导致死锁。理解这个问题的关键在于上面的观察，也就是 bio 的提交（例如 generic_make_request() 中调用注册的 make_request_fn()）允许等待之前提交的 bio 完成。如果它一直等一个位于 generic_make_request() 管理的 current->bio_list 中的 bio 完成，那么就发生了死等。



导致一个 bio 等一个较早的 bio 的依赖关系通常很微妙，而且往往是通过测试发现的而不是代码检查。举一个简单的例子，涉及到偶尔需要使用内存池([mempool](https://lwn.net/Articles/22909/)) 的情况。如果将 bio 提交到对I / O请求的大小或对齐方式有限制的设备，make_request_fn() 的实现可能会选择将 bio 分成两部分来分别处理。bio 层提供了方便的函数（）来做这些，但是这类操作需要再申请一个 bio。在块层中申请内存必须小心，因为当内存紧缺时，Linux使用的一个关键策略是写出脏页 ，数据会经过块层落盘，然后就可以把内存回收。但是脏页回写时还需要申请内存，这个时候就会导致问题了。应对这种情况一种标准机制是使用内存池，内存池会为特定目的预先分配少量内存。从内存池分配可能会等待该内存池的先前用户返回他们使用的内存，但不会等待常规内存回收完成。当使用内存池分配 bio 时，这种等待会引入某种依赖关系，这种依赖关系可能导致 generic_make_request() 死锁。



已经有过几种尝试想用简单的方式解决这些死锁。其中一种体现为 bioset 线程，你可能在 `ps` 的列表中见过。该机制专门针对上述死锁场景，并为用于分配 bio 结构的每个内存池分配一个“救援”线程。当从内存池中申请不到bio 时 `current->bio_list` 中来自该内存池的所有 bio 都被移交给 bioset 线程来处理。这种方法相当繁琐，导致创建了许多几乎从未使用过的线程，并且只处理一个特定的死锁场景。大多数(但是不是全部)死锁场景都涉及将 bios 拆分为两个或多个部分，但它们并不总是涉及等待 mempool 分配。



最近的内核仅在少数情况下才依赖于此，并且通常避免在不需要时创建bioset线程。相反，使用了另一种方法，该方法是通过在 Linux 4.11中对 generic_make_request（）进行更改而引入的。它更通用，并且在运行的系统上开销较小，但是对驱动程序的编写方式提出了要求。



主要的要求是，当一个 bio 被分割时，其中一个应该直接提交到 generic _ make _ request () ，以便在最合适的时间处理它。另一半可以用任何适当的方式处理。这使 generic_make_request() 对发生的事情有了更多一点的控制。利用这种控制的方法是，根据提交的设备堆栈的深度对所有 bios 进行排序。然后，它总是在上层设备之前处理更下层设备的 bio。这个简单的方法消除了所有烦人的死锁。



## 设备队列蓄洪 Device queue plugging

存储设备的每个请求通常开销都不小，所以攒一批请求再一起下发会更有效。当设备速度相对较慢时，它通常会有一个大的待处理请求队列，该队列为确定合适的批次提供了大量机会。当设备速度非常快或慢速设备处于空闲状态时，自然地找到批次的机会相应变少。为了应对这一挑战，Linux块层使用一个称为蓄洪（plugging）的概念。



起初，plugging 只用在空队列中。在向空队列提交请求之前，队列将被塞住，因此一段时间内不会有任何请求流=入下层设备。文件系统来的 bio 可以排队然后合并。塞子可以被文件系统显示的拔出，或者短暂的延时到了后隐式的拔出。希望在此期间可以找到一些合适的请求合并，并且开始的小延迟可以通过最终的大批次提交来弥补。从Linux 2.6.39开始，新的机制（[a new plugging mechanism](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=73c101011926c58) ）已经合入，该机制可以按进程而不是按设备运行。这在多CPU机器上可以更好地扩展。



当文件系统或其他使用块设备的应用要提交请求时通常会用一对函数`blk_start_plug()` 和`blk_finish_plug()` 将 generic_make_request() 相关的代码块包起来。这会设置 current->plug 指向一个数据结构 [struct blk_plug ](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/include/linux/blkdev.h?h=v4.13#n1258)，这个结构体里包含一个 struct blk_plug_cb 的链表（以及一个 struct request 的链表，这个我们下一篇在讲）。由于这些链表是每进程（per-process）的，所以可以做到无锁添加节点。bio 会给到 make_request_fn()，如果发现能提高效率就会把 bio 添加到 plug 中的链表。



像这样进程级的蓄洪是有好处的，有关联的 bios 将容易被识别和聚到一起，并且减少了锁的开销。如果没有这个机制(per-process plugging) 的话，则处理每个 bio都需要一个自旋锁或至少一个原子内存操作。使用 per-process plugging 和话通常可以创建一个每个进程的 bios 链表，然后在最后把这些 bios 通过一次上锁一次性合入公共队列。



## BIO 层及之下

总的来说，bio layer 是很薄的一层，它

它以 struct bio 的形式来接受 I / O 请求，并将其直接传递给适当的 make_request_fn() 函数。它提供了各种支持功能，以简化拆分 bios 和调动子 bios 的工作，并允许队列 plugging。它还执行其他一些简单的任务，例如更新`/proc/vmstat`中的 pgpgin 和 pgpgout 统计信息，但是大多数情况下，它只是使下一级的工作继续进行。



有时，下一层就是最终驱动程序，例如drbd（The Distributed Replicated Block Device）或brd（a RAM based block device）。更常见的下一层是一个中间层，例如 md 和 dm 提供的虚拟设备。也许中间层作为块层的剩余部分是最常见的，我称之为“请求层(request layer)”。该层的一些复杂机制将是本次概述第二部分（[the second part of this overview](https://lwn.net/Articles/738449/).）的主题。



------

## 留言精选

**问：**

Posted Oct 25, 2017 19:12 UTC (Wed) by **edos**

我还是不理解文中说的死锁。俺们套娃块设备并且由于互相依赖导致死锁，爪子可能嘛？



**尼尔答**：

一个可能导致死锁的简单（但极不可能）的情况是：

- 假设我有一个 RAID1，其中每个成员设备是一个 chunk 大小为 4K 的 RAID0。
- 假设一个 8K 的写 bio 到了 RAID1阵列。raid1代码从一个专用内存池中分配了两个 bios，并向每个 RAID0 设备发送了一个 8K 的 bio。这两个 bios 由 generic_make_request() 入队。
- 然后，generic_make_request() 开始处理第一个RAID0 bio。 raid0 代码需要将其拆分为 2 个 4K 的 bios，因此需要从专用池中分配一个 bio，然后将新的 bio 和老 bio（已经减小大小）提交给底层设备。这两个 bios 由还是通过 generic_make_request() 排队。
- 然后，generic_make_request() 开始处理第二个RAID0 bio（新代码会将其排序到列表的末尾，以帮助避免死锁）。同样，raid0 代码需要拆分 bio。

现在，假设没有空闲内存，假设私有内存池有16个预申请的 bios，假设16个线程同时提交这种 8K 写(到 RAID1中的不同地址)。我们最终会有16个线程都试图从同一个私有池中分配第二个 bio，而16个预申请的条目都被占了，每个线程一个，在 generic_make_request() 队列中。申请新的 bio 将等待先前分配出去的 bio 完成，而申请完成前，那些已经分出去的 bio 不会被 generic_make_request() 处理。


