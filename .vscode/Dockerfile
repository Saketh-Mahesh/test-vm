# Use an official Debian Bookworm as a parent image
FROM debian:bookworm

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive

# Update and install initial dependencies
RUN apt update && apt upgrade -y && \
    apt install -y curl apt-transport-https ssl-cert ca-certificates gnupg lsb-release git wget gpg \
    build-essential glibc-source zlib1g-dev libbz2-dev libcurl4-openssl-dev librdkafka-dev automake libtool \
    python3.11 python-is-python3 jupyter python3-pip python3-termcolor python3-numpy python3-matplotlib \
    imagemagick frr git open-vm-tools open-vm-tools-desktop zip bwm-ng wireshark-qt tshark sudo openssh-server vim && \
    mkdir -p /var/run/sshd
#    apt clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN rm /usr/lib/python3.11/EXTERNALLY-MANAGED

# Fix Sudo Issues
RUN echo "mininet ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    useradd -ms /bin/bash mininet && \
    echo 'mininet:mininet' | chpasswd && \
    echo 'PATH="/sbin:$PATH"' >> /home/mininet/.profile && \
    chown mininet:mininet /home/mininet/.profile && \
    chmod 755 /home/mininet

# Configure SSH to allow password authentication and Permit User Login
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#ChallengeResponseAuthentication no/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/UsePAM yes/UsePAM no/' /etc/ssh/sshd_config

# Fix Directory Issues
#RUN sed -i '131 i autologin-user=mininet' /etc/lightdm/lightdm.conf

# Install Mininet and POX
USER mininet
WORKDIR /home/mininet
RUN git clone https://github.com/mininet/mininet.git && \
    ./mininet/util/install.sh -fnpv && \
    cd /home/mininet/pox && \
    git checkout halosaur && \
    chown -R mininet:mininet /home/mininet/pox

## Install Wand
USER mininet
WORKDIR /home/mininet
RUN mkdir -p wand/src && \
    cd /home/mininet/wand/src && \
    curl -LO https://github.com/LibtraceTeam/wandio/archive/refs/tags/4.2.4-1.tar.gz && \
    tar xvzf 4.2.4-1.tar.gz && \
    cd wandio-4.2.4-1 && \
    ./bootstrap.sh && \
    ./configure && \
    make && \
    sudo make install && \
    sudo /sbin/ldconfig && \
    rm -rf /home/mininet/wand

## Install BGP Stream
WORKDIR /home/mininet
RUN mkdir -p bgp/src && \
    cd /home/mininet/bgp/src && \
    curl -LO https://github.com/CAIDA/libbgpstream/releases/download/v2.2.0/libbgpstream-2.2.0.tar.gz && \
    tar xvzf libbgpstream-2.2.0.tar.gz && \
    cd libbgpstream-2.2.0 && \
    sed -i '13436 i #define _GNU_SOURCE' configure && \
    ./configure && \
    make && \
    sudo make install && \
    sudo /sbin/ldconfig && \
    rm -rf /home/mininet/bgp

# Install and Test PyBGPStream
RUN pip install pybgpstream && \
    curl https://raw.githubusercontent.com/CAIDA/pybgpstream/master/examples/tutorial_print.py | python

USER root
EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]
