---
layout: post
title: "ThreadPool - 线程池"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C#
---

使用ThreadPool线程池来实现异步功能

在Unity开发中，很多时候要用到Coroutine/协程来进行复杂的逻辑控制和异步逻辑的实现（如实时反馈的资源加载）。

但协程不是真正的多线程，仍然运行在主线程中。

ThreadPool是.NET自带的线程池静态类，用它可以真正地实现多线程，起到加速计算的作用，同时它的使用比较简单。

ThreadPool也可以简单地实现一个异步功能，以一个利用Gerstner波模拟海面的程序为例，使用CPU计算海面高度场是比较耗时的。

开辟多线程进行这种计算，就可以极大地减轻主线程的负担（避免画面卡顿），同时要注意同步和更新。

``` C#
private void Update()
{
    // 检查运行情况 done变量用来控制同步
    if (enableMultiThread && !done) return;

    // 更新Mesh计算结果
    mesh.SetVertices(vertices);
    mesh.SetUVs(0, uvs);
    mesh.SetIndices(indexs, MeshTopology.Triangles, 0);
    mesh.RecalculateNormals();
    mesh.RecalculateBounds();
    mesh.RecalculateTangents();

    // 新的计算任务
    if (enableMultiThread)
    {
        done = false;

        ThreadPool.QueueUserWorkItem(new WaitCallback(ComputingTask), Time.realtimeSinceStartup);
    }
    else
    {
        UpdateMesh(Time.realtimeSinceStartup);
    }
}

private void ComputingTask(object o)
{
    UpdateMesh((float)o);
    
    done = true;
}
```

![开启异步 FPS 400](/res/notes/threadpool1.png)

(开启异步后，FPS提升至400左右)

![未开启异步 FPS 30](/res/notes/threadpool2.png)

(未开启异步，FPS只有30左右)

由此可以看出，使用ThreadPool来执行海面高度计算，可以避免复杂计算带来的帧数降低

在实际的场景中，可以对一些计算复杂度大、但不要求经常更新的逻辑采用这种方式进行。

以上案例只是开辟一个新线程进行某种特定地运算，而没有真正地利用多线程提高计算效率。

仍然使用线程池，我们把计算的总顶点数分为N份，然后使用QueueUserWorkItem来创建N个线程，就可以提高计算效率。这个问题中顶点都是独立的，不涉及同步问题，只要把总任务细分即可。

多线程优化涉及到计算任务的分配、调度和同步，总的来说是比较复杂的。

ThreadPool使用简单，但是局限性较大，要想灵活地使用多线程，还是需要利用Thread类，但使用较为复杂。

ThreadPool的局限性

1. 线程池创建的线程都是后台线程

2. 线程池创建的线程不能设名称和优先级

3. 不适合一直运行的任务，应用Thread类代替

---