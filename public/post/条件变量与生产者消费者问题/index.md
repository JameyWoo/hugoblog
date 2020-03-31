
本文主要是探讨<<操作系统导论>>一书第30章-条件变量的知识.
书中介绍了条件变量的概念, 并将条件变量运用在生产者消费者问题中. 从最简单的情况开始, 列举了使用条件变量解决生产者消费者问题的几种错误用法.
本文主要是对这几种情况的代码模拟分析, 分为以下四个部分

* 使用 if而非 while且只有一个条件变量
* 使用while但只有一个条件变量
* 使用while且有两个条件变量
* 扩展缓冲区大小(从1到数组)

---

## 使用 if而非 while且只有一个条件变量
书中提供的第一个方案(有问题), 给生产者和消费者共用一个条件变量, 且使用if来判断缓存区

问题: wait的条件使用了if而不是while, 导致如果有多个消费者的情况, 当一个阻塞的消费者被生产者唤醒了, 准备执行但这时另一个消费者抢占执行并进行了消费导致缓冲区空了. 这时切换到第一个消费者消费, 因为缓冲区空了触发断言, 程序错误. 
如果换成while, 那么第二个消费者在醒来的时候, 就会再判断一下条件是否成立, 由于被另一个消费者消费了, 所以它又会调用wait被阻塞.

需要注意wait函数的执行过程. 当一个线程执行wait的时候, 会释放它持有的锁, 在被唤醒并执行的时候, 会重新持有锁. 但, 如果被唤醒且进入了就绪队列, 那么这时它还是处于没有持有锁的状态, 因此其他的消费者可以进行消费. 

看下面这份代码(由于抢占难以正确模拟, 所以使用“先使消费者线程睡眠, 再手动唤醒”的方式来实现抢占, 这种方式是概率性的, 有可能不成功)

```c
    /**
     * 第一版生产者消费者问题:
     * 1. 使用 if 而不是 while
     * 2. 使用一个条件变量
     * @Desc: 这个程序将会模拟上述两个问题带来的一种错误.
     */
    
    #include <assert.h>
    #include <pthread.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    int buffer;
    int count = 0;
    
    pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    pthread_cond_t cond   = PTHREAD_COND_INITIALIZER;
    
    void put(int value) {
        printf("do put\n");
        assert(count == 0);
        count  = 1;
        buffer = value;
    }
    
    int get() {
        printf("do get\n");
        assert(count == 1);
        count = 0;
        return buffer;
    }
    
    void *producer(void *arg) {
        int i = *(int *)arg;
        printf("i am producer %d\n", i);
        pthread_mutex_lock(&mutex);
    
        if (count == 1) {
            pthread_cond_wait(&cond, &mutex);
        }
        put(-1);
        pthread_cond_signal(&cond);
    
        sleep(1);
    
        pthread_mutex_unlock(&mutex);
    }
    
    void *consumer(void *arg) {
        int i = *(int *)arg;
        printf("i am consumer %d\n", i);
        pthread_mutex_lock(&mutex);
        printf("cons %d: count = %d\n", i, count);
    
        if (count == 0) {
            printf("cons %d: 我进入睡眠\n", i);
            pthread_cond_wait(&cond, &mutex);
            printf("cons %d: 我被唤醒了\n", i);
            sleep(1);
        }
        int val = get();
        printf("cons %d: get val = %d\n", i, val);
        pthread_cond_signal(&cond);
    
        pthread_mutex_unlock(&mutex);
    }
    
    int main() {
        pthread_t prod, cons1, cons2;
        int i1 = 1, i2 = 2;
        pthread_create(&cons2, NULL, consumer, &i2);  // 先让 cons1睡眠, 是为了之后cons1可以在cons2之前执行
        pthread_create(&cons1, NULL, consumer, &i1);
        sleep(1);
    
        int p1 = 1;
        pthread_create(&prod, NULL, producer, &p1);
        pthread_cond_signal(&cond);  // 唤醒消费者1以模拟抢占情况
    
        pthread_join(cons1, NULL);
        pthread_join(prod, NULL);
        pthread_join(cons2, NULL);
    }
```
    
注意: 在这份代码中, 生产者和消费者各执行一次

一种可能的执行情况是
![](/images/20200329-1.png)
由于没有while检测, 出发了 assert导致程序异常退出

如果使用了while呢?

程序将会等待一个生产者来唤醒消费者.
![](/images/20200329-2.png)

## 使用while但只有一个条件变量

但在将if变成while之后, 还是没有解决另一个问题: 这个问题是由于生产者和消费者共用一个条件变量引起的

