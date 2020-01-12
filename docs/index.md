### MariaDB Galera Cluster - K8s StatefulSet

By the end of this tutorial you will have configured and deployed a three-node MariaDB Galera cluster into OpenShift OKD.

Galera is a multi-master clustering technology that works with MySQL and MariaDB.  We are going to be using Galera 4 and MariaDB 10.4.

Get more specific information here: [Galera Home](https://galeracluster.com), [MariaDB Home](https://mariadb.com)

Prerequisits:

* Access to an OKD 3.11 cluster with admin rights
* [Docker Desktop](https://www.docker.com/products/docker-desktop), or docker daemon running on a Linux distribution
* This GitHub repository: `git clone https://github.com/cgruver/mariadb-galera-okd.git`
* You need to be comfortable working from a command line.

This tutorial consists of three parts which you should follow in order:

1. [The Containerized MariaDB Image](pages/ContainerBuild.md)
1. [Deploying To OpenShift](pages/OpenShiftDeploy.md)
1. [Using and Managing the Database](pages/UsingTheDatabase.md)
