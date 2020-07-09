---
layout: post
title: "如何使用timerfd-超时timer，定时timer和不定时timer"
author: "Yongqiang Liu"
---

# 如何使用timerfd-超时timer，定时timer和不定时timer



timerfd 是`Linux`为用户程序提供的一个定时器接口。这个接口基于文件描述符，通过文件描述符的可读事件进行超时通知，所以能够被用于`select/poll`的应用场景，本文将通过使用epoll接口实现超时timer，定时timer和不定时timer。

首先对timerfd的相关接口进行说明：

#### 头文件  

```
#include <sys/timerfd.h>
```

#### 创建timerfd的文件描述符：

```
int timerfd_create(int clockid, int flags);
```

参数clockid用来设置timer相关的时钟类型，有一下类型：

```
CLOCK_REALTIME: 可设置的系统范围的实时时钟
CLOCK_MONOTONIC: 不可设置的单调递增时钟
          从过去某个未指定的点开始的时间
           系统启动后更改。

CLOCK_BOOTTIME（从Linux 3.15开始）: 
                  像CLOCK_MONOTONIC一样，这是单调递增的
          时钟。但是，尽管CLOCK_MONOTONIC时钟不
          测量系统挂起的时间，
          CLOCK_BOOTTIME时钟确实包括
          系统已挂起。这对于以下应用程序很有用
          需要保持挂起状态。 CLOCK_REALTIME不适合
          这类应用，因为该时钟受
          系统时钟的不连续更改。
CLOCK_REALTIME_ALARM（从Linux 3.11开始）:
          该时钟类似于CLOCK_REALTIME，但是如果以下情况将唤醒系统
          它被暂停了。呼叫者必须具有CAP_WAKE_ALARM
          以便针对此时钟设置计时器。
CLOCK_BOOTTIME_ALARM（从Linux 3.11开始）:
          该时钟类似于CLOCK_BOOTTIME，但是如果以下情况将唤醒系统
          它被暂停了。呼叫者必须具有CAP_WAKE_ALARM
          以便针对此时钟设置计时器。
```

#### 设置超时时间及间隔：

```
       int timerfd_settime(int fd, int flags,
                           const struct itimerspec *new_value,
                           struct itimerspec *old_value);
timerfd_settime 用来启动或者停止timerfd描述符的计时器，参数new_value为以下结构：
struct timespec {
time_t tv_sec; / *秒* /
long tv_nsec; / *纳秒* /
};
struct itimerspec {
struct timespec it_interval; / *定期计时器的间隔* /
struct timespec it_value; / *初始到期时间* /
};
new_value.it_value指定计时器的初始到期时间，两个字段秒和纳秒。将new_value.it_value的任何字段设置为非零值将使计时器计时。将new_value.it_value两个字段设置为零将使计时器撤销。
将new_value.it_interval的一个或两个字段设置为非零值指定重复计时器的周期（以秒和纳秒为单位）初始到期后到期。如果两个字段都new_value.it_interval为零，计时器仅过期一次，此时由new_value.it_value指定超时时间。
```

#### 获取timerfd的计时器设置： 

```
timerfd_gettime(int fd, struct itimerspec *curr_value);
```

改函数返回当前描述符所对应的计时器设置。

#### 使用timerfd和epoll实现超时单次的timer