一种简单情况就是存在一个生产者和两个消费者, 刚开始时缓冲区为空, 两个消费者分别进行消费并且进入睡眠, 这时生产者生产且因为缓冲区只有一所以进入了睡眠, 并且唤醒了期中一个消费者. 这个消费者进行消费之后执行signal唤醒一个睡眠的线程, 存在一种情况它唤醒的是另一个消费者, 于是这个消费者试图进行消费而被阻塞. 于是这三个线程都进入了睡眠状态, 程序无法继续执行. 

看下面这份代码
```c
/**
 * 第二版生产者消费者问题:
 * 1. 使用一个条件变量
 * @Desc: 如果有一个生产者而有多个消费者且消费者唤醒消费者而不是生产者将会导致所有的线程进入睡眠
 */

#include <assert.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int buffer;
int count = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond   = PTHREAD_COND_INITIALIZER;

void put(int value) {
    printf("do put\n");
    assert(count == 0);
    count  = 1;
    buffer = value;
}

int get() {
    printf("do get\n");
    assert(count == 1);
    count = 0;
    return buffer;
}

void *producer(void *arg) {
    int cnt = 0;
    while (1) {
        int i = *(int *)arg;
        printf("i am producer %d\n", i);
        pthread_mutex_lock(&mutex);

        while (count == 1) {
            printf("proc %d: 我进入睡眠\n", i);
            pthread_cond_wait(&cond, &mutex);
        }
        put(cnt);
        pthread_cond_signal(&cond);

        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    while (1) {
        int i = *(int *)arg;
        printf("i am consumer %d\n", i);
        pthread_mutex_lock(&mutex);
        printf("cons %d: count = %d\n", i, count);

        while (count == 0) {
            printf("cons %d: 我进入睡眠\n", i);
            pthread_cond_wait(&cond, &mutex);
            printf("cons %d: 我被唤醒了\n", i);
            sleep(1);
        }
        printf("cons %d: before get\n", i);
        int val = get();
        printf("cons %d: get val = %d\n", i, val);
        pthread_cond_signal(&cond);

        pthread_mutex_unlock(&mutex);
    }
}

int main() {
    pthread_t prod, cons1, cons2;
    // 刚开始两个消费者都会睡眠, 因为刚开始缓冲区为空
    int c1 = 1, c2 = 2;
    pthread_create(&cons1, NULL, consumer, &c1);
    pthread_create(&cons2, NULL, consumer, &c2);
    sleep(2);
    // 之后消费者加入之后唤醒一个, 然后自己进入睡眠
    int p1 = 1;
    pthread_create(&prod, NULL, producer, &p1);

    pthread_join(cons1, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons2, NULL);
}
```
![](/images/20200329-3.png)
由于 `while(1)`, 本该一直执行下去的程序陷入了暂停的窘境. 其原因就在于消费者唤醒了消费者, 而第二个消费者无法唤醒生产者, 导致三个线程都陷入了睡眠.

## 使用while且有两个条件变量

解决这个问题的方法也很显然, 就是分别提供两个条件变量, 使得生产者只能唤醒消费者, 而消费者只能唤醒生产者.

修改的程序很简单, 只需要添加一个条件变量, 分别为`fill`, `empty`条件变量.

生产者等待empty条件; 消费者等待fill条件

生产者执行后, signal fill条件; 消费者执行后, signal empty条件

