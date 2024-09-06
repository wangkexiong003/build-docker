FROM alpine:3.20 AS builder

ARG UWSGI_VERSION=2.0.26
ARG PYTHON_VERSION=3.10.14
ARG PYTHON_CONFIG_ARGS="--enable-shared --enable-ipv6 --with-system-expat"

WORKDIR /home
RUN apk add bash wget curl git rsync inotify-tools findutils file patch gcc make linux-headers musl-dev
RUN apk add readline-dev bzip2-dev sqlite-dev expat-dev libffi-dev zlib-dev openssl-dev xz-dev ncurses-dev \
            pcre-dev jansson-dev libxml2-dev openssl-dev libcap-dev util-linux-dev tk-dev
RUN curl -fsSL https://pyenv.run | bash
RUN wget https://github.com/unbit/uwsgi/archive/refs/tags/${UWSGI_VERSION}.tar.gz && tar xzvf ${UWSGI_VERSION}.tar.gz

#RUN touch /tmp/monitor.txt && inotifywait -qr -e "create,modify" --format "%w,%f" -d -o /tmp/monitor.txt /usr/local \
#    && bash -c "PYTHON_CONFIGURE_OPTS=\"${PYTHON_CONFIG_ARGS}\" /root/.pyenv/plugins/python-build/bin/python-build ${PYTHON_VERSION} /usr/local"
#RUN cat /tmp/monitor.txt | sort | uniq > /tmp/monitor_files.txt
#RUN while IFS=',' read -r DIR NAME; do \
#      if [ "${NAME}" = "" ]; then \
#        mkdir -p /buildfiles/${DIR}; \
#      else \
#        rsync -av "${DIR}/${NAME}" "/buildfiles/${DIR}" --mkpath; \
#      fi \
#    done < /tmp/monitor_files.txt

RUN find /usr/local ! -type d > /tmp/snapshot_before.txt
RUN bash -c "PYTHON_CONFIGURE_OPTS=\"${PYTHON_CONFIG_ARGS}\" /root/.pyenv/plugins/python-build/bin/python-build ${PYTHON_VERSION} /usr/local"
RUN find /usr/local ! -type d > /tmp/snapshot_after.txt \
    && comm -23 <(sort /tmp/snapshot_after.txt) <(sort /tmp/snapshot_before.txt) > /tmp/files_copy.txt \
    && rsync -av --files-from="/tmp/files_copy.txt" / /buildfiles

RUN bash -c "cd uwsgi-${UWSGI_VERSION} && make package && python uwsgiconfig.py --plugin plugins/python core"
RUN mkdir -p /buildfiles/usr/local/{bin,lib/uwsgi} \
    && cp /home/uwsgi-${UWSGI_VERSION}/uwsgi /buildfiles/usr/local/bin/. \
    && cp /home/uwsgi-${UWSGI_VERSION}/python_plugin.so /buildfiles/usr/local/lib/uwsgi/.
RUN find /buildfiles -name *.pyc -o -name *.pyo | xargs rm -rf
RUN find /buildfiles -exec file {} + | grep -i 'elf.*not stripped' | awk -F: '{print $1}' | xargs strip

FROM alpine:3.20
LABEL maintainer="wangkexiong"

RUN apk add --no-cache bash readline bzip2 sqlite-libs expat libffi zlib openssl xz ncurses pcre jansson libxml2 openssl libcap libuuid tk
RUN mkdir -p /app && adduser -D -H uwsgi

COPY --from=builder /buildfiles /

WORKDIR /app
EXPOSE 3031

CMD if [ -f uwsgi.ini ]; then /bin/bash -l -c "uwsgi --ini uwsgi.ini"; else /bin/bash; fi

