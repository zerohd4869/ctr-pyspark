
[deadline: July 16, Twes]


**Report Writing**

paper list 1, >2000 words, turnitin

- introduction, related work, challenges and new perspectives

**Slides Writing**

choose one paper in list 2, 20+pages

- motivations, methodology, experimental settings, results, conclusions
- point out some potential issues and propose my ideas for future directions or improvements


## Spark

**Spark简介**
    - 一种基于内存计算的大数据并行计算框架，低延时大数据分析
    - 计算模式属于MapReduce，但提供了多种数据集操作类型，编程模型灵活
    - 提供内存计算，将中间结果放到内存，对于迭代运算效率高
    - 基于DAG的任务调度执行机制，优于Hadhoop MapReduce的迭代执行机制

**基本概念**

    - RDD
        - Resillient Distributed Dataset, 弹性分布式数据集
        - 一个RDD就是一个分布式对象集合，本质上是一个只读的分区记录集合，每个RDD可分成多个分区，每个分区就是一个数据集片段
        - 提供了一种高度受限的共享内存模型，即RDD是只读的记录分区的集合，不能直接修改，只能基于稳定的物理存储中的数据集创建RDD，或者通过在其他RDD上执行确定的 转换操作(如map、join和group by)而创建得到新的RDD
        - 提供了一个抽象的数据架构：不同RDD之间的转换操作形成依赖关系，可以实现管道化，避免中间数据存储
        - RDD特性
            - 高效的容错性 中间结果持久化到内存 存放的数据可以是Java对象
            - RDD:血缘关系、重新计算丢失分区、无需回滚系统、重算过程在不同节点之间并行、只记录粗粒度的操作
        - 运行原理
            - 三个执行过程
                - RDD读入外部数据源进行创建
                - RDD经过一系列的转换(Transformation)操作，每一次都会产生不同的RDD，供给下一个转换操作使用
                - 最后一个RDD经过“动作”操作进行转换，并输出到外部数据源
                - 这一系列处理称一个Lineage(血缘关系)：DAG拓扑排序的结果。优点：惰性调用、管道化、避免同步等待、不需要保存中间结果、每次 操作变得简单
        
            - Stage的划分
                - Spark通过分析各个RDD的依赖关系生成了DAG，再通过分析各个RDD中的分区之间的依赖关系来决定如何划分Stage
                - Stage的类型包括两种:ShuffleMapStage和ResultStage

            - RDD在Spark中的三个运行过程
                - 创建RDD对象; 
                - SparkContext负责计算RDD之间的依赖关系，构建DAG; 
                - DAGScheduler负责把DAG图分解成多个Stage，每个Stage中包含了多个Task， 每个Task会被TaskScheduler分发给各个WorkerNode上的Executor去执行。

    - Executor和Task
        - 运行在工作节点（WorkerNode）的一个进程，负责运行Task（工作单元）
        - 1个Application=1个Driver+多个Job
        - 1个Job=多个Stage
        - 1个Stage由多个没有Shuffle关系的Task组成

    - 概念相互关系
        - 当执行一个Application时，Driver会向集群管理器申请资源，启动Executor，并向Executor发送应用程序代码和文件，然后在Executor上执行Task，运 行结束后，执行结果会返回给Driver，或者写到HDFS或者其他数据库中

**Spark运行基本流程**   

    - 四个流程
        - 首先为应用构建起基本的运行环境，即由Driver创建一个SparkContext，进行资源的申请、任务的分配和监控 
        - 资源管理器为Executor分配资源，并启动Executor进程 
        - SparkContext根据RDD的依赖关系构建DAG图，DAG图提交给DAGScheduler解析成 Stage，然后把一个个TaskSet提交给底层调度器TaskScheduler处理;Executor向SparkContext申请Task，Task Scheduler将Task发放给Executor运行，并提供应用程序代码 
        - Task在Executor上运行，把执行结果反馈给TaskScheduler，然后反馈给DAGScheduler，运行完毕后写入数据并释放所有资源

    - 运行架构特点
        - 每个Application都有自己专属的Executor进程，并且该进程在Application运行期间一直驻留。Executor进程以多线程的方式运行Task 
        - Spark运行过程与资源管理器无关，只要能够获取Executor进程并保持通信即可   
        - Task采用了数据本地性和推测执行等优化机制


**RDD基本操作**

Spark的主要操作对象是RDD，RDD可以通过多种方式灵活创建，可通过导入外部数据源建立，或者从其他的RDD转化而来。

在Spark程序中必须创建一个SparkContext对象，该对象是Spark程序的入口，负责创建RDD、启动任务等。在启动Spark Shell后，该对象会自动创建， 可以通过变量sc进行访问。

Spark RDD支持两种类型的操作: 动作(action):在数据集上进行运算，返回计算值。转换(transformation): 基于现有的数据集创建一个新的数据集。

创建RDD有两种方式：读取外部数据集，在驱动器程序中对一个集合进行并行化

    - Action API
        - count collect first 
        - reduce(f) 通过函数func(输入两个参数并返回一个值)聚合数据集中的元素 例如sum
        - foreeach(f) 将数据集中的每个元素传递到函数func中运行
        - take(n) 返回n个元素
        - top(n) 
        - collect() 返回所有元素
        - countByValue() 各元素在RDD中的出现次数

    - Transformation API
        - filter(f) 筛选出满足函数func的元素，并返回一个新的数据集
        - map(f) 将每个元素传递到函数func中，并将结果返回为一个新的数据集
        - flatMap(f) 与map()相似，但每个输入元素都可以映射到0或多个输出结果
        - groupByKey() 应用于(K,V)键值对的数据集时，返回一个新的(K, Iterable<V>)形式的数据集
        - reduceByKey(f) 应用于(K,V)键值对的数据集时，返回一个新的(K, V)形式的数据集，其中的每个 值是将每个key传递到函数func中进行聚合
        - union(r) 含重复的并集 
        - intersection(r) 交集
        - subtract(r) 移除

    - Pair RDD

    - StatsCounter
        - mean sum max min variance stdev 

**DataFrame**
    - DataFrame是一种完全格式化的数据集合，和数据库中的表的概念比较接近，它每列数据必须具有相同的数据类型。也正是由于DataFrame知道数据集合所有的类型信息，DataFrame可以进行列处理优化而获得比RDD更优的性能。 
在内部实现上，DataFrame是由Row对象为元素组成的集合，每个Row对象存储DataFrame的一行，Row对象中记录每个域=>值的映射，因而Row可以被看做是一个结构体类型。可以通过创建多个tuple/list、dict、Row然后构建DataFrame。
    - RDD：没有列名称，只能使用数字来索引；具有map()、reduce()等方法并可指定任意函数进行计算;
    - DataFrame：一定有列名称（即使是默认生成的），可以通过.col_name或者['col_name']来索引列；具有表的相关操作（例如select()、filter()、where()、join），但是没有map()、reduce()等方法。
