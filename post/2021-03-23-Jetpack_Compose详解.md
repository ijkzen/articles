---
title: Jetpack Compose详解
categories: Android原理
---

## 概论

在2021年3月1日，`Jetpack Compose`发布了`Beta`版本，可喜可贺。

`Jetpack Compose`的出现一定是因为当前的`View`体系存在一些问题，且听我娓娓道来；

### View的实质

我们所看到的屏幕实际上一块块大小不一的画布堆叠排版形成的，同时每块画布还要能响应用户操作和更新内容；

所以一块画布有以下属性：

1. 画布大小
2. 画布位置
3. 画布内容
4. 触摸事件
5. 状态更新

### View-based UI 工作过程

原有的`View-based` UI 将上述 5 个属性封装到一个类中，

1. 画布大小，`onMeasure`负责；
2. 画布位置，`onLayout`负责；
3. 画布内容，`onDraw`负责；
4. 触摸事件，`onTouchEvent`负责；
5. 状态更新，下面详细讲解；

我们在`xml`文件中定义页面的`结构树`，这里是声明式的，然后通过 `LayoutInflater` 将 `结构树` 转化为看到的 `视图树`；

然后通过 `findViewById` 获取对应视图的实例，然后通过这个实例调用其成员函数更新其状态，这里是命令式的。



