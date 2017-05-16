FROM ubuntu:xenial

MAINTAINER Ken Sullivan <sullivak@gmail.com>

# Sloth labeler

# for GUI:
# xhost +local:docker
# docker run -it -v /tmp/.X11-unix/:/tmp/.X11-unix/ -e DISPLAY=$DISPLAY --rm sullivak/sloth
# export QT_X11_NO_MITSHM=1

RUN apt-get update && apt-get install -y --no-install-recommends \
  git \
  python-dev \
  python-pip \
  python-numpy \
  python-qt4 \   
  && \
  apt-get clean && \
  apt-get autoremove


RUN pip install --upgrade pip
RUN pip install setuptools
RUN pip --no-cache-dir --timeout=60 install \
  Pillow

WORKDIR /opt
RUN git clone https://github.com/cvhciKIT/sloth.git
WORKDIR /opt/sloth
RUN python setup.py install

ENV QT_X11_NO_MITSHM=1

