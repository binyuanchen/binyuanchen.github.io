---
layout: post
title:  "Just enough configurations for Hadoop applications"
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

DISCLAIMER: as of April 28, 2017, I decided to move the instructions of creating a 3-node CDH Hadoop cluster to this [wiki](https://github.com/binyuanchen/arena-dev-cdh-hadoop/wiki/Instructions-to-setup-a-dockerized-3-node-CDH-Hadoop-cluster-V0.1). Please follow this link if you want to setup. Below I will re-purpose this blog for illustrating the examples of arena-dev-cdh-hadoop with minimal configurations.
{: style="color:red; font-size: 100%; text-align: left;"}

This blog is for developers who write application code (client) that accesses CDH Hadoop system.

Making a client application talk to a CDH Hadoop system, can be a small or big effort, depending on what you want to achieve. If you just want your connect to work with Hadoop, a simple approach is to deploy your client application on a Hadoop gateway node so you get all Hadoop services' client configurations, as well as refreshed configurations when they change, and just include all configurations files in your classpath (or just utilize the system environment variables). You do not have to understand how your client is able to talk to HBase, it just does, because all Hadoop services' client configurations are available to your client.

But, many developers are not satisfied that the client is able to talk to Hadoop, but also want to know exactly what configurations makes that happen.

__Here is the purpose of this blog__: We will illustrate a series of examples. Each example has one specific purpose (such as to create HBase table and read and write, with simple authentication etc.). For each example, we demonstrate and explain the set of configurations that are just enough to make the example work.

Of course, we can't do this without a CDH Hadoop cluster, for you and me to understand each other, please follow the [wiki](https://github.com/binyuanchen/arena-dev-cdh-hadoop/wiki/Instructions-to-setup-a-dockerized-3-node-CDH-Hadoop-cluster-V0.1) to setup a 3-node CDH Hadoop cluster first.

[HBase Client  Example](https://github.com/binyuanchen/arena-dev-cdh-hadoop/blob/master/examples/hbase_example/src/main/java/org/arena_dev_cdh_hadoop/hbase/HBaseClientSimpleExample.java)
---
[TODO]

[HBase Client Example (Kerberos)](https://github.com/binyuanchen/arena-dev-cdh-hadoop/blob/master/examples/hbase_example/src/main/java/org/arena_dev_cdh_hadoop/hbase/HBaseClientKerberosExample.java)
---
[TODO]

[Pig Example](https://github.com/binyuanchen/arena-dev-cdh-hadoop/blob/master/examples/pig_example/src/main/java/org/arena_dev_cdh_hadoop/pig/PigSimpleExample.java)
---
[TODO]

Pig Example (Kerberos)
---
[TODO]

[Spark Example](https://github.com/binyuanchen/arena-dev-cdh-hadoop/blob/master/examples/spark_example/src/main/java/org/arena_dev_cdh_hadoop/spark/ScanUserDataSparkSimpleApp.java)
---
[TODO]

Spark Example (Kerberos)
---
[TODO]

Spark Remote Client Example
---
[TODO]

Spark Remote Client Example (Kerberos)
---
[TODO]

[arena-dev-cdh-hadoop-github]: https://github.com/binyuanchen/arena-dev-cdh-hadoop
