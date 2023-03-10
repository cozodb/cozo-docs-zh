====================================
存储表与事务
====================================

Cozo 使用 **存储表** 来存数据。

---------------------------
存储表
---------------------------

查询存储表时，使用类如 ``*relation[...]`` 或 ``*relation{...}`` 的原子应用（ :doc:`上一章 <queries>` 中已详述）。以下查询选项可用来操作存储表的读写：

.. module:: QueryOp
    :noindex:

.. function:: :create <NAME> <SPEC>

    创建一个名为 ``<NAME>`` 的新存储表，使用 ``<SPEC>`` 作为其列定义。若已有同名的存储表存在，则报错。若同时也给出了 ``?`` 入口规则，则该规则中的数据会被在创建表时插入。这是唯一一个可以省略入口规则的查询选项。

.. function:: :replace <NAME> <SPEC>

    功能与 ``:create`` 类似，也是创建存储表，其区别在于如果已有重名的表存在，则重名的表（包括其中数据）会被删除，然后新表会被建立。若重名表有关联的触发器，则这些触发器会被关联到新表上，即使新表的列定义不同（这可能会使执行触发器时报错，需要手动调整）。使用 ``:replace`` 时入口规则不可省略。

.. function:: :put <NAME> <SPEC>

    将入口查询的表中各行插入名为 ``<NAME>`` 的存储表。若存储表中已有同键的数据，则会被新插入的数据覆盖。

.. function:: :rm <NAME> <SPEC>

    将入口查询表中所有键从名为 ``<NAME>`` 的存储表中删除。在 ``<SPEC>`` 中，只需要声明键的列，不需要值的列。需要删除的键即使在表中不存在也不会报错。

.. function:: :ensure <NAME> <SPEC>

    检查入口查询表中的所有键值都在名为 ``<NAME>`` 的存储表中存在，且当前事务从开始到提交这段时间内没有其他事务更改过这些键值。主要用来保证一些查询的读写一致性。

.. function:: :ensure_not <NAME> <SPEC>

    检查入口查询表中的所有键值都不存在于名为 ``<NAME>`` 的存储表中，且当前事务从开始到提交这段时间内没有其他事务更改过这些键。主要用来保证一些查询的读写一致性。

如果需要删除存储表，需要使用系统操作 ``::remove``；如果需要修改存储表的名称，使用系统操作 ``:rename``。在 :doc:`sysops` 一章中有论述。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
创建与覆盖
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

以上各选项中的列定义 ``<SPEC>`` 格式都相同，但语义略有差异。以下我们先介绍其在 ``:create`` 和 ``:replace`` 中的语义。

列定义最外层是花括号 ``{}`` ，其中每列的定义由逗号隔开，如下例：
::

    ?[address, company_name, department_name, head_count] <- $input_data

    :create dept_info {
        company_name: String,
        department_name: String,
        =>
        head_count: Int,
        address: String,
    }

``=>`` 符号之前的列组成存储表的 **键** ，之后的组成 **值** 。如果所有列都是键，则符号 ``=>`` 可省略。列的顺序，尤其是键列的顺序，是很重要的：数据按照键列的字典排序顺序存在数据库的存储中。

在以上例子中，我们对每一列都声明了类型。如果存入的行中数据的类型与声明的类型不同，系统会先尝试进行类型转换，如果不成功，则报错。如果省略类型声明，则默认的类型为 ``Any?`` ，可存入任何数据。举例来说，上面的例子将所有类型省略，我们就得到：
::

    ?[address, company_name, department_name, head_count] <- $input_data

    :create dept_info { company_name, department_name => head_count, address }

在例子中，入口的绑定变量与列名相同（虽然顺序不同）。如果不同，我们可以指定每列对应的入口绑定：
::

    ?[a, b, count(c)] <- $input_data

    :create dept_info {
        company_name = a,
        department_name = b,
        =>
        head_count = count(c),
        address: String = b
    }

如果入口绑定的变量含有聚合操作算符，则必须显性地指定对应关系，因为诸如 ``count(c)`` 的入口绑定不是合法的列名。另外在上例 ``address`` 列中，我们也可以看到如何同时声明类型和绑定对应。

也可以使用 ``default`` 给列声明默认值：
::

    ?[a, b] <- $input_data

    :create dept_info {
        company_name = a,
        department_name = b,
        =>
        head_count default 0,
        address default ''
    }

默认值可以是一个表达式，这个表达式会对插入的每行重新执行。因此如果默认值是一个生成随机 UUID 的表达式，那每个插入的行都会得到一个不一样的 UUID。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
增删改及约束
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

使用 ``:put`` 、 ``:remove`` 、 ``:ensure`` 、 ``:ensure_not`` 时，当某列在表创建时有默认值或这些列可为空的情况下，这些列可以在列定义中省略，而插入或删除的列值为默认值或空值。在这些操作中声明新的默认值没有任何效果。

在使用 ``:put`` 与 ``:ensure`` 时，给出的列定义，加上默认值，必须足够生成所有的键值列。

