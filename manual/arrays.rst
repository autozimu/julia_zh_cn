.. _man-arrays:

**********
 多维数组
**********

数组是一个存在多维网格中的对象集合。通常，数组包含的对象的类型为 ``Any`` 。对大多数计算而言，数组对象一般更具体为 ``Float64`` 或 ``Int32`` 。

因为性能的原因，Julia 不希望把程序写成向量化的形式。

在 Julia 中，通过引用将参数传递给函数。Julia 的库函数不会修改传递给它的输入。用户写代码时，如果要想做类似的功能，要注意先把输入复制一份儿。

数组
====

基础函数
--------

=============== ========================================================================
函数            说明
=============== ========================================================================
``eltype(A)``   A 中元素的类型
``length(A)``   A 中元素的个数
``ndims(A)``    A 有几个维度
``nnz(A)``      A 中非零元素的个数
``size(A)``     返回一个元素为 A 的维度的多元组
``size(A,n)``   A 在某个维度上的长度
``stride(A,k)`` 在维度 k 上，邻接元素（在内存中）的线性索引距离
``strides(A)``  返回多元组，其元素为在每个维度上，邻接元素（在内存中）的线性索引距离
=============== ========================================================================

构造和初始化
------------

Many functions for constructing and initializing arrays are provided. In
the following list of such functions, calls with a ``dims...`` argument
can either take a single tuple of dimension sizes or a series of
dimension sizes passed as a variable number of arguments.
下列函数中调用的 ``dims...`` 参数，既可以是维度的单多元组，也可以是维度作为可变参数时的一组值。

===================================== =====================================================================
函数                                  说明
===================================== =====================================================================
``Array(type, dims...)``              未初始化的稠密数组
``cell(dims...)``                     未初始化的元胞数组（异构数组）
``zeros(type, dims...)``              指定类型的全 0 数组
``ones(type, dims...)``               指定类型的全 1 数组
``trues(dims...)``                    全 ``true`` 的 ``Bool`` 数组
``falses(dims...)``                   全 ``false`` 的 ``Bool`` 数组
``reshape(A, dims...)``               将数组中的数据按照指定维度排列
``copy(A)``                           复制 ``A``
``deepcopy(A)``                       复制 ``A`` ，并递归复制其元素
``similar(A, element_type, dims...)`` 属性与输入数组（稠密、稀疏等）相同的未初始化数组，但指明了元素类型和维度。
                                      第二、三参数可省略，省略时默认为 ``A`` 的元素类型和维度
``reinterpret(type, A)``              二进制数据与输入数组相同的数组，但指明了元素类型
``rand(dims)``                        在 [0,1) 上独立均匀同分布的 ``Float64`` 类型的随机数组
``randf(dims)``                       在 [0,1) 上独立均匀同分布的 ``Float32`` 类型的随机数组
``randn(dims)``                       ``Float64`` 类型的独立正态同分布的随机数组，均值为 0 ，标准差为 1
``eye(n)``                            ``n`` x ``n`` 单位矩阵
``eye(m, n)``                         ``m`` x ``n`` 单位矩阵
``linspace(start, stop, n)``          从 ``start`` 至 ``stop`` 的由 ``n`` 个元素构成的线性向量
``fill!(A, x)``                       用值 ``x`` 填充数组 ``A``
===================================== =====================================================================

Comprehensions
--------------

Comprehensions 用于构造数组。它的语法类似于数学中的集合标记法： ::

    A = [ F(x,y,...) for x=rx, y=ry, ... ]

``F(x,y,...)`` 根据变量 ``x``, ``y`` 等来求值。这些变量的值可以是任何迭代对象，但大多数情况下，都使用类似于 ``1:n`` 或 ``2:(n-1)`` 的范围对象，或显式指明为类似 ``[1.2, 3.4, 5.7]`` 的数组。它的结果是 N 维稠密数组。

下例计算在维度 1 上，当前元素及左右邻居元素的加权平均数： ::

    julia> const x = rand(8)
    8-element Float64 Array:
     0.276455
     0.614847
     0.0601373
     0.896024
     0.646236
     0.143959
     0.0462343
     0.730987

    julia> [ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]
    6-element Float64 Array:
     0.391572
     0.407786
     0.624605
     0.583114
     0.245097
     0.241854

