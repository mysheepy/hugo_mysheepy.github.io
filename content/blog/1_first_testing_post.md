---
title: "1_first_testing_post"
date: 2024-03-18T17:16:30+08:00
draft: false
tags: ["c", "pthread", "example"]
categories: ["examples"]
slug: "pthread-sample-code-in-c"
---

pthread C sample code:

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <string.h>
#include <stdbool.h>

#define RAND_DELAY true
#define DETACH_TH true

#define THREAD_NUM 300

struct msg
{
    int id;
    char msg[32];
    int *count;
    pthread_mutex_t *mutex;
};

void *counter_func(void *msg_ptr);

int main(int argc, char argv[])
{
    pthread_mutex_t mutex_cnt = PTHREAD_MUTEX_INITIALIZER;
    pthread_t threads[THREAD_NUM];
    struct msg msgs[THREAD_NUM];
    int irets[THREAD_NUM];
    int count = 0;

#if RAND_DELAY
    srand(time(NULL));
#endif

    for (int i = 0; i < THREAD_NUM; i++)
    {
        sprintf((char *)msgs[i].msg, "Thread %d", i);
        msgs[i].count = &count;
        msgs[i].id = i;
        msgs[i].mutex = &mutex_cnt;
    }

    for (int i = 0; i < THREAD_NUM; i++)
    {
#if DETACH_TH
        do
        {
            irets[i] = pthread_create(&threads[i], NULL, counter_func, (void *)&(msgs[i]));
        } while (irets[i] != 0);
        pthread_detach(threads[i]);
#else
        irets[i] = pthread_create(&threads[i], NULL, counter_func, (void *)&(msgs[i]));
#endif
    }

#if !DETACH_TH
    for (int i = 0; i < THREAD_NUM; i++)
        pthread_join(threads[i], NULL);
#endif

#if !DETACH_TH
    for (int i = 0; i < THREAD_NUM; i++)
        if (irets[i] != 0)
            printf("[ERR]: irets[%d] %d\r\n", i, irets[i]);
#endif

#if DETACH_TH
    printf("=====\r\n");
    usleep(30000000);
#endif
    printf("count: %d\r\n", count);

    return EXIT_SUCCESS;
}

void *counter_func(void *msg_ptr)
{
    // pthread_detach(pthread_self());

    struct msg *message = (struct msg *)msg_ptr;
    pthread_mutex_lock(message->mutex);
    printf("%s \n", message->msg);
    int tmp = *(message->count);

#if RAND_DELAY
    int sleep_time = ((rand() % 30) * 10000);
    usleep(sleep_time);
#endif

    tmp += ((message->id) * 2);
    *(message->count) = tmp;

    pthread_mutex_unlock(message->mutex);
}
```
