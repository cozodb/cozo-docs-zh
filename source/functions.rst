========================
函数与算符
========================

函数是表达式的一种。Cozo 中除了读取当前时间的函数和以 ``rand_`` 开头的函数之外，其它所有的函数都是纯函数：给出同样的参数，其值相同。

------------------------------------
不是函数
------------------------------------

首先我们来介绍一些不是函数的构造。函数的求值规则是先对其参数求值，然后将参数的值传入函数的正文，得到一个值作为结果。以下这些构造不是函数，因为其不遵守这个规则。

首先，以下构造不返回值：

* ``var = expr`` 显性地将 ``var`` 与 ``expr`` 归一。注意与算符形式的函数 ``expr1 == expr2`` 的区别。
* ``not clause`` 将原子 ``clause`` 否定。注意与算符形式的函数 ``!expr`` 以及函数 ``negate(expr)`` 的区别。
* ``clause1 or clause2`` 将两个原子进行析取。注意与函数 ``or(expr1, expr2)`` 的区别。
* ``clause1 and clause2`` 将两个原子进行合取。注意与函数 ``and(expr1, expr2)`` 的区别。
* ``clause1, clause2`` 将两个原子进行合取。

以上最后三项中， ``or`` 的优先级最高， ``and`` 次之， ``,`` 最低。 ``and`` 与 ``,`` 的唯一区别就是他们的优先级。

以下构造返回值，但是不是函数：

* ``try(a, b, ...)`` 依次对参数求值，如果求值中没有遇到异常，则直接返回该结果，不对下一个参数求值；若所有参数的求值过程都遇到异常，则抛出最后一个异常。
* ``if(a, b, c)`` 首先对 ``a`` 求值，若为真，则对 ``b`` 求值并返回其结果，否则对 ``c`` 求值并返回其结果。 ``a`` 的求值结果必须是布尔值。
* ``if(a, b)`` 与 ``if(a, b, null)`` 等价。
* ``cond(a1, b1, a2, b2, ...)`` 首先对 ``a1`` 求值，若其值为真，则对 ``b1`` 求值并返回其结果，否则对 ``a2`` 及 ``b2`` 进行相同操作，直到有结果返回。此构造必须给出偶数个参数，且所有的 ``a`` 都必须求值为布尔值。若所有的 ``a`` 的值都为假，则返回空值。若想返回非空的默认值，则可以在最后一对参数的第一个中使用真值作为参数。

------------------------------------
函数的算符表达
------------------------------------

为了书写简便，一些函数有算符表示。以下是二元算符：

* ``a && b`` 等价于 ``and(a, b)``
* ``a || b`` 等价于 ``or(a, b)``
* ``a ^ b`` 等价于 ``pow(a, b)``
* ``a ++ b`` 等价于 ``concat(a, b)``
* ``a + b`` 等价于 ``add(a, b)``
* ``a - b`` 等价于 ``sub(a, b)``
* ``a * b`` 等价于 ``mul(a, b)``
* ``a / b`` 等价于 ``div(a, b)``
* ``a % b`` 等价于 ``mod(a, b)``
* ``a >= b`` 等价于 ``ge(a, b)``
* ``a <= b`` 等价于 ``le(a, b)``
* ``a > b`` 等价于 ``gt(a, b)``
* ``a < b`` 等价于 ``le(a, b)``
* ``a == b`` 等价于 ``eq(a, b)``
* ``a != b`` 等价于 ``neq(a, b)``
* ``a ~ b`` 等价于 ``coalesce(a, b)``

二元算符的优先级如下（以行记，前面的行中所包括的算符优先级高，后面的低；同一行中所有算符有相机相同）：

* ``~``
* ``^``
* ``*``, ``/``
* ``+``, ``-``, ``++``
* ``==``, ``!=``
* ``%``
* ``>=``, ``<=``, ``>``, ``<``
* ``&&``
* ``||``

