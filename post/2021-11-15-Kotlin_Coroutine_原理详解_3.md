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
// 第四个参数为 `Function2` 一个有两个参数一个返回值的函数类型，新增了两个方法；其中 `invokeSuspend` 
// 函数编写了状态机的部分；`create` 函数创建了最初的 `Continuation`；实现 `invoke` 
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

         @NotNull
         public final Continuation create(@Nullable Object value, @NotNull Continuation completion) {
            Intrinsics.checkNotNullParameter(completion, "completion");
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
         if (var1 instanceof <undefinedtype>) {
            $continuation = (<undefinedtype>)var1;
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

