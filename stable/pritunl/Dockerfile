FROM ubuntu:20.04

RUN apt-get update -q &&\
    apt-get install -y apt-transport-https ca-certificates gnupg &&\
    echo 'deb https://repo.pritunl.com/stable/apt focal main' >> /etc/apt/sources.list.d/pritunl.list &&\
    echo 'deb https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.2 multiverse' >> /etc/apt/sources.list.d/mongodb-org-4.2.list &&\
    apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv E162F504A20CDF15827F718D4B7C549A058F8B6B &&\
    apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A &&\
    apt-get update -q &&\
    apt-get install -y locales &&\
    locale-gen en_US en_US.UTF-8 &&\
    dpkg-reconfigure locales &&\
    ln -sf /usr/share/zoneinfo/UTC /etc/localtime &&\
    apt-get update -q &&\
    apt-get install -y pritunl mongodb iptables &&\
    apt-get -y -q autoclean &&\
    apt-get -y -q autoremove

ADD start_pritunl /tmp

EXPOSE 80 443 1194

ENTRYPOINT ["/tmp/start_pritunl"]