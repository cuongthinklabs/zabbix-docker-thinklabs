# syntax=docker/dockerfile:1
ARG MAJOR_VERSION=6.2
ARG RELEASE=3
ARG ZBX_VERSION=${MAJOR_VERSION}.3
ARG BUILD_BASE_IMAGE=zabbix-build-sqlite3:rhel-${ZBX_VERSION}

FROM ${BUILD_BASE_IMAGE} as builder

FROM registry.access.redhat.com/ubi8/ubi-minimal

ARG MAJOR_VERSION
ARG RELEASE
ARG ZBX_VERSION
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES} \
    MIBDIRS=/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL

LABEL description="Zabbix proxy with SQLite3 database support" \
      maintainer="alexey.pustovalov@zabbix.com" \
      name="zabbix/zabbix-proxy-sqlite-62" \
      release="${RELEASE}" \
      run="docker run --name zabbix-proxy -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-proxy-sqlite-62:${ZBX_VERSION}" \
      summary="Zabbix proxy (SQLite3)" \
      url="https://www.zabbix.com/" \
      vendor="Zabbix LLC" \
      version="${MAJOR_VERSION}" \
      io.k8s.description="Zabbix proxy with SQLite3 database support" \
      io.k8s.display-name="Zabbix proxy (SQLite3)" \
      io.openshift.expose-services="10051:10051" \
      io.openshift.tags="zabbix,zabbix-proxy,sqlite" \
      org.label-schema.build-date="${BUILD_DATE}" \
      org.label-schema.description="Zabbix proxy with SQLite3 database support" \
      org.label-schema.docker.cmd="docker run --name zabbix-proxy -p 10051:10051 -d registry.connect.redhat.com/zabbix/zabbix-proxy-sqlite3-62:${ZBX_VERSION}" \
      org.label-schema.license="GPL v2.0" \
      org.label-schema.name="zabbix-proxy-sqlite-rhel" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.url="https://zabbix.com/" \
      org.label-schema.usage="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.label-schema.vcs-ref="${VCS_REF}" \
      org.label-schema.vcs-url="${ZBX_SOURCES}" \
      org.label-schema.vendor="Zabbix LLC" \
      org.label-schema.version="${ZBX_VERSION}"

STOPSIGNAL SIGTERM

COPY ["licenses", "/licenses"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}/src/zabbix_proxy/zabbix_proxy", "/usr/sbin/zabbix_proxy"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}/src/zabbix_get/zabbix_get", "/usr/bin/zabbix_get"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}/src/zabbix_sender/zabbix_sender", "/usr/bin/zabbix_sender"]
COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}/conf/zabbix_proxy.conf", "/etc/zabbix/zabbix_proxy.conf"]

RUN set -eux && \
    INSTALL_PKGS="bash \
            tini \
            traceroute \
            nmap \
            shadow-utils \
            fping \
            iputils \
            libcurl \
            libevent \
            openldap \
            libssh \
            libxml2 \
            net-snmp-agent-libs \
            OpenIPMI-libs \
            pcre2 \
            sqlite-libs \
            unixODBC" && \
    curl -sSL -o /tmp/epel-release-latest-8.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm && \
    rpm -ivh /tmp/epel-release-latest-8.noarch.rpm && \
    rm -rf /tmp/epel-release-latest-8.noarch.rpm && \
    microdnf -y install \
            --disablerepo "*" \
            --enablerepo "ubi-8-baseos" \
            --enablerepo "ubi-8-appstream" \
            --enablerepo "rhel-8-for-x86_64-baseos-rpms" \
            --enablerepo "epel" \
            --setopt=install_weak_deps=0 \
            --best \
            --setopt=tsflags=nodocs \
        ${INSTALL_PKGS} && \
    microdnf -y reinstall \
            --enablerepo "ubi-8-baseos" \
            --setopt=install_weak_deps=0 \
            --best \
            --setopt=tsflags=nodocs \
        tzdata && \
    groupadd \
            --system \
            --gid 1995 \
        zabbix && \
    useradd \
            --system \
            --comment "Zabbix monitoring system" \
            -g zabbix \
            -G root \
            --uid 1997 \
            --shell /sbin/nologin \
            --home-dir /var/lib/zabbix/ \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /var/lib/zabbix/db_data && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    chown --quiet -R zabbix:root /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
    microdnf -y clean all && \
    rm -rf /var/cache/yum /var/lib/yum/yumdb/* /usr/lib/udev/hwdb.d/* && \
    rm -rf /var/cache/dnf /etc/udev/hwdb.bin /root/.pki

EXPOSE 10051/TCP

WORKDIR /var/lib/zabbix

VOLUME ["/var/lib/zabbix/snmptraps"]

COPY ["docker-entrypoint.sh", "/usr/bin/"]

ENTRYPOINT ["/usr/bin/tini", "--", "/usr/bin/docker-entrypoint.sh"]

USER 1997

CMD ["/usr/sbin/zabbix_proxy", "--foreground", "-c", "/etc/zabbix/zabbix_proxy.conf"]
