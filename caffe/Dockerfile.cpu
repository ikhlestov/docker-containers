FROM ubuntu:16.04

LABEL maintainer khlyestovillarion@gmail.com

ARG CAFFE_BRANCH=master
ARG CAFFE_COMMIT=04ab089
ENV CAFFE_ROOT=/root/caffe

# install required dependencies
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        build-essential \
        cmake \
        curl \
        gfortran \
        git \
        hdf5-tools \
        libatlas-base-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libgoogle-glog0v5 \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-core-dev \
        libopencv-dev \
        libopencv-highgui-dev \
        libprotobuf-dev \
        libprotoc-dev \
        libsnappy-dev \
        libsnappy-dev \
        libstdc++6-4.8-dbg \
        opencl-headers \
        protobuf-compiler \
        python-dev
    # ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial.so.10.1.0 /usr/lib/x86_64-linux-gnu/libhdf5.so && \
    # ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial_hl.so.10.0.2 /usr/lib/x86_64-linux-gnu/libhdf5_hl.so
RUN apt-get install --no-install-recommends -y libboost-all-dev && \
    rm -rf /var/lib/apt/lists/*

RUN curl -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

WORKDIR $CAFFE_ROOT
RUN git clone -b ${CAFFE_BRANCH} https://github.com/BVLC/caffe.git . && \
    git reset --hard ${CAFFE_COMMIT} && \
    for req in $(cat python/requirements.txt); do pip install $req; done && \
    pip install protobuf==2.6.1 && \
    cp Makefile.config.example Makefile.config && \
    sed -i '/^# CPU_ONLY := 1/s/^# //' Makefile.config && \
    sed -i '/^# WITH_PYTHON_LAYER := 1/s/^# //' Makefile.config && \
    sed -i 's+/usr/lib/python2.7/dist-packages/numpy/core/include+/usr/local/lib/python2.7/dist-packages/numpy/core/include+g' Makefile.config && \
    sed -i 's+/usr/local/include+/usr/local/include /usr/include/hdf5/serial+g' Makefile.config && \
    sed -i 's+/usr/local/lib /usr/lib+/usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial+g' Makefile.config && \
    make -j"$(nproc)" all py && \
    make pycaffe && \
    make -j"$(nproc)" test && \
    make runtest && \
    make distribute

# Post setup
ENV PYCAFFE_ROOT=$CAFFE_ROOT/python
ENV PYTHONPATH=$PYCAFFE_ROOT:$PYTHONPATH
ENV PATH=$CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

CMD ["/bin/bash"]


