FROM nvidia/cuda-ppc64le:9.1-cudnn7-devel-ubuntu16.04
MAINTAINER Yasuhiro Nakajima <e35294@jp.ibm.com>

# install base
RUN apt-get update && DEBIAN_FRONTEND=noninteractive  apt-get install -y \
    wget curl git apt-transport-https apt-utils vim htop tree zip unzip \
    openssh-server iputils-ping net-tools language-pack-ja-base language-pack-ja fonts-ipafont-mincho build-essential openjdk-8-jdk  sudo&& \
    apt-get clean

# install python3 module
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python python3 python3-pip python3-numpy python3-scipy python3-matplotlib python3-dev python3-wheel python3-setuptools python3-skimage python3-google* && \
    apt-get clean

RUN pip3 install protobuf

# upgrade module
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get clean

#setup python3
RUN update-alternatives --install /usr/bin/python python /usr/bin/python2 10
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 11

# 1. ------------ install package caffe required  ------------------ 

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    libprotobuf-dev libleveldb-dev libsnappy-dev libhdf5-serial-dev protobuf-compiler libatlas-base-dev libgflags-dev libgoogle-glog-dev liblmdb-dev libopencv-dev && \
    apt-get clean

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends libboost-all-dev &&\
    apt-get clean
   
# 2. ------------ Building caffe ------------------ 

RUN ln -s /usr/lib/powerpc64le-linux-gnu/libhdf5_serial.so.10.1.0 /usr/lib/powerpc64le-linux-gnu/libhdf5.so
RUN ln -s /usr/lib/powerpc64le-linux-gnu/libboost_python-py35.so /usr/lib/powerpc64le-linux-gnu/libboost_python3.so

RUN cd /opt &&\
    git clone https://github.com/BVLC/caffe.git
    
COPY CMakeLists.txt /opt/caffe
COPY Makefile.config /opt/caffe
COPY Makefile /opt/caffe

RUN cd /opt/caffe &&\
    make all &&\
    make test &&\
    make pycaffe &&\
    make distribute

ENV PYTHONPATH=/opt/caffe/python:$PYTHONPATH

RUN echo "PYTHONPATH=/opt/caffe/python:$PYTHONPATH" >> /root/.bashrc
RUN ln -s /opt/caffe/build/tools/caffe.bin /usr/bin/caffe

 
# 3. ------------ install nccl  ------------------

RUN cd / && \
   git clone https://github.com/NVIDIA/nccl.git && \
   cd /nccl  && \
   make CUDA_HOME=/usr/local/cuda test 
   
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/nccl/build/lib

#     ------------------------ END --------------------------------

# setup ssh
RUN mkdir /var/run/sshd
RUN echo root:$(echo "passw0rd") | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i '/^AcceptEnv/s/^/#/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN echo "export PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH" >> /root/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/nccl/build/lib" >> /root/.bashrc && \
    echo "export PYTHONPATH=/mxnet/python" >> /root/.bashrc

# setup timezone and locale
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN echo "export LANG=ja_JP.UTF-8" >> /root/.bashrc
RUN echo "export LANGUAGE=ja_JP.UTF-8" >> /root/.bashrc
ENV LANGUAGE=ja_JP
ENV LANG=ja_JP.UTF-8

# import python modules
RUN pip3 install IPython==5.3
RUN pip3 install jupyter
RUN mkdir /root/.jupyter
RUN echo "c.NotebookApp.ip = '*'" > ~/.jupyter/jupyter_notebook_config.py


CMD []
