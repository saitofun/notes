## 基本io模型

同步阻塞; 同步非阻塞; 异步阻塞; 异步非阻塞

### 1.1 同步阻塞

用户进程发起IO, 必须等待IO完成, 否则进程只能等待

### 1.2 同步非阻塞

用户进程发起IO, 可以返回做其他处理, 但是需要用户进程询问IO是否就绪

### 1.3 异步阻塞

用户进程发起IO, 无需等待内核IO操作就绪, 内核完成IO操作后, 会通知用户进程, 用户进程再次发起IO并完成数据拷贝;

### 1.4 异步非阻塞

用户进程发起IO, 无需等待内核IO操作就绪, 内核完成IO操作, 并完成数据拷贝后, 最后通知用户进程.

## UNIX IO模型

IO操作流程
> 等待数据就绪
> 从内核复制数据

IO模型
> 阻塞
> 非阻塞
> IO复用
> 信号驱动
> 异步

### 2.1 阻塞式IO

### 2.2 非阻塞IO

### 2.3 IO复用

用户进程调用select/poll等多路复用接口, 阻塞在该接口调用上

### 2.4 信号驱动

内核在描述符就绪时发送SIGIO信号, 通知用户进程

### 2.5 异步IO

告知内核启动某一个操作, 并让内核在整个操作(包括数据拷贝)完成后通知用户进程.

### 2.6 summary

同步异步: 是针对于应用和内核的交互方式而言. 同步需要用户进程询问, 异步则等待通知.
阻塞非阻塞: 是针对用户进程而言. 阻塞是否需要对某一个IO操作进行等待而无法进行其他操作.

## Linux并发网络编程模型

### C10K问题

Client 100000....
处理多客户端连接时的效率和性能问题.
举例: 如果select操作在连接数增加的情况下, 其性能表现为0(n), 从而导致服务器程序在应对多连接时性能下降.

### 3.1 PPC - Apache模型

process per connection: 为每个连接分配一个进程.
没有根本上解决C10K问题. 进程数的增加依然会导致系统消耗的增加, 加之进程间切换的代价在连接数增加的情况下的系统消耗也不容忽视.

### 3.2 TPC

thread per connection: 每个连接一个线程(like PPC)

### 3.3 select

IO多路复用技术.
问题1: 使用select, 受限于select的实现机制. select模型受限于进程打开的描述符个数`FD_SETSIZE`
解决方法: 通过重新编译内核, 更改`FD_SETSIZE`大小.
问题2: select调用每次都会扫描一个文件描述符的集合, 文件描述符个数增加会导致扫描的速度线性增加.
问题3: 数据拷贝, 也会使效率下降.

### 3.4 poll

poll解决了select问题1, 但是问题2, 3仍然遗留.

### 3.5 pselect

同select

### 3.5 epoll

优势1: 不受限于文件描述符个数限制, 只与内存大小相关
优势2: epoll返回发生变化(活跃的/有事件发生的)IO描述符
优势3: 共享内存的方式, 提高了拷贝数据的效率

## Linux epoll

### 4.1 epoll的提升

见3.5

### 4.2 epoll API

自己manual

## 事件驱动

### 5.1 多线程编程

多线程: 并发, 共享进程间状态, 抢占式多任务, 所有连接共用一个内核线程

> 优势: 多CPU, 可以体现性能.
> 劣势: coding不易, 高性能实现困难; 接口不一致(如windows和Linux接口不同); 抢占式容易造成死锁; 难以调试

### 5.2 事件驱动编程

优势:
> 事件驱动, 通常只是一个执行过程, CPU之间不是并发状态, 处理多任务的时候, 协作式处理任务, 而非抢占式.
> 事件驱动比较容易实现, 只需要用户注册时间, 并完成回调逻辑的设计.
> 事件循环在等待时间的发生, 随之调用处理handler, 这个handler生命周期很短.

劣势:
> 如果handler生命周期较长, 会阻塞用户进程.
> 无法通过handler维护本地状态, 因为handler必须返回

### 5.3 event loop

event loop用于等待和发送, 图示如下

没图

