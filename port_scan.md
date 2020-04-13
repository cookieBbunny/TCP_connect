# TCP端口扫描实验报告

​	TCP的扫描方式是基于连接的三次握手的变化来判断目标端口的状态。通过端口扫描我们可以了解到目标主机都开放了哪些**服务**，甚至能根据服务猜测可能存在某些**漏洞**。

 	其中***TCP connect()扫描***，也称为全连接扫描，这种方式直接连接到目标端口，完成了**TCP三次握手**的过程，这种方式扫描结果比较准确，但速度比较慢而且可轻易被目标系统检测到。

## 1.原理

​	扫描主机通过TCP/IP协议的***三次握手***与目标主机的指定端口建立一次完整的连接。连接由系统调用***connect***开始。如果端口开放，则连接将建立成功；否则，若返回-1则表示端口关闭。全连接扫描方式的核心就是针对不同端口进行TCP连接,根据是否连接成功来判断端口是否打开。

## 2.运行环境

​	微型计算机一台，ubuntu系统Linux下，c

## 3.代码

> scan.c

```c++
#include<netinet/in.h>
#include<arpa/inet.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
 
#define MAXLINE 4098
 
int main(int argc, char **argv)
{
  int sockfd, n;
  struct sockaddr_in servaddr;
 
  //input the IP address and port(from argv[2] to argv[3])
  if (argc != 4) {
    printf("usage: fulfill the cmd\n");
    return -1;
  }
 
  int i;
  //atoi():用于把一个字符串转换为一个整型数据，该函数定义在stdlib.h中
  for (i = atoi(argv[2]); i < atoi(argv[3]); i++) {
    if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
    //sockfd需要在for循环里面，因为TCP的套接字描述符不能重用，需要在每个TCP的connect创建连接之前重新创建一个新的套接字描述符。
      printf("socket error\n");
      return -1;
    }
 
    //include in string.h
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(i);
    //inet_pton include in <arpa/inet.h>
    if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0) {
      printf("inet_pton error\n");
    }
 
    if (connect(sockfd, (struct sockaddr*) &servaddr, sizeof(servaddr)) < 0) {
      //printf("unuseful port: %d\n", i);
      close(sockfd);
      //每次connect之后都需要利用close()把套接字描述符关闭，从而释放系统资源，避免超过可创建描述符达到上限而无法创建新的套接字
      continue;
    }
    else {
      printf("useful port: %d\n", i);
      close(sockfd);
      continue;
    }
  }
 
  exit(0);
}
```



## 4.实验结果

#### (1).对本机进行扫描

> gsn@gsn-virtual-machine:~$ ./scan 192.168.1.121 1 1500
>
> gsn@gsn-virtual-machine:~$ 

因为扫描速度太慢，因此端口的范围只设置了1~1500，在此范围内我的电脑没有端口处于开放状态，与在命令提示符中执行***netstat命令***出现的结果一致。

![](C:\Users\Administrator\Desktop\11.png)

#### (2).对手机进行扫描

> gsn@gsn-virtual-machine:~$ ./scan 192.168.1.104 1 1500
>
> gsn@gsn-virtual-machine:~$ 

手机端口扫描范围也设置在1~1500，此范围内有没有开放状态的端口。

#### (3).对百度进行扫描

> gsn@gsn-virtual-machine:~$ ./scan 123.125.115.110 1 1500
>
> useful port: 80
>
> useful port: 443

***80***端口：80端口是为**HTTP**即超文本传输协议开放的，主要用于WWW即万维网传输信息的协议。

***443***端口：443端口即网页浏览端口，主要是用于**HTTPS服务**，是提供加密和通过安全端口传输的另一种HTTP。在一些对安全性要求较高的网站，比如银行、证券、购物等，都采用HTTPS服务，这样在这些网站上的交换信息，其他人抓包获取到的是加密数据，保证了交易的安全性。



## 5.自我评价

​	通过本次实验，我对TCP connect()扫描有了更深刻了认识，掌握了socket套接字的使用。本次程序使用的是单进程的方式，扫描速度慢、效率太低，因此端口的扫描范围只用1~1500，如果使用多线程同时观察多个套接字会大大提高扫描效率。