FROM centos:latest
MAINTAINER Michael Kavulich <kavulich@ucar.edu>
#
# This Dockerfile compiles GSI from source during "docker build" step
ENV GSI_VERSION 3.7
ENV ENKF_VERSION 1.3

# Make with this many parallel tasks
ENV J 4

# Get basic system tools
RUN yum -y update
RUN yum -y install file ksh hostname m4 make perl tar tcsh time wget vim emacs nano centos-release-scl
# Get devtoolset-7
RUN yum-config-manager --enable rhel-server-rhscl-7-rpms \
 && yum -y install devtoolset-7 
#

# Build Cmake v3 (centos default is v2.8, which is too old)


# Build OpenMPI
RUN mkdir -p /gsi/libs/openmpi/BUILD_DIR
RUN source /opt/rh/devtoolset-7/enable \
 && cd /gsi/libs/openmpi/BUILD_DIR \
 && curl -L -O https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.0.tar.gz \
 && tar -xf openmpi-4.0.0.tar.gz \
 && cd openmpi-4.0.0 \
 && ./configure --prefix=/usr/local \
 && make all install \
 && cd / \
 && rm -rf /gsi/libs/openmpi/BUILD_DIR

# Build Cmake v3 (centos default is v2.8, which is too old)
RUN mkdir -p /gsi/libs/cmake/BUILD_DIR \
 && source /opt/rh/devtoolset-7/enable \
 && cd /gsi/libs/cmake/BUILD_DIR \
 && curl -O -L https://github.com/Kitware/CMake/releases/download/v3.13.3/cmake-3.13.3.tar.gz \
 && tar -xf cmake-3.13.3.tar.gz \
 && cd cmake-3.13.3 \
 && ./bootstrap \
 && make \
 && make install \
 && cd / \
 && rm -rf /gsi/libs/cmake/BUILD_DIR

# Build HDF5 libraries
# Why does bitbucket make this so hard? Is a tarball link too much to ask for???
RUN yum -y install git \
 && source /opt/rh/devtoolset-7/enable \
 && mkdir -p /gsi/libs/hdf5/BUILD_DIR \
 && cd /gsi/libs/hdf5/BUILD_DIR \
 && git clone https://bitbucket.hdfgroup.org/scm/hdffv/hdf5.git \
 && cd hdf5 \
 && git checkout hdf5-1_10_4 \
 && ./configure --enable-fortran --enable-cxx --prefix=/usr/local/ \
 && make install \
 && rm -rf /gsi/libs/hdf5/BUILD_DIR \
 && yum -y erase git
ENV LD_LIBRARY_PATH /usr/local/lib

# Build netCDF C and Fortran libraries
RUN yum -y install libcurl-devel zlib-devel
ENV NETCDF /gsi/libs/netcdf
RUN mkdir -p ${NETCDF}/BUILD_DIR
RUN source /opt/rh/devtoolset-7/enable \
 && cd ${NETCDF}/BUILD_DIR \
 && curl -L -O https://github.com/Unidata/netcdf-c/archive/v4.6.2.tar.gz \
 && curl -L -O https://github.com/Unidata/netcdf-fortran/archive/v4.4.5.tar.gz \
 && tar -xf v4.6.2.tar.gz \
 && cd netcdf-c-4.6.2 \
 && ./configure --prefix=${NETCDF} \
 && make install 
RUN source /opt/rh/devtoolset-7/enable \
 && env \
 && cd ${NETCDF}/BUILD_DIR \
 && tar -xf v4.4.5.tar.gz \
 && cd netcdf-fortran-4.4.5/ \
 && export LD_LIBRARY_PATH=${NETCDF}/lib:${LD_LIBRARY_PATH} \
 && CPPFLAGS=-I${NETCDF}/include LDFLAGS=-L${NETCDF}/lib ./configure --prefix=${NETCDF} \
 && make install

# Build OpenBLAS with LAPACK
RUN mkdir -p /gsi/libs/openblas/BUILD_DIR
RUN source /opt/rh/devtoolset-7/enable \
 && cd /gsi/libs/openblas/BUILD_DIR \
 && curl -L -O https://github.com/xianyi/OpenBLAS/archive/v0.3.5.tar.gz \
 && tar -xf v0.3.5.tar.gz \
 && cd OpenBLAS-0.3.5/ \
 && make \
 && make PREFIX=/usr/ install
 

#
WORKDIR /gsi
#
# Download source code
RUN curl -O -L https://dtcenter.org/com-GSI/users/downloads/GSI_releases/comGSIv3.7_EnKFv1.3.tar.gz \
 && tar -xvf comGSIv3.7_EnKFv1.3.tar.gz 
#

# Set necessary environment variables for GSI build
#
ENV LDFLAGS -lm
ENV LD_LIBRARY_PATH ${NETCDF}/lib:/usr/local/lib/openmpi:/usr/lib/
ENV PATH /usr/local/bin:$PATH
ENV HDF5_ROOT $NETCDF
#
# Build GSI
# 
RUN source /opt/rh/devtoolset-7/enable \
 && env \
 && mkdir gsi_build \
 && cd gsi_build \
 && CPPFLAGS=-I${NETCDF}/include LDFLAGS=-L${NETCDF}/lib \
 && cmake /gsi/comGSIv${GSI_VERSION}_EnKFv${ENKF_VERSION} \
 && make -j ${J}
#
# Setup run directory
RUN mkdir /gsi/gsi_run
#
# Set some useful settings for bashrc/cshrc
RUN echo set -o ignoreeof >> /etc/bashrc \
 && echo set -o ignoreeof >> /etc/csh.cshrc \
RUN set -o ignoreeof
#
# copy in a couple custom scripts
#
#COPY docker-clean /gsi
COPY run-gsi /gsi/gsi_run
#RUN chmod +x /gsi/docker-clean \
# && chmod +x /gsi/gsi_run/run-gsi
#
# set up ssh configuration
#
COPY ssh_config /root/.ssh/config
COPY slave /gsi/slave
RUN yum install -y openssh-clients openssh-server net-tools \
    && yum clean all \
    && mkdir -p /var/run/sshd \
    && ssh-keygen -A \
    && sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#RSAAuthentication yes/RSAAuthentication yes/g' /etc/ssh/sshd_config \
    && sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/g' /etc/ssh/sshd_config \
    && ssh-keygen -f /root/.ssh/id_rsa -t rsa -N '' \
    && chmod 600 /root/.ssh/config \
    && chmod 700 /root/.ssh \
    && cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys \
    && chmod +x /gsi/slave
#
#RUN env
CMD ["/gsi/run-gsi"]
