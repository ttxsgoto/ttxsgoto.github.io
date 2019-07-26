---
title: Airflow动态生成Tasks和dags
date: 2019-07-28 16:21:22
tags:
  - Airflow
categories:
  - 运维
---

#### 动态生成task任务
现在有这样的需求， 需要根据计算结果动态生成任务列表，来执行dags，查找资料后发现主要使用动态生成TaskInstance实例来完成
具体说明如下：
- 将计算得到值使用Variables变量保存
    ```python
    airflow variables --set keyName value
    ```
- 动态生成TaskInstance实例
- 将计算得到的列表遍历task任务

实例如下
```python
# coding: utf8
from datetime import datetime
import airflow
from airflow.operators.python_operator import PythonOperator
import os
from airflow.models import Variable
import logging
from airflow import configuration as conf
from airflow.models import DagBag, TaskInstance
from airflow import DAG, settings
from airflow.operators.bash_operator import BashOperator
 
main_dag_id = 'DynamicWorkflow2'
 
args = {
    'owner': 'airflow',
    'start_date': airflow.utils.dates.days_ago(2),
    'provide_context': True
}
 
dag = DAG(
    main_dag_id,
    schedule_interval=None,
    default_args=args)
 
 
def start(*args, **kwargs):
    dynamicValue = 1
 
    variableValue = Variable.get("DynamicWorkflow_Group1")
    logging.info("Current DynamicWorkflow_Group1 value is " + str(variableValue))
 
    logging.info("Setting the Airflow Variable DynamicWorkflow_Group1 to " + str(dynamicValue))
    os.system('airflow variables --set DynamicWorkflow_Group1 ' + str(dynamicValue))
 
    variableValue = Variable.get("DynamicWorkflow_Group1")
    logging.info("Current DynamicWorkflow_Group1 value is " + str(variableValue))
    for i in range(dynamicValue):
        resetTasksStatus('firstGroup_' + str(i))
 
 
def resetTasksStatus(task_id):
    dag_folder = conf.get('core', 'DAGS_FOLDER')
    dagbag = DagBag(dag_folder)
    check_dag = dagbag.dags[main_dag_id]
    session = settings.Session()
    execution_date = datetime.now()
    my_task = check_dag.get_task(task_id)
    ti = TaskInstance(my_task, execution_date)
    state = ti.current_state()
    logging.info("Current state of " + task_id + " is " + str(state))
    ti.set_state(None, session)
    state = ti.current_state()
    logging.info("Updated state of " + task_id + " is " + str(state))
 
 
def bridge1(*args, **kwargs):
    dynamicValue = 2
 
    variableValue = Variable.get("DynamicWorkflow_Group2")
    logging.info("Current DynamicWorkflow_Group2 value is " + str(variableValue))
 
    logging.info("Setting the Airflow Variable DynamicWorkflow_Group2 to " + str(dynamicValue))
    os.system('airflow variables --set DynamicWorkflow_Group2 ' + str(dynamicValue))
 
    variableValue = Variable.get("DynamicWorkflow_Group2")
    logging.info("Current DynamicWorkflow_Group2 value is " + str(variableValue))
    for i in range(dynamicValue):
        resetTasksStatus('secondGroup_' + str(i))
 
 
def bridge2(*args, **kwargs):
    dynamicValue = 3
 
    variableValue = Variable.get("DynamicWorkflow_Group3")
    logging.info("Current DynamicWorkflow_Group3 value is " + str(variableValue))
 
    logging.info("Setting the Airflow Variable DynamicWorkflow_Group3 to " + str(dynamicValue))
    os.system('airflow variables --set DynamicWorkflow_Group3 ' + str(dynamicValue))
 
    variableValue = Variable.get("DynamicWorkflow_Group3")
    logging.info("Current DynamicWorkflow_Group3 value is " + str(variableValue))
    for i in range(dynamicValue):
        resetTasksStatus('thirdGroup_' + str(i))
 
 
def end(*args, **kwargs):
    logging.info("Ending")
 
 
starting_task = PythonOperator(
    task_id='start',
    dag=dag,
    provide_context=True,
    python_callable=start,
    op_args=[])
 
bridge1_task = PythonOperator(
    task_id='bridge1',
    dag=dag,
    provide_context=True,
    python_callable=bridge1,
    op_args=[])
 
DynamicWorkflow_Group1 = Variable.get("DynamicWorkflow_Group1")
logging.info("The current DynamicWorkflow_Group1 value is " + str(DynamicWorkflow_Group1))
 
 
def doSomeWork(name, index, *args, **kwargs):
    os.system('touch /home/user/airflow/' + str(name) + str(index) + '.txt')
 
 
for index in range(int(DynamicWorkflow_Group1)):
    dynamicTask = PythonOperator(
        task_id='firstGroup_' + str(index),
        dag=dag,
        provide_context=True,
        python_callable=doSomeWork,
        op_args=['firstGroup', index],
    )
 
    starting_task.set_downstream(dynamicTask)
    dynamicTask.set_downstream(bridge1_task)
 
bridge2_task = PythonOperator(
    task_id='bridge2',
    dag=dag,
    provide_context=True,
    python_callable=bridge2,
    op_args=[])
 
DynamicWorkflow_Group2 = Variable.get("DynamicWorkflow_Group2")
logging.info("The current DynamicWorkflow value is " + str(DynamicWorkflow_Group2))
 
for index in range(int(DynamicWorkflow_Group2)):
    dynamicTask = PythonOperator(
        task_id='secondGroup_' + str(index),
        dag=dag,
        provide_context=True,
        python_callable=doSomeWork,
        op_args=['secondGroup', index])
    bridge1_task >> dynamicTask
    dynamicTask >> bridge2_task
 
ending_task = PythonOperator(
    task_id='end',
    dag=dag,
    provide_context=True,
    python_callable=end,
    op_args=[])
 
DynamicWorkflow_Group3 = Variable.get("DynamicWorkflow_Group3")
logging.info("The current DynamicWorkflow value is " + str(DynamicWorkflow_Group3))
 
for index in range(int(DynamicWorkflow_Group3)):
    if index < (int(DynamicWorkflow_Group3) - 1):
        dynamicTask = PythonOperator(
            task_id='thirdGroup_' + str(index),
            dag=dag,
            provide_context=True,
            python_callable=doSomeWork,
            op_args=['thirdGroup', index])
    else:
        dynamicTask = BashOperator(
            task_id='thirdGroup_' + str(index),
            bash_command='touch /home/user/airflow/thirdGroup_' + str(index) + '.txt',
            dag=dag)
 
    bridge2_task >> dynamicTask
    dynamicTask >> ending_task
 
starting_task >> bridge1_task >> bridge2_task >> ending_task
 
```
执行前：
![](https://ttxsgoto.github.io/img/airflow/airflow01.png)
执行后
![](https://ttxsgoto.github.io/img/airflow/airflow02.png)

#### 动态生成Dags
通过获取变量值，globals实现动态生成dags
实例：
```python
# coding: utf-8
from airflow.operators.bash_operator import BashOperator
from datetime import datetime
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
 
 
def create_dag(dag_id,
               schedule,
               dag_number,
               default_args):
    def hello_world_py(*args):
        print('Hello World')
        print('This is DAG: {}'.format(str(dag_number)))
 
    dag = DAG(dag_id,
              schedule_interval=schedule,
              default_args=default_args)
 
    with dag:
        t1 = PythonOperator(
            task_id='hello_world',
            python_callable=hello_world_py,
            dag_number=dag_number)
        t2 = BashOperator(
            task_id='current_date',
            bash_command='date'
        )
        t1 >> t2
 
    return dag
 
 
def get_api_data():
    data = ['test01', 'test02', 'test03']
    return data
 
 
def create_dags(data=None):
    for n in range(len(data)):
        dag_id = 'dynamic_day_{}'.format(data[n])
        default_args = {'owner': 'airflow',
                        'start_date': datetime(2019, 6, 1)}
        schedule = None
        dag_number = n
        globals()[dag_id] = create_dag(dag_id, schedule, dag_number, default_args)
 
 
data = get_api_data()
create_dags(data)
 
```
![](https://ttxsgoto.github.io/img/airflow/airflow03.png)
#### 链接
https://www.linkedin.com/pulse/dynamic-workflows-airflow-kyle-bridenstine/
https://xbuba.com/questions/41517798
https://www.astronomer.io/guides/dynamically-generating-dags/



