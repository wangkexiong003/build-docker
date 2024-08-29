FROM ubuntu:24.04 AS builder

ARG UWSGI_VERSION=2.0.26
ARG PYTHON_VERSION=3.10.14
ARG PYTHON_CONFIG_ARGS="--enable-shared --enable-ipv6 --with-system-expat"

WORKDIR /home
RUN apt-get update
RUN apt-get install -y make build-essential wget curl llvm git file rsync inotify-tools patch
RUN apt-get install -y libssl-dev zlib1g-dev libexpat1-dev libbz2-dev libreadline-dev \
                       libsqlite3-dev libncurses5-dev libffi-dev liblzma-dev xz-utils \
                       libpcre3-dev libssl-dev libcap-dev libxml2-dev libjansson-dev \
                       uuid-dev tk-dev
RUN curl -fsSL https://pyenv.run | bash
RUN wget https://github.com/unbit/uwsgi/archive/refs/tags/${UWSGI_VERSION}.tar.gz && tar xzvf uwsgi-${UWSGI_VERSION}.tar.gz

RUN inotifywait -m -r -e create,modify,delete /usr/local | while read path action file; do \
      mkdir -p "/buildfiles${path}"; rsync -av "${path}/${file}" /buildfiles${path}; \
    done 1>/dev/null &
RUN PYTHON_CONFIGURE_OPTS="${PYTHON_CONFIG_ARGS}" /root/.pyenv/plugins/python-build/bin/python-build ${PYTHON_VERSION} /usr/local
RUN cd uwsgi-${UWSGI_VERSION} && make package && python uwsgiconfig.py --plugin plugins/python core

RUN mkdir -p /buildfiles/usr/local/bin /buildfiles/usr/local/lib/uwsgi
RUN cp /home/uwsgi-${UWSGI_VERSION}/uwsgi /buildfiles/usr/local/bin/. && \
    cp /home/uwsgi-${UWSGI_VERSION}/python_plugin.so /buildfiles/usr/local/lib/uwsgi/.
RUN find /buildfiles -name *.pyc -o -name *.pyo | xargs rm -rf
RUN find /buildfiles -exec file {} + | grep -i 'elf.*not stripped' | awk -F: '{print $1}' | xargs strip

FROM ubuntu:24.04
LABEL maintainer=wangkexiong

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive && export DEBCONF_NONINTERACTIVE_SEEN=true \
    && apt-get install -y libssl3 zlib1g libexpat1 libbz2-1.0 libreadline8 libsqlite3-0 libncurses5 \
                          libffi8 liblzma5 libpcre3 libcap2 libxml2 libjansson4 libuuid1 libtk \
    && apt-get autoremove -y && apt-get clean all \
    && rm -rf /var/{cache,log,lib} /tmp/*
RUN mkdir -p /app && useradd -M uwsgi

COPY --from=builder /buildfiles /

WORKDIR /app
EXPOSE 3031

CMD if [ -f uwsgi.ini ]; then /bin/bash -l -c "uwsgi --ini uwsgi.ini"; else /bin/bash; fi
