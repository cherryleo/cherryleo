---
weight: 14
title: "Borg翻译（三）"
date: "2018-11-29"
lastmod: "2018-11-29"
description:  "Borg如何管理和分配集群硬件资源，Borg上的任务如何访问与监控，任务的优先级如何分配"
categories:  ["kubernetes"]
tags: ["kubernetes"]
---

## 课代表划重点

## Borg架构

## Borg Master
每个 __cell__ 的Borgmaster由两个进程组成：__Borgmaster进程__ 和一个单独的 __调度器（scheduler）__。主Borgmaster进程处理客户端RPC，这些RPC要么改变状态（例如，创建作业），要么提供对数据的只读访问（例如，查找作业）。它还管理系统中的 __所有对象（machines, tasks, allocs等）__ 的状态机，与Borglets通信，并提供WebUI作为Sigma的备份。

Each replica maintains an in-memory copy of most of the state of the cell, and this state is also recorded in a highly-available, distributed, Paxos-based store on the replicas’ local disks. A single elected master per cell serves both as the Paxos leader and the state mutator, handling all operations that change the cell’s state, such as submitting a job or terminating a task on a machine. A master is elected (using Paxos) when the cell is brought up and whenever the elected master fails; it acquires a Chubby lock so other systems can find it. Electing a master and failing-over to the new one typically takes about 10s, but can take up to a minute in a big cell because some in-memory state has to be reconstructed. When a replica recovers from an outage, it dynamically re-synchronizes its state from other Paxos replicas that are up-to-date.

Borgmaster在逻辑上是一个单独的进程，但是实际上被复制了五次（五个副本）。每个副本维护cell的大部分状态的内存副本，并且该状态还记录在副本的本地磁盘上的高可用性、分布式、基于Paxos的存储器中。每个单元中单个选择的主单元同时充当Paxos领导者和状态变异器，处理改变单元状态的所有操作，例如提交作业或终止机器上的任务。当单元格启动时，每当所选的主单元格失败时，都会（使用Paxos）选择主单元；它获取Chubby锁，以便其他系统能够找到它。选择主控程序和故障转移到新主控程序通常需要大约10秒，但是在大单元格中可能需要长达一分钟，因为必须重建一些内存中的状态。当副本从中断恢复时，它会动态地从其他最新的Paxos副本重新同步其状态。

The Borgmaster’s state at a point in time is called a checkpoint, and takes the form of a periodic snapshot plus a change log kept in the Paxos store. Checkpoints have many uses, including restoring a Borgmaster’s state to an arbitrary point in the past (e.g., just before accepting a request that triggered a software defect in Borg so it can be debugged); fixing it by hand in extremis; building a persistent log of events for future queries; and offline simulations.

Borgmaster在某个时间点的状态称为检查点，并采用定期快照和Paxos存储中保存的更改日志的形式。检查点有很多用途，包括将Borgmaster的状态恢复到过去的任意点（例如，刚好在接收到触发Borg中的软件缺陷以便进行调试的请求之前）；在极端情况下手工修复它；为将来的查询构建事件的持久日志；以及离线模拟。

A high-fidelity Borgmaster simulator called Fauxmaster can be used to read checkpoint files, and contains a complete copy of the production Borgmaster code, with stubbed-out interfaces to the Borglets. It accepts RPCs to make state machine changes and perform operations, such as “schedule all pending tasks”, and we use it to debug failures, by interacting with it as if it were a live Borgmaster, with simulated Borglets replaying real interactions from the checkpoint file. A user can step through and observe the changes to the system state that actually occurred in the past. Fauxmaster is also useful for capacity planning (“how many new jobs of this type would fit?”), as well as sanity checks before making a change to a cell’s configuration (“will this change evict any important jobs?”).
一个名为Fauxmaster的高保真Borgmaster模拟器可以用来读取检查点文件，它包含一个完整的产品Borgmaster代码副本，以及到borglet的外接接口。它接受rpc来进行状态机更改和执行操作，例如“调度所有挂起的任务”，我们使用它来调试失败，方法是与它交互，就像它是一个实时的Borgmaster一样，模拟borglet从检查点文件中回放真实的交互。用户可以逐步完成并观察过去实际发生的对系统状态的更改。Fauxmaster还可以用于容量规划(“这种类型的新作业适合多少个?”)，以及在更改单元的配置之前进行完整性检查(“这种更改是否会驱逐任何重要的作业?”)。

