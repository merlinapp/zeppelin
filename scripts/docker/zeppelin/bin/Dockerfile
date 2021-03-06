# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ubuntu:16.04
MAINTAINER Apache Software Foundation <dev@zeppelin.apache.org>

ENV Z_VERSION="0.9.0-SNAPSHOT"

ENV LOG_TAG="[ZEPPELIN_${Z_VERSION}]:" \
    Z_HOME="/zeppelin" \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    ZEPPELIN_ADDR="0.0.0.0"

RUN echo "$LOG_TAG update and install basic packages" && \
    apt-get -y update && \
    apt-get install -y --no-install-recommends locales && \
    locale-gen $LANG && \
    apt-get install -y --no-install-recommends software-properties-common && \
    apt -y autoclean && \
    apt -y dist-upgrade && \
    apt-get install -y --no-install-recommends build-essential && \
    echo "$LOG_TAG install tini related packages" && \
    apt-get install -y --no-install-recommends wget curl grep sed dpkg && \
    TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini_${TINI_VERSION}.deb" > tini.deb && \
    dpkg -i tini.deb && \
    rm tini.deb && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
RUN echo "$LOG_TAG Install java8" && \
    apt-get -y update && \
    apt-get install -y --no-install-recommends openjdk-8-jdk && \
    rm -rf /var/lib/apt/lists/*

# should install conda first before numpy, matploylib since pip and python will be installed by conda
RUN echo "$LOG_TAG Install miniconda3 related packages" && \
    apt-get -y update && \
    apt-get install -y --no-install-recommends bzip2 ca-certificates \
    libglib2.0-0 libxext6 libsm6 libxrender1 \
    git mercurial subversion && \
    echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-4.6.14-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && rm -rf /var/lib/apt/lists/*

ENV PATH /opt/conda/bin:$PATH

RUN echo "$LOG_TAG Install python related packages" && \
    apt-get -y update && \
    apt-get install -y --no-install-recommends python-dev python-pip && \
    apt-get install -y --no-install-recommends gfortran && \
    # numerical/algebra packages
    apt-get install -y --no-install-recommends libblas-dev libatlas-dev liblapack-dev && \
    # font, image
    apt-get install -y --no-install-recommends libpng-dev libfreetype6-dev libxft-dev && \
    # for tkinter
    apt-get install -y --no-install-recommends python-tk libxml2-dev libxslt-dev zlib1g-dev && \
    hash -r && \
    conda config --set always_yes yes --set changeps1 no && \
    conda update -q conda && \
    conda info -a && \
    conda config --add channels conda-forge && \
    pip install -q pycodestyle==2.5.0 && \
    pip install -q numpy==1.17.3 pandas==0.25.0 scipy==1.3.1 grpcio==1.19.0 bkzep==0.6.1 hvplot==0.5.2 protobuf==3.10.0 pandasql==0.7.3 ipython==7.8.0 matplotlib==3.0.3 ipykernel==5.1.2 jupyter_client==5.3.4 bokeh==1.3.4 panel==0.6.0 holoviews==1.12.3 seaborn==0.9.0 plotnine==0.5.1 intake==0.5.3 intake-parquet==0.2.2 altair==3.2.0 pycodestyle==2.5.0 koalas==0.30.0 pyarrow==0.14.1 py4j==0.10.7 pyspark==2.4.5 && \
    rm -rf /root/.cache && rm -rf /var/lib/apt/lists/*

RUN echo "$LOG_TAG Install R related packages" && \
    echo "PATH: $PATH" && \
    ls /opt/conda/bin && \
    echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" | tee -a /etc/apt/sources.list && \
    gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9 && \
    gpg -a --export E084DAB9 | apt-key add - && \
    apt-get -y update && \
    apt-get -y --allow-unauthenticated --no-install-recommends install r-base r-base-dev && \
    R -e "install.packages('evaluate', repos = 'https://cloud.r-project.org')" && \
    R -e "install.packages('knitr', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('ggplot2', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('googleVis', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('data.table', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('IRkernel', repos = 'https://cloud.r-project.org');IRkernel::installspec()" && \
    R -e "install.packages('shiny', repos = 'https://cloud.r-project.org')" && \
    # for devtools, Rcpp
    apt-get -y --no-install-recommends install libcurl4-gnutls-dev libssl-dev && \
    R -e "install.packages('devtools', repos='http://cran.us.r-project.org')" && \
    R -e "install.packages('Rcpp', repos='http://cran.us.r-project.org')" && \
    Rscript -e "library('devtools'); library('Rcpp'); install_github('ramnathv/rCharts')" && \
    #Install kubectl
    apt-get install -y --no-install-recommends apt-transport-https && \
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee -a /etc/apt/sources.list.d/kubernetes.list && \
    apt-get update && \
    apt-get install --no-install-recommends -y kubectl && \
    rm -rf /var/lib/apt/lists/*

RUN echo "$LOG_TAG Cleanup" && \
    apt-get autoclean && \
    apt-get clean


ADD zeppelin-${Z_VERSION}.tar.gz /

RUN echo "$LOG_TAG Download Zeppelin binary" && \
    ln -s /zeppelin-${Z_VERSION} ${Z_HOME} && \
    mkdir -p ${Z_HOME}/logs ${Z_HOME}/run ${Z_HOME}/webapps && \
    # Allow process to edit /etc/passwd, to create a user entry for zeppelin
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd 

COPY log4j.properties ${Z_HOME}/conf/

ARG USER=zeppelin
ARG GROUP=$USER
ARG UID=1000
ARG GID=1000
RUN groupadd -g $GID $GROUP && \
    useradd --no-log-init -u $UID -g $GID $USER && \
    mkdir -p /home/zeppelin && \
    chown -R zeppelin:zeppelin /home/zeppelin && \
    # Give access to some specific folders
    chown -R zeppelin:zeppelin "${Z_HOME}/logs" "${Z_HOME}/run" "${Z_HOME}/notebook" "${Z_HOME}/conf" "${Z_HOME}/webapps" 

USER zeppelin

EXPOSE 8080

ENTRYPOINT [ "/usr/bin/tini", "--" ]
WORKDIR ${Z_HOME}
CMD ["bin/zeppelin.sh"]
