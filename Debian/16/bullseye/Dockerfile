# vim:set ft=dockerfile:
#
# Copyright The CloudNativePG Contributors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM postgres:16.6-bullseye

# Do not split the description, otherwise we will see a blank space in the labels
LABEL name="PostgreSQL Container Images" \
      vendor="The CloudNativePG Contributors" \
      version="${PG_VERSION}" \
      release="25" \
      summary="PostgreSQL Container images." \
      description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 16.6-bullseye."

LABEL org.opencontainers.image.description="This Docker image contains PostgreSQL and Barman Cloud based on Postgres 16.6-bullseye."

COPY requirements.txt /

# Install additional extensions
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		"postgresql-${PG_MAJOR}-pgaudit" \
		"postgresql-${PG_MAJOR}-pgvector" \
		"postgresql-${PG_MAJOR}-pg-failover-slots" \
	; \
	rm -fr /tmp/* ; \
	rm -rf /var/lib/apt/lists/*;

# Install barman-cloud
RUN set -xe; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		python3-pip \
		python3-psycopg2 \
		python3-setuptools \
	; \
	pip3 install  --upgrade pip; \
	# TODO: Remove --no-deps once https://github.com/pypa/pip/issues/9644 is solved
	pip3 install  --no-deps -r requirements.txt; \
	rm -rf /var/lib/apt/lists/*;

# NOTE: this is intended for testing only, not for production use. the imate
# is very large because it includes the full toolchain for building pgsentinel
# from source.

# Set the working directory in the container
WORKDIR /usr/src/pgsentinel
# Install packages to build pgsentinel
RUN set -xe; \
    apt-get update; apt-get install -y --no-install-recommends build-essential curl postgresql-server-dev-${PG_MAJOR}; \
    curl -o pgsentinel.tar.gz -L https://github.com/pgsentinel/pgsentinel/archive/0218c2147daab0d2dbbf08433cb480163d321839.tar.gz; \
    tar -xzf pgsentinel.tar.gz --strip-components=1; \
    cd src; \
    make; \
    make install


# Change the uid of postgres to 26
RUN usermod -u 26 postgres
USER 26
