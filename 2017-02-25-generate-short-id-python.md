---
layout: single
title: "Generate Short Url-Friendly Unique ID in Python"
excerpt: "My exploration for generation methods of shorter and unique ID"
modified: 2017-02-25
tags: [python, develop, bitwise]
comments: true
---


最近的一个项目需要产生唯一ID，而且这个ID可能作为URL的一部分。事情很简单，但几经折腾，最后还是找到比较令人满意的方法。


## Version 0: Base62 Encoded UUID1

用Python完成这个最初的版本只用了一分钟，大体思想就是——用[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)保证ID的唯一性，用[Base62](https://de.wikipedia.org/wiki/Base62)编码确保不需要为URL转义。代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v0: Base62 Encoded UUID1 """
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

初步看起来是唯一的，但是明显的缺点是太长了，长达22个字符。


## Version 1: Compressed XOR UUID1

为了使最终的ID更短，可以考虑将128位的UUID压缩为64位。具体处理过程是——将UUID表示为128位bit的整数，对相邻两位bit进行XOR运算，将计算结果按序拼成64位bit的整数。

这样的处理会导致唯一的ID数从3.403×10<sup>38</sup>种降低到了1.845×10<sup>19</sup>种，但在这种情况下够用了。使用XOR而不是OR或AND运算，是因为XOR运算的结果的信息熵更大。

以16位整数压缩为8位为例，操作步骤见下表——

| Step | Action | Result |
|:----:|:-------|:-------|
|   1  | `num`  | `abcd efgh ijkl mnop` |
|   2  | `num >> 1`  | `0abc defg hijk lmno` |
|   3  | `num ^ (num >> 1)` | `ABCD EFGH IJKL MNOP` |
|   4  | AND mask `0x5555`  | `0B0D 0F0H 0J0L 0N0P` |
|   5  | Compress to the right  | `0000 0000 BDFH JLNP` |

代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v1: Half Unshuffle UUID1 """
    num = uuid1().int
    mask = 0x55555555555555555555555555555555
    x = (num ^ (num >> 1)) & mask
    x = (x | (x >> 1))  & 0x33333333333333333333333333333333
    x = (x | (x >> 2))  & 0x0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F0F
    x = (x | (x >> 4))  & 0x00FF00FF00FF00FF00FF00FF00FF00FF
    x = (x | (x >> 8))  & 0x0000FFFF0000FFFF0000FFFF0000FFFF
    x = (x | (x >> 16)) & 0x00000000FFFFFFFF00000000FFFFFFFF
    x = (x | (x >> 32)) & 0x0000000000000000FFFFFFFFFFFFFFFF
    return base62().encode(x)
{% endhighlight %}

同样连续运行五次，产生的ID长度为11个字符，基本符合要求。

{% highlight text %}
BFuq2iPRm9U
BFtY7Bs4bZM
BFw7yEwowjc
DlRk8ImsWwe
DlQSCmFVMMW
{% endhighlight %}


## Version 2: Shuffled XOR UUID1

上一版本中压缩为64位整数的操作使用了**外完美半理牌**(Outer Perfect Half Unshuffle)算法，考虑用类似的洗牌算法以减少指令数，即——将XOR运算结果的128位整数拆分为两个64位整数，将高位整数的有效位交错的插在低位整数有效位中，构成一个新的64位整数。

以上一版本16位整数的中间结果`0B0D 0F0H 0J0L 0N0P`为例，拆分成`0B0D 0F0H`和`0J0L 0N0P`，交叉后构成8位整数`BJDL FNHP`。代码实现见下——

{% highlight python %}
from uuid import uuid1
from basehash import base62

def generate_short_id():
    """ Short ID generator - v2: Shuffled XOR UUID1 """
    num = uuid1().int
    mask_shuffle = 0x55555555555555555555555555555555
    mask_half = (1 << 64) - 1
    result = (num ^ (num >> 1)) & mask_shuffle
    result = ((mask_half & (result >> 64)) << 1) | (mask_half & result)
    return base62().encode(result)
{% endhighlight %}

运行五次的结果如下，美好的11个字符——

{% highlight text %}
JpJ1kGrbKiQ
JpJ1k7UGqKI
JpJ1hhSz06E
JpJ1hqqJUUM
JbKp7Sn6t34
{% endhighlight %}
