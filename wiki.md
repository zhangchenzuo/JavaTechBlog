# Tech sharing
## airflow
Workflow management platform for data engineering pipeline
Programmatically author, schedule and monitor existing tasks

> concepts

- DAG: Scheduler looks at DAG’s schedules and kick off a DAG run
- Operator: Operator is a template for the task, like the operator will submit a spark job. 
- Task: This task is going to be this operator with parameters. You are taking the operator and instantiate into a task

Dag runs turns into a DagRun, all those tasks which are instances of operators then become task instances

> database info

- Dag info: describes an instance of dag, execution_date, start_date, end_date
- Task info: task_id, dag_id, job_id, execution_date, start_date, end_date

> 生命周期

> component

- Scheduler: work out what task instance need to run.
- Executor: run task and record result.