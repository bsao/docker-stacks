# Copyright (c) Jupyter Development Team.
FROM bsao/minimal-notebook

MAINTAINER Robson Júnior <robson@kimmay.com.br>

USER root

# Spark dependencies
ENV APACHE_SPARK_VERSION 1.6.0
RUN apt-get -y update
RUN apt-get -y install ca-certificates-java
RUN apt-get install -y --no-install-recommends openjdk-7-jre-headless && \
    apt-get clean
RUN wget -qO - http://d3kbcqa49mib13.cloudfront.net/spark-${APACHE_SPARK_VERSION}-bin-hadoop2.4.tgz | tar -xz -C /usr/local/
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop2.4 spark

RUN mkdir /usr/local/spark-classpath
RUN wget -O /usr/local/spark-classpath/elasticsearch-hadoop.jar http://central.maven.org/maven2/org/elasticsearch/elasticsearch-hadoop/2.2.0-m1/elasticsearch-hadoop-2.2.0-m1.jar
RUN wget -O /usr/local/spark-classpath/RedshiftJDBC41.jar https://s3.amazonaws.com/redshift-downloads/drivers/RedshiftJDBC41-1.1.10.1010.jar
RUN wget -O /usr/local/spark-classpath/spark-redshift.jar http://central.maven.org/maven2/com/databricks/spark-redshift_2.10/0.5.2/spark-redshift_2.10-0.5.2.jar
RUN wget -O /usr/local/spark-classpath/postgresql.jar http://central.maven.org/maven2/org/postgresql/postgresql/9.4-1206-jdbc42/postgresql-9.4-1206-jdbc42.jar

# Mesos dependencies
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF && \
    DISTRO=debian && \
    CODENAME=wheezy && \
    echo "deb http://repos.mesosphere.io/${DISTRO} ${CODENAME} main" > /etc/apt/sources.list.d/mesosphere.list && \
    apt-get -y update && \
    apt-get --no-install-recommends -y --force-yes install mesos=0.22.1-1.0.debian78 && \
    apt-get clean

# Spark and Mesos pointers
ENV SPARK_HOME /usr/local/spark
ENV R_LIBS_USER $SPARK_HOME/R/lib
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.8.2.1-src.zip
ENV SPARK_CLASSPATH /usr/local/spark-classpath/elasticsearch-hadoop.jar:/usr/local/spark-classpath/RedshiftJDBC41.jar:/usr/local/spark-classpath/spark-redshift.jar:/usr/local/spark-classpath/postgresql.jar
ENV MESOS_NATIVE_LIBRARY /usr/local/lib/libmesos.so

# R pre-requisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libxrender1 \
    fonts-dejavu \
    gfortran \
    gcc && apt-get clean

USER bsao

# Install Python 3 packages
RUN conda install --yes \
    'ipywidgets=4.0*' \
    'pandas=0.17*' \
    'matplotlib=1.5*' \
    'scipy=0.16*' \
    'seaborn=0.6*' \
    'scikit-learn=0.17*' \
    'bokeh=0.10*' \
    && conda clean -yt

# Install Python 2 packages
RUN conda create -p $CONDA_DIR/envs/python2 python=2.7 \
    'ipython=4.0*' \
    'ipywidgets=4.0*' \
    'pandas=0.17*' \
    'matplotlib=1.5*' \
    'scipy=0.16*' \
    'seaborn=0.6*' \
    'scikit-learn=0.17*' \
    'bokeh=0.10*' \
    pyzmq \
    && conda clean -yt

# R packages
RUN conda config --add channels r
RUN conda install --yes \
    'r-base=3.2*' \
    'r-irkernel=0.4*' \
    'r-ggplot2=1.0*' \
    'r-rcurl=1.95*' && conda clean -yt

# WORKAROUND: symlink version of zmq required by latest rzmq back into conda lib
# https://github.com/jupyter/docker-stacks/issues/55
RUN ln -s /opt/conda/pkgs/zeromq-4.0.*/lib/libzmq.so.4.* /opt/conda/lib/libzmq.so.4
RUN ln -s /opt/conda/pkgs/libsodium-0.4.*/lib/libsodium.so.4.* /opt/conda/lib/libsodium.so.4

USER root

# Install Python 2 kernel spec globally to avoid permission problems when NB_UID
# switching at runtime.
RUN $CONDA_DIR/envs/python2/bin/python \
    $CONDA_DIR/envs/python2/bin/ipython \
    kernelspec install-self
