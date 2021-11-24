---
title: Kotlin Coroutine 原理详解（3）-- 协程的挂起与恢复
categories: Kotlin
---

## 挂起与恢复的功能特征

1. 以同步的方式执行异步代码
2. 线程调度

## 挂起函数

`Kotlin` 使用 `suspend` 关键词标记一个函数为挂起函数；

```kotlin
GlobalScope.launch {
    // 协程体
            aaa()
            bbb()
}

suspend fun aaa(){
        delay(100)
        bbb()
        println("aaa")
    }

suspend fun bbb() {
        delay(100)
        println("bbb")
}
```

很显然我在协程体里依次调用了 `aaa` 和 `bbb` 两个挂起函数，并且在 `aaa` 中调用了一次 `bbb`。

下面是`class` 文件反编译的 `java` 源码，大体查看源码，协程使用状态机把层层嵌套的回调给拍平了。

在协程体里有一个大的状态机，挂起函数内部还有一个小的状态机，在递归地跑完小状态机后，再跑大状态机；

```java
// launch 扩展函数被编译为静态函数；
// 被扩展的 `CoroutineScope` 作为第一个参数；
// 第二三参数分别是 `CoroutineContext` `CoroutineStart`类型；
// 第四个参数类型为 `Function2` 一个有两个参数一个返回值的函数类型，新增了两个方法；其中 `invokeSuspend` 
// 函数编写了状态机的部分；`create` 函数创建了最初的 `Continuation`；实现 `invoke` 启动状态机
// 第五六两个参数不知道有什么用。
BuildersKt.launch$default((CoroutineScope)GlobalScope.INSTANCE, (CoroutineContext)null, (CoroutineStart)null, (Function2)(new Function2((Continuation)null) {
         int label;

         @Nullable
         public final Object invokeSuspend(@NotNull Object $result) {
            Object var2 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
            MainActivity var10000;
            switch(this.label) {
            case 0:
               ResultKt.throwOnFailure($result);
               var10000 = MainActivity.this;
               this.label = 1;
			   // 这里可以看出此处匿名实现的`Function2`不仅仅实现了`Function2`，还实现了`Continuation`接口，这一点并没有在反编译代码中表现；
               if (var10000.aaa(this) == var2) {
                  return var2;
               }
               break;
            case 1:
               ResultKt.throwOnFailure($result);
               break;
            case 2:
               ResultKt.throwOnFailure($result);
               return Unit.INSTANCE;
            default:
               throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
            }

            var10000 = MainActivity.this;
            this.label = 2;
            if (var10000.bbb(this) == var2) {
               return var2;
            } else {
               return Unit.INSTANCE;
            }
         }
		
		// 传入了`value` 参数，但是并没有用到；
		// 传入了`completion`作为上层状态机；
         @NotNull
         public final Continuation create(@Nullable Object value, @NotNull Continuation completion) {
            Intrinsics.checkNotNullParameter(completion, "completion");
			// 此处的 `var3` 同样实现了 `Continuation` 接口；
            Function2 var3 = new <anonymous constructor>(completion);
            return var3;
         }

         public final Object invoke(Object var1, Object var2) {
            return ((<undefinedtype>)this.create(var1, (Continuation)var2)).invokeSuspend(Unit.INSTANCE);
         }
      }), 3, (Object)null);


// 函数签名被修改。
// 在传参的最后添加了一个`Continuation` 的参数
// 函数的返回值修改为`Object`，当该函数能够立即返回时则返回本来的数据类型，此处是 Unit；
// 当函数不能立即返回时，则返回`COROUTINE_SUSPENDED`，来标记当前函数稍后会返回；
 @Nullable
   public final Object aaa(@NotNull Continuation var1) {
      Object $continuation;
      label27: {
	  // 此处 `undefinedtype` 指的是下方所匿名实现的 `ContinuationImpl` 类型；
         if (var1 instanceof <undefinedtype>) {
            $continuation = (<undefinedtype>)var1;
			// 下面的操作是为了保证 `label` 的正确性
            if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
               ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
               break label27;
            }
         }

         $continuation = new ContinuationImpl(var1) {
            // $FF: synthetic field
            Object result;
            int label;
            Object L$0;

            @Nullable
            public final Object invokeSuspend(@NotNull Object $result) {
               this.result = $result;
               this.label |= Integer.MIN_VALUE;
               return MainActivity.this.aaa(this);
            }
         };
      }

      label22: {
         Object $result = ((<undefinedtype>)$continuation).result;
         Object var6 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
         switch(((<undefinedtype>)$continuation).label) {
         case 0:
            ResultKt.throwOnFailure($result);
            ((<undefinedtype>)$continuation).L$0 = this;
            ((<undefinedtype>)$continuation).label = 1;
            if (DelayKt.delay(100L, (Continuation)$continuation) == var6) {
               return var6;
            }
            break;
         case 1:
            this = (MainActivity)((<undefinedtype>)$continuation).L$0;
            ResultKt.throwOnFailure($result);
            break;
         case 2:
            ResultKt.throwOnFailure($result);
            break label22;
         default:
            throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
         }

         ((<undefinedtype>)$continuation).L$0 = null;
         ((<undefinedtype>)$continuation).label = 2;
         if (this.bbb((Continuation)$continuation) == var6) {
            return var6;
         }
      }

      String var2 = "aaa";
      boolean var3 = false;
      System.out.println(var2);
      return Unit.INSTANCE;
   }

   @Nullable
   public final Object bbb(@NotNull Continuation var1) {
      Object $continuation;
      label20: {
         if (var1 instanceof <undefinedtype>) {
            $continuation = (<undefinedtype>)var1;
            if ((((<undefinedtype>)$continuation).label & Integer.MIN_VALUE) != 0) {
               ((<undefinedtype>)$continuation).label -= Integer.MIN_VALUE;
               break label20;
            }
         }

         $continuation = new ContinuationImpl(var1) {
            // $FF: synthetic field
            Object result;
            int label;

            @Nullable
            public final Object invokeSuspend(@NotNull Object $result) {
               this.result = $result;
               this.label |= Integer.MIN_VALUE;
               return MainActivity.this.bbb(this);
            }
         };
      }

      Object $result = ((<undefinedtype>)$continuation).result;
      Object var6 = IntrinsicsKt.getCOROUTINE_SUSPENDED();
      switch(((<undefinedtype>)$continuation).label) {
      case 0:
         ResultKt.throwOnFailure($result);
         ((<undefinedtype>)$continuation).label = 1;
         if (DelayKt.delay(100L, (Continuation)$continuation) == var6) {
            return var6;
         }
         break;
      case 1:
         ResultKt.throwOnFailure($result);
         break;
      default:
         throw new IllegalStateException("call to 'resume' before 'invoke' with coroutine");
      }

      String var2 = "bbb";
      boolean var3 = false;
      System.out.println(var2);
      return Unit.INSTANCE;
   }
```
上面的代码基本说明了协程通过状态机运行的，但是仍然有两点问题：
1. 状态机是如何开始如何结束的？
2. 大状态机如何切换到小状态机，小状态机如何切回大状态机？


