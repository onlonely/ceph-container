# CEPH DAEMON IMAGE
# CEPH VERSION: Luminous
# CEPH VERSION DETAIL: 12.x.x

FROM ubuntu:16.04
MAINTAINER Sébastien Han "seb@redhat.com"

ENV CEPH_VERSION luminous
ENV CONFD_VERSION 0.10.0
ENV KUBECTL_VERSION v1.6.0

# Packages list
ARG PACKAGES="ceph-mon ceph-osd ceph-mds ceph-mgr ceph-base ceph-common radosgw rbd-mirror sharutils etcd s3cmd nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw lvm2"
ARG PURGES="/var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/lib/{dracut,locale,systemd,udev} /usr/bin/hyperkube /usr/bin/etcd /usr/bin/systemd-analyze /etc/{udev,selinux} /usr/lib/{udev,systemd}"
# Add s3cfg file
ADD s3cfg /root/.s3cfg

# Add bootstrap script, ceph defaults key/values for KV store
ADD *.sh ceph.defaults check_zombie_mons.py ./osd_scenarios/* entrypoint.sh.in disabled_scenario /

# install prerequisites
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y wget unzip uuid-runtime python-setuptools udev dmsetup && \
  # Install ceph, ganesha and etcd
  wget -q -O- 'https://download.ceph.com/keys/release.asc' | apt-key add - && \
  echo "deb http://download.ceph.com/debian-$CEPH_VERSION/ xenial main" | tee /etc/apt/sources.list.d/ceph-$CEPH_VERSION.list && \
  echo "deb http://download.ceph.com/nfs-ganesha/deb-V2.5-stable/luminous/ xenial main" | tee /etc/apt/sources.list.d/nfs-ganesha.list && \
  apt-get update && apt-get install -y  --no-install-recommends --force-yes $PACKAGES && \
  dpkg -s $PACKAGES && \
  apt-get clean && rm -rf $PURGES && \
  # Download & install confd
  wget -O /usr/local/bin/confd "https://github.com/kelseyhightower/confd/releases/download/v${CONFD_VERSION}/confd-${CONFD_VERSION}-linux-amd64" && \
  chmod +x /usr/local/bin/confd && mkdir -p /etc/confd/conf.d && mkdir -p /etc/confd/templates && \
  # Install forego
  wget -O /forego.tgz 'https://bin.equinox.io/c/ekMN3bCZFUn/forego-stable-linux-amd64.tgz' && \
  cd /usr/local/bin && tar xfz /forego.tgz && chmod +x /usr/local/bin/forego && rm /forego.tgz && \
  # Install kubectl
  wget -O /usr/local/bin/kubectl "https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
  chmod +x /usr/local/bin/kubectl && \
  # Cleaning container
  bash /clean_container.sh && rm /clean_container.sh

# Modify the entrypoint
RUN bash "/generate_entrypoint.sh" && \
  rm -f /generate_entrypoint.sh && \
  bash -n /*.sh

# Add templates for confd
ADD ./confd/templates/* /etc/confd/templates/
ADD ./confd/conf.d/* /etc/confd/conf.d/

# Add volumes for Ceph config and data
VOLUME ["/etc/ceph","/var/lib/ceph", "/etc/ganesha"]

# Execute the entrypoint
WORKDIR /
ENTRYPOINT ["/entrypoint.sh"]
