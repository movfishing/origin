---
title: CloudComputingLab1
date: 2023-03-29 19:31:03
tags: experiments
categories: CloudComputing
---

# “Super-fast” Sudoku Solving

--- an inelegant solve

### 引言

本实验参考 ~~（照抄）~~ 了/src/Sudoku中的dancing_link算法以及课程Demo示例中线程池的思路 ~~，基本就是缝合而成~~ ，由于第一次写并行程序，可能有很多冗余部分，不太优雅，请见谅。

--- 3.29 ---

暂时只支持百万的总数独谜题个数，后续完善将同步更新。

--- 4.11 ---

完全改善！可以支持任意数量的谜题啦。

### 总体框架

![](/img/CloudComputingLab1/Arch.png)

### BasicVersion

#### 代码片段解读

* JobQueue工作队列

```c
typedef struct {
  char **jobs;
  int **ans;
  int *oplist;//arg to ctrl the order of output
  int head;//first un-solved puz
  int tail;//the last puzzle in queue
  int outCount;//next puzzle to output
  pthread_mutex_t mutex;//maintain var in struct
  pthread_cond_t inputCond;//ctrl worker-thread
  pthread_cond_t outputCond;//ctrl output-thread
  int queueCapacity;//useless, but will be used in next version(i guess...)
  int num_of_job_todo;
  int num_of_job_output;
  int num_of_total_jobs;//work like their names
} JobQueue;
```

* 读取输入

只需要读一个文件名的输入，所以无需单独开线程读输入。

```c
  char* file_name;
  size_t stdin_len = 0;
  ssize_t read;
  FILE* fp;
  int errNum = 0; 
  if((read = getline(&file_name,&stdin_len,stdin)) != -1){
    if(file_name[strlen(file_name)-1] == '\n') 
      file_name[strlen(file_name)-1] = '\0';
    if ((fp =fopen(file_name, "r")) == NULL){
      errNum = errno;
      printf("open fail errno = %d reason = %s \n", errNum, strerror(errNum));//错误检测，debug用
    }
  }
```

* 加入任务

在此版本中，由主线程执行该操作，将刚读取的数独题加入到jobs中。

```c
void enqueueAJob(char *job)
{
  pthread_mutex_lock(&job_queue.mutex);
  job_queue.num_of_job_todo++;
  memcpy(job_queue.jobs[job_queue.tail],job,N*sizeof(job[0]));
  //printf("enqueue success!\n");
  job_queue.tail++;
  job_queue.num_of_total_jobs++;
  pthread_cond_broadcast(&job_queue.inputCond);
  pthread_mutex_unlock(&job_queue.mutex);
}
```

* 输出任务

这里涉及到了一个问题：如何保持顺序输出？这里我采用了`job_queue.oplist`,`job_queue.outCount`,`job_queue.num_of_job_output`三个参数来维持，由于线程执行顺序不确定，但是每个线程所搭载的工作都是明确的，所以根据`job_queue.outCount`对每个输出编号，根据对应编号在`job_queue.oplist`中对应位置的状态来判断下一个输出是否就绪。`job_queue.oplist`在解题函数中会进行更改。

```c
void* dequeueAJob(void *arg)
{
  while(1){
    pthread_mutex_lock(&job_queue.mutex);
    while(job_queue.num_of_job_output<=0 || job_queue.oplist[job_queue.outCount] != 1){
      pthread_cond_wait(&job_queue.outputCond,&job_queue.mutex);
    }
    job_queue.num_of_job_output--;
    copy_i_to_s(job_queue.ans[job_queue.outCount],job_queue.jobs[job_queue.outCount]);
    printf("%s\n",job_queue.jobs[job_queue.outCount]);
    job_queue.outCount++;
    pthread_mutex_unlock(&job_queue.mutex);
  }
}
```

* 工作线程任务

对于每个任务，根据顺序都有一个编号job_id，所以线程获取到自己本次执行的job_id后，就可以释放锁去执行解算法了。这里也对`job_queue.oplist`进行了操作，就是完成工作后，对应位置置1，表示解完毕，可以输出。

