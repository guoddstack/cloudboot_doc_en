*********************************************
OS Template Query
*********************************************

Request
^^^^^^^^^^^^^^^^

.. csv-table:: Request Format
    :header: Field, Description
    :widths: 5, 10

    URL, "http://localhost:8083/api/osinstall/v1/device/getSystemBySn"
    encode, UTF-8
    method, HTTP GET
    payload, text/html

Payload
^^^^^^^^

.. csv-table:: Payload
    :header: Field, Type, Required, Description
    :widths: 5, 5, 5, 15

    sn,string,yes,device serial number
    type,string,yes,"specify result format, could be ``json`` or ``raw``, default is ``raw``"


Request Sample 
^^^^^^^^^^^^^^^

::

   http://localhost:8083/api/osinstall/v1/device/getSystemBySn?sn=test&type=raw




Code Sample (PHP)
^^^^^^^^^^^^^^^^^^

.. code-block:: php

    <?php
        $url = "http://localhost:8083/api/osinstall/v1/device/getSystemBySn?sn=test&type=raw";
        $content = file_get_contents($url);
        echo $content;
    ?>



Sample Response Message
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    install
    url --url=http://mirror.idcos.com/centos/6.7/os/x86_64/
    lang en_US.UTF-8
    keyboard us
    network --onboot yes --device bootif --bootproto dhcp --noipv6
    rootpw  --iscrypted $6$eAdCfx9hZjVMqyS6$BYIbEu4zeKp0KLnz8rLMdU7sQ5o4hQRv55o151iLX7s2kSq.5RVsteGWJlpPMqIRJ8.WUcbZC3duqX0Rt3unK/
    firewall --disabled
    authconfig --enableshadow --passalgo=sha512
    selinux --disabled
    timezone Asia/Shanghai
    text
    reboot
    zerombr
    bootloader --location=mbr --append="console=tty0 biosdevname=0 audit=0 selinux=0"
    clearpart --all --initlabel
    part /boot --fstype=ext4 --size=256 --ondisk=sda
    part swap --size=2048 --ondisk=sda
    part / --fstype=ext4 --size=100 --grow --ondisk=sda

    %packages --ignoremissing
    @base
    @core
    @development

    %pre
    _sn=$(dmidecode -s system-serial-number 2>/dev/null | awk '/^[^#]/ { print $1 }')
    curl -H "Content-Type: application/json" -X POST -d "{\"Sn\":\"$_sn\",\"Title\":\"start os provisioning\",\"InstallProgress\":0.6,\"InstallLog\":\"SW5zdGFsbCBPUwo=\"}" http://osinstall.idcos.com/api/osinstall/v1/report/deviceInstallInfo
    curl -H "Content-Type: application/json" -X POST -d "{\"Sn\":\"$_sn\",\"Title\":\"disk partition and software package installation\",\"InstallProgress\":0.7,\"InstallLog\":\"SW5zdGFsbCBPUwo=\"}" http://osinstall.idcos.com/api/osinstall/v1/report/deviceInstallInfo

    %post
    progress() {
        curl -H "Content-Type: application/json" -X POST -d "{\"Sn\":\"$_sn\",\"Title\":\"$1\",\"InstallProgress\":$2,\"InstallLog\":\"$3\"}" http://osinstall.idcos.com/api/osinstall/v1/report/deviceInstallInfo
    }

    _sn=$(dmidecode -s system-serial-number 2>/dev/null | awk '/^[^#]/ { print $1 }')

    progress "set hostname and networking" 0.8 "Y29uZmlnIG5ldHdvcmsK"

    # config network
    cat > /etc/modprobe.d/disable_ipv6.conf <<EOF
    install ipv6 /bin/true
    EOF

    curl -o /tmp/networkinfo "http://osinstall.idcos.com/api/osinstall/v1/device/getNetworkBySn?sn=${_sn}&type=raw"
    source /tmp/networkinfo

    cat > /etc/sysconfig/network <<EOF
    NETWORKING=yes
    HOSTNAME=$HOSTNAME
    GATEWAY=$GATEWAY
    NOZEROCONF=yes
    NETWORKING_IPV6=no
    IPV6INIT=no
    PEERNTP=no
    EOF

    cat > /etc/sysconfig/network-scripts/ifcfg-eth0 <<EOF
    DEVICE=eth0
    BOOTPROTO=static
    IPADDR=$IPADDR
    NETMASK=$NETMASK
    ONBOOT=yes
    TYPE=Ethernet
    NM_CONTROLLED=no
    EOF

    progress "add user" 0.85 "YWRkIHVzZXIgeXVuamkK"
    useradd yunji

    progress "configure system service" 0.9 "Y29uZmlnIHN5c3RlbSBzZXJ2aWNlCg=="

    # config service
    service=(crond network ntpd rsyslog sshd sysstat)
    chkconfig --list | awk '{ print $1 }' | xargs -n1 -I@ chkconfig @ off
    echo ${service[@]} | xargs -n1 | xargs -I@ chkconfig @ on

    progress "configure system parameter" 0.95 "Y29uZmlnIGJhc2ggcHJvbXB0Cg=="

    # custom bash prompt
    cat >> /etc/profile <<'EOF'

    export LANG=en_US.UTF8
    export PS1='\n\e[1;37m[\e[m\e[1;32m\u\e[m\e[1;33m@\e[m\e[1;35m\H\e[m:\e[4m`pwd`\e[m\e[1;37m]\e[m\e[1;36m\e[m\n\$ '
    export HISTTIMEFORMAT='[%F %T] '
    EOF

    progress "finish" 1 "aW5zdGFsbCBmaW5pc2hlZAo="