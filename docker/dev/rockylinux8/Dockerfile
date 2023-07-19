FROM rockylinux:8.8

# NOTE (esb 7-17-23): Currently rockylinux 8.8 has 4 High-ranked identified 
# vulnerabilities.  Use commands: 
#    docker scout quickview rockylinux:8.8 
#    docker scout cves rockylinux:8.8
#
# All the vulnerabilities are related to python3.6 and setuptools 39.2.0. 
# HOWEVER:  there is no python installed by default.  We will be using 
# Continuum.io's Miniconda3 package manager and using python 3.9 or later... 
# so this vulnerability should not be an issue.

# -----------------------------------------------------------------------------
# Configuration Variables:
# -----------------------------------------------------------------------------

ARG PKG_NAME=mypypackage

# Specific bash installation version of Miniconda3 from anaconda.com
# ARG MINICONDA_PKG=Miniconda3-latest-Linux-aarch64.sh
ARG MINICONDA_PKG=Miniconda3-py311_23.5.2-0-Linux-aarch64.sh

# Verson of python to install into the operational conda env
ARG PYTHON_VERSION=3.9
ARG QT_VERSION=5
# Username for the non-root user to use for non-root testing/running
ARG USERNAME=me

# -----------------------------------------------------------------------------
# Build Shell configuration
# -----------------------------------------------------------------------------

# Set the shell to be a bash login shell -- we will build on the .bashrc as we 
# go and use things added to it in subsequent RUN steps.

SHELL ["/bin/bash", "-l", "-c"]

# -----------------------------------------------------------------------------
# Install System Level Dependencies
# -----------------------------------------------------------------------------

RUN dnf upgrade --refresh -y && \
    dnf install -y \
        which \
        findutils \
        git \
        git-lfs \
        gcc-toolset-11-toolchain && \
    echo "source /opt/rh/gcc-toolset-11/enable" >> /root/.bashrc && \
    dnf install -y \
    https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm && \
    dnf config-manager --enable powertools && \
    dnf install -y \
        xorg-x11-apps \
        xcb-util-renderutil \
        libX11-xcb \
        mesa-dri-drivers \
        mesa-libEGL \
        mesa-libGL \
        mesa-libGLU \
        mesa-libGLw \
        mesa-libOSMesa \
        mesa-libglapi \
        libX11-xcb \
        xcb-util \
        xcb-util-cursor \
        xcb-util-image \
        xcb-util-keysyms \
        xcb-util-wm \
        libxcb \
        xcb-util-renderutil && \
    echo "export DISPLAY=host.docker.internal:0" >> /root/.bashrc && \
    echo "export XDG_RUNTIME_DIR=/tmp/runtime-root" && \
    dnf clean all

# -----------------------------------------------------------------------------
# Install Miniconda3 (conda package manager for Python and system dependencies)
# -----------------------------------------------------------------------------

# Install the specified version of Miniconda3 and configure bash to use it for
# all users.  Note:  /opt/containerd is a shared area for things we want to
# build and keep for root and non-root users as we build docker images.  (Docker
# is one type of containerd image among many like Singularity, Podman, etc.)

ARG SHARED_PATH="/opt/containerd"
ARG SHARED_BASHRC="${SHARED_PATH}/.bashrc"

RUN curl https://repo.anaconda.com/miniconda/${MINICONDA_PKG} > ${MINICONDA_PKG} && \
    bash ${MINICONDA_PKG} -b -p /opt/miniconda3 && \
    /opt/miniconda3/condabin/conda init bash && \
    mkdir -p /opt/containerd && \
    cp /root/.bashrc ${SHARED_BASHRC} && \
    rm -rf ${MINICONDA_PKG}

# -----------------------------------------------------------------------------
# Install Dependencies
# -----------------------------------------------------------------------------
# Create a conda env and Install dependencies using the conda package manager
COPY docker/dev/conda_env.yaml ${SHARED_PATH}/conda_env.yaml
RUN conda env create -f ${SHARED_PATH}/conda_env.yaml && \
    echo "conda activate ${PKG_NAME}_dev" >> ${SHARED_BASHRC}  && \
    cp ${SHARED_BASHRC} /root/.bashrc

COPY requirements_dev.txt ${SHARED_PATH}/requirements_dev.txt
RUN pip install -r ${SHARED_PATH}/requirements_dev.txt

# -----------------------------------------------------------------------------
# Non-root user setup
# -----------------------------------------------------------------------------
# Create the non-root user w/ a home directory and set up bash
# NOTE: everything above the "USER" command below is run as root during build.

RUN groupadd -r ${USERNAME} && useradd -m -r -g ${USERNAME} ${USERNAME}
USER $USERNAME
RUN cp ${SHARED_BASHRC} /home/me/.bashrc

# -----------------------------------------------------------------------------
# Container Image Execution
# -----------------------------------------------------------------------------

WORKDIR /mnt/workspace

# The thing to do when the container runs (CMD or ENTRYPOINT)

CMD [ "/bin/bash" ]