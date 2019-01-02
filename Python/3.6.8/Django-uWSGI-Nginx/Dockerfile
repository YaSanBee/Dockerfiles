# Copyright 2019 YaSanBee
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

# [ GitHub ] - YaSanBee/Dockerfiles/Python/3.6.8/Django-uWSGI-Nginx
#
# [ DockerHub ] - yasanbee/python-django-uwsgi-nginx
#
# VERSION [ 1.1.0 ]

FROM    python:3.6.8

MAINTAINER    YaSanBee    <YaSanBee.ZERG@gmail.com>

ENV    DOCKER_HOME /home/docker

ENV    NGINX_VERSION 1.15.8-1~stretch
ENV    NJS_VERSION   1.15.8.0.2.7-1~stretch

ADD    ./Config/* ${DOCKER_HOME}/Config/


# Install Nginx.
RUN set -x \
     && apt-get update \
     && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 apt-transport-https ca-certificates \
     && \
     NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
     found=''; \
     for server in \
          ha.pool.sks-keyservers.net \
          hkp://keyserver.ubuntu.com:80 \
          hkp://p80.pool.sks-keyservers.net:80 \
          pgp.mit.edu \
     ; do \
          echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
          apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
     done; \
     test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
     apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
     && dpkgArch="$(dpkg --print-architecture)" \
     && nginxPackages=" \
          nginx=${NGINX_VERSION} \
          nginx-module-xslt=${NGINX_VERSION} \
          nginx-module-geoip=${NGINX_VERSION} \
          nginx-module-image-filter=${NGINX_VERSION} \
          nginx-module-njs=${NJS_VERSION} \
     " \
     && case "$dpkgArch" in \
          amd64|i386) \
# arches officialy built by upstream
               echo "deb https://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list.d/nginx.list \
               && apt-get update \
               ;; \
          *) \
# we're on an architecture upstream doesn't officially build for
# let's build binaries from the published source packages
               echo "deb-src https://nginx.org/packages/mainline/debian/ stretch nginx" >> /etc/apt/sources.list.d/nginx.list \
               \
# new directory for storing sources and .deb files
               && tempDir="$(mktemp -d)" \
               && chmod 777 "$tempDir" \
# (777 to ensure APT's "_apt" user can access it too)
               \
# save list of currently-installed packages so build dependencies can be cleanly removed later
               && savedAptMark="$(apt-mark showmanual)" \
               \
# build .deb files from upstream's source packages (which are verified by apt-get)
               && apt-get update \
               && apt-get build-dep -y $nginxPackages \
               && ( \
                    cd "$tempDir" \
                    && DEB_BUILD_OPTIONS="nocheck parallel=$(nproc)" \
                         apt-get source --compile $nginxPackages \
               ) \
# we don't remove APT lists here because they get re-downloaded and removed later
               \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
# (which is done after we install the built packages so we don't have to redownload any overlapping dependencies)
               && apt-mark showmanual | xargs apt-mark auto > /dev/null \
               && { [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; } \
               \
# create a temporary local APT repo to install from (so that dependency resolution can be handled by APT, as it should be)
               && ls -lAFh "$tempDir" \
               && ( cd "$tempDir" && dpkg-scanpackages . > Packages ) \
               && grep '^Package: ' "$tempDir/Packages" \
               && echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list \
# work around the following APT issue by using "Acquire::GzipIndexes=false" (overriding "/etc/apt/apt.conf.d/docker-gzip-indexes")
#   Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
#   ...
#   E: Failed to fetch store:/var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages  Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
               && apt-get -o Acquire::GzipIndexes=false update \
               ;; \
     esac \
     \
     && apt-get install --no-install-recommends --no-install-suggests -y \
                              $nginxPackages \
                              gettext-base \
     && apt-get remove --purge --auto-remove -y apt-transport-https ca-certificates && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
     \
# if we have leftovers from building, let's purge them (including extra, unnecessary build deps)
     && if [ -n "$tempDir" ]; then \
          apt-get purge -y --auto-remove \
          && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
     fi

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
     && ln -sf /dev/stderr /var/log/nginx/error.log

#EXPOSE 80

STOPSIGNAL SIGTERM

#CMD ["nginx", "-g", "daemon off;"]


# Install Supervisor.
RUN    apt-get update \
          && apt-get install -y supervisor \
          && rm -rf /var/lib/apt/lists/*

# Setup Nginx & Supervisor
RUN    rm -rf /etc/nginx/conf.d/* \
                    && cp ${DOCKER_HOME}/Config/Nginx-uWSGI-Django.conf /etc/nginx/conf.d/ \
                    && cp ${DOCKER_HOME}/Config/Supervisor.conf /etc/supervisor/conf.d/ \
                    && echo "daemon off;" >> /etc/nginx/nginx.conf

# Install uWSGI
RUN    pip install uwsgi

# Setup uWSGI
RUN    useradd uwsgi -s /bin/false \
                    && mkdir /var/log/uwsgi \
                    && chown -R uwsgi:uwsgi /var/log/uwsgi \
                    && mkdir /var/run/ZergPools \
                    && chown -R uwsgi:uwsgi /var/run/ZergPools \
                    && mkdir /var/run/uwsgi \
                    && chown -R uwsgi:uwsgi /var/run/uwsgi

# Install Project & Django
ONBUILD    ADD    . ${DOCKER_HOME}/Project/
ONBUILD    RUN    if [ -f ${DOCKER_HOME}/Project/requirements.txt ]; \
                                            then pip install -r ${DOCKER_HOME}/Project/requirements.txt; fi

EXPOSE    8080

CMD    ["/bin/bash"]
