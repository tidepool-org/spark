#
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
#

ARG base_img

FROM $base_img
WORKDIR /

# Reset to root to run installation tasks
USER 0

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN true

RUN mkdir ${SPARK_HOME}/python
# TODO: Investigate running both pip and pip3 via virtualenvs
RUN apt-get install -y build-essential wget
RUN apt-get install -y libreadline-gplv2-dev libncursesw5-dev libssl-dev \
    libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev
RUN cd /opt && wget https://www.python.org/ftp/python/3.8.5/Python-3.8.5.tgz && \
    tar xf Python-3.8.5.tgz \
    && cd Python-3.8.5 \
    && sed -i 's/$(MAKE) run_profile_task//' Makefile.pre.in \
    && ./configure --enable-optimizations --with-ensurepip=install \
    && make \
    && make install \
    && cd .. 
RUN rm -rf Python-3.8.5 \
    && rm /opt/Python-3.8.5.tgz \
    && ln -sf /usr/local/bin/python3 /usr/bin/python \
    && ln -s /usr/local/bin/pip3 /usr/bin/pip \
    && pip install --upgrade pip \
    && apt-get -y purge $(diff -u init_pkgs new_pkgs | grep -E "^\+" | cut -d + -f2- | sed -n '1!p') \
    && apt-get clean \
    && rm -rf \
    /root/.cache/pip \
    && find / -type d -name __pycache__ -exec rm -r {} \+
RUN pip install --upgrade pip setuptools && \
    rm -r /root/.cache && rm -rf /var/cache/apt/*

COPY python/pyspark ${SPARK_HOME}/python/pyspark
COPY python/lib ${SPARK_HOME}/python/lib
ENV PYSPARK_MAJOR_PYTHON_VERSION 3

WORKDIR /opt/spark/work-dir
ENTRYPOINT [ "/opt/entrypoint.sh" ]

# Specify the User that the actual main process will run as
ARG spark_uid=185
USER ${spark_uid}