```
#include <errno.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/epoll.h>
#include <sys/timerfd.h>
#include <time.h>

static char *itimerspec_dump(struct itimerspec *ts);

int randomf(int min, int max) {
    srand(time(NULL));
    return min + rand() / (RAND_MAX / (max - min + 1) + 1);
}

int main()
{
    int tfd, epfd, ret;
    struct epoll_event ev;
    struct itimerspec ts;
    int sec = 10; //设置timer10秒后超时
    uint64_t res;

    printf("testcase start\n");

    tfd = timerfd_create(CLOCK_MONOTONIC, 0); //创建定时器的文件描述符
    if (tfd == -1) {
        printf("timerfd_create() failed: errno=%d\n", errno);
        return EXIT_FAILURE;
    }
    printf("created timerfd %d\n", tfd);

    ts.it_interval.tv_sec = 0;
    ts.it_interval.tv_nsec = 0; //设置interval的两项都为0时，定时器只超时一次
    ts.it_value.tv_sec = sec; //定时器启动10秒后超时
    ts.it_value.tv_nsec = 0;

    epfd = epoll_create(1); //创建epoll对象
    if (epfd == -1) {
        printf("epoll_create() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("created epollfd %d\n", epfd);

    ev.events = EPOLLIN;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, tfd, &ev) == -1) { //向Epoll上注册timerfd的 EPOLLIN事件
        printf("epoll_ctl(ADD) failed: errno=%d\n", errno);
        close(epfd);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("added timerfd to epoll set\n");

    if (timerfd_settime(tfd, 0, &ts, NULL) < 0) { //设置超时时间，并启动定时器
        printf("timerfd_settime() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("set timerfd time=%s\n", itimerspec_dump(&ts));

    sleep(1);

    while(1){
        memset(&ev, 0, sizeof(ev));
        ret = epoll_wait(epfd, &ev, 1, 500); //等待epoll时间
        if (ret < 0) {
            printf("epoll_wait() failed: errno=%d\n", errno);
            close(epfd);
            close(tfd);
            return EXIT_FAILURE;
        }
        printf("waited on epoll, ret=%d\n", ret); //timer 超时后会打印改条记录

        ret = read(tfd, &res, sizeof(res)); //超时后必须读取timer 文件描述符上的数据，否则timerfd无法正常运行
        printf("read() returned %d, res=%" PRIu64 "\n", ret, res);

    }

    if (close(epfd) == -1) { //使用完毕，关闭epoll 文件描述符
        printf("failed to close epollfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    if (close(tfd) == -1) {//使用完毕后关闭timer 的文件描述符
        printf("failed to close timerfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}

static char *
itimerspec_dump(struct itimerspec *ts)
{
    static char buf[1024];

    snprintf(buf, sizeof(buf),
            "itimer: [ interval=%lu s %lu ns, next expire=%lu s %lu ns ]",
            ts->it_interval.tv_sec,
            ts->it_interval.tv_nsec,
            ts->it_value.tv_sec,
            ts->it_value.tv_nsec
           );

    return (buf);
}
```

#### 使用timerfd和epoll实现重复的定时的timer

```
#include <errno.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/epoll.h>
#include <sys/timerfd.h>
#include <time.h>

static char *itimerspec_dump(struct itimerspec *ts);

int randomf(int min, int max) {
    srand(time(NULL));
    return min + rand() / (RAND_MAX / (max - min + 1) + 1);
}

int main()
{
    int tfd, epfd, ret;
    struct epoll_event ev;
    struct itimerspec ts;
    int sec = 10; //设置timer10秒后超时
    uint64_t res;

    printf("testcase start\n");

    tfd = timerfd_create(CLOCK_MONOTONIC, 0); //创建定时器的文件描述符
    if (tfd == -1) {
        printf("timerfd_create() failed: errno=%d\n", errno);
        return EXIT_FAILURE;
    }
    printf("created timerfd %d\n", tfd);

    ts.it_interval.tv_sec = sec; //定时器第一次超时后，将重复每10秒超时一次
    ts.it_interval.tv_nsec = 0;
    ts.it_value.tv_sec = sec; //定时器启动后10秒后超时
    ts.it_value.tv_nsec = 0;

    epfd = epoll_create(1); //创建epoll对象
    if (epfd == -1) {
        printf("epoll_create() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("created epollfd %d\n", epfd);

    ev.events = EPOLLIN;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, tfd, &ev) == -1) { //向Epoll上注册timerfd的 EPOLLIN事件
        printf("epoll_ctl(ADD) failed: errno=%d\n", errno);
        close(epfd);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("added timerfd to epoll set\n");

    if (timerfd_settime(tfd, 0, &ts, NULL) < 0) { //设置超时时间，并启动定时器
        printf("timerfd_settime() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("set timerfd time=%s\n", itimerspec_dump(&ts));

    sleep(1);

    while(1){
        memset(&ev, 0, sizeof(ev));
        ret = epoll_wait(epfd, &ev, 1, 500); //等待epoll时间
        if (ret < 0) {
            printf("epoll_wait() failed: errno=%d\n", errno);
            close(epfd);
            close(tfd);
            return EXIT_FAILURE;
        }
        printf("waited on epoll, ret=%d\n", ret); //timer 超时后会打印改条记录

        ret = read(tfd, &res, sizeof(res)); //超时后必须读取timer 文件描述符上的数据，否则timerfd无法正常运行
        printf("read() returned %d, res=%" PRIu64 "\n", ret, res);

    }

    if (close(epfd) == -1) { //使用完毕，关闭epoll 文件描述符
        printf("failed to close epollfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    if (close(tfd) == -1) {//使用完毕后关闭timer 的文件描述符
        printf("failed to close timerfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}

static char *
itimerspec_dump(struct itimerspec *ts)
{
    static char buf[1024];

    snprintf(buf, sizeof(buf),
            "itimer: [ interval=%lu s %lu ns, next expire=%lu s %lu ns ]",
            ts->it_interval.tv_sec,
            ts->it_interval.tv_nsec,
            ts->it_value.tv_sec,
            ts->it_value.tv_nsec
           );

    return (buf);
}
```

