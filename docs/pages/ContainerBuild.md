### Building the MariaDB Galera Cluster container image

We are going to use docker to build this image.  Then we are going to push it to the image registry of our OpenShift cluster.

If you haven't yet cloned this git repository, we'll do that now.  Then,from the root of this git repository, we'll change directory into the ContainerBuild directory.  Let's examine the files there.

    git clone https://github.com/cgruver/mariadb-galera-okd.git
    cd mariadb-galera-okd/ContainerBuild
    ls -l

    total 40
    -rw-r--r--  1 user  staff  1231 Jan 12 12:33 Dockerfile
    -rw-r--r--  1 user  staff    98 Jan 12 12:30 MariaDB.repo
    -rw-r--r--  1 user  staff    87 Jan 12 12:26 liveness-probe.sh
    -rw-r--r--  1 user  staff  1999 Jan 12 12:37 mariadb-cluster.sh
    -rw-r--r--  1 user  staff    98 Jan 12 12:26 readiness-probe.sh

* `Dockerfile`: This file contains the directions for building our container.
* `MariaDB.repo`: This file is the definition for the MariaDB 10.4 RPM repository.  Since our container is based on a CentOS 7 base image, our package installer is RPM based.
* `liveness-probe.sh`: The script called by the OpenShift runtime to check pod liveness. More on this file and the next one later.  The liveness probe and readiness probe are used by OpenShift to monitor the state of the running container.
* `readiness-probe.sh`: The script called by the OpenShift runtime to check pod readiness.
* `mariadb-cluster.sh`: This is the main script that configures and starts MariaDB.  It is set as the entry point for the container image.

Let's examine the `Dockerfile`:

    FROM centos:7

    ENV HOME=/var/lib/mysql
    EXPOSE 3306 4444 4567 4567/udp 4568

    COPY mariadb-cluster.sh /mariadb-cluster.sh
    COPY readiness-probe.sh /readiness-probe.sh
    COPY liveness-probe.sh /liveness-probe.sh
    COPY MariaDB.repo /etc/yum.repos.d/MariaDB.repo
    RUN groupadd -g 27 mysql && \
       useradd -u 27 -g mysql -d /var/lib/mysql -s /sbin/nologin mysql && \
       yum -y install --setopt=tsflags=nodocs MariaDB-server galera-4 bind-utils hostname rsync sudo && \
       yum -y update && yum clean all && \
       chown -R mysql.0 /var/lib/mysql && \
       chown -R mysql.0 /etc/my.cnf.d && \
       echo 'mysql ALL=(ALL) NOPASSWD: /usr/bin/mkdir -p /var/lib/mysql/data, /usr/bin/chown mysql.0 /var/lib/mysql/data, /usr/bin/mysql_install_db, /usr/sbin/mysqld, /usr/bin/mysql, /usr/bin/galera_recovery, /usr/bin/mysqladmin' >> /etc/sudoers && \
       chown mysql.0 /mariadb-cluster.sh && \
       chown mysql.0 /readiness-probe.sh && \
       chown mysql.0 /liveness-probe.sh && \
       chmod +x /mariadb-cluster.sh && \
       chmod +x /readiness-probe.sh && \
       chmod +x /liveness-probe.sh

    VOLUME /var/lib/mysql
    # NOTE: By default this pod will run as random user on openshift unless a security context is set.
    USER 27

    ENTRYPOINT ["/mariadb-cluster.sh"]

We are using several docker directives here; `FROM`, `ENV`, `EXPOSE`, `COPY`, `RUN`, `VOLUME`, `USER`, and `ENDPOINT`.

For detailed descriptions, please visit the [Docker Documentation](https://docs.docker.com/engine/reference/builder/).

`FROM` tells us that we are starting with a base CentOS version 7 image.  This is the official CentOS image hosted on Docker Hub.  Find it [here](https://hub.docker.com/_/centos).

`ENV` sets runtime environment variables.

`EXPOSE` opens ports that our container will need for communication.

`COPY` does what it says.  It copies files from the build system into the container image.

`RUN` executes commands during the build.  In this case, we are creating the user and group for mariadb, installing the MariaDB and Galera software, setting file ownership and permissions, and setting up sudo rules for the mysql user.

`VOLUME` indicates the mount point for an external volume that will be needed at runtime.

`USER` indicates the uid that we want this container to run as at runtime.

`ENTRYPOINT` is the command that we want to execute when this container is started.

Let's talk a bit more about the `USER` directive.  By default OpenShift will ignore this directive.  It is a security consideration that OpenShift always runs containers as non-root users.  In fact, OpenShift will assign an arbitrary UID to the container when it starts.  For ephemeral images like a Spring Boot application.  This really does not matter.  However, since we are deploying an application that is going to be storing data on a mounted volume, we need the application to be able to access that data after a restart or even a re-deployment of the container image.

To that end, we are going to leverage an OpenShift security context constraint combined with a service account so that our MariaDB database will always run as UID 27.  This will allow us to still access our data across container restarts and re-deployments/upgrades without having to set 777 permissions on the data.

Now, let's build this image and push it to the OpenShift repository.  You need to know the URL of your OpenShift cluster image registry.  It will be something like:

    docker-registry-default.apps.your.cluster.domain.com

Where `apps.your.cluster.domain.com` is the wild-card DNS entry for your OpenShift cluster.

1. Make sure that your Docker daemon is running.

    On CentOS:

        systemctl start docker

    On a desktop OS, start `Docker Desktop`.

1. Log into your image registry: (Assuming that you are already logged into your OpenShift cluster)

        docker login -p $(oc whoami -t) -u admin docker-registry-default.apps.your.cluster.domain.com

1. Build the image:

        docker build -t docker-registry-default.apps.your.cluster.domain.com/openshift/mariadb-galera:10.4 .

1. Push the image to the OpenShift image registry.

        docker push docker-registry-default.apps.your.cluster.domain.com/openshift/mariadb-galera:10.4

### That's It!  We just built a new container image and pushed it into the image registry.

Now, let's deploy a database cluster:

[OpenShift Deployment](OpenShiftDeploy.md)