## 状态机工作原理
`BaseContinuationImpl` 是所有编译器自动生成的类的父类；

```kotlin
internal abstract class BaseContinuationImpl(
    // 包裹当前状态机的上层状态机
    public val completion: Continuation<Any?>?
) : Continuation<Any?>, CoroutineStackFrame, Serializable {
    // 最终实现，不允许重写；
    public final override fun resumeWith(result: Result<Any?>) {
      
        var current = this
        var param = result
        while (true) {
            // Invoke "resume" debug probe on every resumed continuation, so that a debugging library infrastructure
            // can precisely track what part of suspended callstack was already resumed
            probeCoroutineResumed(current)
            with(current) {
                val completion = completion!!
                val outcome: Result<Any?> =
                    try {
                        val outcome = invokeSuspend(param)
                        // 当前状态机调用了挂起函数，不能立即返回，停止循环；
                        if (outcome === COROUTINE_SUSPENDED) return
                        // 当前状态机达到最终状态，拿到了结果；
                        Result.success(outcome)
                    } catch (exception: Throwable) {
                        Result.failure(exception)
                    }
                releaseIntercepted() // this state machine instance is terminating
                // 当前状态机已结束，开始向上递归；
                // 如果当前状态机是 BaseContinuationImpl 的子类，则改变 current 和 param 继续循环；
                if (completion is BaseContinuationImpl) {
                    // unrolling recursion via loop
                    current = completion
                    param = outcome
                } else {
                    // 达到顶层状态机，不是 BaseContinuationImpl 的子类，直接调用它的 resumeWith 函数；
                    completion.resumeWith(outcome)
                    return
                }
            }
        }
    }

   // 包含状态机实现
    protected abstract fun invokeSuspend(result: Result<Any?>): Any?

    ......
}
```

## 线程调度
线程调度分为两个部分：
1. 切到子状态机时，使用指定线程；
2. 切回上层状态机时，变回原来的线程；

