==============================
工具与算法
==============================

固定规则包括工具与算法两种。

只有在编译时启用了 ``graph-algo`` 选项时，以下叙述的算法规则才可用。目前除了浏览器 WASM 之外的所有预编译发布都启用了此选项。

.. module:: Algo
    :noindex:


-------------------
工具
-------------------

.. function:: Constant(data: [...])

    将传入的数组作为表输出。常量规则 ``?[] <- ...`` 其实是 ``?[] <~ Constant(data: ...)`` 的语法糖。

    :param data: 一个元素为数组的数组，用来表示所需的表。

.. function:: ReorderSort(rel[...], out: [...], sort_by: [...], descending: false, break_ties: false, skip: 0, take: 0)

    将传入的规则 ``rel`` 所代表的表排序，然后提取其一些列输出为新的表。

    :param required out: 一个由表达式元素组成的数组，每个元素会依次成为输出表的一列。表达式中的变量会绑定为 ``rel`` 参数中给出的变量。
    :param sort_by: 一个由表达式组成的数组，使用这些表达式生成的值来对 ``rel`` 中的行进行排序。表达式中的变量会绑定为 ``rel`` 参数中给出的变量。
    :param descending: 是否以倒序排序。默认否。
    :param break_ties: 排序时同序的行是否给出不同的行号，即两行序列相同时，行号是 ``1`` 和 ``2`` 还是 ``1`` 和 ``1`` 。默认否，即给出相同的行号。
    :param skip: 输出时跳过多少行。默认为 0.
    :param take: 最多输出多少行。0 表示不做限制。默认为不限制。
    :return: 返回表的列除了 ``out`` 中声明的列之外，还会在前面添加序号列。序号从 1 开始。

    .. TIP::

        此工具与 ``:order`` 、 ``:limit`` 、 ``:offset`` 查询选项功能有一定重合，但是这个工具可以运用于任意规则，而查询选项只能运用于入口规则。在应用于入口规则时，建议使用查询选项。

.. function:: CsvReader(url: ..., types: [...], delimiter: ',', prepend_index: false, has_headers: true)

    从文件或 GET HTTP 请求获取 CSV 格式的数据，并将其转化为表。

    浏览器中的 WASM 实现中没有此工具。另外，如果编译时没有打开 ``requests`` 选项，则不能从网络读取数据。

    :param required url: CSV 文件的 URL。本地文件请使用 ``file://<文件路径>`` 格式的 URL。
    :param required types: 元素为字符串的数组，每个字符串表示一列的类型。与数据库中其它应用类型的地方不同，如果某一列类型为可空类型，而对文件中的数据应用对应的类型转换失败时，此工具会输出空值而不是报错。这么设计的原因是常见的生成 CSV 的程序输出的坏值太多了，如果碰到坏值就报错，那么这个工具其实就没多大作用了。
    :param delimiter: CSV 文件中的分隔符。
    :param prepend_index: 若为真，则输出的第一列为行号。
    :param has_headers: CSV 文件的第一行是否是文件头。若为真，则程序直接跳过第一行，不会对其做任何解析。

.. function:: JsonReader(url: ..., fields: [...], json_lines: true, null_if_absent: false, prepend_index: false)

    从文件或 GET HTTP 请求获取 JSON 格式的数据，并将其转化为表。
    
    浏览器中的 WASM 实现中没有此工具。另外，如果编译时没有打开 ``requests`` 选项，则不能从网络读取数据。

    :param required url: JSON 文件的 URL。本地文件请使用 ``file://<文件路径>`` 格式的 URL。
    :param required fields: 元素为字符串的数组，系统会将 JSON 对象中这些名称的字段提取为列。
    :param json_lines: 若真，则文件应每行包含一个 JSON 对象，否则文件应包含一个大数组，数组中包含多个对象。
    :param null_if_absent: 若真，则所需的字段不存在时默认使用空值，若假，则遇到这种情况时报错。
    :param prepend_index: 若真，则输出的第一列为行号。

------------------------------------
连通性算法
------------------------------------