#### 使用timerfd和epoll实现时间间隔不定时的timer

有些情况下，会有一些特殊的需求，timer的间隔时间不是定时的，此处将以间隔为1~10的随机值为例子。

```
#include <errno.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/epoll.h>
#include <sys/timerfd.h>
#include <time.h>

static char *itimerspec_dump(struct itimerspec *ts);

int randomf(int min, int max) {
    srand(time(NULL));
    return min + rand() / (RAND_MAX / (max - min + 1) + 1);
}

int main()
{
    int tfd, epfd, ret;
    struct epoll_event ev;
    struct itimerspec ts;
    int sec = 10; //设置timer10秒后超时
    uint64_t res;

    printf("testcase start\n");

    tfd = timerfd_create(CLOCK_MONOTONIC, 0); //创建定时器的文件描述符
    if (tfd == -1) {
        printf("timerfd_create() failed: errno=%d\n", errno);
        return EXIT_FAILURE;
    }
    printf("created timerfd %d\n", tfd);

    ts.it_interval.tv_sec = 0;
    ts.it_interval.tv_nsec = 0; //设置interval的两项都为0时，定时器只超时一次
    ts.it_value.tv_sec = sec; //定时器启动10秒后超时
    ts.it_value.tv_nsec = 0;

    epfd = epoll_create(1); //创建epoll对象
    if (epfd == -1) {
        printf("epoll_create() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("created epollfd %d\n", epfd);

    ev.events = EPOLLIN;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, tfd, &ev) == -1) { //向Epoll上注册timerfd的 EPOLLIN事件
        printf("epoll_ctl(ADD) failed: errno=%d\n", errno);
        close(epfd);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("added timerfd to epoll set\n");

    if (timerfd_settime(tfd, 0, &ts, NULL) < 0) { //设置超时时间，并启动定时器
        printf("timerfd_settime() failed: errno=%d\n", errno);
        close(tfd);
        return EXIT_FAILURE;
    }
    printf("set timerfd time=%s\n", itimerspec_dump(&ts));

    sleep(1);

    while(1){
        memset(&ev, 0, sizeof(ev));
        ret = epoll_wait(epfd, &ev, 1, 500); //等待epoll时间
        if (ret < 0) {
            printf("epoll_wait() failed: errno=%d\n", errno);
            close(epfd);
            close(tfd);
            return EXIT_FAILURE;
        }
        printf("waited on epoll, ret=%d\n", ret); //timer 超时后会打印改条记录

        ret = read(tfd, &res, sizeof(res)); //超时后必须读取timer 文件描述符上的数据，否则timerfd无法正常运行
        printf("read() returned %d, res=%" PRIu64 "\n", ret, res);

        ts.it_interval.tv_sec = 0;
        ts.it_interval.tv_nsec = 0;
        ts.it_value.tv_sec = randomf(1, 10); //重新设置定时器的初次超时时间为1~10的随机数，
                                             //相当于再每次超时后重新启动timer计时器
        ts.it_value.tv_nsec = 0;

        if (timerfd_settime(tfd, 0, &ts, NULL) < 0) {
            printf("timerfd_settime() failed: errno=%d\n", errno);
            close(tfd);
            return EXIT_FAILURE;
        }
        printf("set timerfd time=%s\n", itimerspec_dump(&ts));

    }

    if (close(epfd) == -1) { //使用完毕，关闭epoll 文件描述符
        printf("failed to close epollfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    if (close(tfd) == -1) {//使用完毕后关闭timer 的文件描述符
        printf("failed to close timerfd: errno=%d\n", errno);
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}

static char *
itimerspec_dump(struct itimerspec *ts)
{
    static char buf[1024];

    snprintf(buf, sizeof(buf),
            "itimer: [ interval=%lu s %lu ns, next expire=%lu s %lu ns ]",
            ts->it_interval.tv_sec,
            ts->it_interval.tv_nsec,
            ts->it_value.tv_sec,
            ts->it_value.tv_nsec
           );

    return (buf);
}
```