FROM tensorflow/tensorflow:1.0.1-gpu

MAINTAINER Ken Sullivan <ken.sullivan@kohyoung.com>

# Omni container with:
# ML/DL frameworks (TensorFlow, keras, caffe, MXNet)
# Biomedical and 3d image loading and visualization (nibabel, openslide, Mayavi, etc.)
# OpenCV
# PyCharm
# Possible future additions: Neon, CNTK, Torch, Octave
# cobbled together from:
# https://hub.docker.com/r/kongkoro/pycharm/~/dockerfile/
# https://github.com/BVLC/caffe/tree/master/docker
# https://github.com/tensorflow/tensorflow/blob/master/tensorflow/tools/docker/Dockerfile.gpu
# https://hub.docker.com/r/floydhub/dl-docker/~/dockerfile/
# https://github.com/chipgarner/opencv3-webcam-docker/blob/master/Dockerfile
# https://hub.docker.com/r/kaixhin/mxnet/~/dockerfile/
# And various relevant installation documents

# for PyCharm (and other GUI):
# xhost +local:docker
# docker run -it -v /tmp/.X11-unix/:/tmp/.X11-unix/ -e DISPLAY=$DISPLAY -v ~/.PyCharm2016.2:/root/.PyCharm2016.2
# --rm kyra/omni

# NOTE: currently opencv and opencv_contrib using specific bug-fix commits past 3.2.0
ARG OPENCV_VERSION=3.2.0
ARG KERAS_VERSION=1.2.0
ARG CAFFE_CLONE_TAG=rc4
ENV KERAS_BACKEND=tensorflow
ENV PYCHARM_HOME=/opt/pycharm
ENV CAFFE_ROOT=/opt/caffe
ENV OPENCV_ROOT=/opt/opencv
ENV MXNET_ROOT=/opt/mxnet