除 ``^`` 之外，所有二元算符都向左关联，即 ``a / b / c`` 。
``(a / b) / c`` 。 ``^`` 则向右关联： ``a ^ b ^ c`` 等价于 ``a ^ (b ^ c)`` 。

一元算符如下：

* ``-a`` 等价于 ``minus(a)``
* ``!a`` 等价于 ``negate(a)``

在需要改变算符顺序时，可以使用括号。括号优先级最高，其次是一元算符，最后是二元算符。

------------------------
相等与比较
------------------------

.. module:: Func.EqCmp
    :noindex:
    
.. function:: eq(x, y)

    相等。算符形式为 ``x == y`` 。两个参数如果类型不同，则结果为假值。

.. function:: neq(x, y)

    不等。算符形式为 ``x != y`` 。两个参数如果类型不同，则结果为真值。

.. function:: gt(x, y)

    大于。算符形式为 ``x > y`` 。

.. function:: ge(x, y)

    大于等于。算符形式为 ``x >= y`` 。

.. function:: lt(x, y)

    小于。算符形式为 ``x < y`` 。

.. function:: le(x, y)

    小于等于。算符形式为 ``x <= y`` 。

.. NOTE::

    大小比较的两个参数必须隶属于同类型，否则会报错。在 Cozo 中，整数与浮点数的运行时类型相同，都是 ``Number`` 。

.. function:: max(x, ...)

    返回参数中的最大值。所有参数都必须是数字。

.. function:: min(x, ...)

    返回参数中的最小值。所有参数都必须是数字。

------------------------
布尔函数
------------------------

.. module:: Func.Bool
    :noindex:
    
.. function:: and(...)

    接受任意个参数的合取。二元形式等价于 ``x && y`` 。

.. function:: or(...)

    接受任意个参数的析取。二元形式等价于 ``x || y`` 。

.. function:: negate(x)

    否定。等价于 ``!x`` 。

.. function:: assert(x, ...)

    若 ``x`` 为真则返回真，否则抛出异常。给出多个参数时其它参数会包含在异常中，可以作为错误信息。

------------------------
数学函数
------------------------

.. module:: Func.Math
    :noindex:
    
.. function:: add(...)

    多参数形式的加法。二元形式等价于 ``x + y`` 。

.. function:: sub(x, y)

    减法，等价于 ``x - y`` 。

.. function:: mul(...)

    多参数形式的乘法。二元形式等价于 ``x * y`` 。

.. function:: div(x, y)

    除法，等价于 ``x / y`` 。

.. function:: minus(x)

    求负，等价于 ``-x`` 。

.. function:: pow(x, y)

    ``x`` 的 ``y`` 次方。等价于 ``x ^ y`` 。返回浮点数，即使参数都是整数。

.. function:: mod(x, y)

    ``x`` 对 ``y`` 求模（余数）。参数可以是浮点数。返回的值的符号与 ``x`` 相同。等价于 ``x % y`` 。

.. function:: abs(x)

    绝对值。

.. function:: signum(x)

    返回 ``1`` 、 ``0`` 或 ``-1`` 中与所传参数符号一样的数，比如 ``signum(to_float('NEG_INFINITY')) == -1`` ， ``signum(0.0) == 0`` ，但是 ``signum(-0.0) == -1`` 。如果参数为 ``NAN`` 则返回 ``NAN`` 。

.. function:: floor(x)

    向下求整。

.. function:: ceil(x)

    向上求整。

.. function:: round(x)

    四舍五入。当遇到点五时，取离 0 远的值，如 ``round(0.5) == 1.0`` ， ``round(-0.5) == -1.0`` ， ``round(1.4) == 1.0`` 。

.. function:: exp(x)

    指数函数，以自然对数 e 为底。

.. function:: exp2(x)

    指数函数，以 2 为底。即使参数是整数也返回浮点数。

.. function:: ln(x)

    对数函数，以自然对数为底。

.. function:: log2(x)

    对数函数，以 2 为底。

.. function:: log10(x)

    对数函数，以 10 为底。