## Scheduling（调度）
当一个作业被提交时，Borgmaster将作业持久化地记录在Paxos存储中，并将 __作业的任务__ 添加到 __挂起的队列（pending queue）__ 中。调度器异步扫描队列，如果机器上有足够的可用资源满足作业的约束，调度器将向该机器分配任务。（调度器操作的粒度是任务，而不是作业。）扫描过程从高优先级到低优先级，由优先级内的轮询机制进行调整，以确保用户之间的公平性，并避免在大型作业之后出现在 __线头阻塞（head-of-line blocking）__。该调度算法由两部分组成:可行性检验，寻找任务可以在其上运行的机器;评分，选择可行机器中的一台。

The score takes into account user-specified preferences, but is mostly driven by built-in criteria such as minimizing the number and priority of preempted tasks, picking machines that already have a copy of the task’s packages, spreading tasks across power and failure domains, and packing quality including putting a mix of high and low priority tasks onto a single machine to allow the high-priority ones to expand in a load spike.

在可行性检查中，调度器找到一组满足任务约束的机器，并且有足够的“可用”资源——其中包括分配给低优先级可回收任务的资源。在评分中，调度器确定每个可行机器的“优劣”。评分考虑了用户指定的首选项，但主要是由内置标准驱动的，比如最小化抢占任务的数量和优先级、挑选已经具有任务包副本的机器、将任务分布在电源和故障域以及打包质量。包括将高优先级任务和低优先级任务的混合放到单个机器上，以允许高优先级任务在负载峰值中扩展。

比分考虑指定的偏好,但主要是由内置的标准如减少的数量和优先级抢占的任务,选择的机器,已经有一个副本任务的包,传播任务在权力和失败的领域,和包装质量包括将混合的高和低优先级任务到一个机器允许高优先级的扩大在负载峰值。

Borg originally used a variant of E-PVM [4] for scoring, which generates a single cost value across heterogeneous resources and minimizes the change in cost when placing a task. In practice, E-PVM ends up spreading load across all the machines, leaving headroom for load spikes – but at the expense of increased fragmentation, especially for large tasks that need most of the machine; we sometimes call this “worst fit”.

The opposite end of the spectrum is “best fit”, which tries to fill machines as tightly as possible. This leaves some ma- chines empty of user jobs (they still run storage servers), so placing large tasks is straightforward, but the tight packing penalizes any mis-estimations in resource requirements by users or Borg. This hurts applications with bursty loads, and is particularly bad for batch jobs which specify low CPU needs so they can schedule easily and try to run opportunis- tically in unused resources: 20% of non-prod tasks request less than 0.1 CPU cores.

Our current scoring model is a hybrid one that tries to reduce the amount of stranded resources – ones that cannot be used because another resource on the machine is fully allocated. It provides about 3–5% better packing efficiency (defined in [78]) than best fit for our workloads.

If the machine selected by the scoring phase doesn’t have enough available resources to fit the new task, Borg preempts (kills) lower-priority tasks, from lowest to highest priority, until it does. We add the preempted tasks to the scheduler’s pending queue, rather than migrate or hibernate them.3

Task startup latency (the time from job submission to a task running) is an area that has received and continues to receive significant attention. It is highly variable, with the median typically about 25 s. Package installation takes about 80% of the total: one of the known bottlenecks is contention for the local disk where packages are written to. To reduce task startup time, the scheduler prefers to assign tasks to machines that already have the necessary packages (programs and data) installed: most packages are immutable and so can be shared and cached. (This is the only form of data locality supported by the Borg scheduler.) In addition, Borg distributes packages to machines in parallel using tree- and torrent-like protocols.

Additionally, the scheduler uses several techniques to let it scale up to cells with tens of thousands of machines (§3.4).