```c
void* processJobsLongLiveThread(void *arg) {
  while(1)
  {
    pthread_mutex_lock(&job_queue.mutex);
    while(job_queue.num_of_job_todo == 0){
      pthread_cond_wait(&job_queue.inputCond,&job_queue.mutex);
    }
    int job_id;
    job_id = job_queue.head;
    job_queue.head++;
    job_queue.num_of_job_todo--;
    pthread_cond_broadcast(&job_queue.inputCond);
    pthread_mutex_unlock(&job_queue.mutex);

    input(job_queue.jobs[job_id],job_queue.ans[job_id]);
    solve(job_queue.ans[job_id]);

    pthread_mutex_lock(&job_queue.mutex);
    job_queue.oplist[job_id]=1;
    job_queue.num_of_job_output++;
    pthread_cond_broadcast(&job_queue.outputCond);
    pthread_mutex_unlock(&job_queue.mutex);
  }
}
```

* 等待结束函数

线程执行需要时间，所以在主函数最后还要执行一个循环执行的函数进行定时检测，来确保所有输出已完毕，可以结束程序。

```c
void waitForAllJobsDone()
{
    while(1)
  {
    usleep(1000);//Check per 1 ms
    pthread_mutex_lock(&job_queue.mutex);
    if(job_queue.num_of_total_jobs==job_queue.outCount && job_queue.num_of_total_jobs>0)//All jobs done
    {  
      pthread_mutex_unlock(&job_queue.mutex);
      break;
    }
    if(job_queue.num_of_job_todo>0)
      pthread_cond_broadcast(&job_queue.inputCond);
    if(job_queue.num_of_job_output>0)
      pthread_cond_broadcast(&job_queue.outputCond);//保证不死锁
    pthread_mutex_unlock(&job_queue.mutex);
  }
}
```

这里其实还有一个问题，就是解题结果如何输出，因为四个算法返回值类型都是bool，但是他们都对一个全局变量`int board[N]`进行了修改，所以只需要输出这一个全局变量即可。又因为其他三种算法都涉及了许多全局变量，结构也比较复杂难以修改，导致线程之间的切换十分繁琐，而`dancing_link`只有传入solve的一个参数，并对该参数进行了修改，没有引用其余的全局变量，所以我修改了`dancing_link`的参数，使每个任务传入的board为jobs数组对应的部分，从而避免了多线程导致全局变量无法维护的问题，线程可以并行解题。

参考修改:

```c
bool solve_sudoku_dancing_links(int* b)
{
  Dance d(b);
  return d.solve();
}
```

#### 运行结果对比

与main.cc文件的执行结果对比，约有100%的优化(执行时间缩短50%)，有显著效果。(忘记截图，版本已经覆盖，摆烂)

### Advanced Version 1.0

其实这个版本的区别就是需要创建新线程去读取输入，整体框架并无区别。

我创建了两个新线程，一个用于从stdin读取输入的文件名，一个根据读取的文件名去读取对应文件的内容并添加任务。在BasicVersion中这两件事都是由主线程负责的。

#### 代码片段解读

* 新的全局变量与锁

```c
char **files;//files' names
int total_file_num = 0;
int cur_file = 0;

pthread_mutex_t file_mutex;
pthread_cond_t fileCond;
```

* 读取文件名

```c
void* get_files(void *arg){
    char* file_name;
    file_name=(char*)calloc(MAX_FILE_NAME_LENGTH,sizeof(char));
      while(fgets(file_name,MAX_FILE_NAME_LENGTH,stdin)!=NULL){
        if(file_name[0] == '\0' || file_name[0] == '\n')
          return (void*)0;
      if(file_name[strlen(file_name)-1] == '\n')
        file_name[strlen(file_name)-1] = '\0';
      pthread_mutex_lock(&file_mutex);
      memcpy(files[total_file_num],file_name,strlen(file_name)*sizeof(char));
      total_file_num++;
      free(file_name);
      file_name=(char*)malloc(MAX_FILE_NAME_LENGTH*sizeof(char));
      pthread_cond_broadcast(&fileCond);
      pthread_mutex_unlock(&file_mutex);
      }
}
```

