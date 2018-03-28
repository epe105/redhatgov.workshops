FROM docker.io/library/fedora:27

USER root

RUN dnf install -y \
  python \
  python-pip \
  libselinux-python \
  git

WORKDIR /opt/openshift_for_ops
COPY requirements.txt /opt/openshift_for_ops/requirements.txt

RUN pip install --upgrade pip \
  && pip install -r requirements.txt

VOLUME /opt/openshift_for_ops