在使用 ``:rm`` 与 ``:ensure_not`` 时，给出的列定义，加上默认值，必须足够生成所有的键列（值列不需要）。

------------------------------------------------------
连锁查询
------------------------------------------------------

每个提交给数据库的查询文本都在独立的 **事务** 中执行。当需要保证多个操作能够原子的执行时，可以将多个查询放在同一个查询文本中，这时以花括号 ``{}`` 将每个查询包裹起来。每个查询都可以有自己独立的查询选项。执行时，提交的多个查询按照顺序依次执行，直到最后一个查询成功完成，或某个查询报错。整个查询文本的返回结果是最后一个查询的结果。

你可以使用 ``:assert (some|none)`` 、 ``:ensure`` 、 ``:ensure_not`` 这些查询选项来表述事务提交时必须满足的约束条件。

在下例中，我们同时提交了三个查询，这三个查询要么全部成功并将修改写入数据库，要么某个失败而数据库不写入任何数据，且保证在查询提交时有一行数据存在于存储表中：
::

    {
        ?[a, b] <- [[1, 'one'], [3, 'three']]
        :put rel {a => b}
    }
    {
        ?[a] <- [[2]]
        :rm rel {a}
    }
    {
        ?[a, b] <- [[4, 'four']]
        :ensure rel {a => b}
    }

查询事务开始执行时，数据库会对所有数据进行快照，任何对数据库的读行为都只会从快照及当前的更改中获取数据。这意味着在查询中查到的数据要么在查询开始前就已经提交至数据库，要么是当前查询文本修改过的数据，不会查到事务开始后其它事务写入的数据。当前数据提交时，如果多个事务提交了互相矛盾的数据，则会报错。如果写入存储表时激活了这些表的触发器，这些触发器也会在同一个事务中执行。

实际上连锁查询功能本身是由一个迷你语言实现的，这个迷你语言里面的 **表达式** 就是一个个花括号隔开的完成的查询，上面的例子都是由一连串的表达式组成的。这个迷你语言里还有其他语句：

* ``%if <cond> %then ... (%else ...) %end`` 选择性地执行分支。另外还有以 ``%if_not`` 开头的否定形式。 ``<cond>`` 可以是一个表达式，也可以是一个临时表，不管是哪种，总之结果是一个表。如果这个表的第一行的最后一列在执行 ``to_bool`` 函数后为真值，则这个表被认为是真值，如果这个表为空，或其第一行最后一列在执行 ``to_bool`` 后为假值，则这个表被认为是假值。

* ``%loop ... %end`` 用来循环。在循环中你可以使用 ``%break`` 与 ``%continue`` 。你可以在 ``%loop`` 前面加上 ``%mark <marker>``，然后使用 ``%break <marker>`` 或 ``%continue marker`` 来跨层级次跳跃。

* ``%return <表达式或临时表或空>`` 立即返回结果。

* ``%debug <临时表>`` 打印临时表内容到标准输出。

* ``%ignore_error <表达式>`` 执行表达式，忽略所有错误。

* ``%swap <临时表> <另一个临时表>`` 交换两个临时表。

上面所说的 **临时表** 是什么呢？临时表只在事务执行的时候存在，只能被当前事务读写（所以在单个查询中使用临时表没有意义）。创建和修改临时表的方式与存储表相同，除了表的名字以下划线 ``_`` 开头以外。临时表可以说是连锁查询迷你语言中的 **变量**。

让我们举几个例子：
::

    {:create _test {a}}

    %loop
        %if { len[count(x)] := *_test[x]; ?[x] := len[z], x = z >= 10 }
            %then %return _test
        %end
        { ?[a] := a = rand_uuid_v1(); :put _test {a} }
    %end

这里返回的表含有十行随机数据。注意这里生成随机数据 **必须** 使用内联规则。如果使用常量规则，则生成的“随机数”每次都一样：常量规则的正文只会被执行一次，因此会造成此查询死循环。

第二个：
::

    {?[a] <- [[1], [2], [3]]; :replace _test {a}}

    %loop
        { ?[a] := *_test[a]; :limit 1; :rm _test {a} }
        %debug _test

        %if_not _test
        %then %break
        %end
    %end

    %return _test

返回的表为空（非常牵强的从表中删除行的方式）。

最后：
::

    {?[a] <- [[1], [2], [3]]; :replace _test {a}}
    {?[a] <- []; :replace _test2 {a}}
    %swap _test _test2
    %return _test

返回的表也为空，因为两个表被交换了。

这个迷你语言的主要目的是为了可以快速写一些简单的循环算法。当然，因为 Cozo 的 Datalog 本来就是图灵完备的，不用这个迷你语言也可以写任何算法，但是可以不代表好写，也不代表能跑得快。比如说，你可以尝试用基本的查询语言来写佩奇指数算法，你会发现需要使用大量的递归和聚合。但是如果用了这个迷你语言，写起来就很简单。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
多语句事务
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cozo 的部分库（目前包括 Rust、Python、及 NodeJS 的库）以及独立程序同时支持多语句事务：在执行查询时，首先开启一个事务，然后对这个事务进行查询以及修改，最后提交或回滚事务。多语句事务比连锁查询更加灵活，但是使用方法根据不同语言是不同的。请查阅各个语言自己的 Cozo 文档来确定具体如何使用此功能。

