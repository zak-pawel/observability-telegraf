# MIT License
# 
# Copyright (c) 2021 Intel Corporation
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.
# 

ARG TELEGRAF_TAG
ARG FINAL_BASE_IMAGE
FROM telegraf:$TELEGRAF_TAG AS builder

RUN apk update && \
    apk add --update --no-cache \
    build-base~=0.5 \
    bash~=5.2 \
    sudo~=1.9 \
    git~=2.40 \
    alpine-sdk~=1.0 \
    libffi-dev~=3.4 \
    openssl-dev~=3.1 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR "/telegraf_docker/"

COPY ./images/telegraf/script/install_intel_cmt_cat.sh ./

RUN ./install_intel_cmt_cat.sh

# Build final image where telegraf is running.
FROM $FINAL_BASE_IMAGE

RUN set -eux && \
    export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" && \
    apk update && \
    apk add --update --no-cache \
        util-linux~=2.38 \
        bash~=5.2 \
        sudo~=1.9 \
        smartmontools~=7.3 \
        nvme-cli~=2.4 \
        rasdaemon~=0.8 \
        iptables~=1.8 \
        ipmitool~=1.8 \
        su-exec~=0.2 \
        libcap~=2.69 \
        iputils~=20221126 && \
        rm -rf /var/lib/apt/lists/*

RUN addgroup --system telegraf && \
    adduser --system -G telegraf telegraf

WORKDIR "/telegraf_docker/"

COPY ./DEPENDENCIES_SOURCE_CODE_INFO.md .

# Copy dependecies from previous step called builder, where intel-cmt-cat is installed.
COPY --from=builder /usr/local/lib/libpqos.* /usr/lib/
COPY --from=builder /usr/local/bin/ /usr/local/bin/
COPY --from=builder /usr/share/snmp/mibs/ /usr/share/snmp/mibs/
COPY --from=builder /usr/bin/telegraf /usr/bin/telegraf

COPY ./images/telegraf/script/config_privileges.sh .
RUN ./config_privileges.sh

COPY ./images/telegraf/script/intel_entrypoint.sh .

# Override entrypoint
ENTRYPOINT ["sh", "-c", "rasdaemon -r && sleep 10s && ./intel_entrypoint.sh telegraf"]