.. function:: ConnectedComponents(edges[from, to])

    根据边算出顶点的 `连通分量 <https://baike.baidu.com/item/%E8%BF%9E%E9%80%9A%E5%88%86%E9%87%8F/290350>`_ 。

    :return: 第一列为顶点，第二列为其所在连通分量的编号。


.. function:: StronglyConnectedComponent(edges[from, to])

    根据边算出顶点的 `强连通分量 <https://baike.baidu.com/item/%E5%BC%BA%E8%BF%9E%E9%80%9A%E5%88%86%E9%87%8F>`_ 。

    :return: 第一列为顶点，第二列为其所在连通分量的编号。

.. function:: SCC(...)

    见 :func:`Algo.StronglyConnectedComponent` 。

.. function:: MinimumSpanningForestKruskal(edges[from, to, weight?])

    在给出的边上运行 `克鲁斯卡尔算法 <https://baike.baidu.com/item/%E5%85%8B%E9%B2%81%E6%96%AF%E5%8D%A1%E5%B0%94%E7%AE%97%E6%B3%95>`_ 来求 `最小生成树 <https://baike.baidu.com/item/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91>`_ 。边的权重可为负。

    :return: 第一、二列表示一个边，第三列是从树根到第二列顶点的距离。具体哪个顶点会被选为根是不固定的。如果有多个根，则表明图不是连通的。

.. function:: MinimumSpanningTreePrim(edges[from, to, weight?], starting?[idx])

    在给出的边上运行 `普里姆算法 <https://baike.baidu.com/item/Prim>`_ 来求 `最小生成树 <https://baike.baidu.com/item/%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91>`_ 。 ``starting`` 应为一个只有一行、一列的表，其值会作为树根。只有与树根连接的顶点才会被返回。若没有给出 ``starting`` ，则在图不连通时不一定会返回哪个分量。

    :return: 第一、二列表示一个边，第三列是从树根到第二列顶点的距离。

.. function:: TopSort(edges[from, to])

    对所给出的边中的顶点进行 `拓扑排序 <https://baike.baidu.com/item/%E6%8B%93%E6%89%91%E6%8E%92%E5%BA%8F>`_ 。给出的边必须组成一个连通的图。

    :return: 第一列为排序后的序号，第二列为顶点。

------------------------------------
寻路算法
------------------------------------

.. function:: ShortestPathBFS(edges[from, to], starting[start_idx], goals[goal_idx])

    在所给出的边上进行宽度优先搜索，来找出 ``starting`` 中的顶点与 ``goals`` 中的顶点的最短路径。给出的边是有向图的边，每个边的权重都为 1。若有多条最短路径，则返回任意一条。这是最简单的寻路算法：下面有更多的应用更广的寻路算法。

    :return: 第一列为起点，第二列为终点，第三列为最短路径。

.. function:: ShortestPathDijkstra(edges[from, to, weight?], starting[idx], goals[idx], undirected: false, keep_ties: false)

    在给出的边上运行 `戴克斯特拉算法 <https://baike.baidu.com/item/%E6%88%B4%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95/22361204>`_ ，以找出 ``starting`` 中的节点与 ``goals`` 中的节点的最短路径。若给出了权重，则权重必须非负。

    :param undirected: 若真，则给出的边为无向图的边，否则为有向图的边。默认为有向图的边。
    :param keep_ties: 当有多条最短路径时，是否返回所有的最短路径。默认为否，也就是仅返回其中某一条。
    :return: 第一列为起点，第二列为终点，第三列为最短路径的总权重，第四列为最短路径。

.. function:: KShortestPathYen(edges[from, to, weight?], starting[idx], goals[idx], k: expr, undirected: false)

    在给出的边上运行 Yen 算法来找出连接每对 ``starting`` 中的顶点与 ``goals`` 中的顶点的最短的 k 条路径。

    :param required k: 每对顶点返回多少条路径。
    :param undirected: 若真，则给出的边为无向图的边，否则为有向图的边。默认为有向图的边。
    :return: 第一列为起点，第二列为终点，第三列为最短路径的总权重，第四列为最短路径。

