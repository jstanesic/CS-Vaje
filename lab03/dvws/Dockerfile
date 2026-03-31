FROM ubuntu:22.04

# Nastavimo okolje
ENV DEBIAN_FRONTEND=noninteractive

# Namestimo SSH strežnik in nekaj osnovnih orodij
RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    && mkdir /var/run/sshd

# Ustvari uporabnika s šibkim geslom
RUN useradd -m -s /bin/bash testuser \
    && echo 'testuser:test123' | chpasswd \
    && echo 'root:root' | chpasswd

# Omogočimo prijavo z geslom in onemogoči PAM
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

# Nastavimo SSH, da posluša na 22
EXPOSE 22

# Zaženemo SSH strežnik
CMD ["/usr/sbin/sshd", "-D"]