# TaskSet

`TaskSet` is a <<tasks, collection of independent tasks>> of a single <<stageId, stage>> (and a <<stageAttemptId, stage execution attempt>>) that are *missing* (_uncomputed_), i.e. for which computation results are unavailable (as RDD blocks on storage:BlockManager.md[BlockManagers] on executors).

In other words, a TaskSet represents the missing partitions of a stage that (as tasks) can be run right away based on the data that is already on the cluster, e.g. map output files from previous stages, though they may fail if this data becomes unavailable.

NOTE: Since the <<tasks, tasks>> of a TaskSet are only the missing tasks, their number does not necessarily have to be the number of all the tasks of a <<stageId, stage>>. For a brand new stage (that has never been attempted to compute) their numbers are exactly the same.

TaskSet is <<creating-instance, created>> exclusively when `DAGScheduler` is requested to scheduler:DAGScheduler.md#submitMissingTasks[submit the missing tasks of a stage].

NOTE: Once scheduler:DAGScheduler.md#submitMissingTasks[submitted] for execution (to a scheduler:TaskScheduler.md[TaskScheduler]), the execution of the TaskSet is managed by a scheduler:TaskSetManager.md[TaskSetManager] that allows for configuration-properties.md#spark.task.maxFailures[spark.task.maxFailures] (default: `1` for <<local/spark-local.md#, Spark on local>> and `4` for <<spark-cluster.md#, Spark clustered>>).

[[creating-instance]]
TaskSet takes the following to be created:

* [[tasks]] Collection of scheduler:Task.md[tasks] (`Array[Task[_]]`)
* [[stageId]] Stage ID
* [[stageAttemptId]] *Stage execution attempt ID*
* [[priority]] *Priority* (for <<fifo-scheduling, FIFO scheduling>>)
* [[properties]] Key-value *properties*

[[id]]
TaskSet is uniquely identified by an `id` that is the <<stageId, stageId>> followed by the <<stageAttemptId, stageAttemptId>> with the comma (`.`) in-between.

```
[stageId].[stageAttemptId]
```

[[toString]]
A textual representation (`toString`) of TaskSet is *TaskSet [id]*.

```
TaskSet [stageId].[stageAttemptId]
```

== [[fifo-scheduling]] Task Scheduling Prioritization in FIFO Scheduling

The <<priority, priority>> of a TaskSet is exactly the ID of the earliest-created active job that needs the stage (that is given when `DAGScheduler` is requested to scheduler:DAGScheduler.md#submitMissingTasks[submit the missing tasks of a stage]).

Once scheduler:DAGScheduler.md#submitMissingTasks[submitted] for execution (to a scheduler:TaskScheduler.md[TaskScheduler]), the <<priority, priority>> of a TaskSet is the scheduler:TaskSetManager.md#priority[priority] of the `TaskSetManager` (which is a <<spark-scheduler-Schedulable.md#, Schedulable>>) that is used for *task prioritization* (_prioritizing scheduling of tasks_) in the <<spark-scheduler-Pool.md#FIFOSchedulingAlgorithm, FIFO>> scheduling mode.