## Borglet
The Borglet is a local Borg agent that is present on every machine in a cell. It starts and stops tasks; restarts them if they fail; manages local resources by manipulating OS kernel settings; rolls over debug logs; and reports the state of the machine to the Borgmaster and other monitoring systems.

The Borgmaster polls each Borglet every few seconds to retrieve the machine’s current state and send it any outstanding requests. This gives Borgmaster control over the rate of communication, avoids the need for an explicit flow control mechanism, and prevents recovery storms [9].

The elected master is responsible for preparing messages to send to the Borglets and for updating the cell’s state with their responses. For performance scalability, each Borgmaster replica runs a stateless link shard to handle the communication with some of the Borglets; the partitioning is recalculated whenever a Borgmaster election occurs. For resiliency, the Borglet always reports its full state, but the link shards aggregate and compress this information by reporting only differences to the state machines, to reduce the update load at the elected master.

If a Borglet does not respond to several poll messages its machine is marked as down and any tasks it was running are rescheduled on other machines. If communication is restored the Borgmaster tells the Borglet to kill those tasks that have been rescheduled, to avoid duplicates. A Borglet continues normal operation even if it loses contact with the Borgmaster, so currently-running tasks and services stay up even if all Borgmaster replicas fail.

## Scalability
We are not sure where the ultimate scalability limit to Borg’s centralized architecture will come from; so far, every time we have approached a limit, we’ve managed to eliminate it. A single Borgmaster can manage many thousands of ma- chines in a cell, and several cells have arrival rates above 10000 tasks per minute. A busy Borgmaster uses 10–14 CPU cores and up to 50 GiB RAM. We use several tech- niques to achieve this scale.

Early versions of Borgmaster had a simple, synchronous loop that accepted requests, scheduled tasks, and commu- nicated with Borglets. To handle larger cells, we split the scheduler into a separate process so it could operate in paral- lel with the other Borgmaster functions that are replicated for failure tolerance. A scheduler replica operates on a cached copy of the cell state. It repeatedly: retrieves state changes from the elected master (including both assigned and pend- ing work); updates its local copy; does a scheduling pass to assign tasks; and informs the elected master of those as- signments. The master will accept and apply these assign- ments unless they are inappropriate (e.g., based on out of date state), which will cause them to be reconsidered in the scheduler’s next pass. This is quite similar in spirit to the optimistic concurrency control used in Omega [69], and in- deed we recently added the ability for Borg to use different schedulers for different workload types.

To improve response times, we added separate threads to talk to the Borglets and respond to read-only RPCs. For greater performance, we sharded (partitioned) these func- tions across the five Borgmaster replicas §3.3. Together, these keep the 99%ile response time of the UI below 1s and the 95%ile of the Borglet polling interval below 10 s.

Several things make the Borg scheduler more scalable:
Score caching: Evaluating feasibility and scoring a ma- chine is expensive, so Borg caches the scores until the prop- erties of the machine or task change – e.g., a task on the ma- chine terminates, an attribute is altered, or a task’s require- ments change. Ignoring small changes in resource quantities reduces cache invalidations.
Equivalence classes: Tasks in a Borg job usually have identical requirements and constraints, so rather than deter- mining feasibility for every pending task on every machine, and scoring all the feasible machines, Borg only does fea- sibility and scoring for one task per equivalence class – a group of tasks with identical requirements.
Relaxed randomization: It is wasteful to calculate fea- sibility and scores for all the machines in a large cell, so the scheduler examines machines in a random order until it has found “enough” feasible machines to score, and then selects the best within that set. This reduces the amount of scoring and cache invalidations needed when tasks enter and leave the system, and speeds up assignment of tasks to machines. Relaxed randomization is somewhat akin to the batch sam- pling of Sparrow [65] while also handling priorities, preemp- tions, heterogeneity and the costs of package installation.

In our experiments (§5), scheduling a cell’s entire work- load from scratch typically took a few hundred seconds, but did not finish after more than 3 days when the above tech- niques were disabled. Normally, though, an online schedul- ing pass over the pending queue completes in less than half a second.