==============
数据类型
==============

--------------
运行时类型
--------------

Cozo 中的值可以用以下 **运行时类型** 来分类：

* ``Null`` 空值
* ``Bool`` 布尔值
* ``Number`` 数字
* ``String`` 字符串
* ``Bytes`` 字节数组
* ``Uuid`` UUID
* ``List`` 数组
* ``Validity`` 有效性

``Number`` 可以是 ``Float`` （双浮点数）或 ``Int`` （64 位带符号整数）。如果需要的话，Cozo 会自动将 ``Int`` 类型的值转换为 ``Float`` 类型。

``List`` （数组）可以包含任何类型的值，包括其他数组。

可以对 Cozo 中的数值进行全排序，不同运行时类型的值排序顺序根据上表，即： ``null`` 小于 ``true`` 小于 ``[]``。

在相同的类型中，排序逻辑如下：

* ``false < true`` ；
* ``-1 == -1.0 < 0 == 0.0 < 0.5 == 0.5 < 1 == 1.0`` ；
* 数组遵循其元素的字典顺序排序；
* 字节数组遵循字节的字典顺序排序；
* 字符串遵循其 UTF-8 表示字节的字典顺序排序；
* UUID 的排序比较特殊：时间戳相近的 UUIDv1 会排在相近的位置。这可以使使用 UUIDv1 作为键的数据的局部性，增强性能；
* 有效性的存在意义是为了实现历史穿梭查询，其排序顺序在 :doc:`timetravel` 一章中介绍。

.. WARNING::

    ``1 == 1.0`` 的值为真，但是 ``1`` 与 ``1.0`` 是不同的值，所以作为键时，同一个存储表可以同时包含这两个键。在类如 JavaScript 之类的语言中嵌入式地使用 Cozo 时，这尤其容易让人困惑，因为在 JavaScript 中所有的数字都会被转换为浮点数。因此，也因为其它很多原因，我们不推荐用浮点数作为键（但也不禁止）。

----------------
字面表达式
----------------

空值为 ``null``，布尔值 ``false`` 为假， ``true`` 为真。

输入数字时，可以使用常见的十进制，也可以添加 ``0x`` 或 ``-0x`` 前缀来使用 16 进值，添加 ``0o`` 或 ``-0o`` 前缀来使用 8 进制，添加 ``0b`` 或 ``-0b`` 前缀来使用二进制。浮点数的小数点后如果是 0，则可以省略 0，也可以使用常见的科学计数法来表示。所有数字在表示时都可以在任何位置添加额外的下划线 ``_`` 使数字更易读，例如 ``299_792_458`` 米每秒是真空中的光速。

字符串的字面表达式与 JSON 中相同，都以 ``""`` 作为分隔符，转义写法也一模一样。另外也可以用单引号 ``''`` 作为分隔符：除了单双引号之外其他所有转义方式都不变。另外，也可以使用所谓的“原始字符串”：
::

    ___"I'm a raw string"___

原始字符串以任意个下划线加上双引号开始，以双引号加上相同数量的下划线结束，其间所有字符都以字面意思来解释，不经过转义。只要下划线足够多，任何字符串都可以不经过转义表示为原始字符串。

字节数组与 UUID 没有字面表达式，必须使用函数才能创建它们。在满足键列的类型约束时，数据库会自动调用 ``decode_base64`` 及 ``to_uuid`` 函数。

数组以方括号 ``[]`` 作为分隔符，其元素以逗号隔开。最后一个元素之后可以有一个多余的逗号。

------------------------------------------------
列类型
------------------------------------------------

列类型的 **基本类型** 如下：

* ``Int`` 整数
* ``Float`` 浮点数
* ``Bool`` 布尔值
* ``String`` 字符串
* ``Bytes`` 字节数组
* ``Uuid`` UUID
* ``Validity`` 有效性

注意：空值不在上表中。如果在基本类型后添加问号，则类型为 **可空** 的类型：值要么满足类型约束，要么为空。

有两种复合类型： **同质数组** 以方括号加内含元素的类型来表示，如 ``[Int]`` 。也可以指定数组的长度，如 ``[Int; 10]`` 。 **异质数组** 以圆括号加上内含元素的列表来表示，如 ``(Int, Float, String)`` 。异质数组的长度只能是固定的。

特殊类型 ``Any`` 可接受任何非空的值，而 ``Any?`` 则接受所有值。这两种特殊类型都可以作为复合类型中元素的类型。