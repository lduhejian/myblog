---
title: "laravel queue(database) 多个 worker 有竞争关系?"
date: 2019-08-17
draft: false
tags: [laravel, queue, database]
---
我有一个疑问：
laravel 队列在数据库驱动的情形下，会不会有竞争问题出现？
例如：jobs 表中的一条记录被读取，但还未在数据库中删除时，另一个 worker 又读取这条记录 ...

这种问题会出现吗？

**让我们看下 laravel 的源码：**

```
/**
 * Get the next available job for the queue.
 *
 * @param  string|null  $queue
 * @return \Illuminate\Queue\Jobs\DatabaseJobRecord|null
 */
protected function getNextAvailableJob($queue)
{
    $job = $this->database->table($this->table)
                ->lockForUpdate()
                ->where('queue', $this->getQueue($queue))
                ->where(function ($query) {
                    $this->isAvailable($query);
                    $this->isReservedButExpired($query);
                })
                ->orderBy('id', 'asc')
                ->first();

    return $job ? new DatabaseJobRecord((object) $job) : null;
}
```

通过阅读源码，我们知道 laravel 是通过上图中 getNextAvailableJob 获取下一个要执行的 job 的

**让我们再看一下  lockForUpdate:**

* 官网是这样说的

>Alternatively, you may use the lockForUpdate method. A "for update" lock prevents the rows from being modified or from being selected with another shared lock.


* 谷歌中的大神是这么说的

![](/uploads/laravel_queue(database)_多个_worker_有竞争关系？/lockForUpdate引用.png)

**结论A：**

laravel 在取出一个 job 后，对数据库中该条记录上锁，其他 worker 会一直等待 ...

**那要等到什么时候呢？**

```
/**
 * Mark the given job ID as reserved.
 *
 * @param  \Illuminate\Queue\Jobs\DatabaseJobRecord  $job
 * @return \Illuminate\Queue\Jobs\DatabaseJobRecord
 */
protected function markJobAsReserved($job)
{
    $this->database->table($this->table)->where('id', $job->id)->update([
        'reserved_at' => $job->touch(),
        'attempts' => $job->increment(),
    ]);

    return $job;
}
```

**结论B：**

在 laravel 取出一个 job 后，会更新 job reserved_at 和 attempts

**结论A+结论B：**

laravel 在取出一个 job 后，对数据库中该条记录上锁，其他 worker 会一直等待直到该 job 被标记为 reserved.

参考:  
https://laravel.com/docs/5.5/queries#pessimistic-locking  
https://laracasts.com/discuss/channels/general-discussion/how-properly-use-the-lockforupdate-method

