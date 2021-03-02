## Presto任务调度

网上有很多关于MR、Spark的任务调度讨论，Presto的相关介绍却很少. 本文抛砖引玉，希望大家一起讨论Presto的任务调度的设计与实现

## 相关概念

下图中，虚线包围的方块代表SubPlan，其中的方块代表Operator。
![任务调度图](pic-presto-scheduler.png)

### Query

对应提交的一条SQL查询语句，由QueryScheduler负责调度。

### Stage

QueryScheduler会将QueryPlan划分成多个Stage。分成Leaf State和Immediate Stage两类。Presto为了提高查询的效率，默认会将集群中所有的worker节点都会参与leaf stage阶段的任务，因为查询中数据读取的解压/解码过程是最耗时的，但如果数据源是shared nothing类型的，数据读取任务只会下发到数据结点。对于immediate stage，presto会选择一部分worker参与计算。

### Task

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