.. note:: 上例中， ``x`` 被声明为常量，因为对于非常量的全局变量，Julia 的类型推断不怎么样。

可在 comprehension 之前显式指明它的类型。如要避免在前例中声明 ``x`` 为常量，但仍要确保结果类型为 ``Float64`` ，应这样写： ::

    Float64[ 0.25*x[i-1] + 0.5*x[i] + 0.25*x[i+1] for i=2:length(x)-1 ]

使用花括号来替代方括号，可以将它简写为 ``Any`` 类型的数组： ::

    julia> { i/2 for i = 1:3 }
    3-element Any Array:
     0.5
     1.0
     1.5

.. _man-array-indexing:

索引
----

索引 n 维数组 A 的通用语法为： ::

    X = A[I_1, I_2, ..., I_n]

其中 I\_k 可以是：

1. 标量
2. 满足 ``:``, ``a:b``, 或 ``a:b:c`` 格式的 ``Range`` 对象
3. 任意整数向量，包括空向量 ``[]``
4. 布尔值向量

结果 X 的维度通常为 ``(length(I_1), length(I_2), ..., length(I_n))`` ，且 X 的索引 ``(i_1, i_2, ..., i_n)`` 处的值为 ``A[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`` 。缀在后面的标量索引的维度信息被舍弃。如，``A[I, 1]`` 的维度为 ``(length(I),)`` 。由布尔值向量索引的维度长度，是向量中 ``true`` 值的个数。

索引语法与调用 ``getindex`` 等价： ::

    X = getindex(A, I_1, I_2, ..., I_n)

例如： ::

    julia> x = reshape(1:16, 4, 4)
    4x4 Int64 Array
    1 5  9 13
    2 6 10 14
    3 7 11 15
    4 8 12 16

    julia> x[2:3, 2:end-1]
    2x2 Int64 Array
    6 10
    7 11

赋值
----

给 n 维数组 A 赋值的通用语法为： ::

    A[I_1, I_2, ..., I_n] = X

其中 I\_k 可能是：

1. 标量
2. 满足 ``:``, ``a:b``, 或 ``a:b:c`` 格式的 ``Range``  对象
3. 任意整数向量，包括空向量 ``[]``
4. 布尔值向量

X 的维度为 ``(length(I_1), length(I_2), ..., length(I_n))`` ，且 A 在 ``(i_1, i_2, ..., i_n)`` 处的值被覆写为 ``X[I_1[i_1], I_2[i_2], ..., I_n[i_n]]`` 。

索引赋值语法等价于调用 ``setindex!`` ： ::

      setindex!(A, X, I_1, I_2, ..., I_n)

例如： ::

    julia> x = reshape(1:9, 3, 3)
    3x3 Int64 Array
    1 4 7
    2 5 8
    3 6 9

    julia> x[1:2, 2:3] = -1
    3x3 Int64 Array
    1 -1 -1
    2 -1 -1
    3  6  9

连接
----

Arrays can be concatenated along any dimension using the following
functions:
使用下列函数，可在任意维度连接数组：

================ ======================================================
函数             说明
================ ======================================================
``cat(k, A...)`` 沿维度 ``k`` 连接输入的数组
``vcat(A...)``   等价于 ``cat(1, A...)``
``hcat(A...)``   等价于 ``cat(2, A...)``
``hvcat(A...)``
================ ======================================================

连接运算符也可以用来连接数组：

=================== =========
表达式              调用
=================== =========
``[A B C ...]``     ``hcat``
``[A, B, C, ...]``  ``vcat``
``[A B; C D; ...]`` ``hvcat``
=================== =========

向量化的运算符和函数
------------------

数组支持下列运算符。在使用二元运算符时，如果两个输入都是向量，应使用带“点”（逐元素）版本的运算符；如果其中一个输入是标量，两种版本的运算符都可以使用。

1.  一元： ``-``, ``+``, ``!``
2.  二元： ``+``, ``-``, ``*``, ``.*``, ``/``, ``./``,
    ``\``, ``.\``, ``^``, ``.^``, ``div``, ``mod``
3.  比较： ``==``, ``!=``, ``<``, ``<=``, ``>``, ``>=``
4.  一元布尔值或位运算： ``~``
5.  二元布尔值或位运算： ``&``, ``|``, ``$``

下列内置的函数也都是向量化的, 即函数是逐元素版本的： ::

    abs abs2 angle cbrt
    airy airyai airyaiprime airybi airybiprime airyprime
    acos acosh asin asinh atan atan2 atanh
    cos  cosh  sin  sinh  tan  tanh  sinc  cosc
    besselh besseli besselj besselj0 besselj1 besselk bessely bessely0 bessely1
    exp  erf  erfc  exp2  expm1
    beta dawson digamma erfcx erfi
    exponent eta zeta gamma
    hankelh1 hankelh2
     ceil  floor  round  trunc
    iceil ifloor iround itrunc
    isfinite isinf isnan
    lbeta lfact lgamma
    log log10 log1p log2
    copysign max min significand
    sqrt hypot

另外, Julia 提供了 ``@vectorize_1arg`` 和 ``@vectorize_2arg`` 两个宏，分别用来向量化任意的单参数或两个参数的函数。每个宏都接收两个参数，即函数参数的类型和函数名。例如： ::

    julia> square(x) = x^2
    # methods for generic function square
    square(x) at none:1
    
    julia> @vectorize_1arg Number square
    # methods for generic function square
    square{T<:Number}(x::AbstractArray{T<:Number,1}) at operators.jl:216
    square{T<:Number}(x::AbstractArray{T<:Number,2}) at operators.jl:217
    square{T<:Number}(x::AbstractArray{T<:Number,N}) at operators.jl:219
    square(x) at none:1
    
    julia> square([1 2 4; 5 6 7])
    2x3 Int64 Array:
      1   4  16
     25  36  49

Broadcasting
------------

有时要对不同维度的数组进行逐元素的二元运算，如将向量加到矩阵的每一列。低效的方法是，把向量复制成同维度的矩阵： ::

    julia> a = rand(2,1); A = rand(2,3);

    julia> repmat(a,1,3)+A
    2x3 Float64 Array:
     0.848333  1.66714  1.3262 
     1.26743   1.77988  1.13859

维度很大时，效率会很低。Julia 提供 ``broadcast`` 函数，它将数组参数的维度进行扩展，使其匹配另一个数组的对应维度，且不需要额外内存，最后再逐元素调用指定的二元函数： ::

    julia> broadcast(+, a, A)
    2x3 Float64 Array:
     0.848333  1.66714  1.3262 
     1.26743   1.77988  1.13859

    julia> b = rand(1,2)
    1x2 Float64 Array:
     0.629799  0.754948

    julia> broadcast(+, a, b)
    2x2 Float64 Array:
     1.31849  1.44364
     1.56107  1.68622

Elementwise operators such as ``.+`` and ``.*`` perform broadcasting if necessary. There is also a ``broadcast!`` function to specify an explicit destination, and ``broadcast_getindex`` and ``broadcast_setindex!`` that broadcast the indices before indexing.

实现
----

Julia 的基础数组类型是抽象类型 ``AbstractArray{T,n}`` ，其中维度为 ``n`` ，元素类型为 ``T`` 。 ``AbstractVector`` 和 ``AbstractMatrix`` 分别是它 1 维 和 2 维的别名。

``Array{T,n}`` 类型是 ``AbstractArray`` 的特殊实例，它的元素以列序为主序存储。 ``Vector`` 和 ``Matrix`` 是分别是它 1 维 和 2 维的别名。

``SubArray`` 是 ``AbstractArray`` 的特殊实例，它通过引用而不是复制来进行索引。使用 ``sub`` 函数来构造 ``SubArray`` ，它的调用方式与 ``getindex`` 相同（使用数组和一组索引参数）。 ``sub`` 的结果与 ``getindex`` 的结果类似，但它的数据仍留在原地。 ``sub`` 在 ``SubArray`` 对象中保存输入的索引向量，这个向量将被用来间接索引原数组。

``StridedVector`` 和 ``StridedMatrix`` 是为了方便而定义的别名。通过给他们传递 ``Array`` 或 ``SubArray`` 对象，可以使 Julia 大范围调用 BLAS 和 LAPACK 函数，提高索引和内存申请的效率。

下面的例子计算大数组中的一个小块的 QR 分解，无需构造临时变量，直接调用合适的 LAPACK 函数。

.. code-block:: jlcon

    julia> a = rand(10,10)
    10x10 Float64 Array:
     0.763921  0.884854   0.818783   0.519682   …  0.860332  0.882295   0.420202
     0.190079  0.235315   0.0669517  0.020172      0.902405  0.0024219  0.24984
     0.823817  0.0285394  0.390379   0.202234      0.516727  0.247442   0.308572
     0.566851  0.622764   0.0683611  0.372167      0.280587  0.227102   0.145647
     0.151173  0.179177   0.0510514  0.615746      0.322073  0.245435   0.976068
     0.534307  0.493124   0.796481   0.0314695  …  0.843201  0.53461    0.910584
     0.885078  0.891022   0.691548   0.547         0.727538  0.0218296  0.174351
     0.123628  0.833214   0.0224507  0.806369      0.80163   0.457005   0.226993
     0.362621  0.389317   0.702764   0.385856      0.155392  0.497805   0.430512
     0.504046  0.532631   0.477461   0.225632      0.919701  0.0453513  0.505329
    
    julia> b = sub(a, 2:2:8,2:2:4)
    4x2 SubArray of 10x10 Float64 Array:
     0.235315  0.020172
     0.622764  0.372167
     0.493124  0.0314695
     0.833214  0.806369
    
    julia> (q,r) = qr(b);
    
    julia> q
    4x2 Float64 Array:
     -0.200268   0.331205
     -0.530012   0.107555
     -0.41968    0.720129
     -0.709119  -0.600124
    
    julia> r
    2x2 Float64 Array:
     -1.175  -0.786311
      0.0    -0.414549

稀疏矩阵
========

`稀疏矩阵 <http://zh.wikipedia.org/zh-cn/%E7%A8%80%E7%96%8F%E7%9F%A9%E9%98%B5>`_ 是其元素大部分为 0 的矩阵。

列压缩（CSC）存储
-----------------

Julia 中，稀疏矩阵使用 `列压缩（CSC）格式 <http://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_column_.28CSC_or_CCS.29>`_ 。Julia 稀疏矩阵的类型为 ``SparseMatrixCSC{Tv,Ti}`` ，其中 ``Tv`` 是非零元素的类型， ``Ti`` 是整数类型，存储列指针和行索引： ::

    type SparseMatrixCSC{Tv,Ti<:Integer} <: AbstractSparseMatrix{Tv,Ti}
        m::Int                  # Number of rows
        n::Int                  # Number of columns
        colptr::Vector{Ti}      # Column i is in colptr[i]:(colptr[i+1]-1)
        rowval::Vector{Ti}      # Row values of nonzeros
        nzval::Vector{Tv}       # Nonzero values
    end

列压缩存储便于按列简单快速地存取稀疏矩阵的元素，但按行存取则较慢。把非零值插入 CSC 结构等运算，都比较慢，这是因为稀疏矩阵中，在所插入元素后面的元素，都要逐一移位。

构造稀疏矩阵
------------

稠密矩阵有 ``zeros`` 和 ``eye`` 函数，稀疏矩阵对应的函数，在函数名前加 ``sp`` 前缀即可：

::

    julia> spzeros(3,5)
    3x5 sparse matrix with 0 nonzeros:

    julia> speye(3,5)
    3x5 sparse matrix with 3 nonzeros:
        [1, 1]  =  1.0
        [2, 2]  =  1.0
        [3, 3]  =  1.0

``sparse`` 函数是比较常用的构造稀疏矩阵的方法。它输入行索引 ``I`` ，列索引向量 ``J`` ，以及非零值向量 ``V`` 。 ``sparse(I,J,V)`` 构造一个满足 ``S[I[k], J[k]] = V[k]`` 的稀疏矩阵：

::

    julia> I = [1, 4, 3, 5]; J = [4, 7, 18, 9]; V = [1, 2, -5, 3];

    julia> sparse(I,J,V)
    5x18 sparse matrix with 4 nonzeros:
         [1 ,  4]  =  1
         [4 ,  7]  =  2
         [5 ,  9]  =  3
         [3 , 18]  =  -5

与 ``sparse`` 相反的函数为 ``findn`` ，它返回构造稀疏矩阵时的输入：

::

    julia> findn(S)
    ([1, 4, 5, 3],[4, 7, 9, 18])

    julia> findn_nzs(S)
    ([1, 4, 5, 3],[4, 7, 9, 18],[1, 2, 3, -5])

另一个构造稀疏矩阵的方法是，使用 ``sparse`` 函数将稠密矩阵转换为稀疏矩阵：

::

    julia> sparse(eye(5))
    5x5 sparse matrix with 5 nonzeros:
        [1, 1]  =  1.0
        [2, 2]  =  1.0
        [3, 3]  =  1.0
        [4, 4]  =  1.0
        [5, 5]  =  1.0

可以使用 ``dense`` 或 ``full`` 函数做逆操作。 ``issparse`` 函数可用来检查矩阵是否稀疏：

::

    julia> issparse(speye(5))
    true

稀疏矩阵运算
------------

稠密矩阵的算术运算也可以用在稀疏矩阵上。对稀疏矩阵进行赋值运算，是比较费资源的。大多数情况下，建议使用 ``find_nzs`` 函数把稀疏矩阵转换为 ``(I,J,V)`` 格式，在非零数或者稠密向量 ``(I,J,V)`` 的结构上做运算，最后再重构回稀疏矩阵。

稠密矩阵和稀疏矩阵函数对应关系
------------------------------

接下来的表格列出了内置的稀疏矩阵的函数, 及其对应的稠密矩阵的函数。通常，稀疏矩阵的函数，要么返回与输入稀疏矩阵 ``S`` 同样的稀疏度，要么返回   ``d`` 稠密度，例如矩阵的每个元素是非零的概率为 ``d`` 。

详见可以标准库文档的 :ref:`stdlib-sparse` 章节。

+-----------------------+-------------------+----------------------------------------+
| 稀疏矩阵              | 稠密矩阵          | 说明                                   |
+-----------------------+-------------------+----------------------------------------+
| ``spzeros(m,n)``      | ``zeros(m,n)``    | 构造 *m* x *n* 的全 0 矩阵             |
|                       |                   | (``spzeros(m,n)`` 是空矩阵)            |
+-----------------------+-------------------+----------------------------------------+
| ``spones(S)``         | ``ones(m,n)``     | 构造的全 1 矩阵                        |
|                       |                   | 与稠密版本的不同， ``spones``  的稀疏  |
|                       |                   | 度与 *S* 相同                          |
+-----------------------+-------------------+----------------------------------------+
| ``speye(n)``          | ``eye(n)``        | 构造 *m* x *n* 的单位矩阵              |
+-----------------------+-------------------+----------------------------------------+
| ``dense(S)``,         | ``sparse(A)``     | 转换为稀疏矩阵和稠密矩阵                 |
| ``full(S)``           |                   |                                        |
+-----------------------+-------------------+----------------------------------------+
| ``sprand(m,n,d)``     | ``rand(m,n)``     | 构造 *m*-by-*n* 的随机矩阵（稠密度为   |
|                       |                   | *d* ） 独立同分布的非零元素在 [0, 1]   |
|                       |                   | 内均匀分布                             |
+-----------------------+-------------------+----------------------------------------+
| ``sprandn(m,n,d)``    | ``randn(m,n)``    | 构造 *m*-by-*n* 的随机矩阵（稠密度为   |
|                       |                   | *d* ） 独立同分布的非零元素满足标准正  |
|                       |                   | 态（高斯）分布                         |
+-----------------------+-------------------+----------------------------------------+
| ``sprandn(m,n,d,X)``  | ``randn(m,n,X)``  | 构造 *m*-by-*n* 的随机矩阵（稠密度为   |
|                       |                   | *d* ） 独立同分布的非零元素满足 *X* 分 |
|                       |                   | 布。（需要 ``Distributions`` 扩展包）  |
+-----------------------+-------------------+----------------------------------------+
| ``sprandbool(m,n,d)`` | ``randbool(m,n)`` | 构造 *m*-by-*n* 的随机矩阵（稠密度为   |
|                       |                   | *d* ） ，非零 ``Bool``元素的概率为 *d* |
|                       |                   | (``randbool`` 中 *d* =0.5 )            |
+-----------------------+-------------------+----------------------------------------+
