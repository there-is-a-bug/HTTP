# threadpool.c 

## 结构体参数

- 工作体worker
- 线程池CThread_pool



### 工作体worker

- 回调函数

~~~c
void *(*process) (void *arg);
//
using callback = void *(*process) (void *arg)
typedef void *(*process) (void *arg) callback
~~~

简而言之，回调函数就是自身作为参数的函数，**在C语言中，回调函数只能使用函数指针实现******而c++ 11中可用匿名函数实现**（个人猜想，由于函数体巨大，作为参数只能传入口，不可能把所有值传递），这里回调函数的意义在于当线程创建成功之后，或者被唤醒之后，需要该回调函数去进行下一步处理，**相当于传递具体的工作函数**

C++ 11匿名函数简单实现

~~~c++
[](size_t Featurea, size_t Featureb){return Featurea<Featureb ? Featurea : Featureb; };
~~~

匿名函数可做函数指针实现回调函数

- 回调函数参数

~~~c
void *arg
~~~

回调函数的参数，这里用void 强调泛型，即**任何类型的指针都可以直接赋值给arg，而无需转换**

- 指针

~~~c
struct worker *next;
~~~

为工作队列链表做准备



### 线程池CThread_pool

~~~c
//互斥锁
pthread_mutex_t queue_lock;
//条件变量，与互斥锁一起使用
pthread_cond_t queue_ready;
 /*链表结构，线程池中所有等待任务*/
CThread_worker *queue_head;
/*是否销毁线程池*/
int shutdown;
//线程队列指针
pthread_t *threadid;
/*线程池中允许的活动线程数目*/
int max_thread_num;
/*当前等待队列的任务数目*/
int cur_queue_size;
~~~

- 由于c中没有类的概念，因此相应函数在外

~~~
//添加工作函数
void pool_add_worker (void *(*process) (void *arg), void *arg);
//工作线程（轮询）
void *thread_routine (void *arg);
~~~

省去了管理函数和一些常驻线程等，可以看做都是常驻线程

- 共享资源

~~~
static CThread_pool *pool = NULL;
int number_thread[30];  // 创建的线程编号
~~~

pool的意义在于，c没有类的概念，自然没有this指针，他的作用即是代替this指针，同时static限制作用域



#### 一些函数

- 初始化函数（对应构造函数）

~~~
void pool_init(int max_thread_num)
~~~

任务如下：

初始化pool指针

~~~c
pool = (CThread_pool *) malloc (sizeof (CThread_pool));
~~~

初始化锁和条件变量，任务队列头，队列大小，线程池标志位等一系列东西

创建线程与id

~~~
number_thread[i] = i;
        pthread_create (&(pool->threadid[i]), NULL, thread_routine, &number_thread[i]);
~~~



- 任务管理函数（添加任务）

  构造新任务与尾插法

- 工作函数

- 线程池关闭函数