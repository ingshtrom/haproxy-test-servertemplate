FROM hapee-plus-registry.haproxy.com/hapee-business-plus:2.2r1

RUN apt-get update \
  && apt-get install -y cron git wget lua5.3 liblua5.3-dev unzip build-essential libssl-dev software-properties-common socat dnsutils

WORKDIR /tmp
RUN wget https://luarocks.org/releases/luarocks-3.3.0.tar.gz \
  && tar zxpf luarocks-3.3.0.tar.gz \
  && cd luarocks-3.3.0 \
  && ./configure && make && make install
RUN luarocks install luasocket
RUN wget https://github.com/wahern/luaossl/releases/download/rel-20190731/luaossl-20190731-0.src.rock \
  && luarocks install luaossl-20190731-0.src.rock

COPY haproxy.cfg /etc/hapee-2.2/hapee-lb.cfg