完整程序如下
```c
/**
 * 第三版生产者消费者问题
 * 1. 使用while
 * 2. 使用两个条件变量
 * @Desc: 如果有一个生产者而有多个消费者且消费者唤醒消费者而不是生产者将会导致所有的线程进入睡眠
 */

#include <assert.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int buffer;
int count = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t empty   = PTHREAD_COND_INITIALIZER;
pthread_cond_t fill   = PTHREAD_COND_INITIALIZER;

void put(int value) {
    printf("do put\n");
    assert(count == 0);
    count  = 1;
    buffer = value;
}

int get() {
    printf("do get\n");
    assert(count == 1);
    count = 0;
    return buffer;
}

void *producer(void *arg) {
    int cnt = 0;
    while (1) {
        int i = *(int *)arg;
        printf("i am producer %d\n", i);
        pthread_mutex_lock(&mutex);

        while (count == 1) {
            printf("proc %d: 我进入睡眠\n", i);
            pthread_cond_wait(&empty, &mutex);
        }
        put(cnt);
        pthread_cond_signal(&fill);

        pthread_mutex_unlock(&mutex);
    }
}

void *consumer(void *arg) {
    while (1) {
        int i = *(int *)arg;
        printf("i am consumer %d\n", i);
        pthread_mutex_lock(&mutex);
        printf("cons %d: count = %d\n", i, count);

        while (count == 0) {
            printf("cons %d: 我进入睡眠\n", i);
            pthread_cond_wait(&fill, &mutex);
            printf("cons %d: 我被唤醒了\n", i);
            sleep(1);
        }
        printf("cons %d: before get\n", i);
        int val = get();
        printf("cons %d: get val = %d\n", i, val);
        pthread_cond_signal(&empty);

        pthread_mutex_unlock(&mutex);
    }
}

int main() {
    pthread_t prod, cons1, cons2;
    // 刚开始两个消费者都会睡眠, 因为刚开始缓冲区为空
    int c1 = 1, c2 = 2;
    pthread_create(&cons1, NULL, consumer, &c1);
    pthread_create(&cons2, NULL, consumer, &c2);
    sleep(2);
    // 之后消费者加入之后唤醒一个, 然后自己进入睡眠
    int p1 = 1;
    pthread_create(&prod, NULL, producer, &p1);

    pthread_join(cons1, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons2, NULL);
}
```


## 扩展缓冲区大小(从1到数组)
上面探究的是缓冲区只有一个大小的情况, 而实际上缓冲区是有一定空间的, 下面的代码将缓冲区扩展为一个数组

```c
/**
 * 第四版生产者消费者问题
 * 1. 使用while
 * 2. 使用两个条件变量
 * 3. 缓冲区大小不是1, 而是有一定长度
 * @Desc: 如果有一个生产者而有多个消费者且消费者唤醒消费者而不是生产者将会导致所有的线程进入睡眠
 */

#include <assert.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define MAX_SIZE 5

int buffer[MAX_SIZE] = {};
int count = 0;
int loc_put = 0;
int loc_get = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t empty   = PTHREAD_COND_INITIALIZER;
pthread_cond_t fill   = PTHREAD_COND_INITIALIZER;

void put(int value) {
    printf("do put loc = %d\n", loc_put);
    assert(count < MAX_SIZE);
    count += 1;
    buffer[loc_put] = value;
    loc_put = (loc_put + 1) % MAX_SIZE;
}

int get() {
    printf("do get loc = %d\n", loc_get);
    assert(count > 0);
    count -= 1;
    int tmp = buffer[loc_get];
    loc_get = (loc_get + 1) % MAX_SIZE;
    return tmp;
}

void *producer(void *arg) {
    int cnt = 0;
    while (1) {
        int i = *(int *)arg;
        printf("i am producer %d\n", i);
        pthread_mutex_lock(&mutex);

        while (count == MAX_SIZE) {
            printf("proc %d: 我进入睡眠\n", i);
            pthread_cond_wait(&empty, &mutex);
        }
        put(cnt);
        pthread_cond_signal(&fill);

        pthread_mutex_unlock(&mutex);
        cnt++;
    }
}

void *consumer(void *arg) {
    while (1) {
        int i = *(int *)arg;
        printf("i am consumer %d\n", i);
        pthread_mutex_lock(&mutex);
        printf("cons %d: count = %d\n", i, count);

        while (count == 0) {
            printf("cons %d: 我进入睡眠\n", i);
            pthread_cond_wait(&fill, &mutex);
            printf("cons %d: 我被唤醒了\n", i);
            usleep(10000);
        }
        int val = get();
        printf("cons %d: get val = %d\n", i, val);
        pthread_cond_signal(&empty);

        pthread_mutex_unlock(&mutex);
    }
}

int main() {
    pthread_t prod1, prod2, prod3, cons1, cons2;
    // 刚开始两个消费者都会睡眠, 因为刚开始缓冲区为空
    int c1 = 1, c2 = 2;
    pthread_create(&cons1, NULL, consumer, &c1);
    pthread_create(&cons2, NULL, consumer, &c2);
    sleep(2);
    // 之后消费者加入之后唤醒一个, 然后自己进入睡眠
    int p1 = 1, p2 = 2, p3 = 3;
    pthread_create(&prod1, NULL, producer, &p1);
    pthread_create(&prod2, NULL, producer, &p2);
    pthread_create(&prod3, NULL, producer, &p3);

    pthread_join(cons1, NULL);
    pthread_join(cons2, NULL);
    
    pthread_join(prod1, NULL);
    pthread_join(prod2, NULL);
    pthread_join(prod3, NULL);
}
```