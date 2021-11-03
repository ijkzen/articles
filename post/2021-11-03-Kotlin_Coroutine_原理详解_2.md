---
title: Kotlin Coroutine 原理详解（2）-- 协程间的互动
---

## 基础类分析
在讲解协程间是如何进行互动之前需要对协程的一些基础类进行讲解；
### CoroutineContext
协程的上下文，可以看作是一个 `map`，在协程执行过程中充当设置项集合的角色；
#### get(key)
重载运算符方法，根据 `key` 获取相应的 `element`;
#### fold(initial, operation)
将 `initial` 和 `element` 合并并返回 `initial`，不同实现类对 `operation` 操作不同；
#### plus(context)
将两个 `CortouineContext` 合并，并返回一个新的 `CoroutineContext`,此处提供了一个默认实现；
```kotlin
public operator fun plus(context: CoroutineContext): CoroutineContext =
        if (context === EmptyCoroutineContext) this else // fast path -- avoid lambda creation
            context.fold(this) { acc, element ->
                val removed = acc.minusKey(element.key)
                if (removed === EmptyCoroutineContext) element else {
                    // make sure interceptor is always last in the context (and thus is fast to get when present)
                    val interceptor = removed[ContinuationInterceptor]
                    if (interceptor == null) CombinedContext(removed, element) else {
                        val left = removed.minusKey(ContinuationInterceptor)
                        if (left === EmptyCoroutineContext) CombinedContext(element, interceptor) else
                            CombinedContext(CombinedContext(left, element), interceptor)
                    }
                }
            }
```
具体逻辑如下：
为表述方便，将当前 `CoroutineContext` 称为 `left`，将传入的 `CoroutineContext` 称为 `right`;
1. 如果 `right` 不是 `EmptyCoroutineContext`，则返回 `left`;
2. 如果 `right` 不是 `EmptyCoroutineContext`，则 `right` 调用 `fold` 方法，传入 `left` 和 `operation`;

#### minusKey(key)
返回一个不包含指定 `key` 的 `CoroutineContext`;
### Key
操作 `CoroutineContext` 所使用到的键，定义如下：
```kotlin
public interface Key<E : Element>
```
### Element
操作 `CoroutineContext` 所返回的值，继承了 `CoroutineContext` 但是缩小了对 `ContextContext` 的使用范围；
定义如下：
```kotlin
public interface Element : CoroutineContext {
        /**
         * A key of this coroutine context element.
         */
        public val key: Key<*>

        public override operator fun <E : Element> get(key: Key<E>): E? =
            @Suppress("UNCHECKED_CAST")
            if (this.key == key) this as E else null

        public override fun <R> fold(initial: R, operation: (R, Element) -> R): R =
            operation(initial, this)

        public override fun minusKey(key: Key<*>): CoroutineContext =
            if (this.key == key) EmptyCoroutineContext else this
    }
```



接下来来看 `CoroutineContext` 的两个常见的实现类；
### EmptyCoroutineContext
可以看作一个空的`CoroutineContext`，不存在任何元素；这是 `CoroutineContext` 的默认实现；
源码如下：
```kotlin
public object EmptyCoroutineContext : CoroutineContext, Serializable {
    private const val serialVersionUID: Long = 0
    private fun readResolve(): Any = EmptyCoroutineContext
    
    // 没有任何元素，直接返回 null; 下面的重载函数也是相似的实现；
    public override fun <E : Element> get(key: Key<E>): E? = null
    public override fun <R> fold(initial: R, operation: (R, Element) -> R): R = initial
    public override fun plus(context: CoroutineContext): CoroutineContext = context
    public override fun minusKey(key: Key<*>): CoroutineContext = this
    public override fun hashCode(): Int = 0
    public override fun toString(): String = "EmptyCoroutineContext"
}
```

### CombinedContext
```kotlin
// 构造函数接收两个参数
internal class CombinedContext(
    private val left: CoroutineContext,
    private val element: Element
) : CoroutineContext, Serializable {

    // 先查 element，再递归查询 left；
    override fun <E : Element> get(key: Key<E>): E? {
        var cur = this
        while (true) {
            cur.element[key]?.let { return it }
            val next = cur.left
            if (next is CombinedContext) {
                cur = next
            } else {
                return next[key]
            }
        }
    }

    // left 先和 initial 进行递归合并得到 newLeft，然后 newLeft 和 element 进行合并；
    public override fun <R> fold(initial: R, operation: (R, Element) -> R): R =
        operation(left.fold(initial, operation), element)

    // 如果 element 包含 key，则直接返回 left；
    // 如果 left 不包含 key，返回当前 CombinedContext；
    // 如果 left 只包含了 key ，则返回 element；
    // 如果 left 不止包含了 key，则返回 新的 CombinedContext；
    public override fun minusKey(key: Key<*>): CoroutineContext {
        element[key]?.let { return left }
        val newLeft = left.minusKey(key)
        return when {
            newLeft === left -> this
            newLeft === EmptyCoroutineContext -> element
            else -> CombinedContext(newLeft, element)
        }
    }
}
```
从实现来看，`CombinedContext` 有点像链表；

### CoroutineScope
为了方便在协程执行过程中访问 `CoroutineContext` 而创建的接口；
```kotlin
public interface CoroutineScope {
    // 只有一个接口方法，来提供 `CoroutineContext`
    public val coroutineContext: CoroutineContext
}
```

