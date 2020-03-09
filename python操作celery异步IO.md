
**环境说明**：
- python3
- celery == 4.3.0
- eventlet == 0.25.1
- celery的中间人broker基于redis, redis server安装在另一台linux机器上

**步骤**
1. 检查redis server 是否绑定了本地端口(bind)，如果绑定了，那么注释。另外给redis加一个密码(requirepass)，不然外面IP访问不了。
1. 新建两个python文件，分别为task.py和main.py
```python

-----------------------下面是task.py-----------------------------------------------


from celery import Celery
import time


# format:  redis://:password@hostname:port/db_number
REDIS_ADDRESS = 'redis://:111111@10.140.14.105:6379/0'


celery = Celery("tasks", broker=REDIS_ADDRESS, backend=REDIS_ADDRESS)

# 延时函数用来测试celery
@celery.task
def send_mail():
    print('starting.....')
    time.sleep(5)
    print('end......')


-----------------------下面是main.py-----------------------------------------------
from tasks import send_mail

if __name__ == '__main__':
    send_mail.delay()

```

3. 打开cmd，进入到task.py路径

```
# 由于celery版本比较新，在windows上抛的时候要加上 --pool=eventlet，linux貌似不需要
# 貌似有些时候eventlet不好用，改为solo即可

D:\learn_tmp\celery_demo>celery -A tasks.celery --pool=eventlet worker --loglevel=info

 -------------- celery@5CG70740SC v4.3.0 (rhubarb)
---- **** -----
--- * ***  * -- Windows-7-6.1.7601-SP1 2019-11-11 15:34:32
-- * - **** ---
- ** ---------- [config]
- ** ---------- .> app:         tasks:0x4c4da58
- ** ---------- .> transport:   redis://:**@10.140.14.105:6379/0
- ** ---------- .> results:     redis://:**@10.140.14.105:6379/0
- *** --- * --- .> concurrency: 4 (eventlet)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** -----
 -------------- [queues]
                .> celery           exchange=celery(direct) key=celery


[tasks]
  . tasks.send_mail

[2019-11-11 15:34:32,905: INFO/MainProcess] Connected to redis://:**@10.140.14.105:6379/0
[2019-11-11 15:34:32,918: INFO/MainProcess] mingle: searching for neighbors
[2019-11-11 15:34:33,975: INFO/MainProcess] mingle: all alone
[2019-11-11 15:34:34,003: INFO/MainProcess] celery@5CG70740SC ready.
[2019-11-11 15:34:34,016: INFO/MainProcess] pidbox: Connected to redis://:**@10.140.14.105:6379/0.
[2019-11-11 15:34:42,106: INFO/MainProcess] Received task: tasks.send_mail[531ed710-62a9-4e1e-bd8f-694fc3d34219]
[2019-11-11 15:34:42,108: WARNING/MainProcess] starting.....
[2019-11-11 15:34:47,108: WARNING/MainProcess] end......
[2019-11-11 15:34:47,119: INFO/MainProcess] Task tasks.send_mail[531ed710-62a9-4e1e-bd8f-694fc3d34219] succeeded in 5.0230000000447035s: None




# 只要执行下main.py这个文件，celery就会进行工作
[2019-11-11 15:34:42,106: INFO/MainProcess] Received task: tasks.send_mail[531ed710-62a9-4e1e-bd8f-694fc3d34219]
[2019-11-11 15:34:42,108: WARNING/MainProcess] starting.....
[2019-11-11 15:34:47,108: WARNING/MainProcess] end......
[2019-11-11 15:34:47,119: INFO/MainProcess] Task tasks.send_mail[531ed710-62a9-4e1e-bd8f-694fc3d34219] succeeded in 5.0230000000447035s: None


```

