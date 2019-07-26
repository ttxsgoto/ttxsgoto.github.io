---
title: Airflow架构、安装部署
date: 2019-07-26 15:50:35
tags:
  - Airflow
categories:
  - 运维
---

#### 说明
Airflow是一个工作流分配管理系统，通过有向非循环图的方式管理任务流程，设置任务依赖关系和时间调度。Airflow独立于我们要运行的任务，只需要把任务的名字和运行方式提供给Airflow作为一个task就可以， 以代码的方式来定义任务执行流程，可操作性强。

#### Airflow架构
Airflow 是建立在元数据库上的队列系统。数据库存储队列任务的状态，调度器使用这些状态来确定如何将其它任务添加到队列的优先级。此功能由四个主要组件编排
1. 元数据库：这个数据库存储有关任务状态的信息。数据库使用在 SQLAlchemy 中实现的抽象层执行更新。该抽象层将 Airflow 剩余组件功能从数据库中干净地分离了出来。
2. 调度器：调度器是一种使用 DAG 定义结合元数据中的任务状态来决定哪些任务需要被执行以及任务执行优先级的过程。调度器通常作为服务运行。
3. 执行器：Excutor 是一个消息队列进程，它被绑定到调度器中，用于确定实际执行每个任务计划的工作进程。有不同类型的执行器，每个执行器都使用一个指定工作进程的类来执行任务。例如，LocalExecutor 使用与调度器进程在同一台机器上运行的并行进程执行任务。其他像 CeleryExecutor 的执行器使用存在于独立的工作机器集群中的工作进程执行任务。
4. Workers：这些是实际执行任务逻辑的进程，由正在使用的执行器确定。

Airflow 的操作建立于存储任务状态和工作流的元数据库之上（即 DAG）。调度器和执行器将任务发送至队列，让 Worker 进程执行。WebServer 运行（经常与调度器在同一台机器上运行）并与数据库通信，在 Web UI 中呈现任务状态和任务执行日志。每个有色框表明每个组件都可以独立于其他组件存在，这取决于部署配置的类型。

##### 调度器操作
```
0. 从磁盘中加载可用的 DAG 定义（填充 DagBag）
调度器running：
  1. 调度器使用 DAG 定义来标识并且/或者初始化在元数据的 db 中的任何 DagRuns。
  2. 调度器检查与活动 DagRun 关联的 TaskInstance 的状态，解析 TaskInstance 之间的任何依赖，标识需要被执行的 TaskInstance，然后将它们添加至 worker 队列，将新排列的 TaskInstance 状态更新为数据库中的“排队”状态。
  3. 每个可用的 worker 从队列中取一个 TaskInstance，然后开始执行它，将此 TaskInstance 的数据库记录从“排队”更新为“运行”。
  4. 一旦一个 TaskInstance 完成运行，关联的 worker 就会报告到队列并更新数据库中的 TaskInstance 的状态（例如“完成”、“失败”等）。
  5. 调度器根据所有已完成的相关 TaskInstance 的状态更新所有活动 DagRuns 的状态（“运行”、“失败”、“完成”）。
  6. 重复步骤 1-5
 
```

#### 进程说明
```
airflow webserver -p 8090   # web管理页面， 如果添加-D 以后台进程启动
airflow scheduler           # 调度进程
airflow worker              # worker执行进程， -q 指定启用的quene
airflow flower              # 监控celery进程
```

#### airflow 的守护进程是如何一起工作的
1. 调度器 scheduler 会间隔性的去轮询元数据库（Metastore）已注册的 DAG（有向无环图，可理解为作业流）是否需要被执行。如果一个具体的 DAG 根据其调度计划需要被执行，scheduler 守护进程就会先在元数据库创建一个 DagRun 的实例，并触发 DAG 内部的具体 task（任务，可以这样理解：DAG 包含一个或多个task），触发其实并不是真正的去执行任务，而是推送 task 消息至消息队列（即 broker）中，每一个 task 消息都包含此 task 的 DAG ID，task ID，及具体需要被执行的函数。如果 task 是要执行 bash 脚本，那么 task 消息还会包含 bash 脚本的代码。
2. 用户可能在 webserver 上来控制 DAG，比如手动触发一个 DAG 去执行。当用户这样做的时候，一个DagRun 的实例将在元数据库被创建，scheduler 使同 #1 一样的方法去触发 DAG 中具体的 task 。
3. worker 守护进程将会监听消息队列，如果有消息就从消息队列中取出消息，当取出任务消息时，它会更新元数据中的 DagRun 实例的状态为正在运行，并尝试执行 DAG 中的 task，如果 DAG 执行成功，则更新任 DagRun 实例的状态为成功，否则更新状态为失败。

#### 常用CLI命令行接口
```
airflow test DAG_ID TASK_ID EXECUTION_DAT  # 测试任务调用是否可用
airflow list_dags   # 查看dags
airflow list_tasks DAG_ID  # 查看对应dag的task
airflow clear DAG_ID    # 移除dag_id元数据库中的taskinstance记录
airflow resetdb     # 重载数据库，删表在新创建
```

#### 部署说明
- 两台ubuntu 16.04 ,
- 192.168.0.10  (webserver,scheduler,worker,flower)
-  192.168.0.11  (worker)

  说明 两台服务器airflow.cfg配置要一样，对应的dags文件路径也要相同

