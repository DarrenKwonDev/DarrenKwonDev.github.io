+++
title = 'kernel study 03: tracking how load average is calculated'
date = 2023-12-28T04:22:22+09:00
math = true
toc = true
bold = true
draft = false
tags = ["linux", "system_engineering", "korean", "kernel", "scalability"]
+++

{{< box info >}}

Linux kernel v6.7-rc7 기준으로 작성되었습니다.

{{< /box >}}

## Load average

### definition

<blockquote>
/proc/loadavg

<u>The first three fields in this file are load average figures giving the number of jobs in the run queue (state R) or waiting for disk I/O (state D) averaged over 1, 5, and 15 minutes.</u> They are the same as the load average numbers given by uptime(1) and other programs. The fourth field consists of two numbers separated by a slash (/). The first of these is the number of currently runnable kernel scheduling entities (processes, threads). The value after the slash is the number of kernel scheduling entities that currently exist on the system. The fifth field is the PID of the process that was most recently created on the system.

</blockquote>

즉, load average는 1분, 5분, 15분 동안 process status가 R(run queue에 적재) 또는 D(waiting for disk I/O)인 job(task)의 수이다.[^1] 그러나 단순히 task의 수 만으로는 시스템 부하를 판단하기는 어렵다. 현재 가동 중인 cpu 코어의 갯수와 고려하여 다른 측면까지도 고려해야 한다. 본 포스트에서는 시스템 부하를 판단하는 방법에 대해서는 다루지 않고, load average가 어떻게 계산되는지에 대해서만 알아보자.

