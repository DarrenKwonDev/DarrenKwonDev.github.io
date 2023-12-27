+++
title = 'kernel study 02: 프로세스 관련 정보'
date = 2023-12-27T07:23:52+09:00
math = true
toc = true
bold = true
draft = false
tags = ["linux", "system_engineering", "korean", "kernel"]
+++

## process 관련 정보

### top

![test](./top.png)

단순한 사용 방법은 [리눅스 top 정리 및 설명](https://zzsza.github.io/development/2018/07/18/linux-top/)을 참고하는 것이 좋습니다.

#### top의 개선판들

-   [htop](https://github.com/hishamhm/htop), [vtop](https://github.com/MrRio/vtop), [gtop](https://github.com/aksakalli/gtop), [gotop](https://github.com/cjbassi/gotop)

### VIRT, RES, SHR

### memory overcommit

### process status

### niceness[^1]

[^1]: 참고 자료 : https://www.youtube.com/watch?v=GsF8R6DBxSg&ab_channel=HusseinNasser

**renice**

Alters the scheduling priority/niceness of one or more running processes.  
Niceness values range from -20 (most favorable to the process) to 19 (least favorable to the process).  
More information: <https://manned.org/renice>.

```bash
# Change priority of a running process:
renice -n niceness_value -p pid

# Change priority of all processes owned by a user:
renice -n niceness_value -u user

# Change priority of all processes that belong to a process group:
renice -n niceness_value --pgrp process_group
```
