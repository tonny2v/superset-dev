FROM tonny2v/superset-python-nodejs
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

#COPY ./requirements.txt ./
#RUN pip3 install -r requirements.txt

# Create a group and user
RUN addgroup -S superset && adduser -S superset -G superset

# Configure environment
ENV LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# Install extra useful tool for development
RUN apk del openssl openssl-dev # conficts with mariadb-dev (libressl-dev)
RUN apk add --no-cache postgresql-client postgresql-dev redis mariadb-client mariadb-dev

WORKDIR /home/superset

COPY ./requirements.txt ./
COPY ./requirements-dev.txt ./

RUN pip3 install --upgrade setuptools pip \
    && pip3 install -r requirements.txt -r requirements-dev.txt \
    && rm -rf /root/.cache/pip

# Celery 4.2 does not support python 3.7, use master instead
RUN apk add git bash
RUN pip3 install --upgrade git+https://github.com/celery/celery.git

USER superset

COPY --chown=superset:superset superset superset

ENV PATH=/home/superset/superset/bin:$PATH \
    PYTHONPATH=/home/superset/superset/:$PYTHONPATH

RUN cd superset/assets \
    && npm ci \
    && npm run sync-backend \
    && npm run build \
    && rm -rf node_modules

COPY contrib/docker/docker-init.sh .
COPY contrib/docker/docker-entrypoint.sh /entrypoint.sh


ENTRYPOINT ["/entrypoint.sh"]

#HEALTHCHECK CMD ["curl", "-f", "http://localhost:8088/health"]

EXPOSE 8088
