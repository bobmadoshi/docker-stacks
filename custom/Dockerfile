# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

# Ubuntu 18.04 (bionic) from 2018-05-26
# https://github.com/docker-library/official-images/commit/aac6a45b9eb2bffb8102353c350d341a410fb169
ARG BASE_CONTAINER=ubuntu:bionic-20180526@sha256:c8c275751219dadad8fa56b3ac41ca6cb22219ff117ca98fe82b42f24e1ba64e
FROM $BASE_CONTAINER

LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get -yq dist-upgrade \
 && apt-get install -yq --no-install-recommends \
    wget \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
 && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH=$CONDA_DIR/bin:$PATH \
    HOME=/home/$NB_USER

# Add a script that we will use to correct permissions after running certain commands
ADD fix-permissions /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p $CONDA_DIR && \
    chown $NB_USER:$NB_GID $CONDA_DIR && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME && \
    fix-permissions "$(dirname $CONDA_DIR)"

USER $NB_UID

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work && \
    fix-permissions /home/$NB_USER

# Install conda as jovyan and check the md5 sum provided on the download site
ENV MINICONDA_VERSION=4.5.12 \
    CONDA_VERSION=4.6.7

RUN cd /tmp && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "866ae9dff53ad0874e1d1a60b1ad1ef8 *Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    $CONDA_DIR/bin/conda install --quiet --yes conda="${CONDA_VERSION%.*}.*" && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda list python | grep '^python ' | tr -s ' ' | cut -d '.' -f 1,2 | sed 's/$/.*/' >> $CONDA_DIR/conda-meta/pinned && \
    conda clean --all -f -y && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Tini