.. function:: sin(x)

    正弦函数。

.. function:: cos(x)

    余弦函数。

.. function:: tan(x)

    正切函数。

.. function:: asin(x)

    正弦函数的反函数。

.. function:: acos(x)

    余弦函数的反函数。

.. function:: atan(x)

    正切函数的反函数。

.. function:: atan2(x, y)

    正切函数的反函数，同时传入两个参数，对这两个参数的比做反正切，并使用这两个参数的符号来决定返回值的象限。

.. function:: sinh(x)

    双曲正弦函数。

.. function:: cosh(x)

    双曲余弦函数。

.. function:: tanh(x)

    双曲正切函数。

.. function:: asinh(x)

    双曲正弦函数的反函数。

.. function:: acosh(x)

    双曲余弦函数的反函数。

.. function:: atanh(x)

    双曲正切函数的反函数。

.. function:: deg_to_rad(x)

    将角度转换为弧度。

.. function:: rad_to_deg(x)

    将弧度转换为角度。

.. function:: haversine(a_lat, a_lon, b_lat, b_lon)

    给出球面上两点的两对经纬度，使用 `半正矢公式 <https://baike.baidu.com/item/%E5%8D%8A%E6%AD%A3%E7%9F%A2>`_ 来计算他们之间的夹角。经纬度都以弧度给出。由于地图上的经纬度通常以角度给出，下一个函数更常用一些。

.. function:: haversine_deg_input(a_lat, a_lon, b_lat, b_lon)

    与上面的函数的唯一区别是经纬度参数以角度而不是弧度给出。返回的值仍然是弧度而不是角度。

    计算球面表面两点的球面距离时，将返回值乘以球的半径。比如地球的半径为 ``6371`` 公里，或 ``3959`` 英里，或 ``3440`` 海里。

    .. NOTE::

        由于地球并不是精确的球体，所以用此函数来计算距离时会有一定的误差，误差在百分之一之内。

------------------------
字符串函数
------------------------

.. module:: Func.String
    :noindex:

.. function:: length(str)

    返回字符串中含有的 Unicode 字符的数量。参数也可以是数组。

    .. WARNING::

        ``length(str)`` 返回的不是字符串的字节长度，且两个等价的 Unicode 字符串可能规范化形式不同，而导致它们的长度不同。遇到这种情况时建议使用先对字符串使用 ``unicode_normalize`` 函数来保证统一的规范化形式，然后再使用 ``length`` 函数。


.. function:: concat(x, ...)


    串联字符串。二元形式等价于 ``x ++ y`` 。参数也可以都是数组。

.. function:: str_includes(x, y)

    如果字符串 ``x`` 包含 字符串 ``y`` 的内容，则返回真，否则返回假。

.. function:: lowercase(x)

    将字符串转换为小写。支持 Unicode。

.. function:: uppercase(x)

    将字符串转换为大写。支持 Unicode。

.. function:: trim(x)

    删除字符串两头的空白字符。空白字符由 Unicode 标准定义。

.. function:: trim_start(x)

    删除字符串开头的空白字符。空白字符由 Unicode 标准定义。

.. function:: trim_end(x)

    删除字符串结尾的空白字符。空白字符由 Unicode 标准定义。

.. function:: starts_with(x, y)

    检查字符串 ``x`` 是否以 ``y`` 为前缀。

    .. TIP::

        使用 ``starts_with(var, str)`` 而不是等价的正则表达式可以帮助系统更好的优化查询：在一定情况下系统可以使用范围扫描而不是全局扫描。

.. function:: ends_with(x, y)

    检查字符串 ``x`` 是否以 ``y`` 结尾。

.. function:: unicode_normalize(str, norm)

    对字符串 ``str`` 进行 Unicode 规范化。规范化种类 ``norm`` 可以是 ``'nfc'`` 、 ``'nfd'`` 、 ``'nfkc'`` 或 ``'nfkd'`` 。