``
`
#include <sys/epoll.h>
#include <stdlib.h>
#include <errno.h>
#include "poller.h"

struct event_poller
{
	struct epoll_event* events;
	int epfd;
	int max_events;
	int nevents;
	int next_event;
	int stop;
};

event_poller*
poller_init(int max_events)
{
	event_poller* poller;

	poller = (event_poller*)malloc(sizeof(event_poller));
	if (!poller)
	{
		return NULL;
	}


    poller->max_events = max_events;
	poller->events = (struct epoll_event*)malloc(sizeof(struct epoll_event) * max_events);
	if (!poller->events)
	{
	    free(poller);
		return NULL;
	}

	poller->epfd = epoll_create(1024);
	if (poller->epfd == -1)
	{
	    free(poller->events);
		free(poller);
		return NULL;
	}

	poller->nevents = 0;
	poller->next_event = 0;
	poller->stop = 0;

	return poller;
}

void
poller_free(event_poller* poller)
{
	close(poller->epfd);
	free(poller->events);
	free(poller);
}

int
poller_add(event_poller* poller, int fd, int mask)
{
	struct epoll_event ee;
	int op = (mask & POLL_MOD) ? EPOLL_CTL_MOD : EPOLL_CTL_ADD;

	if (mask & POLL_IN) ee.events |= EPOLLIN;
	if (mask & POLL_OUT) ee.events |= EPOLLOUT;
	ee.data.fd = fd;

	if (epoll_ctl(poller->epfd, op, fd, &ee) == -1) return -1;
    return 0;
}

int
poller_remove(event_poller* poller, int fd, int mask)
{
	struct epoll_event ee;
	int op = (mask & POLL_MOD) ? EPOLL_CTL_MOD : EPOLL_CTL_DEL;

	ee.events = 0;
	if (mask & POLL_IN) ee.events |= EPOLLIN;
	if (mask & POLL_OUT) ee.events |= EPOLLOUT;
	ee.data.fd = fd;

	if (epoll_ctl(poller->epfd, op, fd, &ee) == -1) return -1;
    return 0;
}

int
poller_wait(event_poller* poller, int timeout)
{
	int nevents;

    while (1)
    {
		nevents = epoll_wait(poller->epfd, poller->events, poller->max_events, timeout);
		if (nevents == -1 && errno == EINTR)
		{
            continue;
		}
        break;
    }

	poller->nevents = nevents;
	poller->next_event = 0;
	return nevents;
}

int
poller_event(event_poller* poller)
{
    int i;

	if (poller->next_event >= poller->nevents)
	{
		return -1;
	}

	return poller->events[poller->next_event++].data.fd;
}

void poller_stop(event_poller* poller)
{
	poller->stop = 1;
}




//////////////////////////////////////////////////////////////////////////////////////

#include "poller.h"
#include <stdio.h>
#include <unistd.h>

void file_cb(int fd,void *data,int mask)
{
	char buf[51] ={0};
	read(fd, buf, 51);
	printf("I'm file_cb ,here [EventLoop],[fd : %d],[data: %p],[mask: %d] \n",fd,data,mask);
	printf("get %s",buf);
}

int event_loop(event_poller* poller)
{
    int j = 0;

	while (1)
	{
		//printf("waiting for polling %d:\n", ++j);
	    if (poller_wait(poller, 1000) > 0)
	    {
	        int i = 0;
			int ret;
			int fd;

			while ((fd = poller_event(poller)) >= 0)
			{
			    file_cb(fd, "stdin say: ", ++i);
				ret = poller_add(poller, fd, POLL_MOD | POLL_IN);
				if(ret == -1)
				{
					printf("error2!\n");
					return -1;
				}
			}
	    }
	}

	poller_free(poller);
}

int main()
{
    event_poller* poller = poller_init(10);
	int ret;

	ret = poller_add(poller, STDIN_FILENO, POLL_IN | POLL_ADD);
	if (ret == -1)
	{
		printf("error!\n");
		return -1;
	}

	return event_loop(poller);
}

////////////////////////////////////////////////////////////////////////
#ifndef __POLLER_H__
#define __POLLER_H__

#ifdef __cplusplus
extern "C" {
#endif

typedef struct event_poller event_poller;

#define POLL_NONE 0
#define POLL_IN   1
#define POLL_OUT  2
#define POLL_ADD  4
#define POLL_MOD  8

event_poller* poller_init(int max_events);
void          poller_free(event_poller* poller);
int           poller_add(event_poller* poller, int fd, int mask);
int           poller_remove(event_poller* poller, int fd, int mask);
int           poller_wait(event_poller* poller, int timeout);
int			  poller_event(event_poller* poller);

#ifdef __cplusplus
}
#endif

#endif /* __POLLER_H__ */

`

## reactor模式和proactor模式



