```kotlin
public suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    // suspendCoroutineUninterceptedOrReturn 方法由编译器实现，uCont 指的是当前协程实例；
    return suspendCoroutineUninterceptedOrReturn sc@ { uCont ->
        // compute new context
        val oldContext = uCont.context
        val newContext = oldContext + context
        // always check for cancellation of new context
        newContext.ensureActive()
        // FAST PATH #1 -- new context is the same as the old one
        if (newContext === oldContext) {
            val coroutine = ScopeCoroutine(newContext, uCont)
            return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
        }
        // FAST PATH #2 -- the new dispatcher is the same as the old one (something else changed)
        // `equals` is used by design (see equals implementation is wrapper context like ExecutorCoroutineDispatcher)
        if (newContext[ContinuationInterceptor] == oldContext[ContinuationInterceptor]) {
            val coroutine = UndispatchedCoroutine(newContext, uCont)
            // There are changes in the context, so this thread needs to be updated
            withCoroutineContext(newContext, null) {
                return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
            }
        }
        // SLOW PATH -- use new dispatcher
        val coroutine = DispatchedCoroutine(newContext, uCont)
        block.startCoroutineCancellable(coroutine, coroutine)
        coroutine.getResult()
    }
}

// 
internal fun <R, T> (suspend (R) -> T).startCoroutineCancellable(
    receiver: R, completion: Continuation<T>,
    onCancellation: ((cause: Throwable) -> Unit)? = null
) =
    runSafely(completion) {
        createCoroutineUnintercepted(receiver, completion).intercepted().resumeCancellableWith(Result.success(Unit), onCancellation)
    }

// 内部实现
public actual fun <R, T> (suspend R.() -> T).createCoroutineUnintercepted(
    receiver: R,
    completion: Continuation<T>
): Continuation<Unit> {
    val probeCompletion = probeCoroutineCreated(completion)
    return if (this is BaseContinuationImpl)
        create(receiver, probeCompletion)
    else {
        createCoroutineFromSuspendFunction(probeCompletion) {
            (this as Function2<R, Continuation<T>, Any?>).invoke(receiver, it)
        }
    }
}

// ContinuationImpl.kt
// 根据 ContinuationInterceptor 构建新的 Continuation
public fun intercepted(): Continuation<Any?> =
        intercepted
            ?: (context[ContinuationInterceptor]?.interceptContinuation(this) ?: this)
                .also { intercepted = it }

// CoroutineDispatcher.kt
// 构建 DispatchedContinuation 对象
public final override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> =
        DispatchedContinuation(this, continuation)

// DispatchedContinuation.kt
// 扩展方法，根据类型不同特殊处理；
public fun <T> Continuation<T>.resumeCancellableWith(
    result: Result<T>,
    onCancellation: ((cause: Throwable) -> Unit)? = null
): Unit = when (this) {
    is DispatchedContinuation -> resumeCancellableWith(result, onCancellation)
    else -> resumeWith(result)
}

// DispatchedContinuation.kt
    inline fun resumeCancellableWith(
        result: Result<T>,
        noinline onCancellation: ((cause: Throwable) -> Unit)?
    ) {
        val state = result.toState(onCancellation)
        // dispatcher 是 CoroutineDispatcher 的子类，通过构造函数传入；
        if (dispatcher.isDispatchNeeded(context)) {
            _state = state
            resumeMode = MODE_CANCELLABLE
            dispatcher.dispatch(context, this)
        } else {
            executeUnconfined(state, MODE_CANCELLABLE) {
                if (!resumeCancelled(state)) {
                    resumeUndispatchedWith(result)
                }
            }
        }
    }
// CoroutineDispatcher.kt
// DispatchedContinuation 的父类 DispatchedTask 实现了 runnable 接口；
public abstract fun dispatch(context: CoroutineContext, block: Runnable)

// LimitingDispatcher.kt
// 这个类实现了线程池的相关功能
```


```kotlin
// DispatchedContinuation.kt
override fun resumeWith(result: Result<T>) {
        val context = continuation.context
        val state = result.toState()
        // 执行状态机，继续分发到线程池
        if (dispatcher.isDispatchNeeded(context)) {
            _state = state
            resumeMode = MODE_ATOMIC
            dispatcher.dispatch(context, this)
        } else {
           // 状态机结束，调用上层状态机 DispatchedCoroutine 的 resumeWith 方法, 此时仍然在线程池中调用这一方法；
            executeUnconfined(state, MODE_ATOMIC) {
                withCoroutineContext(this.context, countOrElement) {
                    continuation.resumeWith(result)
                }
            }
        }
    }

// AbstractCoroutine.kt
public final override fun resumeWith(result: Result<T>) {
        val state = makeCompletingOnce(result.toState())
        if (state === COMPLETING_WAITING_CHILDREN) return
        afterResume(state)
    }

protected open fun afterResume(state: Any?): Unit = afterCompletion(state)

// DispatchedCoroutine.kt
// 切回之前的线程
override fun afterResume(state: Any?) {
        if (tryResume()) return // completed before getResult invocation -- bail out
        // Resume in a cancellable way because we have to switch back to the original dispatcher
        uCont.intercepted().resumeCancellableWith(recoverResult(state, uCont))
    }
```