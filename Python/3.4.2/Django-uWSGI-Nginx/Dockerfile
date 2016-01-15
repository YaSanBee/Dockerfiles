# Copyright 2016 YaSanBee
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# YaSanBee/Dockerfiles/Python/3.4.2/Django-uWSGI-Nginx
#
# VERSION [ 0.0.1 ]

FROM    python:3.4.2

MAINTAINER    YaSanBee    <YaSanBee.ZERG@gmail.com>

ENV    DOCKER_HOME /home/docker

ADD    ./Config/* ${DOCKER_HOME}/Config

# Install Nginx
RUN    sudo apt-key add ${DOCKER_HOME}/Config/nginx_signing.key \
                    && echo "deb http://nginx.org/packages/debian/ jessie nginx" >> /etc/apt/sources.list \
                    && echo "deb-src http://nginx.org/packages/debian/ jessie nginx" >> /etc/apt/sources.list \
                    && apt-get update \
                    && apt-get install -y nginx \
                    && rm -rf /var/lib/apt/lists/*

# Install uWSGI
RUN    pip install uwsgi

# Install Project
ONBUILD    ADD    . ${DOCKER_HOME}/Project
ONBUILD    RUN    if [ -f ${DOCKER_HOME}/Project/requirements.txt ]; \
                                            then pip install -r ${DOCKER_HOME}/Project/requirements.txt; fi

EXPOSE    80

CMD    ["supervisord", "-n"]
