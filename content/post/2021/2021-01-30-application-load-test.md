---
title: "Application Load Test"
date: 2021-01-30
categories: ["Tools", "Utilities", "Tests"]
draft: false
---
# Application Load Test

When you're developing an application, you need to know how much accessible your application will be when many clients are connected, or if you already have a system in your office and you may want to know if number of employees are increasing during the time, can this application still serve requests?.
It's a good idea to know at least as a range of number how many users your application can handle at the same time.
Luckily, there are plenty of tools exist to let you test your application by supporting variety of protocols.
In this article, we're going to review some CLI HTTP load testing tools which you can start using them today.

### Scenario

Let's imagin we have a web application and we want to make sure it can serve requests without any problem for 1000 requests in total with concurrency of 100 requests at the same time.
So, it means:

* Total: 1000 requests
* Concurrency: 100 requests
Let's say each request takes about 0.5 milliseconds, it means for a 100 requests we expected to get result in 0.5 milliseconds because they're running in concurrent. If requests were in sequential, it would take 50 seconds to be finished (`100 * 0.5 = 50 seconds`).
Now, our expectation will be `(1000 / 100) * 0.5 = 5`. It means in this scenario we expect to get total result in about 5 seconds.

**Note:** these numbers are just an example to give you the idea. For the real projects, please find the numbers in detail.

### [Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html)
[Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html) or ab is a HTTP benchmarking tool that you can test your application with simultaneous users/requests at the same time.
Based on Apache website:

> ab is a tool for benchmarking your Apache Hypertext Transfer Protocol (HTTP) server. It is designed to give you an impression of how your current Apache installation performs. This especially shows you how many requests per second your Apache installation is capable of serving.

To start using Apache Bench, first, you need to install it in your system.
**ArchLinux**

```bash
$ sudo pacman -S apache
```
**Debian/Ubuntu**

```bash
$ sudo apt install apache2-utils
```
**Fedora**

```bash
$ sudo dnf install httpd-tools
```

To know more about installation and other distributions/OS, please check the official website.

Now, it's time to use it:

```bash
$ ab -n 1000 -c 100 http://localhost/myapplication
```

### [Cassowary](https://github.com/rogerwelin/cassowary/)
[cassowary](https://github.com/rogerwelin/cassowary/) is another open source HTTP load testing application which has been developed in [Golang](https://golang.org/).
Based on Cassowary's Github:

> Cassowary is a modern HTTP/S, intuitive & cross-platform load testing tool built in Go for developers, testers and sysadmins. Cassowary draws inspiration from awesome projects like k6, ab & httpstat.

To start using Cassowary, first, you need to install it in your system.
You can easily download the binary version of it from the [release page](https://github.com/rogerwelin/cassowary/releases) and put it in your `PATH` variable, e.g. `/usr/local/bin/`.

>  Note: please don't forget to assign the "execute" permission to the file.

Now, it's time to use it:
```bash
$ cassowary run -u http://localhost/myapplication -n 1000 -c 100
```

### [Bombardier](https://github.com/codesenberg/bombardier)
[Bombardier](https://github.com/codesenberg/bombardier) is also an open source CLI HTTP load testing application which has been developed in Golang.
Based on Bombardier's Github:

> bombardier is a HTTP(S) benchmarking tool. It is written in Go programming language and uses excellent fasthttp instead of Go's default http library, because of its lightning fast performance.

To start using Bombardier, first, you need to install it in your system.
You can easily download the binary version of it from the [release page](https://github.com/codesenberg/bombardier/releases) and put it in your `PATH` variable, e.g. `/usr/local/bin/`.

> Note: please don't forget to assign the "execute" permission to the file.

Now, it's time to use it:

```bash
$ bombardier -n 1000 -c 100 http://localhost/myapplication
```

### [Wrk](https://github.com/wg/wrk)

[Wrk](https://github.com/wg/wrk) is another open source HTTP benchmarking tool which completely developed in C.

Based on wrk's Github:

> wrk is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU. It combines a multithreaded design with scalable event notification systems such as epoll and kqueue.

One benefit on wrk is it supports [LuaJIT](https://luajit.org/) to perform HTTP request genetion:

> An optional LuaJIT script can perform HTTP request generation, response processing, and custom reporting. Details are available in SCRIPTING and several examples are located in [scripts/](https://github.com/wg/wrk/blob/master/scripts).

To start using wrk, first, you need to install it in your system.

**ArchLinux**

```bash
$ yay -S wrk
```

**Debian/Ubuntu**

```bash
$ sudo apt install wrk
```

**Fedora**

```bash
$ dnf copr enable getpagespeed/wrk
$ dnf install wrk
```

To know more about installation and other distributions/OS, please check the its [wiki page](https://github.com/wg/wrk/wiki).

Now, it's time to use it:

```bash
$ wrk -c1000 -t100 http://localhost/myapplication
```

### Conclusion

Load testing or getting benchmark of an application is a very important step of releasing. If you don't have any idea about your application benchmark, maybe it's the time to check it today with these tools. If BlackFriday is coming, does your ecommerce or website support many active users at the same time?
At least you need to have a baseline for your application's load test to know the current status and then you can prepare a plan to improve and optimize it in the future. They're plenty of tools exist for load testing that we didn't dicuss, in the upcoming articles, we'll talk about them.