------------
索引
------------

版本号 0.5 之后的 Cozo 支持为存储表建立关联的索引。在 Cozo 中，索引就是存储表中列的不同排列。如果我们有以下存储表：
::

    :create r {a => b}

但我们经常需要执行查询 ``?[a] := *r{a, b: $value}`` 。如果不使用索引，则查询需要扫描整个表。我们用下列语句建立索引：
::

    ::index create r:idx {b, a}

注意在建立索引时 **不要** 声明函数式依赖（这个例子中的索引也没有函数式依赖）。

Cozo 中的索引就是只读的存储表，因此可以直接查询：
::

    ?[a] := *r:idx {a, b: $value}

在此例子中，之前的查询 ``?[a] := *r{a, b: $value}`` 实际上也会被改写成与上面显性的索引查询相同的执行方案（你可以使用 ``::explain`` 语句来确定这一点）。但是一般来说，Cozo 在决定是否使用索引来改写查询时极端保守：只要有任何可能性使用了索引反而导致性能下降，则 Cozo 不会使用索引。目前来说，只有当使用索引可以避免扫描整个表时 Cozo 才会自动使用索引。这种策略保证了添加索引在任何情况下都不会降低性能。如果你知道一些查询使用索引会更快，但是数据库没有自动改写，你只需要手动使用索引即可。这比使用各种技巧来说服数据库不要使用某个不该使用的索引简单得多。

删除索引：
::

    ::index drop r:idx

在 Cozo 中，创建索引时不需要使用所有的列。但是当所声明的列不包括所有的键列时，系统会自动补齐，也就是说，如果你的表是
::

    :create r {a, b => c, d, e}

而你要求建立如下索引：
::

    ::index create r:i {d, b}

则数据库实际上会建立的索引是：
::

    ::index create r:i {d, b, a}

你可以查看数据库到底建立了哪些列： ``::columns r:i`` 。

索引可以作为固定规则的输入表。如果索引的最后一列的类型为 ``Validity``，则也可以对索引进行历史穿梭查询。

------------------------------------------------------
触发器
------------------------------------------------------

Cozo 支持在存储表上绑定触发器：使用 ``::set_triggers`` 系统操作来设置一个存储表的触发器：
::

    ::set_triggers <REL_NAME>

    on put { <QUERY> }
    on rm { <QUERY> }
    on replace { <QUERY> }
    on put { <QUERY> } # 可以设置任意数量任意种类的触发器

这里面 ``<QUERY>`` 可以是任何查询。

``on put`` 后面的触发器会在数据插入或覆盖后触发： ``:put`` 、 ``:create`` 、 ``:replace`` 均可触发。在触发器中，有两个隐藏的内联表 ``_new[]`` 与 ``_old[]`` 可以在查询中使用，分别包含新插入的行，以及被覆盖的行的旧值。

``on rm`` 触发器会在行被删除时触发：即 ``:rm`` 查询选项可触发。隐藏内联表 ``_new[]`` 与 ``_old[]`` 分别包含删除的键（即使此键在存储表中不存在），以及确实被删除的行的键值。

``on replace`` 触发器会在执行 ``:replace`` 查询选项时触发。此触发器触发后才会触发任何 ``on put`` 触发器。

在设置触发器的 ``::set_triggers`` 系统命令中，所有触发器必须同时一起给出，每次执行此命令会覆盖所设计存储表之前所有的触发器。执行 ``::set_triggers <REL_NAME>`` 命令但不给出任何触发器会删除存储表关联的所有的触发器。

下面给出一个使用触发器来手动建立索引的例子。假设我们有如下原始存储表：
::

    :create rel {a => b}

手动的索引表：
::

    :create rel.rev {b, a}

我们用以下触发器来保证索引表的同步性：
::

    ::set_triggers rel

    on put {
        ?[a, b] := _new[a, b]

        :put rel.rev{ b, a }
    }
    on rm {
        ?[a, b] := _old[a, b]

        :rm rel.rev{ b, a }
    }

现在索引表就建好了，在查询中我们可以使用 ``*rel.rev{..}`` 来取代 ``*rel{..}`` ，以执行对索引的查询。

注意，与自动索引不同，有部分导入数据的 API 执行时不会激活触发器。另外，如果你使用触发器来手动同步索引，你需要手动导入索引建立之前的历史数据。

.. WARNING::

    触发器 **不会** 激活其它的触发器，也就是说如果一个触发器修改了某个表，而那个表有其它的触发器，则后面这些触发器不会被运行。这个处理方式与早期的版本中的处理方式不同。我们修改了之前的处理方式，因为触发器连锁反应造成的问题比解决的问题更多。