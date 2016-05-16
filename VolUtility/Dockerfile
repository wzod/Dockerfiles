#
# This Docker image encapsulates the VolUtility Web Interface for Volatility 
# Memory Analysis framework by Kevin Breen from 
# https://github.com/kevthehermit/VolUtility
#
# VolUtility provides a web interface for the Volatility Framework (version 2.5) by The 
# Volatility Foundation from http://www.volatilityfoundation.org/#!releases/component_71401
#
# To run this image after installing Docker, use the following command:
# sudo docker run --rm -it -v ~/memdumps:/home/nonroot/memdumps remnux/volutility bash
#
# To access the VolUtility  web interface from outside the Docker container, use the command:
#
# sudo docker run --rm -it -v ~/memdumps:/home/nonroot/memdumps -p 8000:8000 remnux/volutility bash
#
# Before running VolUtility, create the ~/memdumps directory and make it world-accessible
# (“chmod a+xwr").

FROM remnux/volatility
MAINTAINER Zod (@wzod)

ENV DEBIAN_FRONTEND noninteractive
ENV LC_ALL C

USER root
RUN apt-get -qq update && apt-get install -y software-properties-common && \
  apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927 && \
  echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list && \
  apt-add-repository -y universe && \
  apt-get -qq update && apt-get install -y --fix-missing \
  git \
  mongodb-org \
  python-dev \ 
  python-pip

# Install additional dependencies
RUN cd /tmp && \
  pip install django \
  distorm3 \ 
  pymongo \ 
  pycrypto

# Setup Volatility and environment
RUN cd /home/nonroot/volatility-2.5 && \
  python setup.py build && \
  python setup.py install && \
  bash -c 'mkdir -pv /data/{db,logs}' && \
  chown -R nonroot /data

# Clone VolUtility from GitHub repo and load default configuration
USER nonroot
RUN cd /home/nonroot && \
  git clone https://github.com/kevthehermit/VolUtility && \
  cd VolUtility && \
  cp volutility.conf.sample volutility.conf

# Clean up
USER root
RUN ldconfig && \
  apt-get remove -y --purge build-essential libtool && \
  apt-get autoremove -y --purge && \
  apt-get clean -y && \
  rm -rf /tmp/* && \
  rm -rf /var/lib/apt/lists/*

# Define mountable directories.
VOLUME ["/data/db"]

# Expose ports
EXPOSE 27017

USER nonroot
ENV HOME /home/nonroot
ENV USER nonroot
WORKDIR /home/nonroot/VolUtility

ENTRYPOINT /usr/bin/mongod --fork --logpath /data/logs/VolUtility.log --logappend --dbpath /data/db && /bin/bash
