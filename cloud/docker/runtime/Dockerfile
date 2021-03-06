# Container image #1: wcm-runtime.
# This Dockerfile builds the runtime environment for the whole cell model.
#
# To build this image locally from the wcEcoli/ project root directory:
#
#     > docker build -f cloud/docker/runtime/Dockerfile -t wcm-runtime .
#
# Add option `--build-arg from=ABC` to build from a different base image but
# DO NOT USE an alpine base since the simulation math comes out different!
ARG from=python:2.7.16
FROM ${from}

RUN apt-get update \
    && apt-get install -y swig python-cvxopt \
        gfortran llvm cmake ncurses-dev libreadline7 libreadline-dev nano

RUN echo "alias ls='ls --color=auto'" >> ~/.bashrc

# Install openblas v0.3.5 (not the buggy v0.3.3 nor v0.3.6).
# Future: `apt-get install libopenblas-dev` but currently that gets v0.2.19.
RUN (mkdir -p openblas && cd openblas \
    && curl -SL https://github.com/xianyi/OpenBLAS/archive/v0.3.5.tar.gz | tar -xz \
    && cd OpenBLAS* \
    && make FC=gfortran \
    && make PREFIX=/usr install) \
    && rm -r openblas

# Install glpk into /usr/local/lib/.
RUN (mkdir -p glpk && cd glpk \
    && curl -SL ftp://ftp.gnu.org/gnu/glpk/glpk-4.65.tar.gz | tar -xz \
    && cd glpk-4.65 \
    && ./configure && make install) \
    && rm -r glpk

ENV CVXOPT_BUILD_GLPK=1 \
    CVXOPT_GLPK_LIB_DIR=/usr/local/lib \
    CVXOPT_GLPK_INC_DIR=/usr/local/include \
    CVXOPT_BLAS_LIB=openblas \
    CVXOPT_LAPACK_LIB=openblas \
    OPENBLAS_NUM_THREADS=1

# Install all the pips within one Docker layer and don't cache the downloads.
COPY requirements.txt /
RUN pip install --no-cache-dir 'numpy==1.14.6' --no-binary numpy \
    && pip install --no-cache-dir -r requirements.txt --no-binary numpy,scipy,cvxopt

CMD ["/bin/bash"]