RUN conda install --quiet --yes 'tini=0.18.0' && \
    conda list tini | grep tini | tr -s ' ' | cut -d ' ' -f 1,2 >> $CONDA_DIR/conda-meta/pinned && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN conda install --quiet --yes
    '_r-mutex=1.0.0=anacondar_1' \
    'alembic=1.0.8=py_0' \
    'arrow-cpp=0.13.0=py37hfe0260e_2' \
    'asn1crypto=0.24.0=py37_1003' \
    'async_generator=1.10=py_0' \
    'attrs=19.1.0=py_0' \
    'backcall=0.1.0=py_0' \
    'beautifulsoup4=4.7.1=py37_1001' \
    'binutils_impl_linux-64=2.31.1=h6176602_1' \
    'binutils_linux-64=2.31.1=h6176602_3' \
    'blas=2.8=openblas' \
    'bleach=3.1.0=py_0' \
    'bokeh=1.0.4=py37_1000' \
    'boost-cpp=1.68.0=h11c811c_1000' \
    'brotli=1.0.7=hf484d3e_1000' \
    'bwidget=1.9.11=1' \
    'bzip2=1.0.6=h14c3975_1002' \
    'ca-certificates=2019.3.9=hecc5488_0' \
    'cairo=1.16.0=ha4e643d_1000' \
    'certifi=2019.3.9=py37_0' \
    'cffi=1.12.3=py37h8022711_0' \
    'chardet=3.0.4=py37_1003' \
    'click=7.0=py_0' \
    'cloudpickle=0.8.1=py_0' \
    'conda=4.6.14=py37_0' \
    'conda-env=2.6.0=1' \
    'configurable-http-proxy=1.3.0=0' \
    'cryptography=2.6.1=py37h72c5cf5_0' \
    'curl=7.64.1=hf8cf82a_0' \
    'cycler=0.10.0=py_1' \
    'cython=0.29.7=py37he1b5a44_0' \
    'cytoolz=0.9.0.1=py37h14c3975_1001' \
    'dask=1.1.5=py_0' \
    'dask-core=1.1.5=py_0' \
    'dbus=1.13.6=he372182_0' \
    'decorator=4.4.0=py_0' \
    'defusedxml=0.5.0=py_1' \
    'dill=0.2.9=py37_0' \
    'distributed=1.28.0=py37_0' \
    'entrypoints=0.3=py37_1000' \
    'et_xmlfile=1.0.1=py37_0' \
    'expat=2.2.5=hf484d3e_1002' \
    'fastcache=1.1.0=py37h516909a_0' \
    'fontconfig=2.13.1=he4413a7_1000' \
    'freetype=2.10.0=he983fc9_0' \
    'gcc_impl_linux-64=7.3.0=habb00fd_1' \
    'gcc_linux-64=7.3.0=h553295d_3' \
    'gettext=0.19.8.1=hc5be6a0_1002' \
    'gflags=2.2.2=hf484d3e_1001' \
    'gfortran_impl_linux-64=7.3.0=hdf63c60_1' \
    'gfortran_linux-64=7.3.0=h553295d_3' \
    'glib=2.58.3=hf63aee3_1001' \
    'glog=0.3.5=hf484d3e_1001' \
    'gmp=6.1.2=hf484d3e_1000' \
    'gmpy2=2.0.8=py37hb20f59a_1002' \
    'graphite2=1.3.13=hf484d3e_1000' \
    'gsl=2.4=h294904e_1006' \
    'gst-plugins-base=1.14.4=hdf3bae2_1001' \
    'gstreamer=1.14.4=h66beb1c_1001' \
    'gxx_impl_linux-64=7.3.0=hdf63c60_1' \
    'gxx_linux-64=7.3.0=h553295d_3' \
    'h5py=2.9.0=nompi_py37hf008753_1102' \
    'harfbuzz=2.4.0=h37c48d4_0' \
    'hdf5=1.10.4=nompi_h3c11f04_1106' \
    'heapdict=1.0.0=py37_1000' \
    'icu=58.2=hf484d3e_1000' \
    'idna=2.8=py37_1000' \
    'imageio=2.5.0=py37_0' \
    'ipykernel=5.1.0=py37h24bf2e0_1002' \
    'ipython=7.5.0=py37h24bf2e0_0' \
    'ipython_genutils=0.2.0=py_1' \
    'ipywidgets=7.4.2=py_0' \
    'jaydebeapi=1.1.1=py_0' \
    'jdcal=1.4.1=py_0' \
    'jedi=0.13.3=py37_0' \
    'jinja2=2.10.1=py_0' \
    'jpeg=9c=h14c3975_1001' \
    'jpype1=0.6.3=py37ha8d69ae_1000' \
    'jsonschema=3.0.1=py37_0' \
    'jupyter_client=5.2.4=py_3' \
    'jupyter_core=4.4.0=py_0' \
    'jupyterhub=0.9.6=py37_0' \
    'jupyterlab=0.35.5=py37_0' \
    'jupyterlab_server=0.2.0=py_0' \
    'kiwisolver=1.1.0=py37hc9558a2_0' \
    'krb5=1.16.3=h05b26f9_1001' \
    'libblas=3.8.0=8_openblas' \
    'libcblas=3.8.0=8_openblas' \
    'libcurl=7.64.1=hda55be3_0' \
    'libedit=3.1.20170329=hf8c457e_1001' \
    'libffi=3.2.1=he1b5a44_1006' \
    'libgcc-ng=8.2.0=hdf63c60_1' \
    'libgfortran-ng=7.3.0=hdf63c60_0' \
    'libiconv=1.15=h516909a_1005' \
    'liblapack=3.8.0=8_openblas' \
    'liblapacke=3.8.0=8_openblas' \
    'libpng=1.6.37=hed695b0_0' \
    'libprotobuf=3.7.1=h8b12597_0' \
    'libsodium=1.0.16=h14c3975_1001' \
    'libssh2=1.8.2=h22169c7_2' \
    'libstdcxx-ng=8.2.0=hdf63c60_1' \
    'libtiff=4.0.10=h648cc4a_1001' \
    'libuuid=2.32.1=h14c3975_1000' \
    'libxcb=1.13=h14c3975_1002' \
    'libxml2=2.9.9=h13577e0_0' \
    'llvmlite=0.27.1=py37hdbcaa40_0' \
    'locket=0.2.0=py_2' \
    'lz4-c=1.8.3=he1b5a44_1001' \
    'make=4.2.1=h14c3975_2004' \
    'mako=1.0.7=py_1' \
    'markupsafe=1.1.1=py37h14c3975_0' \
    'matplotlib=3.0.3=py37_1' \
    'matplotlib-base=3.0.3=py37h5f35d83_1' \
    'mistune=0.8.4=py37h14c3975_1000' \
    'mpc=1.1.0=hb20f59a_1006' \
    'mpfr=4.0.2=ha14ba45_0' \
    'mpmath=1.1.0=py_0' \
    'msgpack-python=0.6.1=py37h6bb024c_0' \
    'nbconvert=5.5.0=py_0' \
    'nbformat=4.4.0=py_1' \
    'ncurses=6.1=hf484d3e_1002' \
    'networkx=2.3=py_0' \
    'nodejs=11.14.0=he1b5a44_1' \
    'notebook=5.7.8=py37_0' \
    'numba=0.42.1=py37hf484d3e_0' \
    'numexpr=2.6.9=py37h637b7d7_1000' \
    'numpy=1.15.4=py37h8b7e671_1002' \
    'olefile=0.46=py_0' \
    'openblas=0.3.6=h6e990d7_1' \
    'openpyxl=2.6.1=py_0' \
    'openssl=1.1.1b=h14c3975_1' \
    'packaging=19.0=py_0' \
    'pamela=1.0.0=py_0' \
    'pandas=0.24.2=py37hf484d3e_0' \
    'pandoc=2.7.2=0' \
    'pandocfilters=1.4.2=py_1' \
    'pango=1.40.14=h4ea9474_1004' \
    'parquet-cpp=1.5.1=2' \
    'parso=0.4.0=py_0' \
    'partd=0.3.9=py_0' \
    'patsy=0.5.1=py_0' \
    'pcre=8.41=hf484d3e_1003' \
    'pendulum=2.0.4=py37_1000' \
    'pexpect=4.7.0=py37_0' \
    'pickleshare=0.7.5=py37_1000' \
    'pillow=6.0.0=py37he7afcd5_0' \
    'pip=19.1=py37_0' \
    'pixman=0.34.0=h14c3975_1003' \
    'prometheus_client=0.6.0=py_0' \
    'prompt_toolkit=2.0.9=py_0' \
    'protobuf=3.7.1=py37he1b5a44_0' \
    'psutil=5.6.2=py37h516909a_0' \
    'pthread-stubs=0.4=h14c3975_1001' \
    'ptyprocess=0.6.0=py_1001' \
    'pyarrow=0.13.0=py37h0978efd_0' \
    'pycosat=0.6.3=py37h14c3975_1001' \
    'pycparser=2.19=py37_1' \
    'pycurl=7.43.0.2=py37h16ce93b_0' \
    'pygments=2.3.1=py_0' \
    'pyopenssl=19.0.0=py37_0' \
    'pyparsing=2.4.0=py_0' \
    'pyqt=5.9.2=py37hcca6a23_0' \
    'pyrsistent=0.15.1=py37h516909a_0' \
    'pysocks=1.6.8=py37_1002' \
    'python=3.7.3=h5b0a415_0' \
    'python-dateutil=2.8.0=py_0' \
    'python-editor=1.0.4=py_0' \
    'python-oauth2=1.1.0=py37h28b3542_1' \
    'pytz=2019.1=py_0' \
    'pytzdata=2019.1=py_0' \
    'pywavelets=1.0.3=py37hd352d35_1' \
    'pyyaml=5.1=py37h14c3975_0' \
    'pyzmq=18.0.1=py37hc4ba49a_1' \
    'qt=5.9.7=h52cfd70_1' \
    'r-askpass=1.1=r351h96ca727_0' \
    'r-assertthat=0.2.1=r351h6115d3f_0' \
    'r-backports=1.1.4=r351hcdcec82_0' \
    'r-base=3.5.1=h271c98b_1006' \
    'r-base64enc=0.1_3=r351h96ca727_1002' \
    'r-bh=1.69.0_1=r351h6115d3f_0' \
    'r-bit=1.1_14=r351hcdcec82_0' \
    'r-bit64=0.9_7=r351h96ca727_1000' \
    'r-bitops=1.0_6=r351h96ca727_1002' \
    'r-blob=1.1.1=r351_1001' \
    'r-broom=0.5.2=r351h6115d3f_0' \
    'r-callr=3.2.0=r351h6115d3f_0' \
    'r-caret=6.0_82=r351hcdcec82_0' \
    'r-cellranger=1.1.0=r351h6115d3f_1001' \
    'r-class=7.3_15=r351h96ca727_1000' \
    'r-cli=1.1.0=r351h6115d3f_0' \
    'r-clipr=0.6.0=r351h6115d3f_0' \
    'r-codetools=0.2_16=r351h6115d3f_1000' \
    'r-colorspace=1.4_1=r351hcdcec82_0' \
    'r-config=0.3=r351h6115d3f_1001' \
    'r-crayon=1.3.4=r351h6115d3f_1001' \
    'r-curl=3.3=r351h96ca727_0' \
    'r-data.table=1.12.2=r351hcdcec82_0' \
    'r-dbi=1.0.0=r351h6115d3f_1001' \
    'r-dbplyr=1.4.0=r351h6115d3f_0' \
    'r-devtools=1.13.6=r351h6115d3f_1' \
    'r-digest=0.6.18=r351h96ca727_1000' \
    'r-dplyr=0.8.0.1=r351h29659fb_1000' \
    'r-ellipsis=0.1.0=r351h96ca727_0' \
    'r-evaluate=0.13=r351h6115d3f_1' \
    'r-fansi=0.4.0=r351h96ca727_1000' \
    'r-forcats=0.4.0=r351h6115d3f_0' \
    'r-foreach=1.4.4=r351h6115d3f_1001' \
    'r-forecast=8.2=r351hf484d3e_1001' \
    'r-forge=0.2.0=r351h6115d3f_0' \
    'r-fracdiff=1.4_2=r351h96ca727_1002' \
    'r-fs=1.3.0=r351h0357c0b_0' \
    'r-generics=0.0.2=r351h6115d3f_1001' \
    'r-ggplot2=3.1.1=r351h6115d3f_0' \
    'r-git2r=0.25.2=r351h5ca76e2_0' \
    'r-glue=1.3.1=r351hcdcec82_0' \
    'r-gower=0.2.0=r351h96ca727_0' \
    'r-gtable=0.3.0=r351h6115d3f_0' \
    'r-haven=2.1.0=r351h29659fb_0' \
    'r-hexbin=1.27.2=r351ha65eedd_1002' \
    'r-highr=0.8=r351h6115d3f_0' \
    'r-hms=0.4.2=r351h6115d3f_1000' \
    'r-htmltools=0.3.6=r351hf484d3e_1002' \
    'r-htmlwidgets=1.2=r351h6115d3f_1000' \
    'r-httpuv=1.5.1=r351h0357c0b_0' \
    'r-httr=1.4.0=r351h6115d3f_1000' \
    'r-ipred=0.9_8=r351h96ca727_1000' \
    'r-irdisplay=0.7=r351_1000' \
    'r-irkernel=0.8.15=r351h6115d3f_1001' \
    'r-iterators=1.0.10=r351h6115d3f_1001' \
    'r-jsonlite=1.6=r351h96ca727_1000' \
    'r-kernsmooth=2.23_15=r351ha65eedd_1002' \
    'r-knitr=1.22=r351h6115d3f_0' \
    'r-labeling=0.3=r351h6115d3f_1001' \
    'r-later=0.8.0=r351h29659fb_0' \
    'r-lattice=0.20_38=r351h96ca727_1000' \
    'r-lava=1.6.5=r351h6115d3f_0' \
    'r-lazyeval=0.2.2=r351hcdcec82_0' \
    'r-lmtest=0.9_36=r351ha65eedd_1000' \
    'r-lubridate=1.7.4=r351h29659fb_1001' \
    'r-magrittr=1.5=r351h6115d3f_1001' \
    'r-markdown=0.9=r351h96ca727_1000' \
    'r-mass=7.3_51.4=r351hcdcec82_0' \
    'r-matrix=1.2_17=r351hcdcec82_0' \
    'r-memoise=1.1.0=r351h6115d3f_1001' \
    'r-mgcv=1.8_28=r351hcdcec82_0' \
    'r-mime=0.6=r351h96ca727_1000' \
    'r-modelmetrics=1.2.2=r351h0357c0b_0' \
    'r-modelr=0.1.4=r351h6115d3f_0' \
    'r-munsell=0.5.0=r351h6115d3f_1001' \
    'r-nlme=3.1_139=r351h9bbef5b_0' \
    'r-nnet=7.3_12=r351h96ca727_1002' \
    'r-numderiv=2016.8_1=r351h6115d3f_1001' \
    'r-nycflights13=1.0.0=r351_1000' \
    'r-openssl=1.3=r351h9c8475f_0' \
    'r-pbdzmq=0.3_3=r351h47a35c4_1001' \
    'r-pillar=1.3.1=r351h6115d3f_1000' \
    'r-pkgconfig=2.0.2=r351h6115d3f_1001' \
    'r-plogr=0.2.0=r351h6115d3f_1001' \
    'r-plyr=1.8.4=r351h29659fb_1002' \
    'r-prettyunits=1.0.2=r351h6115d3f_1001' \
    'r-processx=3.3.1=r351hcdcec82_0' \
    'r-prodlim=2018.04.18=r351h29659fb_1002' \
    'r-progress=1.2.0=r351h6115d3f_1002' \
    'r-promises=1.0.1=r351h29659fb_1000' \
    'r-ps=1.3.0=r351h96ca727_1000' \
    'r-purrr=0.3.2=r351hcdcec82_0' \
    'r-quadprog=1.5_5=r351ha65eedd_1002' \
    'r-quantmod=0.4_13=r351h6115d3f_1000' \
    'r-r2d3=0.2.3=r351h6115d3f_1000' \
    'r-r6=2.4.0=r351h6115d3f_0' \
    'r-randomforest=4.6_14=r351ha65eedd_1000' \
    'r-rappdirs=0.3.1=r351h96ca727_1002' \
    'r-rcolorbrewer=1.1_2=r351h6115d3f_1001' \
    'r-rcpp=1.0.1=r351h0357c0b_0' \
    'r-rcpparmadillo=0.9.400.2.0=r351h0357c0b_0' \
    'r-rcpproll=0.3.0=r351h29659fb_1000' \
    'r-rcurl=1.95_4.12=r351hcdcec82_0' \
    'r-readr=1.3.1=r351h29659fb_1000' \
    'r-readxl=1.3.1=r351h0357c0b_0' \
    'r-recipes=0.1.5=r351h6115d3f_0' \
    'r-rematch=1.0.1=r351h6115d3f_1001' \
    'r-repr=1.0.0=r351h6115d3f_0' \
    'r-reprex=0.2.1=r351h6115d3f_1000' \
    'r-reshape2=1.4.3=r351h29659fb_1003' \
    'r-rlang=0.3.4=r351hcdcec82_0' \
    'r-rmarkdown=1.11=r351h6115d3f_1000' \
    'r-rpart=4.1_15=r351hcdcec82_0' \
    'r-rprojroot=1.3_2=r351h6115d3f_1001' \
    'r-rsqlite=2.1.1=r351h29659fb_1000' \
    'r-rstudioapi=0.10=r351h6115d3f_0' \
    'r-rvest=0.3.3=r351h6115d3f_0' \
    'r-scales=1.0.0=r351h29659fb_1001' \
    'r-selectr=0.4_1=r351h6115d3f_1000' \
    'r-shiny=1.2.0=r351_1000' \
    'r-sourcetools=0.1.7=r351hf484d3e_1000' \
    'r-sparklyr=0.9.4=r351h6115d3f_0' \
    'r-squarem=2017.10_1=r351h6115d3f_1001' \
    'r-stringi=1.4.3=r351h0357c0b_0' \
    'r-stringr=1.4.0=r351h6115d3f_0' \
    'r-survival=2.44_1.1=r351hcdcec82_0' \
    'r-sys=3.2=r351hcdcec82_0' \
    'r-tibble=2.1.1=r351hcdcec82_0' \
    'r-tidyr=0.8.3=r351h0357c0b_0' \
    'r-tidyselect=0.2.5=r351h29659fb_1000' \
    'r-tidyverse=1.2.1=r351h6115d3f_1001' \
    'r-timedate=3043.102=r351h6115d3f_1000' \
    'r-tinytex=0.12=r351h6115d3f_0' \
    'r-tseries=0.10_46=r351h9ac9557_1000' \
    'r-ttr=0.23_4=r351ha65eedd_1000' \
    'r-utf8=1.1.4=r351h96ca727_1000' \
    'r-uuid=0.1_2=r351h96ca727_1001' \
    'r-viridislite=0.3.0=r351h6115d3f_1001' \
    'r-whisker=0.3_2=r351h6115d3f_1001' \
    'r-withr=2.1.2=r351h6115d3f_1000' \
    'r-xfun=0.6=r351h6115d3f_0' \
    'r-xml2=1.2.0=r351h29659fb_1002' \
    'r-xtable=1.8_3=r351_2000' \
    'r-xts=0.11_1=r351h96ca727_1000' \
    'r-yaml=2.2.0=r351h96ca727_1001' \
    'r-zoo=1.8_5=r351hcdcec82_0' \
    're2=2019.04.01=he1b5a44_0' \
    'readline=7.0=hf8c457e_1001' \
    'requests=2.21.0=py37_1000' \
    'rpy2=2.9.4=py37r351h941a26a_1' \
    'ruamel_yaml=0.15.71=py37h14c3975_1000' \
    'scikit-learn=0.20.3=py37ha8026db_1' \
    'scipy=1.2.1=py37h09a28d5_1' \
    'send2trash=1.5.0=py_0' \
    'setuptools=41.0.1=py37_0' \
    'sip=4.19.8=py37hf484d3e_1000' \
    'six=1.12.0=py37_1000' \
    'snappy=1.1.7=hf484d3e_1002' \
    'sortedcontainers=2.1.0=py_0' \
    'soupsieve=1.9.1=py37_0' \
    'sqlalchemy=1.3.3=py37h516909a_0' \
    'sqlite=3.26.0=h67949de_1001' \
    'statsmodels=0.9.0=py37h3010b51_1000' \
    'sympy=1.3=py37_1000' \
    'tblib=1.3.2=py_1' \
    'terminado=0.8.2=py37_0' \
    'testpath=0.4.2=py_1001' \
    'thrift-cpp=0.12.0=h0a07b25_1002' \
    'time=1.8=h516909a_0' \
    'tini=0.18.0=h14c3975_1001' \
    'tk=8.6.9=h84994c4_1001' \
    'tktable=2.10=h14c3975_0' \
    'toolz=0.9.0=py_1' \
    'tornado=6.0.2=py37h516909a_0' \
    'traitlets=4.3.2=py37_1000' \
    'turbodbc=3.1.1=py37h64dbffa_0' \
    'tzlocal=1.5.1=py_0' \
    'unixodbc=2.3.7=h227dcee_1000' \
    'urllib3=1.24.2=py37_0' \
    'vincent=0.4.4=py_1' \
    'wcwidth=0.1.7=py_1' \
    'webencodings=0.5.1=py_1' \
    'wheel=0.33.1=py37_0' \
    'widgetsnbextension=3.4.2=py37_1000' \
    'xlrd=1.2.0=py_0' \
    'xlsxwriter=1.1.8=py_0' \
    'xorg-kbproto=1.0.7=h14c3975_1002' \
    'xorg-libice=1.0.9=h516909a_1004' \
    'xorg-libsm=1.2.3=h84519dc_1000' \
    'xorg-libx11=1.6.7=h14c3975_1000' \
    'xorg-libxau=1.0.9=h14c3975_0' \
    'xorg-libxdmcp=1.1.3=h516909a_0' \
    'xorg-libxext=1.3.4=h516909a_0' \
    'xorg-libxrender=0.9.10=h516909a_1002' \
    'xorg-renderproto=0.11.1=h14c3975_1002' \
    'xorg-xextproto=7.3.0=h14c3975_1002' \
    'xorg-xproto=7.0.31=h14c3975_1007' \
    'xz=5.2.4=h14c3975_1001' \
    'yaml=0.1.7=h14c3975_1001' \
    'zeromq=4.3.1=hf484d3e_1000' \
    'zict=0.1.4=py_0' \
    'zlib=1.2.11=h14c3975_1004' \
    'zstd=1.3.3=1' && \
RUN pip install --quiet --yes
    'cached-property==1.5.1' \
    'colorama==0.4.1' \
    'dnspython==1.16.0' \
    'exchangelib==1.12.4' \
    'future==0.17.1' \
    'ipython-cypher==0.2.6' \
    'isodate==0.6.0' \
    'lxml==4.3.3' \
    'neo4j==1.7.3' \
    'neo4jrestclient==2.1.1' \
    'neo4jupyter==0.1.2' \
    'neobolt==1.7.12' \
    'neotime==1.7.4' \
    'ntlm-auth==1.3.0' \
    'prettytable==0.7.2' \
    'py2neo==4.3.0' \
    'requests-ntlm==1.1.0' && \
    conda clean --all -f -y && \
    jupyter labextension install @jupyterlab/hub-extension@^0.12.0 && \
    npm cache clean --force && \
    jupyter notebook --generate-config && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

USER root

EXPOSE 8888
WORKDIR $HOME

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Add local files as late as possible to avoid cache busting
COPY start.sh /usr/local/bin/
COPY start-notebook.sh /usr/local/bin/
COPY start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/
RUN fix-permissions /etc/jupyter/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
