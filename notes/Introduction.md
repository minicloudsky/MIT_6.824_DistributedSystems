## 分布式系统

1. 分布式系统是什么？

   - 多台协作的计算机

   - 存储大型网站，MapReduce，p2p 共享网络等很多关键的基础设施是分布式的
   
2. 建设分布式系统的原因
   
- 通过并行增加容量
  
- 通过复制实现错误容忍(容错)
  

   - 使计算在物理上靠近外部实体

   
   - 通过隔离实现安全
   
3. 但是很多并发部分，复杂的交互，必须处理局部错误，难以实现性能潜力

4. 很多性能问题并不能简单通过扩展实现

   快速响应单一用户请求

   所有用户想更新相同的数据

   这些通常需要更好的设计而不仅仅是更多的服务器

5. MapReduce

   #### MapReduce 任务抽象

   ![](./imgs/mapreduce.png)
   
   ​                                     MapReduce 过程
   
   ```
     input is (already) split into M files
     Input1 -> Map -> a,1 b,1
     Input2 -> Map ->     b,1
     Input3 -> Map -> a,1     c,1
                       |   |   |
                       |   |   -> Reduce -> c,1
                       |   -----> Reduce -> b,2
                       ---------> Reduce -> a,2
     MR calls Map() for each input file, produces set of k2,v2
       "intermediate" data
       each Map() call is a "task"
     MR gathers all intermediate v2's for a given k2,
       and passes each key + values to a Reduce call
     final output is set of <k2,v3> pairs from Reduce()s
   
   Example: word count
     input is thousands of text files
     Map(k, v)
       split v into words
       for each word w
         emit(w, "1")
     Reduce(k, v)
       emit(len(v))
   
   MapReduce scales well:
     N "worker" computers get you Nx throughput.
    Maps()s can run in parallel, since they don't interact.
       Same for Reduce()s.
  So you can get more throughput by buying more computers.
   ```
   
   #### MapReduce 隐藏了很多细节:
   
   - 发送应用代码到服务器
- 追踪哪个任务已经完成
   - 将数据从 Map 移动到 Reduce
- 在服务器之间进行平衡加载调整
   - 恢复失败任务
   
   #### MapReduce 限制了应用能做的

   - 没有交互或者状态(除了通过中间输出)
- 没有迭代，没有多阶段管道
   - 没有做到实时数据或者流数据处理
   
   #### 输入和输出存储在 GFS 集群文件系统中，MR 需要巨大的并行输入和输出吞吐量
   
- GFS 需要把大型文件分割到多个服务器上，分成 64 MB 的文件块
   - Map 并行读取
- Reduce 并行写入
   - GFS 也把每个文件复制到2-3个服务器上，GFS 对 MapReduce 来说是巨大的胜利

   #### 限制性能的是什么？

   CPU ？ 内存 ？ 硬盘 ？ 网络带宽 ？

   在 2004 年他们认为是被网络带宽限制

   对于排序来说，Map 从 GFS 读取文件，Reduce 读取 Map 的输出可能和 输入一样大，Reduce 再把文件写入 GFS

   在 MR 的 all -to -all shuffle过程中，一半以上的流量通过根交换机

   论文中的根交换机：100-200 GB/s,一共1800台服务器，也就是 55MB/s,55 Mb 很小，远低于磁盘或内存速度

   今天，网络和根交换机相对于CPU /磁盘快得多

   #### 论文中图 1 的详情
   
   一个主节点，将任务分配给 worker 节点并且记住任务进度
   
   1. 主节点分给 worker节点 Map 任务直到所有的 Map 任务完成，Map 节点把输出（中间结果数据）到本地磁盘上，Map 分割输出，通过 hash分成每一个 Reduce 任务
   2. 等所有 Map 完成后，主节点分配Reduce 任务到