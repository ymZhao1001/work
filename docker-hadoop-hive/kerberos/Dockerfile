#FROM debian:jessie
FROM ubuntu:20.04

ENV REALM=EXAMPLE.COM

# Install necessary dependencies
ENV DEBIAN_FRONTEND=noninteractive
RUN set -ex && \
   # sed -i 's#http://deb.debian.org#https://mirrors.ustc.edu.cn#g' /etc/apt/sources.list && \
    #sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends krb5-kdc krb5-admin-server

# Add configurations
COPY kerberos/conf/ /etc/
RUN mkdir -p /var/log/kerberos/

# Add entrypoint
COPY kerberos/entrypoint.sh /tmp/entrypoint.sh
RUN chmod +x /tmp/entrypoint.sh
ENTRYPOINT ["/tmp/entrypoint.sh"]

EXPOSE 749 750 88