.. function:: BreadthFirstSearch(edges[from, to], nodes[idx, ...], starting?[idx], condition: expr, limit: 1)

    在所给的边上运行宽度优先搜索，从 ``starting`` 中的顶点开始搜索。若 ``starting`` 未给出，则默认为边中所包含的所有顶点（计算量可能会非常大）。

    :param required condition: 表示停止搜索条件的表达式。表达式中的变量绑定为 ``nodes`` 参数给出的变量。表达式的值应为布尔值，当值为真时表示找到了所需结果。
    :param limit: 找到多少个所需结果后停止搜索。默认为 1。
    :return: 第一列为起点，第二列为终点，第三列为找到的路径。

.. function:: BFS(...)

    见 :func:`Algo.BreadthFirstSearch` 。


.. function:: DepthFirstSearch(edges[from, to], nodes[idx, ...], starting?[idx], condition: expr, limit: 1)

    在所给的边上运行深度优先搜索，从 ``starting`` 中的顶点开始搜索。若 ``starting`` 未给出，则默认为边中所包含的所有顶点（计算量可能会非常大）。

    :param required condition: 表示停止搜索条件的表达式。表达式中的变量绑定为 ``nodes`` 参数给出的变量。表达式的值应为布尔值，当值为真时表示找到了所需结果。
    :param limit: 找到多少个所需结果后停止搜索。默认为 1。
    :return: 第一列为起点，第二列为终点，第三列为找到的路径。

.. function:: DFS(...)

    见 :func:`Algo.DepthFirstSearch` 。

.. function:: ShortestPathAStar(edges[from, to, weight], nodes[idx, ...], starting[idx], goals[idx], heuristic: expr)

    在给出的边上执行 `A\* 算法 <https://baike.baidu.com/item/A%2A%E7%AE%97%E6%B3%95/215793>`_ ，以找出 ``starting`` 中每个顶点到 ``goals`` 中每个顶点的最短路径。给出的边 ``edges`` 必须是有向图的边，且每条边都有非负的权重值。

    :param required heuristic: 启发式的表达式。表达式中的变量将会绑定为 ``goals`` 与 ``nodes`` 参数中给出的变量。启发式求值后应得到一个数值，这个数值应是当前顶点到当前终点的最短路径权重的一个下限。若启发式求值后的数值不是下限，则算法可能会返回错误的结果。

    :return: 第一列为起点，第二列为终点，第三列为最短路径的总权重，第四列为最短路径。

    .. TIP::

        A\* 算法的性能受启发式的影响极大，好的启发式会大大提速算法。由于边的权重非负， ``0`` 是一个合法的启发式，但在这种情况下实际上应该使用戴克斯特拉算法。

        很多好的启发式实际上是由顶点所在空间的距离函数（度量）所决定的测地线长度，比如说球面上两个点之间的测度线， 在这时 :func:`Func.Math.haversine_deg_input` 函数可以用来表示启发式（但注意半径的单位一定要和数据中的单位匹配）。另一个例子是曼哈顿网格空间中的最短距离。

        虽然给出不满足条件的启发式可能会得出错误的结果，但误差的上限决定于启发式对于实际最短距离高估的值。如果这个值不大的话，在一些场景下错误的结果也是可以用的。

-------------------------------------
社区发现算法
-------------------------------------

.. function:: ClusteringCoefficients(edges[from, to, weight?])

    根据所给出的边，计算其中所含每个顶点的聚合系数。

    :return: 第一列为顶点，第二列为聚合系数，第三列为包含此顶点的三角形的数量，第四列为包含该顶点的边的数量。