* 读取文件并加入任务

```c
void* enqth_job(void* arg){
  while(1){
    FILE* fp;
    int errNum = 0; 
    pthread_mutex_lock(&file_mutex);
    while(cur_file == total_file_num){
      pthread_cond_wait(&fileCond,&file_mutex);
    }
    if ((fp =fopen(files[cur_file], "r")) == NULL){
        errNum = errno;
        printf("open fail errno = %d reason = %s \n", errNum, strerror(errNum));
    }
    cur_file++;
    pthread_mutex_unlock(&file_mutex);

    while (fgets(puzzle, sizeof puzzle, fp) != NULL) {
      if (strlen(puzzle) >= N) {
        enqueueAJob(puzzle);
      }
    }
    fclose(fp);
  }
}
```

都是比较简单的函数，随便看看。

#### 运行结果

运行时间似乎不太稳定，第一次900多ms，第二次700多ms，第三次300多ms,~~它好像在自己进化~~

![](/img/CloudComputingLab1/execRes.png)

### 主程序代码(Advanced Version 1.0)

sudoku.h与算法代码请移步[果果老师仓库](https://github.com/1989chenguo/CloudComputingLabs)查看(本实验仓库应要求设为了Private)

```c
#include <assert.h>
#include <stdint.h>
#include <stdio.h>
#include <string.h>
#include <sys/time.h>
#include <unistd.h>
#include <sys/types.h>
#include <pthread.h>
#include <stdlib.h>
#include <errno.h>

#include "sudoku.h"

#define MAX_PUZZLE_NUM 1000000
#define MAX_FILE_NUM 100
#define MAX_FILE_NAME_LENGTH 100

typedef struct {
  char **jobs;
  int **ans;
  int *oplist;
  int head;//first un-solved puz
  int tail;//the last puzzle in queue
  int outCount;//next puzzle to output
  pthread_mutex_t mutex;
  pthread_cond_t inputCond;
  pthread_cond_t outputCond;
  int queueCapacity;
  int num_of_job_todo;
  int num_of_job_output;
  int num_of_total_jobs;
} JobQueue;

int64_t now()
{
  struct timeval tv;
  gettimeofday(&tv, NULL);
  return tv.tv_sec * 1000000 + tv.tv_usec;
}

JobQueue job_queue;

char puzzle[128];
bool (*solve)(int*) = solve_sudoku_dancing_links;
char **files;
int total_file_num = 0;
int cur_file = 0;

pthread_mutex_t file_mutex;
pthread_cond_t fileCond;

void initJobQueue()
{
  job_queue.jobs=(char**)malloc(MAX_PUZZLE_NUM*sizeof(job_queue.jobs[0]));
  job_queue.ans=(int**)malloc(MAX_PUZZLE_NUM*sizeof(job_queue.ans[0]));
  job_queue.oplist=(int*)malloc(MAX_PUZZLE_NUM*sizeof(int));
  for(int i=0;i<MAX_PUZZLE_NUM;i++){
    job_queue.jobs[i]=(char*)malloc(N*sizeof(job_queue.jobs[0][0]));
    job_queue.ans[i]=(int*)malloc(N*sizeof(job_queue.ans[0][0]));
    job_queue.oplist[i]=0;
  }
  job_queue.head=0;
  job_queue.tail=0;
  job_queue.outCount=0;
  job_queue.queueCapacity=MAX_PUZZLE_NUM; 
  job_queue.num_of_job_todo=0;
  job_queue.num_of_job_output=0;
  job_queue.num_of_total_jobs=0;
  pthread_mutex_init(&job_queue.mutex,NULL);
  pthread_cond_init(&job_queue.inputCond,NULL);
  pthread_cond_init(&job_queue.outputCond,NULL);
}

void* dequeueAJob(void *arg)
{
  while(1){
    pthread_mutex_lock(&job_queue.mutex);
    while(job_queue.num_of_job_output<=0 || job_queue.oplist[job_queue.outCount] != 1){
      pthread_cond_wait(&job_queue.outputCond,&job_queue.mutex);
    }
    job_queue.num_of_job_output--;
    copy_i_to_s(job_queue.ans[job_queue.outCount],job_queue.jobs[job_queue.outCount]);
    printf("%s\n",job_queue.jobs[job_queue.outCount]);
    job_queue.outCount++;
    pthread_mutex_unlock(&job_queue.mutex);
  }
}

void copy_i_to_s(int* in,char* out){
  for (int cell = 0; cell < N; ++cell) {
    out[cell] = in[cell] + '0';
    assert('0' <= out[cell] && out[cell] <= '9');
  }
}

void enqueueAJob(char *job)
{
  pthread_mutex_lock(&job_queue.mutex);
  job_queue.num_of_job_todo++;
  memcpy(job_queue.jobs[job_queue.tail],job,N*sizeof(job[0]));
  job_queue.tail++;
  job_queue.num_of_total_jobs++;
  pthread_cond_broadcast(&job_queue.inputCond);
  pthread_mutex_unlock(&job_queue.mutex);
}

void* processJobsLongLiveThread(void *arg) {
  while(1)
  {
    pthread_mutex_lock(&job_queue.mutex);
    while(job_queue.num_of_job_todo == 0){
      pthread_cond_wait(&job_queue.inputCond,&job_queue.mutex);
    }
    int job_id;
    job_id = job_queue.head;
    job_queue.head++;
    job_queue.num_of_job_todo--;
    pthread_cond_broadcast(&job_queue.inputCond);
    pthread_mutex_unlock(&job_queue.mutex);

    input(job_queue.jobs[job_id],job_queue.ans[job_id]);
    solve(job_queue.ans[job_id]);  

    pthread_mutex_lock(&job_queue.mutex);
    job_queue.oplist[job_id]=1;
    job_queue.num_of_job_output++;
    pthread_cond_broadcast(&job_queue.outputCond);
    pthread_mutex_unlock(&job_queue.mutex);
  }
}

void waitForAllJobsDone()
{
    while(1)
  {
    usleep(1000);//Check per 1 ms
    pthread_mutex_lock(&job_queue.mutex);
    if(job_queue.num_of_total_jobs==job_queue.outCount && job_queue.num_of_total_jobs>0)//All jobs done
    {  
      pthread_mutex_unlock(&job_queue.mutex);
      break;
    }
    if(job_queue.num_of_job_todo>0)
      pthread_cond_broadcast(&job_queue.inputCond);
    if(job_queue.num_of_job_output>0)
      pthread_cond_broadcast(&job_queue.outputCond);
    pthread_mutex_unlock(&job_queue.mutex);
  }
}

void* get_files(void *arg){
    char* file_name;
    file_name=(char*)calloc(MAX_FILE_NAME_LENGTH,sizeof(char));
      while(fgets(file_name,MAX_FILE_NAME_LENGTH,stdin)!=NULL){
        if(file_name[0] == '\0' || file_name[0] == '\n')
          return (void*)0;
      if(file_name[strlen(file_name)-1] == '\n')
        file_name[strlen(file_name)-1] = '\0';
      pthread_mutex_lock(&file_mutex);
      memcpy(files[total_file_num],file_name,strlen(file_name)*sizeof(char));
      total_file_num++;
      free(file_name);
      file_name=(char*)malloc(MAX_FILE_NAME_LENGTH*sizeof(char));
      pthread_cond_broadcast(&fileCond);
      pthread_mutex_unlock(&file_mutex);
      }
}

void* enqth_job(void* arg){
  while(1){
    FILE* fp;
    int errNum = 0; 
    pthread_mutex_lock(&file_mutex);
    while(cur_file == total_file_num){
      pthread_cond_wait(&fileCond,&file_mutex);
    }
    if ((fp =fopen(files[cur_file], "r")) == NULL){
        errNum = errno;
        printf("open fail errno = %d reason = %s \n", errNum, strerror(errNum));
    }
    cur_file++;
    pthread_mutex_unlock(&file_mutex);

    while (fgets(puzzle, sizeof puzzle, fp) != NULL) {
      if (strlen(puzzle) >= N) {
        enqueueAJob(puzzle);
      }
    }
    fclose(fp);
  }
}

int main()
{
  pthread_mutex_init(&file_mutex,NULL);
  pthread_cond_init(&fileCond,NULL);

  files=(char**)malloc(MAX_FILE_NUM*sizeof(files[0]));
  for(int f = 0;f<MAX_FILE_NUM;f++){
    files[f]=(char*)malloc(MAX_FILE_NAME_LENGTH*sizeof(char));
  }

  int enableCPUNum_ = sysconf(_SC_NPROCESSORS_ONLN);
  int numOfWorkerThread=enableCPUNum_;
  initJobQueue();

  pthread_t th[numOfWorkerThread];
  for(int i=0;i<numOfWorkerThread;i++)
  {
    if(pthread_create(&th[i], NULL, processJobsLongLiveThread,NULL)!=0)
    {
      perror("pthread_create failed");
      exit(1);
    }
  }

  pthread_t out_th;
  if(pthread_create(&out_th, NULL, dequeueAJob,NULL)!=0)
    {
      perror("pthread_create failed");
      exit(1);
    }

  pthread_t enq_th;
  if(pthread_create(&enq_th, NULL, enqth_job,NULL)!=0)
    {
      perror("pthread_create failed");
      exit(1);
    }

    pthread_t in_th;
  if(pthread_create(&in_th, NULL, get_files,NULL)!=0)
    {
      perror("pthread_create failed");
      exit(1);
    }

  waitForAllJobsDone();
  return 0;
}
```

### AdvancedVersion2.0

其实很简单，但是却拖了很久）

相较于1.0版本，成功支持任意数量的谜题数！(但是暂且设置成为了仅支持1000000个文件输入，总计能支持1000000*INF的谜题数)

代码改动也仅仅只添加了一个条件变量：maxPuzzleCond，当当前已完成与待完成的工作量加起来到达MAX_PUZZLE_NUM时，就会先等待所有任务完成，再初始化工作队列的相关成员，从头开始。

部分更改的代码:

```c++
void enqueueAJob(char *job)
{
  pthread_mutex_lock(&job_queue.mutex);
  while(job_queue.num_of_total_jobs == MAX_PUZZLE_NUM)
    pthread_cond_wait(&maxPuzzleCond,&job_queue.mutex);  //new code
  job_queue.num_of_job_todo++;
  memcpy(job_queue.jobs[job_queue.tail],job,N*sizeof(job[0]));
  job_queue.tail++;
  job_queue.num_of_total_jobs++;
  pthread_cond_broadcast(&job_queue.inputCond);
  pthread_mutex_unlock(&job_queue.mutex);
}
```

```c++
void* dequeueAJob(void *arg)
{
  while(1){
    pthread_mutex_lock(&job_queue.mutex);
    while(job_queue.num_of_job_output<=0 || job_queue.oplist[job_queue.outCount] != 1){
      pthread_cond_wait(&job_queue.outputCond,&job_queue.mutex);
    }
    job_queue.num_of_job_output--;
    copy_i_to_s(job_queue.ans[job_queue.outCount],job_queue.jobs[job_queue.outCount]);
    printf("%s\n",job_queue.jobs[job_queue.outCount]);
    job_queue.outCount++;
    if(job_queue.num_of_total_jobs == MAX_PUZZLE_NUM && job_queue.outCount == MAX_PUZZLE_NUM){  //new code
      job_queue.num_of_job_todo = 0;
      job_queue.tail=0;
      job_queue.head=0;
      job_queue.outCount=0;
      job_queue.num_of_job_output=0;
      job_queue.num_of_total_jobs=0;
      for(int i=0;i<MAX_PUZZLE_NUM;i++){
        job_queue.oplist[i]=0;
      }
      pthread_cond_broadcast(&maxPuzzleCond);
    }
    pthread_mutex_unlock(&job_queue.mutex);
  }
}
```

