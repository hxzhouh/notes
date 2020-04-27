# jstat命令简介

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。

命令的格式如下：

jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

# 参数

>**以下的统计空间单位,未标明的 都是KB**



| 命令选项                   | 参数                  |
| -------------------------- | --------------------- |
| **类加载统计**             | jstat -class 1        |
| **编译统计**               | jstat -compiler  1    |
| **垃圾回收统计**           | jstat -gc             |
| **堆内存统计**             | jstat -gccapacity     |
| **新生代垃圾回收统计**     | jstat -gcnew          |
| **新生代内存统计**         | jstat -gcnewcapacity  |
| **老年代垃圾回收统计**     | jstat -gcold          |
| **老年代内存统计**         | jstat -gcoldcapacity  |
| **JDK8 下 元数据空间统计** | jstat -gcmetacapacity |
| **总结垃圾回收统计**       |                       |
|                            |                       |


## 类加载统计

1. 命令

   `jstact -class 1`

   

   ![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200330100154.png)

2.  结果说明

   1. Loaded:加载class的数量
   2. Bytes：所占用空间大小
   3. Unloaded：未加载数量
   4. Bytes:未加载占用空间
   5. Time：时间

## **垃圾回收统计**

1. 命令

   `jstack -gc 1 `

   ![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200330100651.png)

2. 结果说明

   1. S0C：第一个幸存区的大小
   2. S1C：第二个幸存区的大小
   3. S0U：第一个幸存区的使用大小
   4. S1U：第二个幸存区的使用大小
   5. EC：伊甸园区的大小
   6. EU：伊甸园区的使用大小
   7. OC：老年代大小
   8. OU：老年代使用大小
   9. MC：方法区大小
   10. MU：方法区使用大小
   11. CCSC:压缩类空间大小
   12. CCSU:压缩类空间使用大小
   13. YGC：年轻代垃圾回收次数
   14. YGCT：年轻代垃圾回收消耗时间
   15. FGC：老年代垃圾回收次数
   16. FGCT：老年代垃圾回收消耗时间
   17. GCT：垃圾回收消耗总时间

### **堆内存统计**

1. 命令

   ```shell
   jstat -gccapacity
   ```

2. 结果说明