.. function:: chars(str)

    返回字符串中所含的 Unicode 字符。

.. function:: from_substrings(list)

    将一个字符串的数组组合成一个字符串。可以说是 ``chars`` 的逆函数。

    .. WARNING::

        由于 Unicode 的复杂性，Cozo 中的字符串不能以整数作为索引来查询特定位置的字符。如果查询时需要此功能，则需要先使用 ``chars`` 将其转化为数组。

--------------------------
数组函数
--------------------------

.. module:: Func.List
    :noindex:

.. function:: list(x, ...)

    将参数组成一个数组。 ``list(1, 2, 3)`` 等价于 ``[1, 2, 3]`` 。

.. function:: is_in(el, list)

    测试元素是否在数组中。

.. function:: first(l)

    提取数组中的第一个元素。空数组返回空值。

.. function:: last(l)

    提取数组中的最后一个元素。空数组返回空值。

.. function:: get(l, n)

    返回数组中索引为 ``n`` 的元素，索引为整数，从 0 开始。若索引在范围之外则报错。

.. function:: maybe_get(l, n)

    返回数组中索引为 ``n`` 的元素，索引为整数，从 0 开始。若索引在范围之外则返回空值。

.. function:: length(list)

    返回数组的长度。也可以对字节数组及字符串使用。

.. function:: slice(l, start, end)

    从索引值 ``start`` 开始（含）到索引值 ``end`` 为止（不含），取参数数组的子数组。索引值可以为负数，意义为从数组结尾开始计算的索引。例： ``slice([1, 2, 3, 4], 1, 3) == [2, 3]`` 、 ``slice([1, 2, 3, 4], 1, -1) == [2, 3]`` 。

.. function:: concat(x, ...)

    将参数数组组成一个数组。二元形式等价于 ``x ++ y`` 。参数也可以是字符串。

.. function:: prepend(l, x)

    将元素 ``x`` 插入 ``l`` 的最前端。

.. function:: append(l, x)

    将元素 ``x`` 插入 ``l`` 的最后端。

.. function:: reverse(l)

    倒转数组。

.. function:: sorted(l)

    对数组进行排序，返回排序后的结果。

.. function:: chunks(l, n)

    将数组切为长度为 ``n`` 的多个数组，最后一个数组可能长度不够，例： ``chunks([1, 2, 3, 4, 5], 2) == [[1, 2], [3, 4], [5]]`` 。

.. function:: chunks_exact(l, n)

    将数组切为长度为 ``n`` 的多个数组，如果最后一个数组长度不够则舍弃之，例： ``chunks([1, 2, 3, 4, 5], 2) == [[1, 2], [3, 4]]`` 。

.. function:: windows(l, n)

    返回数组中长度为 ``n`` 的滑动窗口，例： ``windows([1, 2, 3, 4, 5], 3) == [[1, 2, 3], [2, 3, 4], [3, 4, 5]]`` 。

.. function:: union(x, y, ...)

    返回给定参数（每个参数都代表一个集合）的联合。

.. function:: intersection(x, y, ...)

    返回给定参数（每个参数都代表一个集合）的交叉。

.. function:: difference(x, y, ...)

    返回第一个参数对其它参数（每个参数都代表一个集合）的差异。

----------------
二进制函数
----------------

.. module:: Func.Bin
    :noindex:

.. function:: length(bytes)

    返回字节数组的长度。也接受字符串及数组为参数。

.. function:: bit_and(x, y)

    返回两个字节数组比特级别的与。两个字节数组长度必须一致。

.. function:: bit_or(x, y)

    返回两个字节数组比特级别的或。两个字节数组长度必须一致。

.. function:: bit_not(x)

    返回字节数组比特级别的非。

.. function:: bit_xor(x, y)

    返回两个字节数组比特级别的排他或。两个字节数组长度必须一致。

.. function:: pack_bits([...])

    将一个包含布尔值的数组转换为一个字节数组。若参数中的数组长度不能被 8 整除，则以假值补足再转换。

