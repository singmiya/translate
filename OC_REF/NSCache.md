
为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# NSCache
你用来短暂存储临时键值对的可变集合，且在资源不足时可以把它回收掉。

---

## 概述
缓存对象同其他可变集合的几个不同的地方：

* `NSCache`类包含多种自动回收策略，这确保缓存不会耗费太多系统内存。如果其他应用需要内存，这些策略会从缓存中移除一些数据，以减少它对内存的占用。
* 你可以用不同的线程在不需要对缓存加锁的情况下向缓存中添加、移除和查询数据。
* 不像[NSMutableDictionary](https://developer.apple.com/documentation/foundation/nsmutabledictionary)对象，缓存不会复制放入其中的键对象。

通常的，你使用`NSCache`对象去短暂的存储需要花费高成本创建的临时对象。重用这些对象可以带来性能上的好处，因为他们的值不需要重新计算。然而，对应用来不是很关键的对象会在内存紧张的时候废弃掉。如果废弃掉，在再需要这些值的时候就需要重新计算。

具有子部分的对象不再使用的会被回收掉，这可以采用[NSDiscardableContent](#dcontent)去改善缓存回收行为。默认情况下，他们的内容在缓存中的[NSDiscardableContent](#dcontent)对象在他们的内容被丢弃后可以自动的移除掉，尽管这个自动移除策略可以改变。如果一个[NSDiscardableContent](#dcontent)对象存入到了缓存中，缓存在移除它的时候会对它调用`discardContentIfPossible`。

---

## 属性
### name
`实例属性`
缓存名称

#### 声明
```
@property(copy) NSString *name;
```
#### 讨论
默认值是一个空字符串（""）

---

### countLimit
缓存可以保存对象的最大值

#### 声明
```
@property NSUInteger countLimit;
```
#### 讨论
如果等于0，那么就没有数量限制。默认值是0.

这不是严格的限制，如果缓存超出限制，缓存中的对象可以立即回收或者永远不回收，这取决于缓存的详细实现。

---

### totalCostLimit
在开始回收对象之前的缓存可以容纳的最大值

#### 声明
```
@property NSUInteger totalCostLimit;
```
#### 讨论
如果为0，则没有最大值限制。默认值是0.

当你往缓存中添加对象的时候，可能会对对象传递一个指明的成本，例如对象的字节大小。如果向缓存中添加这个对象导致缓存的总成本超过`totalCostLimit`，缓存可能会自动回收对象知道它的总成本低于`totalCostLimit`。缓存回收对象的顺序是无法确定的。

这个限制并不严格，如果缓存超过限制，缓存中的对象可能会被立即回收、也可能是晚些时候或者永远不会，这都取决于缓存的具体实现。

---

### evictsObjectsWithDiscardedContent
缓存是否会自动回收内容废弃的对象

#### 声明
```
@property BOOL evictsObjectsWithDiscardedContent;
```
#### 讨论
如果是YES的话，缓存会自动回收内容已废弃的对象。如果是NO，则不会。默认值是YES。

### delegate
缓存的委托

#### 声明
```
@property (assign) id<NSCacheDelegate> delegate;
```
#### 讨论
这个委托必须遵守`NSCacheDelegate`协议。

---


## 协议
#### [NSDiscardableContent](#dcontent)
#### [NSCacheDelegate](#cdelegate)

## 方法
### - objectForKey:
返回同给定的键值相关联的值。
#### 声明
```
- (ObjectType)objectForKey:(KeyType)key;
```

#### 参数
> **key**
标识一个值的对象。

#### 返回值
与键相关的值，或者如果与键相关的值不存在的话，则返回nil。

---

### setObject:forKey:
为缓存中特定的键设置值。
#### 声明
```
- (void)setObject:(ObjectType)obj forKey:(KeyType)key;
```

#### 参数
> **obj**
	存储在缓存中的对象

---
> **key**
	同值相关的键。
	
#### 讨论
不像`NSMutableDictionary`对象那样，缓存不会拷贝放在其中的键对象。

---

### setObject:forKey:cost:
为缓存中特定的键设置值，同时为键值对设置特定的成本（占用空间）。
#### 声明
```
- (void)setObject:(ObjectType)obj forKey:(KeyType)key cost:(NSUInteger)g;
```
#### 参数
> **obj**
	存储在缓存中的对象

---
> **key**
同值关联的键。

---
> **num**
	同键值对相关的成本（占用空间）。

#### 讨论
一个用来计算包含在缓存中所有对象的占用空间总值的`cost`值。当内存有限或当总的成本（占用空间）超过允许的总成本（占用空间）时，缓存可以启动一个回收过程把其中的一些元素移除。然而，这个回收过程不按照指定的顺序进行。因此，如果你试图篡改成本（占用空间）值去达到特定的行为，这样可能会对你的程序没有益处。通常，明显的成本是字节码的值。如果这些信息不是现成的，你不应该费力的去计算它，因为这样会增加使用缓存的成本。如果你没有任何有用的信息，你可以想成本中传递0或者简单地使用`setObject:forKey:`方法，这个方法不需要传递成本值。

不像`NSMutableDictionary`对象，缓存不会拷贝放进去的键对象。

### removeObjectForKey:
移除缓存中的特定键对应的值。

#### 声明
```
- (void)removeObjectForKey:(KeyType)key;
```
#### 参数
> **key**
	标识将要移除值的键。

---

### removeAllObjects
清空缓存
#### 声明
```
- (void)removeAllObjects;
```

---

# <span id="dcontent">NSDiscardableContent</span>
如果你实现了这个协议，在类对象中的属性没有被使用的时候会将其空间废弃掉，从而让应用的内存占用更加小。

---

## 概述
`NSDiscardableContent`对象的生命周期取决于“计数”变量。`NSDiscardableContent`对象是一个可清除的内存块，用于跟踪它是否正被其他对象使用。当这个内存正被读取或还被使用时，它的计数变量将被增加或者设置为1。当未被使用或废弃时，计数变量会被设置为0。

当计数器是0时，如果当时内存紧张内存块会被废弃。为了要废弃内容，在对象中调用`discardContentIfPossible`方法，如果计数变量为0时将会释放相关的内存。

默认情况下，`NSDiscardableContent`对象初始化的时候把计数变量设置为1，用以保证它们不会立即被内存管理系统回收。由此，你必须对计数变量的状态进行跟踪。调用`beginContentAccess`方法会对计数器加1，这样可以保证对象不会被回收。当你不再需要这个对象时，调用`endContentAccess`方法会递减计数器。

`Foundation`框架包含`NSPurgeableData`类，这个类提供这个协议的默认实现。

---

## 方法
### beginContntAccess
返回一个布尔值，它用来指示可废弃的内容是否可用、是否可以成功访问。
`Required`

#### 声明

```
- (BOOL)beginContentAccess;
```

#### 返回值
如果可废弃的内容仍然可用且可以成功访问，则YES；否则返回NO
#### 讨论
如果需要且打算使用对象的内存，可以调用此方法。这个方法会增加计数器的值，以此来保护对象的内存不会被回收。实现了可以决定这个方法重新创建已经废弃的内容，并在创建成功后返回YES。如果在未调用`beginContentAccess`方法的时候使用`NSDiscardableContent`对象，这个协议的实现者应该会出现异常。

---

### endContentAccess
如果可回收的内容不在被访问，则可以调用此方法
`Required`
#### 声明
``` 
- (void)endContentAccess;
``` 	
#### 讨论
这个方法会递减这个对象的计数变量，这通常将会把计数器的值减到0，如果需要的话，它可以回收对象的可回收内容。

---

### discardContentIfPossible
如果获取到的计数器值为0，调用这个方法去回收接收者的内容。
#### 声明
```
- (void)discardContentIfPossible;
```
#### 讨论
如果获取到的计数器的值为0，这个方法会回收对象的内容。否则，什么都不做。

### isContentDiscarded
返回一个布尔值，表明内容是否已被回收。
`Required`
#### 声明
```
- (BOOL)isContentDiscarded;
```
#### 返回值
如果内容已被回收，怎返回YES；否则返回NO。


# <span id="cdelegate">NSCacheDelegate</span>
当打算去回收对象或者把对象从缓存中移除的时候，`NSCache`对象的委托实现这个协议去执行特定的动作。

## 方法
### cache:willEvictObject:
当打算回收对象或者把对象从缓存中回收的时候，可以调用这个方法。
#### 声明
```
- (void)cache:(NSCache *)cache willEvictObject:(id)obj;
```
#### 参数
> **cache**
 与感兴趣的对象相关联的缓存。

---

> **obj**
缓存中感兴趣的对象。

#### 讨论
不可能在这个委托方法的实现中修改缓存。

