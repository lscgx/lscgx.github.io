### centos7 下安装 SS

#### 1、 安装 cmake

> ```shell
> yum install -y gcc gcc-c++ make automake
> yum install -y wget
> wget http://www.cmake.org/files/v2.8/cmake-2.8.10.2.tar.gz
> tar -zxvf cmake-2.8.10.2.tar.gz
> cd cmake-2.8.10.2
> ./bootstrap
> gmake
> gmake install
> ```

#### 2 、安装 libsodium

>```shell
>yum install m2crypto gcc -y
>wget -N --no-check-certificate https://github.com/jedisct1/libsodium/releases/download/1.0.18-RELEASE/libsodium-1.0.18.tar.gz
>tar zfvx libsodium-1.0.18.tar.gz
>cd libsodium-1.0.18
>./configure
>make && make install
>echo "include ld.so.conf.d/*.conf" > /etc/ld.so.conf
>echo "/lib" >> /etc/ld.so.conf
>echo "/usr/lib64" >> /etc/ld.so.conf
>echo "/usr/local/lib" >> /etc/ld.so.conf
>ldconfig
>cd ..
>```

#### 3、安装 python

**直接安装**

>```shell
>yum install python36
>ln -s /usr/bin/python3 /usr/local/bin/python
>```

**源码安装**

>```shell
>#安装编译相关工具
>yum -y groupinstall "Development tools" 
>yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel 
>yum install libffi-devel -y
>```
>
>下载安装包解压（视情况下载 2.7 版本）
>
>```shell
>cd #回到用户目录
>wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
>tar -xvJf  Python-3.7.0.tar.xz
>```
>
>```shell
>yum -y install zlib* # 可选
>```
>
>编译安装
>
>```shell
>mkdir /usr/local/python3 #创建编译安装目录
>cd Python-3.7.0rm -
>./configure --prefix=/usr/local/python3
>make && make install
>```
>
>创建软连接
>
>```shell
>ln -s /usr/local/python3/bin/python3 /usr/local/bin/python#3
>ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip#3
>```
>
>软连接操作（可选）
>
>```shell
>ln -s  源文件  软连接名    #新增
>ln –snf  源文件    软连接  #修改
>rm -rf 软连接名			  #删除软连接
>rm -rf 源文件             #删除源文件
>```
>
>验证是否成功
>
>```shell
>python3 -V
>pip3 -V
>```
>
>安装 setuptools
>
>```shell
>pip install setuptools
># or
>wget https://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz
>tar zxvf setuptools-0.6c11.tar.gz
>cd setuptools-0.6c11
>python setup.py install
>```
>
>

#### 4 、安装 ss

> ```shell
> wget --no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
> # https://lscgx.github.io/files/shadowsocks.sh
> chmod +x shadowsocks.sh
> ./shadowsocks.sh 2>&1 | tee shadowsocks.log
> ```
>
> > 上面脚本中检测 /usr/bin/ssserver 和 /usr/local/bin/ssserver 
> >
> > ss 安装到当前 python 下，所以如果安装python的时候自定义了路径，则需要修改脚本或修改python安装路径使之相同。 
>
> 配置
>
> ```shell
> vi /etc/shadowsocks.json
> ```
>
> ```json
> {
>     "server":"0.0.0.0",
>     "local_address":"0.0.0.0", 
>     "local_port":1080,
>     "port_password":{
>          "40000":"lcg0407",
>          "40001":"lcg0407",
>          "40002":"lcg0407",
>          "40003":"lcg0407",
>          "40004":"lcg0407",
>          "40005":"lcg0407",
>          "40006":"lcg0407",
>          "40007":"lcg0407"
>     },
>     "timeout":300,
>     "method":"aes-256-cfb",
>     "fast_open": false
> }
> ```
>
> 
>
> 命令
>
> ```shell
> 启动：/etc/init.d/shadowsocks start
> 停止：/etc/init.d/shadowsocks stop
> 重启：/etc/init.d/shadowsocks restart
> 状态：/etc/init.d/shadowsocks status
> ```

#### 5、BBR加速

> ```shell
> wget — no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
> chmod +x bbr.sh
> ./bbr.sh
> ```
>
> 安装完后，按提示重启 VPS，输入 Y 回车重启。稍候 1min 等待重启完成，再重新连接 Xshell。
>
> 重启后输入 `lsmod | grep bbr` ，出现 tcp_bbr 即说明 BBR 已经启动。

