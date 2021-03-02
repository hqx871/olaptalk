## Presto任务调度

网上有很多关于MR、Spark的任务调度讨论，Presto的相关介绍却很少. 本文抛砖引玉，希望大家一起讨论Presto的任务调度的设计与实现

## 相关概念

下图中，虚线包围的方块代表SubPlan，其中的方块代表Operator。
![任务调度图](pic-presto-scheduler.png)

### Query

对应提交的一条SQL查询语句，由QueryScheduler负责调度。

### Stage

QueryScheduler会将QueryPlan划分成多个Stage。分成Leaf State和Immediate Stage两类。Presto为了提高查询的效率，默认会将集群中所有的worker节点都会参与leaf stage阶段的任务，因为查询中数据读取的解压/解码过程是最耗时的，但如果数据源是shared nothing类型的，数据读取任务只会下发到数据结点。对于immediate stage，presto会选择一部分worker参与计算。
stage由StageScheduler负责调度。stage的调度有两种策略:
- all-at-once: 直接将所有的stage的任务下发到worker，形成数据处理的拓扑图。
- phased: 由于stage可能有循环依赖，需要先将有向图转成有向无环图。操作就是将有相互依赖的stage合并成一个新的结点，从而得到新的有向无环图。

### Task/Pipeline
StageScheduler会将stage衍生成多个并行执行的task任务。task下发到worker后，由TaskScheduler负责调度。调度逻辑不同于spark，spark是将每个task任务运行结束现执行下一个task，执行过程中只有一个线程。而presto会将task分成多个pipeline，pipeline的调度过程类似于**操作系统的进度调度**。任务执行一次只处理一个page，执行中可能由于资源不到位或者数据未就绪而被block，从而可以在每个operator中退出。处理完一个page后被放回任务队列等待下次时间片。
这种更细粒度的调度可以使得每个查询都得到充分的执行，所以官方推荐中说明可以通过增加cpu的方式减少查询延迟。
另外，这种presto会统计task每次执行的时长，将运行时间长的任务划分到低优先级，时间少的划到高优化级，从而保证了快得到更多的资源，因为presto认为查询用时越少的任务优先级越高。

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/hqx871/olaptalk/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
