FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && apt-get install -y \
    openssh-server \
    python3 \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install execline
RUN apt-get update && apt-get install -y execline

# Setup SSH
RUN mkdir /var/run/sshd
RUN echo 'root:root' | chpasswd

# Generate SSH host keys
RUN ssh-keygen -A

# Allow root login + password auth (Only for testing)
RUN sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?UsePAM .*/UsePAM no/' /etc/ssh/sshd_config

# Install s6-overlay
ADD https://github.com/just-containers/s6-overlay/releases/latest/download/s6-overlay-noarch.tar.xz /tmp/
ADD https://github.com/just-containers/s6-overlay/releases/latest/download/s6-overlay-x86_64.tar.xz /tmp/

RUN apt-get update && apt-get install -y xz-utils \
    && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
    && tar -C / -Jxpf /tmp/s6-overlay-x86_64.tar.xz


# Copy services
COPY s6-overlay/ /etc/s6-overlay/
COPY echo_server.py /app/echo_server.py

# Expose ports
EXPOSE 22 8000

# Use s6 init
ENTRYPOINT ["/init"]