.. function:: unpack_bits(x)

    将字节数组转换为布尔值的数组。

.. function:: encode_base64(b)

    将字节数组使用 Base64 编码为字符串。

    .. NOTE::
        对列进行类型转化时，若列的类型为字节数组，则会自动套用此函数。

.. function:: decode_base64(str)

    尝试将字节使用 Base64 编码解码为字节数组。


--------------------------------
类型检查与转换函数
--------------------------------

.. module:: Func.Typing
    :noindex:

.. function:: coalesce(x, ...)

    聚凝算符，即返回第一个非空的值。若所有值都为空则返回空。二元形式等价于 ``x ~ y`` 。

.. function:: to_string(x)

    将参数转换为字符串。如参数本身就是字符串，则不做变更，否则使用 JSON 的字符串表示形式。

.. function:: to_float(x)

    将参数转换为浮点数。不管参数是什么，此函数都不会抛出异常，当无法转换时会返回特殊的浮点数 ``NAN``。以下是一些可转换的特殊字符串：

    * ``INF`` 转换为正无穷大；
    * ``NEG_INF`` 转换为负无穷大；
    * ``NAN`` 转换为 ``NAN`` （两个 ``NAN`` 不相等：若要检查值是否为 ``NAN``，需要使用 ``is_nan`` 函数）；
    * ``PI`` 转换为圆周率（3.14159...）；
    * ``E`` 转换为自然对数的底（欧拉常数之一，2.71828...）。

    空值与假值转换为 ``0.0`` ，真值转换为 ``1.0`` 。

.. function:: to_int(x)

    将参数转换为整数。当参数为有效性时，提取有效性中的整数时间戳。

.. function:: to_unity(x)

    将参数转换为 ``0`` 或 ``1`` ：空值、假值、 ``0`` 、 ``0.0`` 、 ``""`` 、 ``[]`` 、空字节数组转换为 ``0`` ，其余都转换为 ``1`` 。

.. function:: to_bool(x)

    将参数转换为布尔值。以下转换为假值，其他所有值转换为真值：

    * ``null``
    * ``false``
    * ``0`` ， ``0.0``
    * ``""`` 空字符串
    * 空字节数组
    * 空 UUID （所有字节都为 0）
    * ``[]`` 空数组
    * 所有行为值为假的有效性

.. function:: to_uuid(x)

    将参数转换为 UUID。如果参数不是 UUID 或合法的 UUID 字符串表示，则报错。

.. function:: uuid_timestamp(x)

    从 UUID v1 中提取时间戳的浮点数，以秒为单位。如果 UUID 版本不是 1，则返回空值。若参数不是 UUID 则报错。

.. function:: is_null(x)

    测试参数是否为空值。

.. function:: is_int(x)

    测试参数是否为整数。

.. function:: is_float(x)

    测试参数是否为浮点数。

.. function:: is_finite(x)

    测试参数是否为有限的数字。

.. function:: is_infinite(x)

    测试参数是否为无穷的浮点数。

.. function:: is_nan(x)

    测试参数是否是特殊的浮点数 ``NAN`` 。

.. function:: is_num(x)

    测试参数是否为数字。

.. function:: is_bytes(x)

    测试参数是否为字节数组。

.. function:: is_list(x)

    测试参数是否为数组。

.. function:: is_string(x)

    测试参数是否为字符串。

.. function:: is_uuid(x)

    测试参数是否为 UUID。

-----------------
随机函数
-----------------

.. module:: Func.Rand
    :noindex:

.. function:: rand_float()

    返回在闭区间 [0, 1] 内均匀采样的浮点数。

.. function:: rand_bernoulli(p)

    返回随机的布尔值，以几率 ``p`` 返回真值。

.. function:: rand_int(lower, upper)

    返回所给闭区间内的随机整数，均匀采样。

.. function:: rand_choose(list)

    随机返回数组中的一个元素，随机采样。若数组为空则返回空值。

