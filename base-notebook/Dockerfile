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

ADD fix-permissions /usr/local/bin/fix-permissions
# Create jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN groupadd wheel -g 11 && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
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
    conda clean -tipsy && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Tini
RUN conda install --quiet --yes 'tini=0.18.0' && \
    conda list tini | grep tini | tr -s ' ' | cut -d ' ' -f 1,2 >> $CONDA_DIR/conda-meta/pinned && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN conda install --quiet --yes \
    'notebook=5.7.5' \
    'jupyterhub=0.9.4' \
    '_r-mutex=1.0.0' \
    'anypytools=1.1.3' \
    'appdirs=1.4.3' \
    'arrow-cpp=0.12.1' \
    'asn1crypto=0.24.0' \
    'atomicwrites=1.3.0' \
    'attrs=19.1.0' \
    'automat=0.7.0' \
    'backcall=0.1.0' \
    'beautifulsoup4=4.7.1' \
    'binutils_impl_linux-64=2.31.1' \
    'binutils_linux-64=2.31.1' \
    'bleach=3.1.0' \
    'blosc=1.8.1' \
    'boost-cpp=1.68.0' \
    'bzip2=1.0.6' \
    'ca-certificates=2019.3.9' \
    'cached-property=1.5.1' \
    'cairo=1.14.12' \
    'certifi=2019.3.9' \
    'cffi=1.12.2' \
    'chardet=3.0.4' \
    'colorama=0.4.1' \
    'constantly=15.1.0' \
    'cryptography=2.6.1' \
    'cssselect=1.0.3' \
    'curl=7.64.0' \
    'cypari=2.3.0' \
    'decorator=4.3.2' \
    'defusedxml=0.5.0' \
    'django=2.2.1' \
    'dnspython=1.16.0' \
    'entrypoints=0.3' \
    'et_xmlfile=1.0.1' \
    'exchangelib=1.12.2' \
    'fontconfig=2.13.0' \
    'freetype=2.9.1' \
    'fribidi=1.0.5' \
    'future=0.17.1' \
    'fxrays=1.3.3' \
    'gcc_impl_linux-64=7.3.0' \
    'gcc_linux-64=7.3.0' \
    'get=2019.4.13' \
    'gfortran_impl_linux-64=7.3.0' \
    'gfortran_linux-64=7.3.0' \
    'glib=2.56.2' \
    'glo=0.0.0.dev1' \
    'gmp=6.1.2' \
    'graphite2=1.3.13' \
    'gxx_impl_linux-64=7.3.0' \
    'gxx_linux-64=7.3.0' \
    'h5py=2.9.0' \
    'harfbuzz=1.8.8' \
    'hdf5=1.10.4' \
    'hyperlink=19.0.0' \
    'icu=58.2' \
    'idna=2.8' \
    'image=1.5.27' \
    'incremental=17.5.0' \
    'ipykernel=5.1.0' \
    'ipython=7.3.0' \
    'ipython_genutils=0.2.0' \
    'isodate=0.6.0' \
    'jdcal=1.4.1' \
    'jedi=0.13.3' \
    'jinja2=2.10' \
    'jsonschema=3.0.1' \
    'jupyter_client=5.2.4' \
    'jupyter_core=4.4.0' \
    'krb5=1.16.1' \
    'libblas=3.8.0' \
    'libcblas=3.8.0' \
    'libcurl=7.64.0' \
    'libedit=3.1.20181209' \
    'libffi=3.2.1' \
    'libgcc-ng=8.2.0' \
    'libgfortran-ng=7.3.0' \
    'libiconv=1.15' \
    'libpng=1.6.36' \
    'libprotobuf=3.6.1' \
    'libsodium=1.0.16' \
    'libssh2=1.8.0' \
    'libstdcxx-ng=8.2.0' \
    'libuuid=1.0.3' \
    'libxcb=1.13' \
    'libxml2=2.9.9' \
    'lml=0.0.9' \
    'lxml=4.3.3' \
    'lzo=2.10' \
    'make=4.2.1' \
    'markupsafe=1.1.1' \
    'mistune=0.8.4' \
    'mock=3.0.3' \
    'more-itertools=7.0.0' \
    'mro-base=3.5.1' \
    'mro-base_impl=3.5.1' \
    'mro-basics=3.5.1' \
    'nbconvert=5.4.1' \
    'nbformat=4.4.0' \
    'ncurses=6.1' \
    'networkx=2.3' \
    'notebook=5.7.6' \
    'ntlm-auth=1.2.0' \
    'numexpr=2.6.9' \
    'numpy=1.16.2' \
    'openblas=0.3.5' \
    'openpyxl=2.5.14' \
    'openssl=1.1.1b' \
    'pandas=0.24.2' \
    'pandoc=1.0.2' \
    'pandocfilters=1.4.2' \
    'pango=1.42.4' \
    'parquet-cpp=1.5.1' \
    'parse=1.12.0' \
    'parsel=1.5.1' \
    'parso=0.3.4' \
    'pbr=5.2.0' \
    'pcre=8.43' \
    'pendulum=2.0.4' \
    'pexpect=4.6.0' \
    'pickleshare=0.7.5' \
    'pillow=6.0.0' \
    'pip=19.1' \
    'pixman=0.38.0' \
    'plink=2.2' \
    'pluggy=0.9.0' \
    'ply=3.11' \
    'post=2019.4.13' \
    'prometheus_client=0.6.0' \
    'prompt_toolkit=2.0.9' \
    'ptyprocess=0.6.0' \
    'public=2019.4.13' \
    'py=1.8.0' \
    'pyarrow=0.12.1' \
    'pyasn1=0.4.5' \
    'pyasn1-modules=0.2.5' \
    'pybind11=2.2.4' \
    'pycparser=2.19' \
    'pydispatcher=2.0.5' \
    'pydoe=0.3.8' \
    'pyexcel-io=0.5.17' \
    'pyexcel-xlsx=0.5.7' \
    'pygments=2.3.1' \
    'pygments-anyscript=1.2.1' \
    'pyhamcrest=1.9.0' \
    'pyodbc=4.0.24' \
    'pyopenssl=19.0.0' \
    'pypng=0.0.19' \
    'pyreadline=2.1' \
    'pyrsistent=0.14.11' \
    'pysocks=1.6.8' \
    'pytables=3.5.1' \
    'pytest=4.4.1' \
    'pytest-runner=4.4' \
    'python=3.7.2' \
    'python-dateutil=2.8.0' \
    'pytz=2018.9' \
    'pytzdata=2019.1' \
    'pywinpty=0.5.5' \
    'pyzmq=17.1.2' \
    'query-string=2019.4.13' \
    'queuelib=1.5.0' \
    'r-abind=1.4_5' \
    'r-assertthat=0.2.0' \
    'r-backports=1.1.2' \
    'r-base64enc=0.1_3' \
    'r-bh=1.66.0_1' \
    'r-bindr=0.1.1' \
    'r-bindrcpp=0.2.2' \
    'r-boot=1.3_20' \
    'r-broom=0.5.0' \
    'r-callr=2.0.4' \
    'r-caret=6.0_80' \
    'r-cellranger=1.1.0' \
    'r-checkpoint=0.4.4' \
    'r-class=7.3_14' \
    'r-cli=1.0.0' \
    'r-clipr=0.4.1' \
    'r-cluster=2.0.7_1' \
    'r-codetools=0.2_15' \
    'r-colorspace=1.3_2' \
    'r-crayon=1.3.4' \
    'r-curl=3.2' \
    'r-cvst=0.2_2' \
    'r-data.table=1.11.4' \
    'r-dbi=1.0.0' \
    'r-dbplyr=1.2.2' \
    'r-ddalpha=1.3.4' \
    'r-deoptimr=1.0_8' \
    'r-deployrrserve=9.0.0' \
    'r-dichromat=2.0_0' \
    'r-digest=0.6.15' \
    'r-dimred=0.1.0' \
    'r-doparallel=1.0.13' \
    'r-dplyr=0.7.6' \
    'r-drr=0.0.3' \
    'r-essentials=3.5.1' \
    'r-evaluate=0.11' \
    'r-fansi=0.2.3' \
    'r-forcats=0.3.0' \
    'r-foreach=1.5.0' \
    'r-foreign=0.8_70' \
    'r-formatr=1.5' \
    'r-geometry=0.3_6' \
    'r-ggplot2=3.0.0' \
    'r-glmnet=2.0_16' \
    'r-glue=1.3.0' \
    'r-gower=0.1.2' \
    'r-gtable=0.2.0' \
    'r-haven=1.1.2' \
    'r-hexbin=1.27.2' \
    'r-highr=0.7' \
    'r-hms=0.4.2' \
    'r-htmltools=0.3.6' \
    'r-htmlwidgets=1.2' \
    'r-httpuv=1.4.5' \
    'r-httr=1.3.1' \
    'r-ipred=0.9_6' \
    'r-irdisplay=0.5.0' \
    'r-irkernel=0.8.11' \
    'r-iterators=1.0.10' \
    'r-jsonlite=1.5' \
    'r-kernlab=0.9_26' \
    'r-kernsmooth=2.23_15' \
    'r-knitr=1.20' \
    'r-labeling=0.3' \
    'r-later=0.7.3' \
    'r-lattice=0.20_35' \
    'r-lava=1.6.2' \
    'r-lazyeval=0.2.1' \
    'r-lubridate=1.7.4' \
    'r-magic=1.5_8' \
    'r-magrittr=1.5' \
    'r-maps=3.3.0' \
    'r-markdown=0.8' \
    'r-mass=7.3_49' \
    'r-matrix=1.2_14' \
    'r-mgcv=1.8_23' \
    'r-microsoftr=3.5.0.108' \
    'r-mime=0.5' \
    'r-modelmetrics=1.1.0' \
    'r-modelr=0.1.2' \
    'r-munsell=0.5.0' \
    'r-nlme=3.1_137' \
    'r-nnet=7.3_12' \
    'r-numderiv=2016.8_1' \
    'r-openssl=1.0.2' \
    'r-pbdzmq=0.3_3' \
    'r-pillar=1.3.0' \
    'r-pkgconfig=2.0.1' \
    'r-plogr=0.2.0' \
    'r-pls=2.6_0' \
    'r-plyr=1.8.4' \
    'r-png=0.1_7' \
    'r-praise=1.0.0' \
    'r-processx=3.1.0' \
    'r-prodlim=2018.04.18' \
    'r-promises=1.0.1' \
    'r-purrr=0.2.5' \
    'r-quantmod=0.4_13' \
    'r-r6=2.2.2' \
    'r-randomforest=4.6_14' \
    'r-rbokeh=0.6.3' \
    'r-rcolorbrewer=1.1_2' \
    'r-rcpp=0.12.18' \
    'r-rcpproll=0.3.0' \
    'r-readr=1.1.1' \
    'r-readxl=1.1.0' \
    'r-recipes=0.1.3' \
    'r-recommended=3.5.1' \
    'r-rematch=1.0.1' \
    'r-repr=0.15.0' \
    'r-reprex=0.2.0' \
    'r-reshape2=1.4.3' \
    'r-revoioq=10.0.0' \
    'r-revomods=11.0.0' \
    'r-revoutils=11.0.0' \
    'r-revoutilsmath=11.0.0' \
    'r-rlang=0.2.1' \
    'r-rmarkdown=1.10' \
    'r-robustbase=0.93_2' \
    'r-rpart=4.1_13' \
    'r-rprojroot=1.3_2' \
    'r-rstudioapi=0.7' \
    'r-runit=0.4.26' \
    'r-rvest=0.3.2' \
    'r-scales=0.5.0' \
    'r-selectr=0.4_1' \
    'r-sfsmisc=1.1_2' \
    'r-shiny=1.1.0' \
    'r-sourcetools=0.1.7' \
    'r-spatial=7.3_11' \
    'r-squarem=2017.10_1' \
    'r-stringi=1.2.4' \
    'r-stringr=1.3.1' \
    'r-survival=2.41_3' \
    'r-testthat=2.0.0' \
    'r-tibble=1.4.2' \
    'r-tidyr=0.8.1' \
    'r-tidyselect=0.2.4' \
    'r-tidyverse=1.2.1' \
    'r-timedate=3043.102' \
    'r-tinytex=0.6' \
    'r-ttr=0.23_3' \
    'r-utf8=1.1.4' \
    'r-uuid=0.1_2' \
    'r-viridislite=0.3.0' \
    'r-whisker=0.3_2' \
    'r-withr=2.1.2' \
    'r-xfun=0.3' \
    'r-xml2=1.2.0' \
    'r-xtable=1.8_2' \
    'r-xts=0.11_0' \
    'r-yaml=2.2.0' \
    'r-zoo=1.8_3' \
    'readline=7.0' \
    'request=2019.4.13' \
    'requests=2.21.0' \
    'requests-ntlm=1.1.0' \
    'scipy=1.2.1' \
    'scrapy=1.6.0' \
    'send2trash=1.5.0' \
    'service-identity=18.1.0' \
    'setuptools=40.8.0' \
    'six=1.12.0' \
    'snappy=2.6.1' \
    'snappy-manifolds=1.0' \
    'soupsieve=1.9.1' \
    'spherogram=1.8.1' \
    'sqlite=3.27.2' \
    'sqlparse=0.3.0' \
    'terminado=0.8.1' \
    'testpath=0.4.2' \
    'thrift-cpp=0.12.0' \
    'tk=8.6.8' \
    'tornado=6.0.1' \
    'traitlets=4.3.2' \
    'turbodbc=3.1.0' \
    'twisted=19.2.0' \
    'tzlocal=1.5.1' \
    'unixodbc=2.3.7' \
    'urllib3=1.24.1' \
    'vc=2018.7.10' \
    'w3lib=1.20.0' \
    'wcwidth=0.1.7' \
    'webencodings=0.5.1' \
    'wheel=0.33.1' \
    'win-inet-pton=1.1.0' \
    'wincertstore=0.2' \
    'xlsxwriter=1.1.7' \
    'xz=5.2.4' \
    'zeromq=4.2.5' \
    'zlib=1.2.11' \
    'zope-interface=4.6.0' \
    'jupyterlab=0.35.4' && \
    conda clean -tipsy && \
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
