---
slug: 03-load-testing
id: odtuyjaghogf
type: challenge
title: 'Cluster Performance: Scaling applications horizontally with containers'
tabs:
- title: Terminal 1
  type: terminal
  hostname: crc
- title: Terminal 2
  type: terminal
  hostname: crc
- title: Visual Editor
  type: code
  hostname: crc
  path: /root/labs
difficulty: intermediate
timelimit: 675
---
In this exercise, you will scale and load test a distributed application using a tool called Apache Bench. Apache Bench (ab command) is a tool for benchmarking HTTP servers. It is designed to give you an impression of how your current Apache installation perform. In particular ab shows you how many requests per second your Apache installation is capable of serving. The ab tool is especially useful with a Kubernetes cluster, where you can scale up web servers with a single command and provide more capacity to handle more requests per second.

Before we test, copy the URL address to connect to for several tests:

```
echo "http://$(oc get svc | grep wpfrontend | awk '{print $3}')/wp-admin/install.php"
```

From *Terminal 2*, start toolbox to install AB benchmarking tool from Apache Tools inside a container:

```
toolbox
```

Install AB:

```
dnf install -y httpd-tools
```

Copy the URL from the Terminal 1 into an environment variable so we can use it multiple times:

```
export SITE=<URL_COPIED>
```

Test with ab before we scale the application to get a base line. Take note of the "Time taken for tests" section:

```
ab -n10 -c 3 -k -H "Accept-Encoding: gzip, deflate" $SITE
```


Take note of the following sections in the output:

- Time taken for tests
- Requests per second
- Percentage of the requests served within a certain time (ms)


Go to the web interface. Scale the Wordpress Deployment up to three containers. Click the up arrow:

- Workloads -> Deployments -> wordpress -> Up Arrow


Test with ab again. How did this affect our benchmarking and why?

```
ab -n10 -c 3 -k -H "Accept-Encoding: gzip, deflate" $SITE
```

Come back in *Terminal 1*, now lets scale up more with command line instead of the web interface:

```
oc scale --replicas=5 rc/wordpress
```


Come back to *Terminal 2* to test with ab. How did this affect our benchmarking and why?

```
ab -n10 -c 3 -k -H "Accept-Encoding: gzip, deflate" $SITE
```


In *Terminal 1*, scale the application back down to one pod:

```
oc scale --replicas=1 rc/wordpress
```

From *Terminal 2*, test with ab. How did this affect our benchmarking and why?

```
ab -n10 -c 3 -k -H "Accept-Encoding: gzip, deflate" $SITE
```

Counter intuitively, you may have seen slower response times when you scaled the pods up. This is probably because we are running Kubernetes on a single node. The requests are bing sent through a reverse proxy and distributed to multiple pods all running on the same server for this lab. If this were a real OpenShift cluster with 100s or 1000s of nodes, you may have seen scale out performance where response times went down.

Sometimes horizontal scaling can have counter intuitive effects. Sometimes great care must be taken with applications to get the performance characteristics that we need. The world of container orchestration opens up an entirely new kind of performance tuning and you will need new skills to tackle this challenge.
