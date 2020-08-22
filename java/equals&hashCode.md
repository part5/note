### ==
* 基础类型==比较值是否相同
* 引用类型比较地址是否相同

### equals()
* equals()默认使用==比较两个变量

### hashCode()
> 将集合分为多个区域，每个元素都对应一个区域，查找元素时，先确定元素所在的区域，再到这个区域里查找，这样比和集合中的每一个元素比对要快捷的多。

- hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为**散列值**，可以类比取余操作
- hashCode返回的不是对象的存储地址，而是哈希表中的位置
- 哈希表中同一个哈希值可能对应多个对象，一样可以减少比较次数(equals())
- 哈希数据结构的集合查找、插入速度非常快，但遍历不方便
- 哈希表包含所有的哈希值

### 为什么重写了equals()后一定要重写hashCode()
> 重写了equals()则表示不再以内存地址判断两个对象是否相等，此时如果不重写hashCode()，会出现存入两个hashCode相同，但内存地址不同的key，如果此时两个key的equals返回true，则存入第二个时就会覆盖第一个key。

* equals默认比较对象的内存地址，hasCode默认返回散列表中的位置

### 为了Hash类集合重复判断，equals()&hashCode()要遵循如下规则
* 若两个对象equals()为true，则hashCode()一定相同
* 若两个对象hashCode()相同，则equals()不一定为true
* 若两个对象hashCode()不同，则equals()一定为false
* 若两个对象equals()为false，则hashCode()不一定不同
* 两个对象`equals()为true`是`hashCode()相同`的充分不必要条件
* 充分的一定能得出结论，必要的是结论的一部分
