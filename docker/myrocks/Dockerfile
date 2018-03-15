# TODO: Clean up the directory structure, it's a mess.

# Warning: Requires at least 3.5GB to build, maybe a bit more.

#http://myrocks.io/docs/getting-started/

FROM centos:7.2.1511
#https://hub.docker.com/_/centos/
MAINTAINER Lamden Calrissian version: 0.1

# Yum prereqs for Docker
RUN yum clean all
RUN yum install -y yum-plugin-ovl
RUN yum install -y deltarpm

# Install Git
RUN yum install -y git

# Download Facebook's mysql git repo (includes myrocks)
RUN mkdir -p /opt/myrocks
RUN cd /opt/myrocks && git clone https://github.com/facebook/mysql-5.6.git
RUN cd /opt/myrocks/mysql-5.6 && git submodule init && git submodule update

# Get build deps from package management
RUN yum install -y cmake gcc-c++ bzip2-devel libaio-devel bison zlib-devel snappy-devel
RUN yum install -y gflags-devel readline-devel ncurses-devel openssl-devel lz4-devel gdb git
# Deps missing from rocksdb documentation
RUN yum install -y make boost-devel

# Build
RUN cd /opt/myrocks/mysql-5.6 && cmake . -DCMAKE_BUILD_TYPE=Debug \
-DWITH_SSL=system -DWITH_ZLIB=bundled -DMYSQL_MAINTAINER_MODE=1 \
-DENABLED_LOCAL_INFILE=1 -DENABLE_DTRACE=0 -DCMAKE_CXX_FLAGS="-march=native"
RUN cd /opt/myrocks/mysql-5.6 && make -j8
RUN cd /opt/myrocks/mysql-5.6 && make install DESTDIR="/opt/mysql"

# Instal dependency for mysql install
RUN yum install -y "perl(Data::Dumper)"

# Clean Yum cache to reduce artifact size
RUN yum clean all

# Copy config and install db
COPY ./my.cnf /etc/opt/my.cnf

RUN cd /opt/mysql/usr/local/mysql/ && ./scripts/mysql_install_db --defaults-file=/etc/opt/my.cnf

# Open mysql daemon port
EXPOSE 3306

RUN useradd -ms /bin/bash mysql
RUN chown -R mysql:mysql /opt/mysql/usr/local/mysql/data

ARG CACHEBUST=1
RUN cd /opt/mysql/usr/local/mysql/ && ./bin/mysqld_safe & \
echo "Waiting for mysqld to start..." && \
sleep 5 && \
PW=$(< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-10};echo;) && \
U='seneca_test' && \
DB='seneca_test' && \
cd /opt/mysql/usr/local/mysql/ && ./bin/mysql -e "CREATE DATABASE $DB /*\!40100 DEFAULT CHARACTER SET utf8 */;" && \
cd /opt/mysql/usr/local/mysql/ && ./bin/mysql -e "CREATE USER $U@localhost IDENTIFIED BY '$PW';" && \
cd /opt/mysql/usr/local/mysql/ && ./bin/mysql -e "GRANT ALL ON *.* to $U@'%' IDENTIFIED BY '$PW';" && \
cd /opt/mysql/usr/local/mysql/ && ./bin/mysql -e "FLUSH PRIVILEGES;" && \
P=$(ps aux | grep /bin/mysqld | grep -v grep | awk '{print $2}') && \
kill -9 $P && \
echo "" && \
echo "Listening on 3306, login with mysql --protocol=TCP -u $U --password=$PW" && \
echo "Kill with 'docker kill', mysql is very stubborn, it doesn't close on sigint/sigkill" && \
echo ""

CMD cd /opt/mysql/usr/local/mysql/ && ./bin/mysqld_safe

#mysql --protocol=TCP -u test --password=test
#docker run -it --entrypoint /bin/bash temp
#docker kill `docker ps --format "table {{.Names}}" --filter "ancestor=temp" | tail -n +2`