FROM ubuntu:18.04

# Install cmake
RUN apt-get update && apt-get install -y wget make build-essential libcurl4-gnutls-dev zlib1g-dev
RUN wget --no-check-certificate https://github.com/Kitware/CMake/releases/download/v3.14.4/cmake-3.14.4.tar.gz && tar xvzf cmake-3.14.4.tar.gz
RUN cd cmake-3.14.4 && ./bootstrap --system-curl && make && make install

# Install opencv
RUN apt-get update && apt-get install -y git wget unzip yasm pkg-config libswscale-dev libtbb2 libtbb-dev \
    libjpeg-dev libpng-dev libtiff-dev libavformat-dev libpq-dev
RUN apt-get install -y python3-pip && pip3 install -U pip && pip3 install numpy

WORKDIR /
ENV OPENCV_VERSION="4.1.1"
RUN wget https://github.com/opencv/opencv/archive/${OPENCV_VERSION}.zip && unzip ${OPENCV_VERSION}.zip \
    && wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/${OPENCV_VERSION}.zip \
    && unzip opencv_contrib.zip \
    && mkdir /opencv-${OPENCV_VERSION}/cmake_binary && cd /opencv-${OPENCV_VERSION}/cmake_binary \
    && cmake -DBUILD_TIFF=ON \
      -DBUILD_opencv_java=OFF \
      -DWITH_CUDA=OFF \
      -DWITH_OPENGL=ON \
      -DWITH_OPENCL=ON \
      -DWITH_IPP=ON \
      -DWITH_TBB=ON \
      -DWITH_EIGEN=ON \
      -DWITH_V4L=ON \
      -DBUILD_TESTS=OFF \
      -DBUILD_PERF_TESTS=OFF \
      -DBUILD_opencv_xfeatures2d=OFF \
      -DCMAKE_BUILD_TYPE=RELEASE \
      -DOPENCV_EXTRA_MODULES_PATH=/opencv_contrib-${OPENCV_VERSION}/modules \
      -DCMAKE_INSTALL_PREFIX=$(python3 -c "import sys; print(sys.prefix)") \
      -DPYTHON_EXECUTABLE=$(which python3) \
      -DPYTHON_INCLUDE_DIR=$(python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())") \
      -DPYTHON_PACKAGES_PATH=$(python3 -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())") \
      .. \
    && make install \
    && rm /${OPENCV_VERSION}.zip /opencv_contrib.zip \
    && rm -rf /opencv-${OPENCV_VERSION} /opencv_contrib-${OPENCV_VERSION}
RUN ln -s /usr/lib/python3.6/dist-packages/cv2/python-3.6/cv2.cpython-36m-aarch64-linux-gnu.so /usr/lib/python3.6/dist-packages/cv2/python-3.6/cv2.so

# Install dldt
RUN apt-get install -y --no-install-recommends build-essential cmake curl wget libssl-dev ca-certificates git libboost-regex-dev libgtk2.0-dev \
    pkg-config unzip automake libtool autoconf libcairo2-dev libpango1.0-dev libglib2.0-dev libgtk2.0-dev libswscale-dev \
    libavcodec-dev libavformat-dev libgstreamer1.0-0 gstreamer1.0-plugins-base libusb-1.0-0-dev libopenblas-dev libpng-dev
WORKDIR /
RUN git clone -b 2020 https://github.com/opencv/dldt.git && cd /dldt/inference-engine && git submodule init && git submodule update --recursive
RUN sed -i -- "s/return std::move(ret);/return ret;/g" /dldt/inference-engine/thirdparty/ade/sources/ade/source/execution_engine.cpp
ENV OpenCV_DIR=/usr/lib
RUN pip3 install cython
RUN cd /dldt && mkdir build && cd build && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DENABLE_MKL_DNN=OFF \
    -DENABLE_CLDNN=OFF \
    -DENABLE_GNA=OFF \
    -DENABLE_SSE42=OFF \
    -DTHREADING=SEQ \
    -DENABLE_PYTHON=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python3.6 \
    -DPYTHON_LIBRARY=/usr/lib/aarch64-linux-gnu/libpython3.6m.so \
    -DPYTHON_INCLUDE_DIR=/usr/include/python3.6 .. && make -j4 && make install && cd .. && rm -r build

ENV PYTHONPATH=/dldt/bin/aarch64/Release/lib/python_api/python3.6

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp* /root/.cache/pip/wheels/*
