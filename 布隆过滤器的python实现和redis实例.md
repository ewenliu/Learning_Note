# 布隆过滤器的python实现和redis实例

#### 布隆过滤器是什么

> 布隆过滤器是一种比较巧妙的概率型数据结构（probabilistic data structure），它实际上是一个很长的二进制向量和一系列随机映射函数，特点是高效地插入和查询，可以用来告诉你 “某样东西一定不存在或者可能存在”。


#### 缓存穿透
查询一个一定不存在的值，这个值不仅不在redis，也不在数据库，导致每次查询请求都走数据库，并且还查不到(比如请求用户ID为-1的用户)，这就是缓存穿透

#### 布隆过滤器的使用场景
主要作用是用来设定一个规则用来判断Input存在与否

典型应用场景如下：

- 地址过滤，URL过滤等
- 黑名单过滤
- redis**缓存穿透**

#### 布隆过滤器的原理

原理这里就不细说了，感兴趣的可以百度，有很多说的很好的博文详细介绍了原理，这里以偏概全的提一下大概。

**过滤规则新增**

> 前提：布隆过滤器的二进制向量初始值全部置为0

当输入等于"google.com"且布隆过滤器hash函数是3个的时候，假设布隆过滤器生成的hash值是1，4，7，那么将过滤器的二进制向量的1，4，7位置置为1，新增完毕。

**过滤判断**

当输入等于"baidu.com"时，假设布隆过滤器生成的hash值是1，4，8，可以发现，二进制向量的第8位不为1（1，4，7为1），那么判定"baidu.com"不存在。

#### Python实现

```python
# -*- coding: utf-8 -*-#

# 提供二进制向量
from bitarray import bitarray
# 提供hash函数mmh3.hash()
import mmh3


class BloomFilter(set):
    """
    size: 二进制向量长度
    hash_count: hash函数个数

    一般来说，hash函数的个数要满足（hash函数个数=二进制长度*ln2/插入的元素个数）
    """
    def __init__(self, size, hash_count):
        super(BloomFilter, self).__init__()
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)
        self.size = size
        self.hash_count = hash_count

    def __len__(self):
        return self.size

    # 使得BloomFilter可迭代
    def __iter__(self):
        return iter(self.bit_array)

    def add(self, item):
        for ii in range(self.hash_count):
            # 假设hash完的值是22，size为20，那么取模结果为2，将二进制向量第2位置为1
            index = mmh3.hash(item, ii) % self.size
            self.bit_array[index] = 1

        return self

    # 可以使用 xx in BloomFilter方式来判断元素是否在过滤器内（有小概率会误判不存在的元素也存在, 但是已存在的元素绝对不会误判为不存在）
    def __contains__(self, item):
        out = True
        for ii in range(self.hash_count):
            index = mmh3.hash(item, ii) % self.size
            if self.bit_array[index] == 0:
                out = False

        return out


def main():
    bloom = BloomFilter(100, 10)
    animals = ['dog', 'cat', 'giraffe', 'fly', 'mosquito', 'horse', 'eagle',
               'bird', 'bison', 'boar', 'butterfly', 'ant', 'anaconda', 'bear',
               'chicken', 'dolphin', 'donkey', 'crow', 'crocodile']
    # 新增过滤规则
    for animal in animals:
        bloom.add(animal)

    # 已经插入的动物应该绝对满足存在
    for animal in animals:
        if animal in bloom:
            print('{}在布隆过滤器中......'.format(animal))
        else:
            print('{}不在布隆过滤器中......'.format(animal))

    print('\n')
    print('这是分割线', '-'*100)
    print('\n')

    # 未插入的动物，虽然'sheep'不在过滤规则里，但是确判定为存在，由此可见布隆过滤器有一定的误判几率（宁可误杀不可放走）
    other_animals = ['badger', 'cow', 'pig', 'sheep', 'bee', 'wolf', 'fox',
                     'whale', 'shark', 'fish', 'turkey', 'duck', 'dove',
                     'deer', 'elephant', 'frog', 'falcon', 'goat', 'gorilla',
                     'hawk']
    for other_animal in other_animals:
        if other_animal in bloom:
            print('{}在布隆过滤器中......'.format(other_animal))
        else:
            print('{}不在布隆过滤器中......'.format(other_animal))


if __name__ == '__main__':
    main()

```

**增强准确性**

- 增加hash个数，增大hash随机性，减少hash碰撞的概率
- 增加二进制向量的长度，使hash值分布更均匀化

