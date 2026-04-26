FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive
ENV container docker

# Install systemd + services
RUN apt-get update && apt-get install -y \
    systemd \
    systemd-sysv \
    openssh-server \
    openssh-client \
    python3 \
    && apt-get clean

# Remove unnecessary systemd units to reduce overhead
RUN rm -f /lib/systemd/system/multi-user.target.wants/* \
    /etc/systemd/system/*.wants/* \
    /lib/systemd/system/local-fs.target.wants/* \
    /lib/systemd/system/sockets.target.wants/*udev* \
    /lib/systemd/system/sockets.target.wants/*initctl* \
    /lib/systemd/system/basic.target.wants/* \
    /lib/systemd/system/anaconda.target.wants/* \
    /lib/systemd/system/plymouth* \
    /lib/systemd/system/systemd-update-utmp*

# Setup SSH directory and permissions
RUN mkdir -p /var/run/sshd \
    && chmod 755 /var/run/sshd

# Set root password
RUN echo 'root:root' | chpasswd

# Generate SSH host keys
RUN ssh-keygen -A

# Allow root login + password auth (Only for testing)
RUN sed -i 's/^#\?PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?UsePAM .*/UsePAM no/' /etc/ssh/sshd_config \
 && sed -i 's/^#\?PermitEmptyPasswords .*/PermitEmptyPasswords no/' /etc/ssh/sshd_config

# Copy app
COPY echo_server.py /opt/echo_server.py

# Copy systemd service files
COPY systemd/echo-server.service /etc/systemd/system/
COPY systemd/sshd.service /etc/systemd/system/

# Create symlinks for multi-user.target.wants (for manual enabling)
RUN mkdir -p /lib/systemd/system/multi-user.target.wants \
    && ln -sf /etc/systemd/system/echo-server.service /lib/systemd/system/multi-user.target.wants/echo-server.service \
    && ln -sf /etc/systemd/system/sshd.service /lib/systemd/system/multi-user.target.wants/sshd.service

EXPOSE 22 8000

STOPSIGNAL SIGRTMIN+3

CMD ["/sbin/init"]