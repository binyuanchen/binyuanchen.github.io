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

This blog is for developers who work on application code that accesses CDH Hadoop system. Here we talk about how to create a three-node (yes, only 3, no more and no less) CDH Hadoop cluster running on your local Mac machine, using open source project [arena-dev-cdh-hadoop](https://github.com/binyuanchen/arena-dev-cdh-hadoop).

Some highlight of such a cluster,

* it runs on your local Mac,
* it is a cluster with more than one node, all nodes are running as docker containers,
* if you screw the cluster, you can quickly setup a new one using deployer script (python),
* the cluster is generic enough in the sense that once you verified your application code works well with the cluster, it doesn't take much effort to make it work with your production cluster,
* the cluster is not for performance or production use.

With such a cluster, you no longer worry about confusions caused when your application code writes into a Jenkins or automation testing Hadoop cluster which is shared by automation tests.

System and Software Requirements
------------------------------
* Mac OS X (Yosemite and above)
* 20GB+ memory (other than your regular usage)

> Tips: I managed to create a three-node cluster with 12GB memory. But for any practical use, I recommend to have at lease 20GB memory dedicated for creating a three-node cluster.

* 120GB+ disk space (other than your regular usage)
* Python 2.7+
* docker engine and docker-machine (for example, you can install ['Docker For Mac'](https://docs.docker.com/docker-for-mac/))
* virtualbox (See https://www.virtualbox.org/wiki/Downloads for downloading for OSX hosts)
* [paramiko](http://www.paramiko.org/installing.html).
* `git clone https://github.com/binyuanchen/arena-dev-cdh-hadoop.git` (later we will refer to the absolute path to the root directory of this cloned project as \<ARENA-SRC-ROOT\>)


Part 1: Prepare the infrastructure
---------------------------


<a name="Step11">Step 1.1</a>
====

open a terminal window and create 4 tabs in it and name the tabs as __mh-keystore__, __mgr1__, __mgr2__ and __mgr3__, later I will refer to these tabs as __tab mgr1__ or alike, when we will execute some commands against docker hosts.

An illustration of the four tabs is,

![Four Tabs example]({{ site.url }}/images/fourtabs.png)

<a name="Step12">Step 1.2</a>
====

in tab mgr1, execute below four commands, sequentially,

```bash
docker-machine create -d virtualbox mh-keystore
docker-machine create -d virtualbox --virtualbox-disk-size "40960" --virtualbox-memory "8192" mgr1
docker-machine create -d virtualbox --virtualbox-disk-size "40960" --virtualbox-memory "6144" mgr2
docker-machine create -d virtualbox --virtualbox-disk-size "40960" --virtualbox-memory "6144" mgr3
```

> This creats four docker hosts, with names mh-keystore, mgr1, mgr2 and mgr3. mgr1 is configured to have 40GB of disk and 8GB of memory, mgr2 and mgr3 have the same disk, but with 6GB of memory each.

After these four hosts are created successfully, collect the ip addresses assigned to these hosts,

```bash
docker-machine ip mh-keystore
```
192.168.99.100
```bash
docker-machine ip mgr1
```
192.168.99.101
```bash
docker-machine ip mgr2
```
192.168.99.102
```bash
docker-machine ip mgr3
```
192.168.99.103

> The above ips are the results on my machine, you may get different ips. But it is nevertheless important to remember the mappings between the docker host names and their assigned ips, as we will directly use these ips below without mentioning which hosts they belong to.

<a name="Step13">Step 1.3</a>
====

In tab mh-keystore, execute,
{% highlight bash %}
eval $(docker-machine env mh-keystore)
{% endhighlight %}

In tab mgr1, execute,
{% highlight bash %}
eval $(docker-machine env mgr1)
{% endhighlight %}

In tab mgr2, execute,
{% highlight bash %}
eval $(docker-machine env mgr2)
{% endhighlight %}

In tab mgr3, execute,
{% highlight bash %}
eval $(docker-machine env mgr3)
{% endhighlight %}

<a name="Step14">Step 1.4</a>
====

In tab mh-keystore, execute,
```bash
docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap
```
> This creates and runs the consul store. We use this container to support the creation of cross hosts overlay network.

<a name="Step15">Step 1.5</a>
====

In tab mgr1, ssh into host mgr1, via
```bash
docker-machine ssh mgr1
```
, then inside host mgr1, modify boot2docker profile to point to the consul store and specify this host's advertise ip.

So suppose before modify, the profile on mgr1 looks like this,
```bash
docker@mgr1:~$ cat /var/lib/boot2docker/profile

EXTRA_ARGS='
--label provider=virtualbox

'
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=auto
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
```
, then you will modify the profile so it will look like this,

```bash
docker@mgr1:~$ cat /var/lib/boot2docker/profile

EXTRA_ARGS='
--label provider=virtualbox
--cluster-store=consul://192.168.99.100:8500/network
--cluster-advertise=192.168.99.101:2376
'
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=auto
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
```
Note the added 2 lines in the EXTRA_ARGS,

```bash
--cluster-store=consul://192.168.99.100:8500/network
--cluster-advertise=192.168.99.101:2376
```

Exit the ssh session from mgr1 after changing the profile.

Similarly, modify the profile of host mgr2 (in tab mgr2) and host mgr3 (in tab mgr3), so the profile looks like these:

```bash
docker@mgr2:~$ cat /var/lib/boot2docker/profile

EXTRA_ARGS='
--label provider=virtualbox
--cluster-store=consul://192.168.99.100:8500/network
--cluster-advertise=192.168.99.102:2376
'
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=auto
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
```

, and modify the profile of host mgr3 in tab mgr3, so the

```bash
docker@mgr3:~$ cat /var/lib/boot2docker/profile

EXTRA_ARGS='
--label provider=virtualbox
--cluster-store=consul://192.168.99.100:8500/network
--cluster-advertise=192.168.99.103:2376
'
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=auto
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
```

<a name="Step16">Step 1.6</a>
====

Restart hosts to pick up the changes in previous step. Note __you must perform the host restarts in sequential, one at a time__. Verify after the restart of each host that its assigned ip has not changed

In tab mgr1, execute,

```bash
docker-machine restart mgr1
docker-machine ip mgr1
```
192.168.99.101

```bash
docker-machine restart mgr2
docker-machine ip mgr2
```
192.168.99.102

```bash
docker-machine restart mgr3
docker-machine ip mgr3
```
192.168.99.103

At this point, you can verify that all three mgr hosts has successfully connected to the consul store, by browsing to url,
```http
http://192.168.99.100:8500/ui/#/dc1/nodes/consul
```
See an example of the UI results with 3 connected hosts below,

![Consul UI showing connected hosts]({{ site.url }}/images/consul_ui_1.png)

<a name="Step17">Step 1.7</a>
====

Create an overlay network, named 'net1', all arena hadoop containers created later will communicate over this network.

In tab mgr1, execute,

```bash
docker network create -d overlay --subnet=10.10.10.0/24 net1
```

Verify host mgr can see this network by issuing command in tab mgr2,

```bash
docker network ls
```

Part 2: Deploying containers
------------------------------------

All arena hadoop docker container images are available at [docker hub](hub.docker.io).

<a name="Step21">Step 2.1</a> Create and run cmserver node, and its proxy
====

In tab mgr1, execute,

```bash
docker run -d \
--net net1 \
--privileged=true \
--name cmc1 \
-h cmc1.net1 \
binyuanchen/cmserver:0.1

docker run \
--name proxy1 \
--net net1 \
-p 2222:2222 \
-p 80:80 \
-p 1004:1004 \
-p 2181:2181 \
-p 7180:7180 \
-p 8020:8020 \
-p 8021:8021 \
-p 8042:8042 \
-p 8088:8088 \
-p 8085:8085 \
-p 8888:8888 \
-p 9001:9001 \
-p 9095:9095 \
-p 11000:11000 \
-p 19888:19888 \
-p 10002:10002 \
-p 18088:18088 \
-p 50010:50010 \
-p 50060:50060 \
-p 50030:50030 \
-p 50070:50070 \
-p 50075:50075 \
-p 60000:60000 \
-p 60010:60010 \
-p 60020:60020 \
-p 60030:60030 \
-e PROXIED_SERVER=cmc1 -e OVERLAY_NET=net1 \
-d binyuanchen/cmproxy:0.1
```

> The cmserver node is given a name 'cmc1', its proxy is given a name 'proxy1'.

To (wait and) verify cmserver is up (the cloudera manager running on it is ready), tail the docker log of container cmc1, in tab mgr1,

```bash
docker logs -f cmc1
```

until below message is seen, which indicates cmc1 is ready,

```bash
[16:24:51][DEPLOYER] Waiting for CM to be started on port 7180
[16:24:56][DEPLOYER] Waiting for CM to be started on port 7180
[16:25:01][DEPLOYER] waitcm - DONE
2017-03-29 16:25:01,610 INFO exited: cmlauncher (exit status 0; expected)
```

<a name="Step22">Step 2.2</a> Create and run two cmagent nodes, and their proxies
====

In tab mgr2, execute,

```bash
docker run -d \
--net net1 \
--privileged=true \
--name cmc2 \
-h cmc2.net1 \
binyuanchen/cmagent:0.1

docker run \
--name proxy2 \
--net net1 \
-p 2222:2222 \
-p 80:80 \
-p 1004:1004 \
-p 2181:2181 \
-p 7180:7180 \
-p 8020:8020 \
-p 8021:8021 \
-p 8042:8042 \
-p 8088:8088 \
-p 8085:8085 \
-p 8888:8888 \
-p 9001:9001 \
-p 9095:9095 \
-p 11000:11000 \
-p 19888:19888 \
-p 10002:10002 \
-p 18088:18088 \
-p 50010:50010 \
-p 50060:50060 \
-p 50030:50030 \
-p 50070:50070 \
-p 50075:50075 \
-p 60000:60000 \
-p 60010:60010 \
-p 60020:60020 \
-p 60030:60030 \
-e PROXIED_SERVER=cmc2 -e OVERLAY_NET=net1 \
-d binyuanchen/cmproxy:0.1
```

In tab mgr3, execute,

```bash
docker run -d \
--net net1 \
--privileged=true \
--name cmc3 \
-h cmc3.net1 \
binyuanchen/cmagent:0.1

docker run \
--name proxy3 \
--net net1 \
-p 2222:2222 \
-p 80:80 \
-p 1004:1004 \
-p 2181:2181 \
-p 7180:7180 \
-p 8020:8020 \
-p 8021:8021 \
-p 8042:8042 \
-p 8088:8088 \
-p 8085:8085 \
-p 8888:8888 \
-p 9001:9001 \
-p 9095:9095 \
-p 11000:11000 \
-p 19888:19888 \
-p 10002:10002 \
-p 18088:18088 \
-p 50010:50010 \
-p 50060:50060 \
-p 50030:50030 \
-p 50070:50070 \
-p 50075:50075 \
-p 60000:60000 \
-p 60010:60010 \
-p 60020:60020 \
-p 60030:60030 \
-e PROXIED_SERVER=cmc3 -e OVERLAY_NET=net1 \
-d binyuanchen/cmproxy:0.1
```

At this point, you can verify the connectivity between the docker containers running on different hosts:

First, in tab mgr1, login to container cmc1,

```bash
docker exec -it cmc1 bash
```
, then ssh into container cmc2,

```bash
ssh root@cmc2
```
> The ssh login password is 'root' (excluding the single quote on either end).

If the ssh login succeeds, the communication over the overlay network 'net1' between container cmc1 and cmc2 on two different hosts works.

<a name="Step23">Step 2.3</a> Modify /etc/hosts on your Mac.
====

Adding these entries below to /etc/hosts on your Mac to allow applications run on your Mac can access the arena hadoop containers via their names.

```bash
192.168.99.101	cmc1.net1 cmc1
192.168.99.102  cmc2.net1 cmc2
192.168.99.103  cmc3.net1 cmc3
```

Part 3: pick a cluster config and deploy Hadoop cluster
------------------------------------

Arena-dev-cdh-hadoop provides deploy scripts to create a three-node CDH Hadoop clusters with different configurations. The cluster configurations are specified in config files. Here is a summary of how the cluster will be configured per config file. All config files are under \<ARENA-SRC-ROOT\>/deploy/ directory.


|----------------+----------------------------------+-----------------|
|  Scenario/Config   |  Cluster Spec                    | to deploy,      |
|:---------------:|:---------------------------------|:----------------|
| config_1.json  |Simple AuthN,<br>Zookeeper,<br>HDFS (NN, SN, DN),<br> HBASE(MASTER, RS) | [Follow](#Step31S1)      |
|----------------+----------------------------------+-----------------|
| config_2.json  |Simple AuthN,<br>Zookeeper,<br>HDFS(NN, DN, JN, HA/Nameservice),<br>* Nameservice id: nameservice1<br>* namenode1: active on cmc1<br>* namenode2: standby on cmc2<br>HBASE(Master, RS),<br>Pig,<br>MR1 (mapreduce 0.20),<br>* JobTracler HA, and two FCs | [Follow](#Step31S2)    |
|----------------+----------------------------------+-----------------|
| config_3.json  |Simple AuthN,<br>Zookeeper,<br>HDFS(NN, DN, JN, HA/Nameservice),<br>* Nameservice id: nameservice1<br>* namenode1: active on cmc1<br>* namenode2: standby on cmc2<br>HBASE(Master, RS),<br>Pig,<br>MR1 (mapreduce 0.20),<br>* JobTracler HA, and two FCs <br>Yarn,<br>Spark |  [Follow](#Step31S3)   |
|----------------+----------------------------------+-----------------|

<a name="Step31">Step 3.1</a> deploy cluster using deployer, depending on which config you pick
====

You will deploy your Hadoop cluster by picking any one of below scenarios.

<a name="Step31S1">__Scenario-1__</a> __If you are deploying cluster with config_1.json__

In tab mgr1,

```bash
cd <ARENA-SRC-ROOT>/deploy/
```
, then execute,

```bash
python deployer.py \
--cm_user admin \
--cm_pass admin \
--cm_api_entrypoint cmc1.net1:7180 \
--cluster_name Cluster1 \
--cm_api_version v12 \
--cmserver cmc1.net1 \
--cmagents cmc2.net1,cmc3.net1 \
--ssh_user root \
--ssh_pass root \
--ext_ssh_port 2222 \
--cdh_parcel 5.7.1-1.cdh5.7.1.p0.11 \
--cdh_version CDH5 \
--cdh_full_version 5.7.1 \
--cm_repo_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.7.1" \
--gpg_key_custom_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera" \
--config_file_location ./config_1.json \
--substitution_file_location ./substitution.json \
--app_superuser appadmin
```

<a name="Step31S2">__Scenario-2__</a> __If you are deploying cluster with config_2.json__

In tab mgr1,

```bash
cd <ARENA-SRC-ROOT>/deploy/
```
, then execute,

```bash
python deployer.py \
--cm_user admin \
--cm_pass admin \
--cm_api_entrypoint cmc1.net1:7180 \
--cluster_name Cluster1 \
--cm_api_version v12 \
--cmserver cmc1.net1 \
--cmagents cmc2.net1,cmc3.net1 \
--ssh_user root \
--ssh_pass root \
--ext_ssh_port 2222 \
--cdh_parcel 5.7.1-1.cdh5.7.1.p0.11 \
--cdh_version CDH5 \
--cdh_full_version 5.7.1 \
--cm_repo_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.7.1" \
--gpg_key_custom_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera" \
--config_file_location ./config_2.json \
--substitution_file_location ./substitution.json \
--app_superuser appadmin
```

<a name="Step31S3">__Scenario-3__</a> __If you are deploying cluster with config_3.json__

In tab mgr1,

```bash
cd <ARENA-SRC-ROOT>/deploy/
```
, then execute,

```bash
python deployer.py \
--cm_user admin \
--cm_pass admin \
--cm_api_entrypoint cmc1.net1:7180 \
--cluster_name Cluster1 \
--cm_api_version v12 \
--cmserver cmc1.net1 \
--cmagents cmc2.net1,cmc3.net1 \
--ssh_user root \
--ssh_pass root \
--ext_ssh_port 2222 \
--cdh_parcel 5.7.1-1.cdh5.7.1.p0.11 \
--cdh_version CDH5 \
--cdh_full_version 5.7.1 \
--cm_repo_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.7.1" \
--gpg_key_custom_url "http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/RPM-GPG-KEY-cloudera" \
--config_file_location ./config_3k.json \
--substitution_file_location ./substitution.json \
--app_superuser appadmin
```

<a name="Step32">Step 3.2</a> Login to Cloudera Manager UI (running on cmc1) to check cluster status
====

On your Mac, open a browser and access this URL, login with admin:admin.

```bash
http://cmc1.net1:7180/cmf/login
```

If above steps are completed without issues, you should see the admin UI similar to this (assuming you are deploying with config_3.json):

![CM Admin UI]({{ site.url }}/images/cmui.png)

At this point, your deployment is done and your have the cluster.

Manual changes after cluster deployment
------------------------------

Depending on what your client application is doing against the cluster, you may need to do one (or several) of the below manual changes after cluster deployment.

<a name="manual">MANUAL-1</a> grant everyone full access to /data
====

In tab mgr1,

```bash
docker exec -it cmc1 bash
```
, then execute,

```bash
chmod 777 -R /data/
```

Do the same for mgr2 and mgr3.

Limitations and known issues
------------------------------
[TODO]


[arena-dev-cdh-hadoop-github]: https://github.com/binyuanchen/arena-dev-cdh-hadoop