.. function:: rand_uuid_v1()

    生成一个随机的 UUID v1（包含当前时间戳）。在浏览器中的时间戳精度比原生程序的低很多。

.. function:: rand_uuid_v4()

    生成一个随机的 UUID v4。

------------------
正则表达式函数
------------------

.. module:: Func.Regex
    :noindex:

.. function:: regex_matches(x, reg)

    测试字符串能否被正则表达式匹配。

.. function:: regex_replace(x, reg, y)

    将字符串 ``x`` 中被正则表达式匹配上的第一处替换为 ``y`` 。

.. function:: regex_replace_all(x, reg, y)

    将字符串 ``x`` 中被正则表达式匹配上的所有地方都替换为 ``y`` 。

.. function:: regex_extract(x, reg)

    将字符串中所有被正则表达式匹配上的地方放在一个数组中返回。

.. function:: regex_extract_first(x, reg)

    返回字符串中被正则表达式匹配上的第一处。如果没有匹配则返回空值。


^^^^^^^^^^^^^^^^^
正则表达式语法
^^^^^^^^^^^^^^^^^

单个字符：
::
    .             除了换行之外的任何字符
    \d            数字 (\p{Nd})
    \D            非数字
    \pN           单个字母表示的 Unicode 字符类
    \p{Greek}     Unicode 字符类
    \PN           单个字母表示的 Unicode 字符类的补集
    \P{Greek}     Unicode 字符类的补集

字符集：
::
    [xyz]         单个字符 x 或 y 或 z
    [^xyz]        除了 x 、 y 、 z 以外的所有单个字符
    [a-z]         在 a-z 范围内的单个字符
    [[:alpha:]]   ASCII 字符类（[A-Za-z]）
    [[:^alpha:]]  ASCII 字符类的补集（[^A-Za-z]）
    [x[^xyz]]     包含潜逃的字符类
    [a-y&&xyz]    交集（匹配 x 或 y）
    [0-9&&[^4]]   使用交集与补集来做差异
    [0-9--4]      差异（匹配 0-9，但是 4 除外）
    [a-g~~b-h]    对称差异（仅匹配 a 与 h）
    [\[\]]        字符集中的转义（匹配 [ 或 ]）

组合：
::
    xy    串联（x 后面紧接着 y）
    x|y   交替（x 或者 y，都可以的时候优先 x）

重复：
::
    x*        零或多个 x（贪婪匹配）
    x+        一或多个 x（贪婪匹配）
    x?        零或一个 x（贪婪匹配）
    x*?       零或多个 x（惰性匹配）
    x+?       一或多个 x（惰性匹配）
    x??       零或一个 x（惰性匹配）
    x{n,m}    至少 n 个，至多 m 个 x（贪婪匹配）
    x{n,}     至少 n 个 x（贪婪匹配）
    x{n}      正好 n 个 x（贪婪匹配）
    x{n,m}?   至少 n 个，至多 m 个 x（惰性匹配）
    x{n,}?    至少 n 个 x（惰性匹配）
    x{n}?     正好 n 个 x（惰性匹配）

空匹配::
    ^     文本起始处
    $     文本结束处
    \A    仅文本起始处
    \z    仅文本结束处
    \b    Unicode 词语边界（以 \w 开始，以 \W、\A 或 \z 结束）
    \B    不是 Unicode 词语边界


--------------------
时间戳函数
--------------------

.. function:: now()

    返回当前的 UNIX 时间戳（以秒计，浮点数）。浏览器中的精度比原生程序的低得多。

.. function:: format_timestamp(ts, tz?)

    将浮点数 UNIX 时间戳 ``ts`` （以秒计）根据 RFC 3339 标准转换为字符串。若 ``ts`` 为有效性，则使用其中以微秒计的整数时间戳。

    可选的第二个参数指定字符串显示的市区，格式为 UNIX 系统中的格式。

.. function:: parse_timestamp(str)

    根据 RFC 3339 标准将字符串转换为浮点数时间戳（以秒计）。