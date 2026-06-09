FROM frrouting/frr:latest

RUN apk add --no-cache openssh && \
    ssh-keygen -A && \
    adduser -D oxidized && \
    echo "oxidized:oxidized123" | chpasswd && \
    echo "PermitRootLogin yes" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "" > /etc/motd

# Script de arranque que lanza SSH y FRR
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
