ARG BASE_TARGET=base
FROM nvidia/cuda:10.2-devel-ubuntu18.04 as base

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get clean && apt-get update
RUN apt-get install -y curl locales git-all autoconf automake make cmake wget unzip

RUN locale-gen en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# Install openssl
FROM base as openssl
ADD ./common/install_openssl.sh install_openssl.sh
RUN bash ./install_openssl.sh && rm install_openssl.sh

# Install python
FROM base as python
ADD common/install_cpython.sh install_cpython.sh
RUN apt-get update -y && \
    apt-get install build-essential gdb lcov libbz2-dev libffi-dev \
        libgdbm-dev liblzma-dev libncurses5-dev libreadline6-dev \
        libsqlite3-dev libssl-dev lzma lzma-dev tk-dev uuid-dev zlib1g-dev -y && \
    bash ./install_cpython.sh && \
    rm install_cpython.sh && \
    apt-get clean

FROM base as intel
# Install MKL
ADD ./common/install_mkl.sh install_mkl.sh
RUN bash ./install_mkl.sh && rm install_mkl.sh

FROM base as cuda
ADD ./common/install_cuda.sh install_cuda.sh
ADD ./common/install_magma.sh install_magma.sh
ENV CUDA_HOME /usr/local/cuda

FROM base as conda
ADD ./common/install_conda.sh install_conda.sh
RUN bash ./install_conda.sh && rm install_conda.sh
RUN /opt/conda/bin/conda install -y cmake=3.14

FROM cuda as cuda9.2
RUN bash ./install_cuda.sh 9.2
RUN bash ./install_magma.sh 9.2

FROM cuda as cuda10.1
RUN bash ./install_cuda.sh 10.1
RUN bash ./install_magma.sh 10.1

FROM cuda as cuda10.2
RUN bash ./install_cuda.sh 10.2
RUN bash ./install_magma.sh 10.2

FROM cuda as cuda11.0
RUN bash ./install_cuda.sh 11.0
RUN bash ./install_magma.sh 11.0

FROM cuda as cuda11.1
RUN bash ./install_cuda.sh 11.1
RUN bash ./install_magma.sh 11.1

FROM cuda as cuda11.2
RUN bash ./install_cuda.sh 11.2
RUN bash ./install_magma.sh 11.2

FROM cuda as cuda11.3
RUN bash ./install_cuda.sh 11.3
RUN bash ./install_magma.sh 11.3

FROM ${BASE_TARGET} as final
# Install LLVM
COPY --from=pytorch/llvm:9.0.1 /opt/llvm              /opt/llvm
COPY --from=pytorch/llvm:9.0.1 /opt/llvm_no_cxx11_abi /opt/llvm_no_cxx11_abi
COPY --from=openssl            /opt/openssl           /opt/openssl
# Install patchelf
ADD ./common/install_patchelf.sh install_patchelf.sh
RUN bash ./install_patchelf.sh && rm install_patchelf.sh
COPY --from=intel /opt/intel /opt/intel
# Install Anaconda
COPY --from=conda /opt/conda /opt/conda
# Install python
COPY --from=python /opt/python    /opt/python
COPY --from=python /opt/_internal /opt/_internal
ENV PATH /opt/conda/bin:$PATH
