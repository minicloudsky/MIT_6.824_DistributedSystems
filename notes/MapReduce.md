## MapReduce

​		让非分布式计算的人员也可以写出在成千上万的分布式计算机上面运行的程序，通过写`Map` 和 `Reduce` 函数就可以做分布式计算

1. `Map`，文件作为输入数据，生成一个 key value 列表

   Map（k,v）

   splict v into disk

   for each word in w:

      emit(w,1)

2. Reduce，对多个 Map的结果进行合并，得到最终的结果

   Reduce(k,v)

   emit(len(v))

3. shuffle: 行存储变为列存储负责 reduce

