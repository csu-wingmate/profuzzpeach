
# 使用 Ubuntu 20.04 作为基础镜像
FROM ubuntu:20.04

# 设置非交互式前端，防止apt-get安装过程中出现交互式提示
ENV DEBIAN_FRONTEND=noninteractive

# 更新包列表，安装所需软件包，配置时区
RUN apt-get update && \
    apt-get install -y sudo vim && \
    echo "set nu" >>  ~/.vimrc && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
    tzdata \
    coreutils apt-utils wget curl openssl ca-certificates bash-completion \
    joe nano \
    unzip p7zip \
    fping hping3 httping thc-ipv6 gdb \
    tcpdump wireshark-common \
    locales-all \
    git build-essential strace tcpdump \
    ruby doxygen libxml2-utils less openjdk-8-jre xsltproc asciidoctor \
    nodejs node-typescript wget \
    apt-transport-https dirmngr gnupg ca-certificates apt-utils \
    cmake dos2unix \
    libglib2.0-dev libcairo2-dev \
    autoconf \
    llvm llvm-dev clang \
    && echo "tzdata tzdata/Areas select Asia" | debconf-set-selections \
    && echo "tzdata tzdata/Zones/Asia select Shanghai" | debconf-set-selections \
    && echo "wireshark-common wireshark-common/install-setuid boolean true" | debconf-set-selections \
    && ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata

# 设置时区环境变量
ENV TZ=Asia/Shanghai

# 设置工作目录
WORKDIR /root/

# 安装其他依赖包
RUN apt-get install -y \
    gnutls-dev libgnutls28-dev lcov

# 克隆 pcguard-cov 仓库
RUN git clone https://github.com/csu-wingmate/pcguard-cov.git

# 切换到 pcguard-cov 目录并解压文件
WORKDIR /root/pcguard-cov
RUN unzip -o pcguard-cov.zip
# 编译 pcguard-cov
RUN make
RUN cp collect.py /root/collect.py

# 切换到 llvm_mode 目录并编译
WORKDIR /root/pcguard-cov/llvm_mode
RUN AFL_TRACE_PC=1 make

WORKDIR /root
RUN git clone git://thekelleys.org.uk/dnsmasq.git dns
RUN cd dns && CC=/root/pcguard-cov/afl-clang-fast CXX=/root/pcguard-cov/afl-clang-fast++ AFL_USE_ASAN=1 make
RUN cd dns && CC=/root/pcguard-cov/afl-clang-fast CXX=/root/pcguard-cov/afl-clang-fast++ AFL_USE_ASAN=1 make install

WORKDIR /var/log/
RUN mkdir /var/log/dnsmasq && \
    touch /var/log/dnsmasq/dnsmasq.log

WORKDIR /root
# 容器启动命令
CMD ["bash"]
