# NSCache
你用来短暂存储临时键值对的可变集合，且在资源不足时可以把它回收掉。

---

## 概述
缓存对象同其他可变集合的几个不同的地方：

* `NSCache`类包含多种自动回收策略，这确保缓存不会耗费太多系统内存。如果其他应用需要内存，这些策略会从缓存中移除一些数据，以减少它对内存的占用。
* 你可以用不同的线程在不需要对缓存加锁的情况下向缓存中添加、移除和查询数据。
* 不像[NSMutableDictionary](/documentation/foundation/nscache)对象，缓存不会复制放入其中的键对象。

通常的，你使用`NSCache`对象去短暂的存储需要花费高代价创建的临时对象。重用这些对象可以带来性能上的好处，因为他们的值不需要重新计算。然而，对应用来不是很关键的对象会在内存紧张的时候废弃掉。如果废弃掉，在再需要这些值的时候就需要重新计算。

具有子部分的对象不再使用的会被回收掉，这可以采用[NSDiscardableContent](/documentation/foundation/nscache)去改善缓存回收行为。默认情况下，他们的内容在缓存中的[NSDiscardableContent](/documentation/foundation/nscache)对象在他们的内容被丢弃后可以自动的移除掉，尽管这个自动移除策略可以改变。如果一个[NSDiscardableContent](/documentation/foundation/nscache)对象存入到了缓存中，缓存在移除它的时候会对它调用[discardContentIfPossible](/documentation/foundation/nscache)。

## 主题
### 属性
> **name** `实例属性`
缓存名称

> ---
**声明**
```
@property(copy) NSString *name;
```
---
**讨论**
默认值是一个空字符串（""）

---
> **countLimit**
缓存可以保存对象的最大值

> ---
**声明**
```
@property NSUInteger countLimit;
```
**讨论**
如果等于0，那么就没有数量限制。默认值是0.

> 这不是严格的限制，如果缓存超出限制，缓存中的对象可以立即回收或者永远不回收，这取决于缓存的详细实现。

---
> **totalCostLimit**
在开始回收对象之前的缓存可以容纳的最大值

> ---
**声明**
```
@property NSUInteger totalCostLimit;
```
**讨论**
如果为0，则没有最大值限制。默认值是0.

> 当你往缓存中添加对象的时候，可能会对对象传递一个指明的成本，例如对象的字节大小。如果向缓存中添加这个对象导致缓存的总成本超过`totalCostLimit`，缓存可能会自动回收对象知道它的总成本低于`totalCostLimit`。缓存回收对象的顺序是无法确定的。

> 这个限制并不严格，如果缓存超过限制，缓存中的对象可能会被立即回收、也可能是晚些时候或者永远不会，这都取决于缓存的具体实现。

---

> **evictsObjectsWithDiscardedContent**
缓存是否会自动回收内容废弃的对象

> ---
> **声明**
```
@property BOOL evictsObjectsWithDiscardedContent;
```
**讨论**
如果是YES的话，缓存会自动回收内容已废弃的对象。如果是NO，则不会。默认值是YES。


### 方法