RUN apt-get update && apt-get install -y --no-install-recommends \
  libtiff5=4.0.6-1 \
  libtiffxx5=4.0.6-1 \
  libtiff5-dev=4.0.6-1 \
  build-essential \
  checkinstall \
  cmake \
  gfortran \
  git \
  graphviz \
  libavcodec-dev \
  libavformat-dev \
  libboost-all-dev \
  libdc1394-22-dev \
  libeigen3-dev \
  libgflags-dev \
  libgoogle-glog-dev \
  libgraphviz-dev \
  libgstreamer0.10-dev \
  libgstreamer-plugins-base0.10-dev \
  libgtk2.0-dev \
  libhdf5-serial-dev \
  libjasper-dev \
  libjpeg-dev \
  liblapack-dev \
  liblapacke-dev \
  libleveldb-dev \
  liblmdb-dev \
  libmp3lame-dev \
  libopenblas-dev \
  libopencore-amrnb-dev \
  libopencore-amrwb-dev \
  libprotobuf-dev \
  libqt4-dev \
  libsnappy-dev \
  libswscale-dev \
  libtbb-dev \
  libtheora-dev \
  libv4l-dev \
  libvorbis-dev \
  libxrender1 \
  libxtst6 \
  libxvidcore-dev \
  openslide-tools \
  protobuf-compiler \
  pkg-config \
  python-dev \
  python-h5py \
  python-numpy \
  python-tk \
  python-vtk \
  python-yaml \
  python-pycurl \
  qt4-default \
  unzip \
  v4l-utils \
  wget \
  x264 \
  yasm \
  && \
  apt-get clean && \
  apt-get autoremove && \
  rm -rf /var/lib/apt/lists/*

# Install opencv and opencv_contrib
# Checkout specified versions
# NOTE: Ran into https://github.com/opencv/opencv/issues/7970, fixed in https://github.com/opencv/opencv/pull/7999
# opencv Fix commit: 3668a01fca0858c32b
# NOTE: Ran into https://github.com/opencv/opencv/issues/7949 aka https://github.com/opencv/opencv_contrib/issues/919
# opencv_contrib Fix commit abf44fcccfe2f281b
WORKDIR $OPENCV_ROOT
# Ideally would: RUN git clone -b ${OPENCV_VERSION} --single-branch --depth 1 https://github.com/opencv/opencv.git && \
RUN git clone https://github.com/opencv/opencv.git && cd opencv && git checkout 3668a01fca0858c32b
WORKDIR $OPENCV_ROOT
RUN git clone https://github.com/opencv/opencv_contrib.git && cd opencv_contrib && git checkout abf44fcccfe2f281b
WORKDIR $OPENCV_ROOT/opencv/build
# build, install, and cleanup in one RUN command/layer to avoid gigantic image
RUN cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D WITH_TBB=ON \
    -D BUILD_NEW_PYTHON_SUPPORT=ON \
    -D WITH_V4L=ON \
    -D BUILD_TIFF=ON \
    -D WITH_QT=4 \
    -D OPENCV_EXTRA_MODULES_PATH=$OPENCV_ROOT/opencv_contrib/modules \
    -D WITH_OPENGL=ON .. && \
  make -j4 && \
  make install && \
  echo "/usr/local/lib" | tee -a /etc/ld.so.conf.d/opencv.conf && \
  ldconfig && \
  cp $OPENCV_ROOT/opencv/build/lib/cv2.so /usr/lib/python2.7/dist-packages/cv2.so && \
  rm -rf $OPENCV_ROOT/opencv/build && \
  rm -rf $OPENCV_ROOT/opencv/.git && \
  rm -rf $OPENCV_ROOT/opencv_contrib/.git

# Install keras
RUN pip --no-cache-dir install --no-dependencies git+git://github.com/fchollet/keras.git@${KERAS_VERSION}

# Install other pip
RUN pip --no-cache-dir --timeout=60 install \
    graphviz \
    line_profiler \
    mayavi \
    nibabel \
    openslide-python \
    Pillow \
    pydicom \
    pyside \
    scikit-image \
    SimpleITK \
    sklearn \
    theano \
    && \
    python -m ipykernel.kernelspec

# Install caffe
WORKDIR $CAFFE_ROOT
RUN git clone -b ${CAFFE_CLONE_TAG} --depth 1 https://github.com/BVLC/caffe.git . && \
    pip install --upgrade pip && \
    cd python && for req in $(cat requirements.txt) pydot; do pip install $req; done && cd .. && \
    git clone https://github.com/NVIDIA/nccl.git && cd nccl && make -j install && cd .. && rm -rf nccl && \
    mkdir build && cd build && \
    cmake -DUSE_CUDNN=1 -DUSE_NCCL=1 -DBLAS=open .. && \
    make -j"$(nproc)" && \
    rm -rf $CAFFE_ROOT/.git

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

# MXNet install: merging kaixhan's Dockerfile, mxnet's github Dockerfiles,
# mxnet/setup-utils/install-mxnet-ubuntu-r.sh and documentation
# Clone MXNet repo and move into it
WORKDIR $MXNET_ROOT
RUN git clone --recursive https://github.com/dmlc/mxnet . && \
  # Copy config.mk
  cp make/config.mk config.mk && \
  # Switch from atlas to OpenBLAS
  sed -i 's/USE_BLAS = atlas/USE_BLAS = openblas/g' config.mk && \
  # Set up GPU
  echo "USE_CUDA=1" >>config.mk && \
  echo "USE_CUDA_PATH=/usr/local/cuda" >>config.mk && \
  echo "USE_CUDNN=1" >>config.mk && \
  make -j"$(nproc)" && \
  cd $MXNET_ROOT/python && python setup.py install && \
  echo "export PYTHONPATH=$MXNET_ROOT/python:$PYTHONPATH" >> /root/.bashrc && \
  rm -rf $MXNET_ROOT/build && \
  rm -rf $MXNET_ROOT/.git

# Install pycharm
RUN wget https://download.jetbrains.com/python/pycharm-community-2017.2.2.tar.gz -O /tmp/pycharm.tar.gz &&\
    mkdir -p ${PYCHARM_HOME} && tar -xzvf /tmp/pycharm.tar.gz -C ${PYCHARM_HOME} --strip=1 &&\
    ln -s ${PYCHARM_HOME}/bin/pycharm.sh /bin/pycharm &&\
    rm -rf /tmp/pycharm.tar.gz

# Convenience scripts
WORKDIR /root
COPY pycharm_quiet.sh .
COPY qopen.sh .
RUN chmod +x pycharm_quiet.sh
RUN chmod +x qopen.sh
