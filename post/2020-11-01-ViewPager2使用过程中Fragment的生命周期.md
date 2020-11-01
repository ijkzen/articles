---
title: ViewPager2使用过程中Fragment的生命周期
categories: android
---

## offscreenPageLimit = 0 的情况
此种情况表示不会预先加载其他的Fragment，仅仅加载第一个Fragment，其生命周期如下：

```
Test fragment 0 onAttach 
Test fragment 0 onCreate 
Test fragment 0 onCreateView 
Test fragment 0 OnActivityCreated 
Test fragment 0 onStart 
Test fragment 0 onResume 
```

### 切换到其他的Fragment

这俩个Fragment的生命周期如下：

```
Test fragment 1 onAttach 
Test fragment 1 onCreate 
Test fragment 1 onCreateView 
Test fragment 1 OnActivityCreated 
Test fragment 1 onStart 
Test fragment 0 onPause 
Test fragment 1 onResume 
```

等第二个Fragment执行到`onStart`后，执行第一个Fragment的`onPause`函数。

### 返回到原来的Fragment

前后两个Fragment的生命周期如下：

```
Test fragment 1 onPause 
Test fragment 0 onResume
```

先执行当前Fragment的`onPause`函数，然后执行原来的Fragment的`onResume`函数。

## offscreenPageLimit != 0 的情况

此种情况将会根据`offscreenPageLimit`的值加载不可见的Fragment，这几个Fragment的生命周如下：

```
Test fragment 0 onAttach 
Test fragment 0 onCreate 
Test fragment 0 onCreateView 
Test fragment 0 OnActivityCreated 
Test fragment 0 onStart 
Test fragment 0 onResume 
Test fragment 1 onAttach 
Test fragment 1 onCreate 
Test fragment 1 onCreateView 
Test fragment 1 OnActivityCreated 
Test fragment 1 onStart 
Test fragment 2 onAttach 
Test fragment 2 onCreate 
Test fragment 2 onCreateView 
Test fragment 2 OnActivityCreated 
Test fragment 2 onStart 
```

这几个Fragment会依次执行其生命周期，但是只有可见的Fragment执行到了`onResume`函数。

## Container Fragment及Child Fragment的生命周期

### Child Fragment入栈

入栈调用的是`replace()`函数。

commitChildFragment操作在Container Fragment的`onCreateView`函数中执行，这两个Fragment的生命周期如下：

```
container fragment onAttach
container fragment onCreate
container fragment onCreateView
container fragment onActivityCreated
Child fragment 0 onAttach 
Child fragment 0 onCreate 
Child fragment 0 onCreateView
Child fragment 0 OnActivityCreated 
container fragment onStart
Child fragment 0 onStart 
container fragment onResume
Child fragment 0 onResume 
```

虽然`commitChildFragment`在`onCreateView`执行，但是child Fragment真正地开始执行生命周期是在`container Fragment`的`onActivityCreated`函数之后。

### Child Fragment 再入栈

前后两个Fragment生命周期如下，`Child Fragment 0`先入栈，`Child Fragment 1`后入栈。

```
Child fragment 1 onAttach 
Child fragment 1 onCreate 
Child fragment 0 onPause 
Child fragment 0 onStop 
Child fragment 0 onDestroyView 
Child Fragment 1 onCreateView
Child fragment 1 OnActivityCreated 
Child fragment 1 onStart 
Child fragment 1 onResume 
```

### Child Fragment 出栈

```
Child fragment 1 onPause 
Child fragment 1 onStop 
Child fragment 1 onDestroyView 
Child fragment 1 onDestroy 
Child fragment 1 onDetach 
Child fragment 0 onCreateView
Child fragment 0 OnActivityCreated 
Child fragment 0 onStart 
Child fragment 0 onResume 
```

