FROM registry.access.redhat.com/ubi8/ubi:8.5

ENV HOME=/var/lib/mysql
EXPOSE 3306 4444 4567 4567/udp 4568

COPY mariadb-cluster.sh /mariadb-cluster.sh
COPY readiness-probe.sh /readiness-probe.sh
COPY liveness-probe.sh /liveness-probe.sh
COPY MariaDB.repo /etc/yum.repos.d/MariaDB.repo
RUN dnf -y --enablerepo=ubi-8-baseos --enablerepo=ubi-8-appstream --enablerepo=ubi-8-codeready-builder --nodocs install MariaDB-server galera-4 bind-utils hostname rsync sudo shadow-utils && \
    dnf -y update && dnf -y clean all && \
    groupadd -g 27 mysql && \
    useradd -u 27 -g mysql -d /var/lib/mysql -s /sbin/nologin mysql && \
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