##### Environmental dependence
```python
# python3.5 && pip3
sudo apt-get update
ln -s /usr/bin/python3 /usr/bin/python
sudo apt install python3-pip -y
pip3 install --upgrade pip
 
# Docker Install
sudo apt-get install -y apt-transport-https ca-certificates
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo "deb https://mirrors.tuna.tsinghua.edu.cn/docker/apt/repo ubuntu-trusty main" \
| sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update
apt-cache policy docker-engine
sudo apt-get install -y linux-image-extra-$(uname -r) linux-image-extra-virtual
sudo apt install -y docker.io
```
##### Install Airflow
```python
pip3 install apache-airflow
pip3 install 'apache-airflow[celery]'
pip3 install -U werkzeug Jinja2
pip3 install flask_bcrypt
 
####
修改对应的配置(dags路径)
executor = CeleryExecutor
demo_mode = False
sql_alchemy_conn = mysql://root:admin@192.168.0.10:3306/airflow
broker_url = pyamqp://airflow:airflow@192.168.0.10:5672/airflow
sql_alchemy_conn = mysql://root:admin@192.168.0.10:3306/airflow
 
```

##### Run Docker(Mysql && RabbitMQ)
```python
- docker mysql
    docker pull mysql
    docker run -it -d --name airflow-mysql -v /data/airflow_mysql_data/:/var/lib/mysql/ -p 3306:3306 -e MYSQL_ROOT_PASSWORD=admin --restart=always -d mysql
 
 
- docker rabbitmq
    docker pull rabbitmq
    docker run -it -d -p 5672:5672 -p 15672:15672 -v /data/airflow_rabbitmq_data:/var/lib/rabbitmq --restart=always --name airflow_rabbitmq rabbitmq
    #创建一个RabbitMQ用户
    rabbitmqctl add_user airflow airflow
    #创建一个RabbitMQ虚拟主机
    rabbitmqctl add_vhost airflow
    #将这个用户赋予admin的角色
    rabbitmqctl set_user_tags airflow admin
    #允许这个用户访问这个虚拟主机
    rabbitmqctl set_permissions -p airflow airflow ".*" ".*" ".*"
    # no usage
    rabbitmq-plugins enable rabbitmq_management
```
##### Supervisor管理进程
```python
sudo apt-get install supervisor
 
192.168.0.10配置文件如下：
[program:airflow_webserver]
directory=/home/ubuntu/airflow/
command=/usr/local/bin/airflow webserver
autostart=true
autorestart=true
redirect_stderr=true
environment=AIRFLOW_HOME="/home/ubuntu/airflow",HOME="/home/ubuntu/"
user=ubuntu
 
stdout_logfile=/var/log/supervisor/airflow_webserver.out
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
stderr_logfile=/var/log/supervisor/airflow_webserver.err
stderr_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
[program:airflow_scheduler]
directory=/home/ubuntu/airflow/
command=/usr/local/bin/airflow scheduler
autostart=true
autorestart=true
redirect_stderr=true
environment=AIRFLOW_HOME="/home/ubuntu/airflow",HOME="/home/ubuntu/"
user=ubuntu
 
stdout_logfile=/var/log/supervisor/airflow_scheduler.out
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
stderr_logfile=/var/log/supervisor/airflow_scheduler.err
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
 
[program:airflow_worker]
directory=/home/ubuntu/airflow/
command=/usr/local/bin/airflow worker
autostart=true
autorestart=true
redirect_stderr=true
environment=AIRFLOW_HOME="/home/ubuntu/airflow",HOME="/home/ubuntu/"
user=ubuntu
 
stdout_logfile=/var/log/supervisor/airflow_worker.out
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
stderr_logfile=/var/log/supervisor/airflow_worker.err
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
 
[program:airflow_flower]
directory=/home/ubuntu/airflow/
command=/usr/local/bin/airflow flower
autostart=true
autorestart=true
redirect_stderr=true
environment=AIRFLOW_HOME="/home/ubuntu/airflow",HOME="/home/ubuntu/"
user=ubuntu
 
stdout_logfile=/var/log/supervisor/airflow_flower.out
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
stderr_logfile=/var/log/supervisor/airflow_flower.err
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
 
 
192.168.0.11配置文件如下：
[program:airflow_worker]
directory=/home/ubuntu/airflow/
command=/usr/local/bin/airflow worker
autostart=true
autorestart=true
redirect_stderr=true
environment=AIRFLOW_HOME="/home/ubuntu/airflow",HOME="/home/ubuntu/"
user=ubuntu
 
stdout_logfile=/var/log/supervisor/airflow_worker.out
stdout_logfile_maxbytes=50MB
stdout_logfile_backups=10
 
stderr_logfile=/var/log/supervisor/airflow_worker.err
stderr_logfile_maxbytes=50MB
stderr_logfile_backups=10
 
```

#### 遇到的问题
1. locale.Error: unsupported locale setting
```python
    https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting
    export LC_ALL="en_US.UTF-8"
    export LC_CTYPE="en_US.UTF-8"
    sudo dpkg-reconfigure locales
```
2. ImportError: No module named 'MySQLdb'
```python
    sudo apt-get install libmysqlclient-dev -y
    sudo pip3 install mysqlclient
```
3. airflow.exceptions.AirflowException: No module named 'flask_bcrypt'
```python
    pip3 install flask_bcrypt
```
4. UnicodeDecodeError: 'ascii' codec can't decode byte 0xe8 in position 3199: ordinal not in range(128)
```python
字符集问题
/etc/default/locale 
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh"
LC_ALL="zh_CN.UTF-8"
```

#### 参考链接
https://medium.com/@dustinstansbury/understanding-apache-airflows-key-concepts-a96efed52b1a
https://blog.csdn.net/youzi_yun/article/details/90141362
























