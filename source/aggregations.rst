==============
聚合
==============

Cozo 中的聚合可以理解为作用于一连串值然后返回单个值（聚合值）的函数。

Cozo 中的聚合分为两类， **普通聚合** 和 **半晶格聚合** 。所有的半晶格聚合都同时也是普通聚合，但是底层的实现方式不一样。只有半晶格聚合才能用在自递归的规则中。

半晶格聚合满足额外的代数性质：

    幂等性
        对单个值 ``a`` 进行聚合，值一定为 ``a`` 本身；
    交换律
        ``a`` 与 ``b`` 的聚合等于 ``b`` 与 ``a`` 的聚合；
    结合律
        聚合时怎么打括号得到的结果都一样。

在含有半晶格聚合的自递归规则中，理论上来说正文中的变量还需要满足一些额外的安全限制。常见的使用方式里这些限制都是被满足的，但随意使用归一来根据递归结果生成新的值可能会违反这些限制：Cozo 不会（也无法）检查这些有问题的查询。

------------------------------------
半晶格聚合
------------------------------------

.. module:: Aggr.SemiLattice
    :noindex:

.. function:: min(x)

    最小值。

.. function:: max(x)

    最大值。

.. function:: and(var)

    所有值的逻辑与。

.. function:: or(var)

    所有值的逻辑或。

.. function:: union(var)

    所有值（每一个都是数组）的并集。

.. function:: intersection(var)

    所有值（每一个都是数组）的交集。

.. function:: choice(var)

    从给出的值中抽取一个非空值作为聚合值。若所有值都为空，则聚合值为空。

.. function:: min_cost([data, cost])

    聚合值为 ``cost`` 最小的 ``data`` 。

.. function:: shortest(var)

    聚合值为长度最短的那个数组。

.. function:: bit_and(var)

    比特级别的“与”聚合值。被聚合的值必须都是字节数组。

.. function:: bit_or(var)

    比特级别的“或”聚合值。被聚合的值必须都是字节数组。

---------------------
普通聚合
---------------------

.. module:: Aggr.Ord
    :noindex:

.. function:: count(var)

    计数。

.. function:: count_unique(var)

    计数，重复值只计算一次。

.. function:: collect(var)

    将所有值聚合为一个数组。

.. function:: unique(var)

    将所有值聚合为一个数组，重复值只保留一份。

.. function:: group_count(var)

    对被聚合值按照值进行计数，列：值依次为 ``'a'`` 、 ``'b'`` 、 ``'c'`` 、 ``'c'`` 、 ``'a'`` 、 ``'c'`` ，聚合值为 ``[['a', 2], ['b', 1], ['c', 3]]`` 。

.. function:: bit_xor(var)

    比特级别的“排他或”聚合。值必须都为字节数组。

.. function:: latest_by([data, time])

    聚合值为 ``time`` 最大的 ``data`` 。与 ``min_cost`` 类似，但是 ``min_cost`` 中数组里第二个值只能是数字，这里可以是任何类型，且 ``min_cost`` 用的是最小，这里是最大。

.. function:: smallest_by([data, cost])

    聚合值为 ``cost`` 最小的 ``data`` 。与 ``min_cost`` 类似，但是 ``min_cost`` 中数组里第二个值只能是数字，这里可以是任何类型。当 ``cost`` 为空值时，直接舍弃此数组。

.. function:: choice_rand(var)

    从值中随机均匀取样一个值作为聚合值。

    .. NOTE::
        此聚合不是半晶格聚合，因为如果不保存状态，则无法保证均匀采样，而目前的半晶格聚合实现都是无状态的。

^^^^^^^^^^^^^^^^^^^^^^^^^
统计聚合
^^^^^^^^^^^^^^^^^^^^^^^^^

.. function:: mean(x)

    平均数。

.. function:: sum(x)

    求和。

.. function:: product(x)

    求积。

.. function:: variance(x)

    （样本）方差。

.. function:: std_dev(x)

    （样本）标准差。