.. function:: CommunityDetectionLouvain(edges[from, to, weight?], undirected: false, max_iter: 10, delta: 0.0001, keep_depth?: depth)

    在给出的边上运行 Louvain 算法找出社区结构。

    :param undirected: 图是否是无向图。默认为否，即有向图。
    :param max_iter: 在算法的每个纪元内运行的最大迭代次数。默认为 10。
    :param delta: 模块性变更多少以上才认为其代表了有效的社区效应。
    :param keep_depth: 返回多少层社区。默认返回所有层级。
    :return: 第一列是表示社区的一个数组，第二列是包含在这个社区中的一个顶点。数组的长度小于等于要求输出的社区层数，其结构表示社区与子社区的内部结构。

.. function:: LabelPropagation(edges[from, to, weight?], undirected: false, max_iter: 10)

    在所给出的边上运行 `标签传播算法 <https://baike.baidu.com/item/%E6%A0%87%E7%AD%BE%E4%BC%A0%E6%92%AD%E7%AE%97%E6%B3%95/2497898>`_ 来找出社区。

    :param undirected: 图是否是无向图。默认为否，即有向图。
    :param max_iter: 最大迭代次数。默认为 10。
    :return: 第一列是社区的标签（整数），第二列是这个社区包含的一个顶点。

-------------------------------------
中心度量算法
-------------------------------------

.. function:: DegreeCentrality(edges[from, to])

    使用给出的边计算其中所含顶点的度中心性。这个没有任何复杂计算，因此非常快，在拿到新图想要了解其结构时第一个运行的应该就是此算法。

    :return: 第一列是顶点，第二列是包含此顶点的边的数量，第三列是由此顶点出发的边的数量，第四列是到达此顶点的边的数量。

.. function:: PageRank(edges[from, to, weight?], undirected: false, theta: 0.85, epsilon: 0.0001, iterations: 10)

    在所给的边上运行 `佩奇排名 <https://baike.baidu.com/item/google%20pagerank?fromtitle=pagerank&fromid=111004>`_ 算法。

    :param undirected: 图是否是无向图。默认为否，即有向图。
    :param theta: 0 与 1 之间的一个小数，表示显性给出的边在（假设的）所有边中的比例。默认为 0.8。为 1 时表明严格意义上网络中不存在未发现、未给出的隐藏边。
    :param epsilon: 迭代中最小变化阈值。Pagerank 值的提升如果低于这个阈值，则认为迭代已经收敛。默认为 0.05。
    :param iterations: 最大迭代次数。若提前收敛，则在此次数到达前返回结果。默认为 20。

    :return: 第一列为顶点，第二列为佩奇排名的值。

.. function:: ClosenessCentrality(edges[from, to, weight?], undirected: false)

    使用给出的边计算其中所含顶点的临近中心性。边可以同时给出权重。

    :param undirected: 图是否是无向图。默认为否，即有向图。
    :return: 第一列为顶点，第二列为临近中心性。

.. function:: BetweennessCentrality(edges[from, to, weight?], undirected: false)

    使用给出的边计算其中所含顶点的介中心性。边可以同时给出权重。

    :param undirected: 图是否是无向图。默认为否，即有向图。
    :return: 第一列为顶点，第二列为介中心性。

    .. WARNING::

        此算法复杂度非常高，因此无法在大中型网络上运行。对于大中型网络，建议先使用社区发现算法将其聚合为中小型的超级网络，再运行此算法。

------------------
杂项
------------------

.. function:: RandomWalk(edges[from, to, ...], nodes[idx, ...], starting[idx], steps: 10, weight?: expr, iterations: 1)

    在给出的边上进行随机游走，起点为 ``starting`` 中的顶点。

    :param required steps: 游走步数。若走入了死胡同，则实际返回的步数可能会比这个值短。
    :param weight: 表达式，其中变量绑定为 ``nodes`` 与 ``edges`` 参数中给出的变量，每次执行时分别代表了当前所在顶点与下一步可选的边。应返回非负浮点数，代表这条边的权重，权重越大选择这条边的几率越大。若未给出此选项，则每步都对所有可选边均匀抽样。
    :param iterations: 由 ``starting`` 中的每个顶点开始随机游走的次数。
    :return: 第一列是序号，第二列是开始的节点，第三列是游走的路线（表示为包含节点的一个数组）。