#### 6 、配置 iptables

> ```shell
> systemctl stop firewalld
> systemctl mask firewalld
> # 安装iptables-services
> yum install iptables-services
> # 设置开机启动
> systemctl enable iptables
> ```

> ```shell
> #查看iptables现有规则
> iptables -L -n
> #先允许所有,不然有可能会杯具
> iptables -P INPUT ACCEPT
> #清空所有默认规则
> iptables -F
> #清空所有自定义规则
> iptables -X
> #所有计数器归0
> iptables -Z
> #允许来自于lo接口的数据包(本地访问)
> iptables -A INPUT -i lo -j ACCEPT
> #开放22端口
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> #开放21端口(FTP)
> iptables -A INPUT -p tcp --dport 21 -j ACCEPT
> #开放80端口(HTTP)
> iptables -A INPUT -p tcp --dport 80 -j ACCEPT
> #开放443端口(HTTPS)
> iptables -A INPUT -p tcp --dport 443 -j ACCEPT
> #允许ping
> iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
> #允许接受本机请求之后的返回数据 RELATED,是为FTP设置的
> iptables -A INPUT -m state --state  RELATED,ESTABLISHED -j ACCEPT
> #其他入站一律丢弃
> iptables -P INPUT DROP
> #所有出站一律绿灯
> iptables -P OUTPUT ACCEPT
> #所有转发一律丢弃
> iptables -P FORWARD DROP
> 
> #如果要添加内网ip信任（接受其所有TCP请求）
> iptables -A INPUT -p tcp -s 45.96.174.68 -j ACCEPT
> #过滤所有非以上规则的请求
> iptables -P INPUT DROP
> #要封停一个IP，使用下面这条命令：
> iptables -I INPUT -s ***.***.***.*** -j DROP
> #要解封一个IP，使用下面这条命令:
> iptables -D INPUT -s ***.***.***.*** -j DROP
> 
> #保存上述规则
> service iptables save
> 
> #注册iptables服务
> #相当于以前的chkconfig iptables on
> systemctl enable iptables.service
> #开启服务
> systemctl start iptables.service
> #查看状态
> systemctl status iptables.service
> 
> ```
>
> 直接复制版本
>
> ```shell
> #!/bin/sh
> iptables -P INPUT ACCEPT
> iptables -F
> iptables -X
> iptables -Z
> iptables -A INPUT -i lo -j ACCEPT
> iptables -A INPUT -p tcp --dport 22 -j ACCEPT
> iptables -A INPUT -p tcp --dport 21 -j ACCEPT
> iptables -A INPUT -p tcp --dport 80 -j ACCEPT
> iptables -A INPUT -p tcp --dport 443 -j ACCEPT
> iptables -A INPUT -p tcp --dport 40000:40100 -j ACCEPT
> iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
> iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
> iptables -P INPUT DROP
> iptables -P OUTPUT ACCEPT
> iptables -P FORWARD DROP
> service iptables save
> systemctl restart iptables.service
> ```
>
> 检查端口情况
>
> ```shell 
> yum install -y telnet
> telnet **.**.**.** port
> ```

#### 7 、 shadows.sh 脚本

