## MySQL基础

### 基本架构图

![](pic/01.png)

MySQL 大致可以分为Server层和存储引擎层两部分

Server层包含连接器、分析器、优化器、执行器、查询缓存等，以及所有的内置函数（如日期、时间

数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

#### InnoDB体系架构

![](pic/02.png)

InnoDB存储引擎有多个内存块组成了一个大的内存池。

后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。此外将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下InnoDB能恢复到正常状态。

##### 内存结构

![](pic/03.png)
