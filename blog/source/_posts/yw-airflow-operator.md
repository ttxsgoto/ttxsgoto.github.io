---
title: Airflow变量和执行器
date: 2019-07-27 16:15:08
tags:
  - Airflow
categories:
  - 运维
---
#### 问题描述
最近在调研Airflow demo相关的问题和解决方案， 主要问题有：

- Dags中任务启动时，参数如何传递
- Task任务之间的依赖关系，返回值如何被其他task使用
- 运行docker程序
- Http API请求实现

#### 具体说明
##### Dags中任务启动时，参数如何传递
Airflow中可以使用Variables来定义变量来传递参数，该变量为全局变量
```python
# 设置变量
airflow variables --set keyName value # 或者管理UI设置
 
# 获取变量
from airflow.models import Variable
message = Variable.get('message')
```
##### Task任务之间的依赖关系，返回值如何被其他task使用
通过xcom来返回给后面的task任务使用任务的返回值，使用kwargs['task_instance'].xcom_pull(task_ids='run_task')来获取run_task任务的返回值
```python
# coding: utf8
 
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import timedelta, datetime
from airflow.models import Variable
 
default_args = {
    'owner': 'airflow',
    'description': 'Use of the Xcom',
    'depend_on_past': False,
    'start_date': datetime(2019, 6, 3),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=30)
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
    # 'wait_for_downstream': False,
    # 'dag': dag,
    # 'adhoc':False,
    # 'sla': timedelta(hours=2),
    # 'execution_timeout': timedelta(seconds=300),
    # 'on_failure_callback': some_function,
    # 'on_success_callback': some_other_function,
    # 'on_retry_callback': another_function,
    # 'trigger_rule': u'all_success'
}
 
dag = DAG(
    'xcom_demo',
    default_args=default_args,
    schedule_interval=None
)
 
 
def run_this_func(**kwargs):
    message = Variable.get('message')
    print('message------>', message)
    return message
 
 
run_task = PythonOperator(
    task_id='run_task',
    provide_context=True,
    python_callable=run_this_func,
    dag=dag
)
 
 
def print_hello(**context):
    before_data = context['task_instance'].xcom_pull(task_ids='run_task')
    return before_data
 
 
hello_operator = PythonOperator(
    task_id='hello_task',
    python_callable=print_hello,
    provide_context=True,
    dag=dag,
)
 
 
def three(**kwargs):
    frist_data = kwargs['task_instance'].xcom_pull(task_ids='run_task')
    two_data = kwargs['task_instance'].xcom_pull(task_ids='hello_task')
    return frist_data, two_data
 
 
last_operator = PythonOperator(
    task_id='last_task',
    python_callable=three,
    provide_context=True,
    dag=dag,
)
 
run_task >> hello_operator >> last_operator  # xcoms
 
if __name__ == "__main__":
    dag.cli()
 
 
```
##### 运行docker程序
执行器DockerOperator 完成docker运行
```python
# coding: utf-8
 
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta
from airflow.operators.docker_operator import DockerOperator
 
default_args = {
    'owner': 'airflow',
    'description': 'Use of the DockerOperator',
    'depend_on_past': False,
    'start_date': datetime(2019, 6, 3),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=30)
}
 
image = 'docker.api:0.1.0'
volumes = ['/home/user/data:/data']
run_commend = 'cd /data/ && ./run.sh'
with DAG('docker_demo', default_args=default_args, schedule_interval=None, catchup=False) as dag:
    t1 = BashOperator(
        task_id='print_current_date',
        bash_command='date'
    )
 
    t2 = DockerOperator(
        task_id='dpt_docker',
        image=image,
        auto_remove=True,
        command=run_commend,
        force_pull=True,
        volumes=volumes,
        # network_mode='bridge'
    )
 
    t3 = BashOperator(
        task_id='print_hello',
        bash_command='echo "hello world"'
    )
 
    t1 >> t2 >> t3
 
if __name__ == "__main__":
    dag.cli()
 
```
##### Http API请求实现
执行器SimpleHttpOperator 完成http api请求
```python
# coding: utf-8
import json
import os
import time
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from airflow.utils.trigger_rule import TriggerRule
from datetime import datetime, timedelta
from airflow.operators.http_operator import SimpleHttpOperator
 
from airflow.models import Variable
 
default_args = {
    'owner': 'airflow',
    'description': 'Use of the SimpleHttpOperator',
    'depend_on_past': False,
    'start_date': datetime(2019, 6, 3),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=30)
}
 
http_add = 'http://127.0.0.1:8888'
api = '/person/'
url = 'http://1027.0.0.1:8889'
login_api = '/user/login/'
get_task_api = '/task/'
os.environ[
    'AIRFLOW_CONN_HTTP_TEST'] = http_add  # 这里定义不同的接口规则,在SimpleHttpOperator中http_conn_id需要指定IRFLOW_CONN_*对应的内容,默认接口指向google的api
os.environ['AIRFLOW_CONN_TEST_HTTP'] = url
 
token = ''
 
 
def get_http_data(**context):
    token_data = context['task_instance'].xcom_pull(task_ids='post_login')
    token_dict = json.loads(token_data)
    token = token_dict['data']['token']
    Variable.set('token', token)
    return token
 
 
def get_data(**context):
    time.sleep(10)
    return token
 
 
with DAG('http_api_demo',
         default_args=default_args,
         schedule_interval="5 * * * *",
         catchup=False) as dag:
    t1 = BashOperator(
        task_id='print_current_date',
        bash_command='date'
    )
 
    t2 = SimpleHttpOperator(
        task_id='get_person',
        http_conn_id='http_test',
        method='GET',
        headers={"Content-Type": "application/json"},
        endpoint=api,
        xcom_push=True,  # 将结果通过xcom传递给下一个task
        response_check=lambda response: True if response.status_code == 200 else False,
    )
    t3 = SimpleHttpOperator(
        task_id='post_login',
        http_conn_id='test_http',
        method='POST',
        headers={
            "X-Requested-With": 'XMLHttpRequest',
            "Accept": "application/json",
            "Content-Type": "application/json; charset=UTF-8"
        },
        endpoint=login_api,
        xcom_push=True,  # 将结果通过xcom传递给下一个task
        response_check=lambda response: True if response.status_code == 200 else False,
        data=json.dumps({'username': 'admin', 'password': 'admin'}),
    )
    # PostgresOperator
    t4 = SimpleHttpOperator(
        task_id='get_task',
        http_conn_id='test_http',
        method='GET',
        headers={
            "X-Requested-With": 'XMLHttpRequest',
            "Accept": "application/json",
            "Content-Type": "application/json; charset=UTF-8",
            "Authorization": 'jwt {}'.format(Variable.get('token')),  # 这里需要获取到login的token
        },
        endpoint=get_task_api,
        xcom_push=True,  # 将结果通过xcom传递给下一个task
        response_check=lambda response: True if response.status_code == 200 else False,
        trigger_rule=TriggerRule.NONE_FAILED
    )
 
    t5 = PythonOperator(
        task_id='data_task',
        python_callable=get_http_data,
        provide_context=True,
        # trigger_rule=TriggerRule.ONE_SUCCESS
    )
    t6 = PythonOperator(
        task_id='test_data',
        python_callable=get_data,
        provide_context=True
    )
    t7 = PythonOperator(
        task_id='sleep_data',
        python_callable=get_data,
        provide_context=True,
    )
 
    t1 >> t2 >> t3 >> [t5, t6] >> t4 >> t7
 
if __name__ == "__main__":
    dag.cli()
 
```

