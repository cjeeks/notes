### Hybird 模型 (混合模型)
#### 参考文档 &ensp;&ensp; [Kylin Hybird](http://kylin.apache.org/blog/2015/09/25/hybrid-model/)
1. Problem
对于一个将要进行查询的 sql, kylin 会选择一个一个 cube 来响应对应的 query.
样例: 
&ensp;&ensp;&ensp;&ensp;假如我们有一个 cube 叫做 Cube_V1, 这个已经构建了两个月了。现在，用户想要增加新的维度以及度量来满足他们 当前的需求。所以他们又建了 Cube_V2。
由于一些原因，用户想要保留 Cube_V1, 同时希望 Cube_V2 在Cube_V1 最后的账期开始进行计算。    

    * 历史数据已经被删除了，Cube_V2 不能从最开始的时候进行构建。
    * cube 很大，从头构建需要花大量的时间。
    * 新的维度以及度量只是从某天开始有，之前的历史数据中是没有相应的字段信息的。 
    * 用户觉得历史数据中，没有相应的字段也是可以接受的。
#### Hybird Model

::: hljs-center

![HyBird](../../imgs/HyBird.PNG)

:::

Hybird 模型是由多个其它的模型混合而成的。
他相当于其它其它表的视图。 Hybird 模型会分发请求到具体的 cube 中，再将所有结果进行 union all.

#### FAQ
1. 什么时候 hybird 模型会被选中来响应相应的查询？
&ensp;&ensp; 如果 hybird 依赖的 cube 能够响应对应的查询，这个 hybird 模型会自动选中。
2. hybird 模型如何响应？
&ensp;&ensp; hybird 模型会分发查询到对应的子 cube 中。如果某个子 cube 满足所有的维度，他的结果就会被返还给用户。最后 kylin 会从 hybird 聚合所有的结果, 返回给用户。
3. hybird 模型不要求所有的 cube 有相同的数据模型。
4. hybird 模型不支持嵌套。hybird 模型的实现只能依赖于子 cube 不能依赖于 子 hybird 模型。
5. hybird 作用主要是用来实现 类似于 union 这种 操作的。
6. 如果 hybird 的子 cube 被置为 disable 了 ，那么这个 cube 将不会被查询