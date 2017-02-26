---
layout: single
title: "Generate Short Url-Friendly Unique ID in Python"
excerpt: "My exploration for generation methods of shorter and unique ID"
modified: 2017-02-25
tags: [python, develop, bitwise]
comments: true
---


最近的一个项目需要大量生成唯一的ID，而且ID可以作为URL的一部分。需求很简单，实现稍曲折，最终还是找到了令人满意的方法。

实验基于一台[Standard D3_v2](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/series/#d-series)规模的Azure虚拟机，使用Ubuntu 16.04.1操作系统，Linux内核版本是4.4.0-64-generic，Python版本是3.5.2。


## Version 0: Base62-Encoded UUID1

用Python实现这个初步的版本只需一分钟，大体思想就是——用[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)保证ID的唯一性，用[Base62](https://de.wikipedia.org/wiki/Base62)编码确保用于URL时不需要转义。代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v0: Base62-Encoded UUID1 """
    return base62().encode(uuid1().int)
{% endhighlight %}

运行五次获得的结果如下——

{% highlight text %}
4u0bvZpU6zDOWLmmlUgMns
4u0bvbUWZHMsplKiMcc06u
4u0bvd9Z1ZWN9AsdxkXdPw
4u0bveobTrfrSaQZYsTGiy
4u0bvgTdw9pLlzyVA0Ou20
{% endhighlight %}

初步看起来是唯一的，但是缺点是明显太长了，平均22个字符的长度。


## Version 1: Half Unshuffled UUID1

为了使生成的ID更短，可以考虑将128位的UUID压缩为64位。具体处理过程是——将UUID表示为128位bit的整数，对相邻两位bit进行XOR运算，将计算结果按序拼成64位bit的整数。

这样的处理会导致理论上可以生成的唯一ID的总数从3.403×10<sup>38</sup>种降低到了1.845×10<sup>19</sup>种，但在当前情况下绝对够用了。使用XOR而不是OR或AND运算，是因为XOR运算结果的信息熵更大。

以16位整数压缩为8位为例，操作步骤见下表——

| Step | Action                 | Result                |
|:----:|:-----------------------|:----------------------|
|   1  | Get `num`              | `abcd efgh ijkl mnop` |
|   2  | Get `num >> 1`         | `0abc defg hijk lmno` |
|   3  | `num` XOR `num >> 1`   | `ABCD EFGH IJKL MNOP` |
|   4  | AND mask `0x5555`      | `0B0D 0F0H 0J0L 0N0P` |
|   5  | Compress to the right  | `0000 0000 BDFH JLNP` |

代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v1: Half Unshuffled UUID1 """
    n = uuid1().int
    x = (n ^ (n >> 1))  & 0x55555555555555555555555555555555
    x = (x | (x >> 1))  & 0x33333333333333333333333333333333
    x = (x | (x >> 2))  & 0x0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F
    x = (x | (x >> 4))  & 0x00FF00FF00FF00FF00FF00FF00FF00FF
    x = (x | (x >> 8))  & 0x0000FFFF0000FFFF0000FFFF0000FFFF
    x = (x | (x >> 16)) & 0x00000000FFFFFFFF00000000FFFFFFFF
    x = (x | (x >> 32)) & 0x0000000000000000FFFFFFFFFFFFFFFF
    return base62().encode(x)
{% endhighlight %}

同样连续运行五次，产生的ID平均长度约11个字符，基本符合对长度的要求。

{% highlight text %}
BFuq2iPRm9U
BFtY7Bs4bZM
BFw7yEwowjc
DlRk8ImsWwe
DlQSCmFVMMW
{% endhighlight %}


## Version 2: Shuffled XOR UUID1

上一版本中压缩为64位整数的操作使用了**外完美半理牌**(Outer Perfect Half Unshuffle)算法，考虑用类似的洗牌算法以减少指令数，即——将XOR运算结果的128位整数拆分为两个64位整数，将高位整数的有效位交错地插在低位整数有效位中，构成一个新的64位整数。

以上一版本16位整数的中间结果`0B0D 0F0H 0J0L 0N0P`为例，拆分成`0B0D 0F0H`和`0J0L 0N0P`，交叉后构成8位整数`BJDL FNHP`。代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v2: Shuffled XOR UUID1 """
    num = uuid1().int
    mask = (1 << 64) - 1
    result = (num ^ (num >> 1)) & 0x55555555555555555555555555555555
    result = ((mask & (result >> 64)) << 1) | (mask & result)
    return base62().encode(result)
{% endhighlight %}

运行五次的结果如下，依然是美好的11个字符——

{% highlight text %}
JpJ1kGrbKiQ
JpJ1k7UGqKI
JpJ1hhSz06E
JpJ1hqqJUUM
JbKp7Sn6t34
{% endhighlight %}


## Version 3: Shuffled XOR UUID4

既然长度已经达标，那么来检查**唯一性**这一要求。通过在同一机器短时间内多次调用的方法，依次测试了上述三个版本，结果表明后两个会出现大量的碰撞。以Version 2为例，下列32个UUID转化后均为`9EPwBcQLEfG`——

{% highlight text %}
2a44b540-fb79-11e6-be32-000d3a80d4c8
2a44b543-fb79-11e6-be32-000d3a80d4c8
2a44b54c-fb79-11e6-be32-000d3a80d4c8
2a44b54f-fb79-11e6-be32-000d3a80d4c8
2a44b570-fb79-11e6-be32-000d3a80d4c8
2a44b573-fb79-11e6-be32-000d3a80d4c8
2a44b57c-fb79-11e6-be32-000d3a80d4c8
2a44b57f-fb79-11e6-be32-000d3a80d4c8
2a44b580-fb79-11e6-be32-000d3a80d4c8
2a44b583-fb79-11e6-be32-000d3a80d4c8
2a44b58c-fb79-11e6-be32-000d3a80d4c8
2a44b58f-fb79-11e6-be32-000d3a80d4c8
2a44b5b0-fb79-11e6-be32-000d3a80d4c8
2a44b5b3-fb79-11e6-be32-000d3a80d4c8
2a44b5bc-fb79-11e6-be32-000d3a80d4c8
2a44b5bf-fb79-11e6-be32-000d3a80d4c8
2a44b640-fb79-11e6-be32-000d3a80d4c8
2a44b643-fb79-11e6-be32-000d3a80d4c8
2a44b64c-fb79-11e6-be32-000d3a80d4c8
2a44b64f-fb79-11e6-be32-000d3a80d4c8
2a44b670-fb79-11e6-be32-000d3a80d4c8
2a44b673-fb79-11e6-be32-000d3a80d4c8
2a44b67c-fb79-11e6-be32-000d3a80d4c8
2a44b67f-fb79-11e6-be32-000d3a80d4c8
2a44b680-fb79-11e6-be32-000d3a80d4c8
2a44b683-fb79-11e6-be32-000d3a80d4c8
2a44b68c-fb79-11e6-be32-000d3a80d4c8
2a44b68f-fb79-11e6-be32-000d3a80d4c8
2a44b6b0-fb79-11e6-be32-000d3a80d4c8
2a44b6b3-fb79-11e6-be32-000d3a80d4c8
2a44b6bc-fb79-11e6-be32-000d3a80d4c8
2a44b6bf-fb79-11e6-be32-000d3a80d4c8
{% endhighlight %}

出现这一情况是因为生成UUID时采用的是[**UUID Version 1**](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_1_.28date-time_and_MAC_address.29)生成算法，其中最低的48位用于标识机器上，其次有14位随机数，高位是拆分后的60位时间戳。在[Python](https://docs.python.org/3.5/library/uuid.html#uuid.uuid1)的实现中，默认情况下仅有时间戳会发生变化——直接导致了时间相近时，碰撞容易多次发生。

直接的改进方案是通过为UUID Version 1强制生成14位随机数来减小碰撞的概率，亦可放弃机器特征信息直接使用完全由随机数实现的[**UUID Version 4**](https://en.wikipedia.org/wiki/Universally_unique_identifier#Version_4_.28random.29)生成算法。基于后者的代码实现如下——

{% highlight python %}
from uuid import uuid4
from basehash import base62

def generate_short_id():
    """ Short ID generator - v3: Shuffled XOR UUID4 """
    num = uuid4().int
    mask = (1 << 64) - 1
    result = (num ^ (num >> 1)) & 0x55555555555555555555555555555555
    result = ((mask & (result >> 64)) << 1) | (mask & result)
    return base62().encode(result)
{% endhighlight %}

连续运行五次的结果见下——

{% highlight text %}
AEe4VHJJUjb
IPLt6fJLmPX
8SXYILvcgZL
7aukApGhDka
L08tR6LLDqM
{% endhighlight %}

另在单机上测试了8,000,000次，无任何碰撞。


## Version 4: Urandom

既然只需要64位整数，是否可以在UUID生成的时候进行缩减呢？我查阅了[Python源代码](https://www.python.org/downloads/source/)中对`uuid4`函数的实现。Python以2.7.11和3.5.1版本为界，前后主要有两个显著不同的`uuid4`函数的实现版本——

Python 3.5.0的`/Lib/uuid.py`中`uuid4`函数的实现如下：

{% highlight python %}
def uuid4():
    """Generate a random UUID."""

    # When the system provides a version-4 UUID generator, use it.
    if _uuid_generate_random:
        _buffer = ctypes.create_string_buffer(16)
        _uuid_generate_random(_buffer)
        return UUID(bytes=bytes_(_buffer.raw))

    # Otherwise, get randomness from urandom or the 'random' module.
    try:
        import os
        return UUID(bytes=os.urandom(16), version=4)
    except Exception:
        import random
        return UUID(int=random.getrandbits(128), version=4)
{% endhighlight %}

而Python 3.5.1中`uuid4`的实现则变成了：

{% highlight python %}
def uuid4():
    """Generate a random UUID."""
    return UUID(bytes=os.urandom(16), version=4)
{% endhighlight %}

两者的主要区别在于前者在`urandom`函数的基础上，还调用libuuid库和`random`伪随机数两种方法。但这一实现在2015年10月底的[Issue 25515](https://bugs.python.org/issue25515)中被认为不够安全高效，并被[修改](https://bugs.python.org/review/25515/)为[目前仅使用`urandom`函数的版本](https://github.com/python/cpython/blob/master/Lib/uuid.py)。

参照此方法仅用[urandom](https://docs.python.org/3.5/library/os.html#os.urandom)实现，同样一行代码解决问题——

{% highlight python %}
from os import urandom
from struct import unpack
from basehash import base62

def generate_short_id():
    """ Short ID generator - v4: Urandom """
    return base62().encode(unpack("<Q", urandom(8))[0])
{% endhighlight %}

连续运行五次的结果如下，零碰撞的结果也令人满意——

{% highlight text %}
JWDypmB3wt
ANbbxzr6B4f
AcNhBlo6G0S
Dh3aVhCtQUm
4dCcrJwi9VL
{% endhighlight %}


# Evaluation

唯一性的问题已经解决，现在来重新评估一下各个版本产生ID的实际长度。在单机上用以上各个版本各生成200,000个不同的ID，ID长度的统计结果如下——

| ver.             | len=8            | len=9            | len=10           | len=11           | len=22           | avg. len.        |
|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|
| v0               | 0 / 0%           | 0 / 0%           | 0 / 0%           | 0 / 0%           | 200000 / 100%    | 22               |
| v1               | 4 / 0.002%       | 142 / 0.071%     | 8774 / 4.387%    | 191080 / 95.54%  | 0 / 0%           | 10.955           |
| v2               | 0 / 0%           | 0 / 0%           | 0 / 0%           | 200000 / 100%    | 0 / 0%           | 11               |
| v3               | 0 / 0%           | 0 / 0%           | 0 / 0%           | 200000 / 100%    | 0 / 0%           | 11               |
| v4               | 0 / 0%           | 148 / 0.074%     | 8931 / 4.466%    | 190921 / 95.461% | 0 / 0%           | 10.954           |

由上表可以看出，在当前环境下Version 4产生的ID平均长度最短。长度不均是因为整数高位的0在Base62编码过程中被省略了。例如，0x0013c3bb0fd02d5c编码后的长度为9个字符，而0xe05dc018dbcac150则为11个字符。

基于Python的[timeit](https://docs.python.org/3.5/library/timeit.html)库测得的时间消耗如下——

| ver.             | times=10000      | times=50000      | times=100000     | times=200000     | times=500000     |
|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|
| v0               | 234.456µs        | 240.602µs        | 240.618µs        | 240.929µs        | 240.218µs        |
| v1               | 237.585µs        | 237.863µs        | 238.270µs        | 237.965µs        | 238.010µs        |
| v2               | 236.660µs        | 237.336µs        | 236.844µs        | 237.302µs        | 237.722µs        |
| v3               | 235.845µs        | 234.734µs        | 235.397µs        | 235.122µs        | 235.229µs        |
| v4               | 224.070µs        | 224.375µs        | 225.760µs        | 226.288µs        | 225.901µs        |

各个版本的平均耗时在数量上相差不大，主要是因为Base62编码过程相对耗时，共同减去这一过程，测得的结果如下——

| ver.             | times=10000      | times=50000      | times=100000     | times=200000     | times=500000     |
|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|
| v0_raw           | 3.617µs          | 3.699µs          | 3.803µs          | 3.726µs          | 3.748µs          |
| v1_raw           | 4.820µs          | 4.782µs          | 4.848µs          | 4.872µs          | 4.830µs          |
| v2_raw           | 4.246µs          | 4.245µs          | 4.295µs          | 4.278µs          | 4.270µs          |
| v3_raw           | 4.458µs          | 4.602µs          | 4.609µs          | 4.638µs          | 4.656µs          |
| v4_raw           | 1.269µs          | 1.309µs          | 1.312µs          | 1.303µs          | 1.311µs          |

可以看出在当前环境下Version 4的效率是最高的，同时也发现Base62编码部分是性能的瓶颈，需要进一步优化。


## Version 5: Base62-Encoded Urandom

在查阅了Base62所使用的[BaseHash库](https://github.com/bnlucas/python-basehash)的源代码后，我发现瓶颈在于类的[初始化过程](https://github.com/bnlucas/python-basehash/blob/a79581fda56895e65bdada92d90e70ca45f00c06/basehash/__init__.py#L31)上（代码见下）——无论是否会使用`hash`或`unhash`函数，初始化`base62`类时都不可以避免的需要执行这个费时`next_prime`：

{% highlight python %}
self.prime = next_prime(int((self.maximum + 1) * self.generator))
{% endhighlight %}

尝试在**site-packages/basehash/__init__.py**中把这行赋值改为`None`，性能瞬间提高了。当然，这样hack并不能成为正式的方案，于是考虑将Base62编码直接替换手写的简单版本，不再依赖BaseHash库。代码实现见下——

{% highlight python %}
import string
from os import urandom
from struct import unpack

def generate_short_id():
    """ Short ID generator - v5: Base62-Encoded Urandom """
    num = unpack("<Q", urandom(8))[0]
    if num <= 0:
        result = "0"
    else:
        alphabet = string.digits + string.ascii_uppercase + string.ascii_lowercase
        key = []
        while num > 0:
            num, rem = divmod(num, 62)
            key.append(alphabet[rem])
        result = "".join(reversed(key))
    return result
{% endhighlight %}

和前一版本的性能对比数据如下——

| ver.             | times=10000      | times=50000      | times=100000     | times=200000     | times=500000     |
|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|:----------------:|
| v4_raw           | 1.210µs          | 1.219µs          | 1.209µs          | 1.232µs          | 1.237µs          |
| v5_raw           | 1.188µs          | 1.195µs          | 1.188µs          | 1.220µs          | 1.221µs          |
| v4               | 224.785µs        | 221.538µs        | 222.862µs        | 223.577µs        | 223.554µs        |
| v5               | 5.216µs          | 5.140µs          | 5.134µs          | 5.119µs          | 5.153µs          |

结果非常理想，相对前一版本达到了43倍的性能提升，平均每秒可以产生19万ID。


## Conclusion

Version 5就是最终的版本，它在长度、唯一性和效率上均已符合我的预期。完整的源代码和实验脚本，可以[从GitHub下载](https://github.com/vejuhust/blog-code/tree/master/python-short-id-generator)。

马斯洛（[Abraham Maslow](https://en.wikipedia.org/wiki/Abraham_Maslow)）说过一句名言*“In any given moment we have two options: to step forward into growth or back into safety.”*。这谈论的大概也是我们工程师的日常吧——是往前一步在挑战中成长，还是退回到安全区就此妥协。
