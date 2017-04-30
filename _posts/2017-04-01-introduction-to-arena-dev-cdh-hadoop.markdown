---
layout: post
title:  "Introduction to arena-dev-cdh-hadoop"
date:   2017-04-01 12:16:49 -0800
categories: posts update
---

<style>
table{
    border-collapse: collapse;
    border-spacing: 1;
    border:2px solid #000000;
}
th{
    border:2px solid #000000;
}
td{
    border:1px solid #000000;
}
</style>

Initiative
---

This blog is for developers who write application code (client) that accesses CDH Hadoop system. We talk about how to create a three-node CDH Hadoop cluster running on your local Mac machine, using open source project [arena-dev-cdh-hadoop](https://github.com/binyuanchen/arena-dev-cdh-hadoop).

Some highlight of such a cluster,

* it runs on your local Mac,
* it is a cluster with more than one node, all nodes are running as docker containers,
* if you screw the cluster, you can quickly setup a new one using deployer script (python),
* the cluster is generic enough in the sense that once you verified your application code works well with the cluster, it doesn't take much effort to make it work with your production cluster,
* the cluster is not for performance or production use.

With such a cluster, you no longer worry about confusions caused when your application code writes into a Jenkins or automation testing Hadoop cluster which is shared by automation tests.

As of April 28, 2017, I moved most of the setup steps in the blog to the project wiki at [arena-dev-cdh-hadoop wiki](https://github.com/binyuanchen/arena-dev-cdh-hadoop/wiki).

Here is an example of the cloudera manager admin UI after you set this cluster up:

![CM Admin UI]({{ site.url }}/images/cmui.png)



[arena-dev-cdh-hadoop-github]: https://github.com/binyuanchen/arena-dev-cdh-hadoop