```shell
#!/usr/bin/env bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
#=================================================================#
#   System Required:  CentOS 6+, Debian 7+, Ubuntu 12+            #
#   Description: One click Install Shadowsocks-Python server      #
#   Author: Teddysun <i@teddysun.com>                             #
#   Thanks: @clowwindy <https://twitter.com/clowwindy>            #
#   Intro:  https://teddysun.com/342.html                         #
#=================================================================#

clear
echo
echo "#############################################################"
echo "# One click Install Shadowsocks-Python server               #"
echo "# Intro: https://teddysun.com/342.html                      #"
echo "# Author: Teddysun <i@teddysun.com>                         #"
echo "# Github: https://github.com/shadowsocks/shadowsocks        #"
echo "#############################################################"
echo

# Current folder
cur_dir=`pwd`
# Stream Ciphers
ciphers=(
aes-256-gcm
aes-192-gcm
aes-128-gcm
aes-256-ctr
aes-192-ctr
aes-128-ctr
aes-256-cfb
aes-192-cfb
aes-128-cfb
camellia-128-cfb
camellia-192-cfb
camellia-256-cfb
chacha20-ietf-poly1305
chacha20-ietf
chacha20
rc4-md5
)
# Color
red='\033[0;31m'
green='\033[0;32m'
yellow='\033[0;33m'
plain='\033[0m'

# Make sure only root can run our script
[[ $EUID -ne 0 ]] && echo -e "[${red}Error${plain}] This script must be run as root!" && exit 1

# Disable selinux
disable_selinux(){
    if [ -s /etc/selinux/config ] && grep 'SELINUX=enforcing' /etc/selinux/config; then
        sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
        setenforce 0
    fi
}

#Check system
check_sys(){
    local checkType=$1
    local value=$2

    local release=''
    local systemPackage=''

    if [[ -f /etc/redhat-release ]]; then
        release="centos"
        systemPackage="yum"
    elif cat /etc/issue | grep -Eqi "debian"; then
        release="debian"
        systemPackage="apt"
    elif cat /etc/issue | grep -Eqi "ubuntu"; then
        release="ubuntu"
        systemPackage="apt"
    elif cat /etc/issue | grep -Eqi "centos|red hat|redhat"; then
        release="centos"
        systemPackage="yum"
    elif cat /proc/version | grep -Eqi "debian"; then
        release="debian"
        systemPackage="apt"
    elif cat /proc/version | grep -Eqi "ubuntu"; then
        release="ubuntu"
        systemPackage="apt"
    elif cat /proc/version | grep -Eqi "centos|red hat|redhat"; then
        release="centos"
        systemPackage="yum"
    fi

    if [[ ${checkType} == "sysRelease" ]]; then
        if [ "$value" == "$release" ]; then
            return 0
        else
            return 1
        fi
    elif [[ ${checkType} == "packageManager" ]]; then
        if [ "$value" == "$systemPackage" ]; then
            return 0
        else
            return 1
        fi
    fi
}

# Get version
getversion(){
    if [[ -s /etc/redhat-release ]]; then
        grep -oE  "[0-9.]+" /etc/redhat-release
    else
        grep -oE  "[0-9.]+" /etc/issue
    fi
}

# CentOS version
centosversion(){
    if check_sys sysRelease centos; then
        local code=$1
        local version="$(getversion)"
        local main_ver=${version%%.*}
        if [ "$main_ver" == "$code" ]; then
            return 0
        else
            return 1
        fi
    else
        return 1
    fi
}

# Get public IP address
get_ip(){
    local IP=$( ip addr | egrep -o '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | egrep -v "^192\.168|^172\.1[6-9]\.|^172\.2[0-9]\.|^172\.3[0-2]\.|^10\.|^127\.|^255\.|^0\." | head -n 1 )
    [ -z ${IP} ] && IP=$( wget -qO- -t1 -T2 ipv4.icanhazip.com )
    [ -z ${IP} ] && IP=$( wget -qO- -t1 -T2 ipinfo.io/ip )
    [ ! -z ${IP} ] && echo ${IP} || echo
}

get_char(){
    SAVEDSTTY=`stty -g`
    stty -echo
    stty cbreak
    dd if=/dev/tty bs=1 count=1 2> /dev/null
    stty -raw
    stty echo
    stty $SAVEDSTTY
}

# Pre-installation settings
pre_install(){
    if check_sys packageManager yum || check_sys packageManager apt; then
        # Not support CentOS 5
        if centosversion 5; then
            echo -e "$[{red}Error${plain}] Not supported CentOS 5, please change to CentOS 6+/Debian 7+/Ubuntu 12+ and try again."
            exit 1
        fi
    else
        echo -e "[${red}Error${plain}] Your OS is not supported. please change OS to CentOS/Debian/Ubuntu and try again."
        exit 1
    fi
    # Set shadowsocks config password
    echo "Please input password for shadowsocks-python"
    read -p "(Default password: teddysun.com):" shadowsockspwd
    [ -z "${shadowsockspwd}" ] && shadowsockspwd="teddysun.com"
    echo
    echo "---------------------------"
    echo "password = ${shadowsockspwd}"
    echo "---------------------------"
    echo
    # Set shadowsocks config port
    while true
    do
    echo "Please input port for shadowsocks-python [1-65535]"
    read -p "(Default port: 8989):" shadowsocksport
    [ -z "$shadowsocksport" ] && shadowsocksport="8989"
    expr ${shadowsocksport} + 1 &>/dev/null
    if [ $? -eq 0 ]; then
        if [ ${shadowsocksport} -ge 1 ] && [ ${shadowsocksport} -le 65535 ]; then
            echo
            echo "---------------------------"
            echo "port = ${shadowsocksport}"
            echo "---------------------------"
            echo
            break
        else
            echo -e "[${red}Error${plain}] Input error, please input a number between 1 and 65535"
        fi
    else
        echo -e "[${red}Error${plain}] Input error, please input a number between 1 and 65535"
    fi
    done

    # Set shadowsocks config stream ciphers
    while true
    do
    echo -e "Please select stream cipher for shadowsocks-python:"
    for ((i=1;i<=${#ciphers[@]};i++ )); do
        hint="${ciphers[$i-1]}"
        echo -e "${green}${i}${plain}) ${hint}"
    done
    read -p "Which cipher you'd select(Default: ${ciphers[0]}):" pick
    [ -z "$pick" ] && pick=1
    expr ${pick} + 1 &>/dev/null
    if [ $? -ne 0 ]; then
        echo -e "[${red}Error${plain}] Input error, please input a number"
        continue
    fi
    if [[ "$pick" -lt 1 || "$pick" -gt ${#ciphers[@]} ]]; then
        echo -e "[${red}Error${plain}] Input error, please input a number between 1 and ${#ciphers[@]}"
        continue
    fi
    shadowsockscipher=${ciphers[$pick-1]}
    echo
    echo "---------------------------"
    echo "cipher = ${shadowsockscipher}"
    echo "---------------------------"
    echo
    break
    done

    echo
    echo "Press any key to start...or Press Ctrl+C to cancel"
    char=`get_char`
    # Install necessary dependencies
    if check_sys packageManager yum; then
        yum install -y python python-devel python-setuptools openssl openssl-devel curl wget unzip gcc automake autoconf make libtool
    elif check_sys packageManager apt; then
        apt-get -y update
        apt-get -y install python python-dev python-setuptools openssl libssl-dev curl wget unzip gcc automake autoconf make libtool
    fi
    cd ${cur_dir}
}

# Download files
download_files(){
    # Download libsodium file
    if ! wget --no-check-certificate -O libsodium-1.0.13.tar.gz https://github.com/jedisct1/libsodium/releases/download/1.0.13/libsodium-1.0.13.tar.gz; then
        echo -e "[${red}Error${plain}] Failed to download libsodium-1.0.13.tar.gz!"
        exit 1
    fi
    # Download Shadowsocks file
    if ! wget --no-check-certificate -O shadowsocks-master.zip https://github.com/shadowsocks/shadowsocks/archive/master.zip; then
        echo -e "[${red}Error${plain}] Failed to download shadowsocks python file!"
        exit 1
    fi
    # Download Shadowsocks init script
    if check_sys packageManager yum; then
        if ! wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks -O /etc/init.d/shadowsocks; then
            echo -e "[${red}Error${plain}] Failed to download shadowsocks chkconfig file!"
            exit 1
        fi
    elif check_sys packageManager apt; then
        if ! wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-debian -O /etc/init.d/shadowsocks; then
            echo -e "[${red}Error${plain}] Failed to download shadowsocks chkconfig file!"
            exit 1
        fi
    fi
}

# Config shadowsocks
config_shadowsocks(){
    cat > /etc/shadowsocks.json<<-EOF
{
    "server":"0.0.0.0",
    "server_port":${shadowsocksport},
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"${shadowsockspwd}",
    "timeout":300,
    "method":"${shadowsockscipher}",
    "fast_open":false
}
EOF
}

# Firewall set
firewall_set(){
    echo "firewall set start..."
    if centosversion 6; then
        /etc/init.d/iptables status > /dev/null 2>&1
        if [ $? -eq 0 ]; then
            iptables -L -n | grep -i ${shadowsocksport} > /dev/null 2>&1
            if [ $? -ne 0 ]; then
                iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport ${shadowsocksport} -j ACCEPT
                iptables -I INPUT -m state --state NEW -m udp -p udp --dport ${shadowsocksport} -j ACCEPT
                /etc/init.d/iptables save
                /etc/init.d/iptables restart
            else
                echo "port ${shadowsocksport} has already been set up."
            fi
        else
            echo -e "[${yellow}Warning${plain}] iptables looks like shutdown or not installed, please manually set it if necessary."
        fi
    elif centosversion 7; then
        systemctl status firewalld > /dev/null 2>&1
        if [ $? -eq 0 ]; then
            firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/tcp
            firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/udp
            firewall-cmd --reload
        else
            echo "Firewalld looks like not running, try to start..."
            systemctl start firewalld
            if [ $? -eq 0 ]; then
                firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/tcp
                firewall-cmd --permanent --zone=public --add-port=${shadowsocksport}/udp
                firewall-cmd --reload
            else
                echo -e "[${yellow}Warning${plain}] Try to start firewalld failed. please enable port ${shadowsocksport} manually if necessary."
            fi
        fi
    fi
    echo "firewall set completed..."
}

# Install Shadowsocks
install(){
    # Install libsodium
    if [ ! -f /usr/lib/libsodium.a ]; then
        cd ${cur_dir}
        tar zxf libsodium-1.0.13.tar.gz
        cd libsodium-1.0.13
        ./configure --prefix=/usr && make && make install
        if [ $? -ne 0 ]; then
            echo -e "[${red}Error${plain}] libsodium install failed!"
            install_cleanup
            exit 1
        fi
    fi

    ldconfig
    # Install Shadowsocks
    cd ${cur_dir}
    unzip -q shadowsocks-master.zip
    if [ $? -ne 0 ];then
        echo -e "[${red}Error${plain}] unzip shadowsocks-master.zip failed! please check unzip command."
        install_cleanup
        exit 1
    fi

    cd ${cur_dir}/shadowsocks-master
    python setup.py install --record /usr/local/shadowsocks_install.log

    if [ -f /usr/bin/ssserver ] || [ -f /usr/local/bin/ssserver ]; then
        chmod +x /etc/init.d/shadowsocks
        if check_sys packageManager yum; then
            chkconfig --add shadowsocks
            chkconfig shadowsocks on
        elif check_sys packageManager apt; then
            update-rc.d -f shadowsocks defaults
        fi
        /etc/init.d/shadowsocks start
    else
        echo
        echo -e "[${red}Error${plain}] Shadowsocks install failed! please visit https://teddysun.com/342.html and contact."
        install_cleanup
        exit 1
    fi

    clear
    echo
    echo -e "Congratulations, Shadowsocks-python server install completed!"
    echo -e "Your Server IP        : \033[41;37m $(get_ip) \033[0m"
    echo -e "Your Server Port      : \033[41;37m ${shadowsocksport} \033[0m"
    echo -e "Your Password         : \033[41;37m ${shadowsockspwd} \033[0m"
    echo -e "Your Encryption Method: \033[41;37m ${shadowsockscipher} \033[0m"
    echo
    echo "Welcome to visit:https://teddysun.com/342.html"
    echo "Enjoy it!"
    echo
}

# Install cleanup
install_cleanup(){
    cd ${cur_dir}
    rm -rf shadowsocks-master.zip shadowsocks-master libsodium-1.0.13.tar.gz libsodium-1.0.13
}

# Uninstall Shadowsocks
uninstall_shadowsocks(){
    printf "Are you sure uninstall Shadowsocks? (y/n) "
    printf "\n"
    read -p "(Default: n):" answer
    [ -z ${answer} ] && answer="n"
    if [ "${answer}" == "y" ] || [ "${answer}" == "Y" ]; then
        ps -ef | grep -v grep | grep -i "ssserver" > /dev/null 2>&1
        if [ $? -eq 0 ]; then
            /etc/init.d/shadowsocks stop
        fi
        if check_sys packageManager yum; then
            chkconfig --del shadowsocks
        elif check_sys packageManager apt; then
            update-rc.d -f shadowsocks remove
        fi
        # delete config file
        rm -f /etc/shadowsocks.json
        rm -f /var/run/shadowsocks.pid
        rm -f /etc/init.d/shadowsocks
        rm -f /var/log/shadowsocks.log
        if [ -f /usr/local/shadowsocks_install.log ]; then
            cat /usr/local/shadowsocks_install.log | xargs rm -rf
        fi
        echo "Shadowsocks uninstall success!"
    else
        echo
        echo "uninstall cancelled, nothing to do..."
        echo
    fi
}

# Install Shadowsocks-python
install_shadowsocks(){
    disable_selinux
    pre_install
    download_files
    config_shadowsocks
    if check_sys packageManager yum; then
        firewall_set
    fi
    install
    install_cleanup
}

# Initialization step
action=$1
[ -z $1 ] && action=install
case "$action" in
    install|uninstall)
        ${action}_shadowsocks
        ;;
    *)
        echo "Arguments error! [${action}]"
        echo "Usage: `basename $0` [install|uninstall]"
    ;;
esac

```