[^1]: process status에 대해서는 [이전 포스트](https://darrenkwondev.github.io/posts/2023-12-27_kernel_study_02.md/#process-status)를 참고하자.

load average는 다음과 같이 다양한 방법으로 조회할 수 있다.

```bash
cat /proc/loadavg
uptime
top -b | head -1
```

## tracking how load average is calculated

### strace

```bash
strace uptime
```

출력된 덤프 하단을 보면 다음과 같은 점들을 확인할 수 있습니다.

```text {hl_lines=[11]}
# uptime 실행
execve("/usr/bin/uptime", ["uptime"], 0xfffff2486300 /* 24 vars */) = 0

# 메모리 할당 및 매핑
brk(NULL)                               = 0xaaaaf84c2000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xffffaf954000

... 생략

# /proc/loadavg를 RDONLY로 open하여 fd 4에 할당
openat(AT_FDCWD, "/proc/loadavg", O_RDONLY) = 4

# fd 4의 시작으로 SEEK_SET. 처음부터 파일을 읽을 것을 위함.
lseek(4, 0, SEEK_SET)                   = 0
read(4, "0.00 0.00 0.00 3/190 26203\n", 8191) = 27

... 생략
+++ exited with 0 +++
```

결국 uptime은 /proc/loadavg를 읽어오는 것에 불과합니다.

top에 출력되는 load average도 마찬가지입니다.  
`strace -e openat top -b -n 1`를 실행하여 덤프 하단을 살펴보면 /proc/loadavg를 읽어오는 것을 확인할 수 있습니다.

그렇다면 구체적으로 /proc/loadavg이 어떻게 계산되는지 kernel을 살펴보겠습니다.

### loadavg_proc_show

[/fs/proc/loadavg.c](https://elixir.bootlin.com/linux/v6.7-rc7/source/fs/proc/loadavg.c) 파일의 코드에는 다음과 같은 내용이 존재합니다. 짧은 코드니 header를 제외한 전부를 첨부해보겠습니다.

```c
// /fs/proc/loadavg.c

static int loadavg_proc_show(struct seq_file *m, void *v)
{
	unsigned long avnrun[3];

	get_avenrun(avnrun, FIXED_1/200, 0);

	seq_printf(m, "%lu.%02lu %lu.%02lu %lu.%02lu %u/%d %d\n",
		LOAD_INT(avnrun[0]), LOAD_FRAC(avnrun[0]),
		LOAD_INT(avnrun[1]), LOAD_FRAC(avnrun[1]),
		LOAD_INT(avnrun[2]), LOAD_FRAC(avnrun[2]),
		nr_running(), nr_threads,
		idr_get_cursor(&task_active_pid_ns(current)->idr) - 1);
	return 0;
}

static int __init proc_loadavg_init(void)
{
	struct proc_dir_entry *pde;

	pde = proc_create_single("loadavg", 0, NULL, loadavg_proc_show);
	pde_make_permanent(pde);
	return 0;
}
fs_initcall(proc_loadavg_init);
```

`cat /proc/loadavg`의 결과를 고려해보면 0.01 0.01 0.00 1/188 26290 와 같은 꼴로 출력되었는데 loadavg_proc_show 함수의 seq_printf 호출에서 이 형식을 확인해볼 수 있습니다.

그러나 이는 수식이 아니고 출력 형식입니다. 실제 계산하는 과정을 확인하기 위해 get_avenrun 함수를 살펴보겠습니다.

### exponentially decaying average of (R + D)

구체적인 계산식은 kernel/sched/loadavg.c의 [get_avenrun](https://elixir.bootlin.com/linux/v6.7-rc7/source/kernel/sched/loadavg.c#L71) 함수에서 확인할 수 있습니다.

해당 파일에 접근하면, 상단의 주석에서 load average의 계산 결과에 대한 설명을 참고할 수 있습니다.

```text
The global load average is an exponentially decaying average of nr_running +
nr_uninterruptible.

Once every LOAD_FREQ:
    nr_active = 0;
    for_each_possible_cpu(cpu)
    nr_active += cpu_of(cpu)->nr_running + cpu_of(cpu)->nr_uninterruptible;
    avenrun[n] = avenrun[0] * exp_n + nr_active * (1 - exp_n)
```

위의 psuedo code를 보면 load average가 우리가 알고 있는 R, D 상태의 task 수를 적당히 가공한 수치임을 확인할 수 있습니다.

새로운 avenrun(avenrun[n])을 계산하기 위해 이전 시점의 avenrun(avenrun[0])을 활용하며, 이전 시점 avenrun[0]는 exp_n에 의해 지수적으로 감소하며[^2], 새로운 R, D 상태의 프로세스의 수를 의미하는 nr_active는 (1 - exp_n)에 의해 지수적으로 증가합니다.

[^2]: decay는 일반적으로 최근에 더욱 가중치를 주고 싶어 과거의 데이터의 가중치를 감소시키는 일종의 수학적 trick입니다.

지수적 감소를 자연상수 `e`를 이용하여 표현하면 다음과 같습니다.

{{< box important >}}
실제로 자연 상수를 통해 지수적 감소를 표현하지는 않습니다. 정확한 값에 대해서는 후술됩니다!
{{< /box >}}

$$
active \space process = R + D \\\
\space \\\
avenrun[n] = avenrun[0] \times e^{-\lambda t} + (R + D) \times (1 - e^{-\lambda t})
$$

결국 단순히 R, D 상태의 task 수만으로 load average를 계산하는 것이 아니라  
<u>load average is an exponentially decaying average of nr_running + nr_uninterruptible.</u> 라는 사실을 확인할 수 있습니다.

구체적으로 계산되는 코드를 찾기 위해 get_avenrun 함수를 살펴보겠습니다.

```c
/**
 * get_avenrun - get the load average array
 * @loads:	pointer to dest load array
 * @offset:	offset to add
 * @shift:	shift count to shift the result left
 *
 * These values are estimates at best, so no need for locking.
 */
void get_avenrun(unsigned long *loads, unsigned long offset, int shift)
{
	loads[0] = (avenrun[0] + offset) << shift;
	loads[1] = (avenrun[1] + offset) << shift;
	loads[2] = (avenrun[2] + offset) << shift;
}
```

loads 배열에 avenrun 배열의 원소값을 조작해서 계산하는 것을 확인할 수 있습니다.  
이제는 avenrun 배열이 초기화 되거나 활용되는 부분을 찾아야 합니다.  
찾아보면, 동일 파일인 /kernel/sched/loadavg.c에서 avenrun 가 선언되었으며 calc_global_load 에서 계산되는 것을 확인할 수 있습니다.

```c
// /kernel/sched/loadavg.c

/* Variables and functions for calc_load */
atomic_long_t calc_load_tasks;
unsigned long calc_load_update;
unsigned long avenrun[3]; // 여기에서 초기화 됩니다.
```

```c
// /kernel/sched/loadavg.c

/*
 * calc_load - update the avenrun load estimates 10 ticks after the
 * CPUs have updated calc_load_tasks.
 *
 * Called from the global timer code.
 */
void calc_global_load(void)
{
	unsigned long sample_window;
	long active, delta;

	sample_window = READ_ONCE(calc_load_update);
	if (time_before(jiffies, sample_window + 10))
		return;

	/*
	 * Fold the 'old' NO_HZ-delta to include all NO_HZ CPUs.
	 */
	delta = calc_load_nohz_read();
	if (delta)
		atomic_long_add(delta, &calc_load_tasks);

	active = atomic_long_read(&calc_load_tasks);
	active = active > 0 ? active * FIXED_1 : 0;

	avenrun[0] = calc_load(avenrun[0], EXP_1, active);
	avenrun[1] = calc_load(avenrun[1], EXP_5, active);
	avenrun[2] = calc_load(avenrun[2], EXP_15, active);

	WRITE_ONCE(calc_load_update, sample_window + LOAD_FREQ);

	/*
	 * In case we went to NO_HZ for multiple LOAD_FREQ intervals
	 * catch up in bulk.
	 */
	calc_global_nohz();
}
```

여기서 avenrun은 calc_load 함수에 의해 여러 값과 함께 계산이 되는 것으로 보입니다.

매크로 값은 아래와 같이 확인됩니다. 지수적 감소를 위한 exp_n 값으로 보입니다.

```c
// /include/linux/sched/loadavg.h
#define EXP_1		1884		/* 1/exp(5sec/1min) as fixed-point */
#define EXP_5		2014		/* 1/exp(5sec/5min) */
#define EXP_15		2037		/* 1/exp(5sec/15min) */
```

active의 경우에는 앞서 살펴본 active process (R + D)의 갯수를 의미하는 것으로 보입니다.
`active = atomic_long_read(&calc_load_tasks);` 와 같은 함수의 형식으로 미루어보아, atomic_long_read는 atomic read를 지원하기 위한 함수로 보이며 calc_load_tasks가 핵심적인 값으로 보입니다.

### conclusion : calc_load

최종적으로는 다음과 같이 계산되는 것을 확인할 수 있습니다.  
주석에서 설명하던 exponential decay average를 계산하는 것을 확인할 수 있습니다.

```c
// include/linux/sched/loadavg.h

calc_load(unsigned long load, unsigned long exp, unsigned long active)
{
	unsigned long newload;

	newload = load * exp + active * (FIXED_1 - exp);
	if (active >= load)
		newload += FIXED_1 - 1;

	return newload / FIXED_1;
}
```

-   load: 이전의 부하 평균 값입니다.
-   exp: 지수적 감쇠를 위한 계수로, 과거 데이터의 현재 값에 대한 영향을 조절합니다. (EXP_1, EXP_5, EXP_15)
-   active: 현재 활성화된 (실행 중 또는 실행 대기 중인) 프로세스의 수입니다.

`newload = load * exp + active * (FIXED_1 - exp);`  
과거 부하 평균(load)에 지수적 감쇠 계수(exp)를 곱하고, 현재 활성화된 프로세스 수(active)에 1 - exp를 곱하여 두 값을 합산합니다. 여기서 FIXED_1은 고정 소수점 연산을 위한 상수입니다.

`if (active >= load) newload += FIXED_1 - 1;`  
만약 현재 활성화된 프로세스 수가 이전 부하 평균보다 크거나 같은 경우, newload에 추가적인 값(FIXED_1-1)을 더합니다. 이는 부하가 증가하는 상황에서 부하 평균을 보다 빠르게 반응하도록 하는 조정입니다.

`return newload / FIXED_1;`  
마지막으로, 계산된 새로운 부하 평균(newload)을 FIXED_1로 나누어 반환합니다. 이는 고정 소수점 연산을 실제 부동 소수점 값으로 변환하는 단계입니다.
