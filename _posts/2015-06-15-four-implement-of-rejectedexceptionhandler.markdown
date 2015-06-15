---
layout: post
title:  "ThreadPoolExecutor 的四种 RejectedExecutionHandler"
date:   2015-06-15 
categories: java
---

ThreadPoolExecutor 的四种 RejectedExecutionHandler

当queue填满后会触发RejectedExecutionHandler,ThreadPoolExecutor提供了四种实现：

* CallerRunsPolicy

{% highlight java %}
 if (!e.isShutdown()) {
        r.run();
    }
{% endhighlight%}

executor调度主线程直接执行该任务。当使用线程池进行异步任务处理时，要谨慎使用该policy.如果进行异步任务调度时，使用该策略，如某任务执行过慢导致队列满了，任务执行会转交给主线程，任务将变成同步，会导致主线程堵塞。

* AbortPolicy

{% highlight java %}
throw new RejectedExecutionException("Task " + r.toString() +
                " rejected from " +
                e.toString());
{% endhighlight%}

中止任务，并抛出RejectedException(继承RuntimeException)。ThreadPoolExecutor默认的rejectedexceptionhandler
{% highlight java %}
 private static final RejectedExecutionHandler defaultHandler =
        new AbortPolicy();
{% endhighlight%}

* DiscardPolicy
抛弃试图新增的任务，不抛出任何异常

* DiscardOldestPolicy
{% highlight java %}
 if (!e.isShutdown()) {
        e.getQueue().poll();
        e.execute(r);
 }
{% endhighlight%}
移除队列中最老的一个任务，然后继续执行新增的任务