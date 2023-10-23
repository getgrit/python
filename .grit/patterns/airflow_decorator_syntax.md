---
title: Use modern decorator syntax to define airflow DAGs.
---

Airflow supports decorators 

```grit
engine marzano(0.1)
language python

function_returning_DAG($return_type, $body, $name, $parameters, $kwargs) =>
    `@dag($kwargs)
def $name($parameters):
    $body`

```

## Works as expected on functions that return DAGs

```python
def do_my_thing() -> DAG:
    dag = DAG(description="My cool DAG")
    def do_thing(**context: T.Any) -> bool:
        return not aws_rds.db_exists(region=get_variable(EnvVarKeys.TARGET))
    def do_thing_two(**context: T.Any) -> bool:
        pass
    other_operator = ShortCircuitOperator(
        dag=dag,
        task_id='do_db_thing',
        python_callable=do_thing,
        provide_context=True,
    )
    operator_two = PythonOperator(python_callable=do_thing_two)
    other_operator >> operator_two
    do_thing >> do_thing_two
```

```python
@dag(description="My cool DAG")
def do_my_thing():
    
    @task(task_id='do_db_thing', provide_context=True)
    def do_thing(**context: T.Any) -> bool:
        return not aws_rds.db_exists(region=get_variable(EnvVarKeys.TARGET))
    @task()
    def do_thing_two(**context: T.Any) -> bool:
        pass
    other_operator = do_thing
    operator_two = do_thing_two
    other_operator >> operator_two
    do_thing >> do_thing_two
```