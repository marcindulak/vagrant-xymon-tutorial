# -*- mode: ruby -*-
# vi: set ft=ruby :

XYMONVER="4.3.19"

Vagrant.configure(2) do |config|
  # centos6 (32-bit) client
  config.vm.define "centos6" do |centos6|
    centos6.vm.box = "puppetlabs/centos-6.6-32-nocm"
    centos6.vm.box_url = 'puppetlabs/centos-6.6-32-nocm'
    centos6.vm.network "private_network", ip: "192.168.0.10"
    centos6.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # centos7 client
  config.vm.define "centos7" do |centos7|
    centos7.vm.box = "puppetlabs/centos-7.0-64-nocm"
    centos7.vm.box_url = 'puppetlabs/centos-7.0-64-nocm'
    centos7.vm.network "private_network", ip: "192.168.0.20"
    centos7.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # ubuntu client
  config.vm.define "ubuntu14" do |ubuntu14|
    ubuntu14.vm.box = "puppetlabs/ubuntu-14.04-64-nocm"
    ubuntu14.vm.box_url = 'puppetlabs/ubuntu-14.04-64-nocm'
    ubuntu14.vm.network "private_network", ip: "192.168.0.30"
    ubuntu14.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # bbserver
  config.vm.define "bbserver" do |bbserver|
    bbserver.vm.box = "puppetlabs/centos-6.6-64-nocm"
    bbserver.vm.box_url = 'puppetlabs/centos-6.6-64-nocm'
    bbserver.vm.network "private_network", ip: "192.168.0.5"
    bbserver.vm.network "forwarded_port", guest: 80, host: 8080
    bbserver.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # stop iptables
  $service_iptables_stop = <<SCRIPT
service iptables stop
SCRIPT
  # stop firewalld
  $systemctl_stop_firewalld = <<SCRIPT
systemctl stop firewalld.service
SCRIPT
  # common settings on all machines
  $etc_hosts = <<SCRIPT
cat <<END >> /etc/hosts
192.168.0.5 bbserver
192.168.0.10 centos6
192.168.0.20 centos7
192.168.0.30 ubuntu14
END
SCRIPT
  $epel6 = <<SCRIPT
yum -y install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
SCRIPT
  $epel7 = <<SCRIPT
yum -y install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
SCRIPT
  # build xymon RPMS
  $xymon_build_rpms = <<SCRIPT
XYMONVER=$1
yum install -y gcc make wget rpm-build
yum install -y pcre-devel openssl-devel openldap-devel rrdtool-devel
#yum install -y yp-tools  # FIXME - provides ypmatch required by xymon-4.3.19/configure.client
yum install -y c-ares-devel  # FIXME - provides libcares.so
wget http://sourceforge.net/projects/xymon/files/Xymon/${XYMONVER}/xymon-${XYMONVER}.tar.gz
tar zxf xymon-${XYMONVER}.tar.gz
cd xymon-${XYMONVER}
sh ./build/makerpm.sh ${XYMONVER}
SCRIPT
  $xymon_configure_local_rpms_repo = <<SCRIPT
XYMONVER=$1
yum clean all
yum -y install createrepo
rpmbuild=`readlink -f ~vagrant/xymon-${XYMONVER}/rpmbuild/RPMS`
createrepo ${rpmbuild}
cat <<END > /etc/yum.repos.d/xymon.repo
[xymon]
name=CentOS-$releasever - Xymon locally built RPMS
baseurl=file://${rpmbuild}
enabled=1
gpgcheck=0
END
SCRIPT
  # build xymon deb
  $xymon_build_deb = <<SCRIPT
XYMONVER=$1
apt-get update
apt-get install -y rrdtool librrd-dev libpcre3-dev libssl-dev ldap-utils libldap2-dev
apt-get install -y dpkg-dev  # FIXME
apt-get install -y libc-ares-dev  # FIXME
wget http://sourceforge.net/projects/xymon/files/Xymon/${XYMONVER}/xymon-${XYMONVER}.tar.gz
tar zxf xymon-${XYMONVER}.tar.gz
cd xymon-${XYMONVER}
sh ./build/makedeb.sh ${XYMONVER}
SCRIPT
  $xymon_configure_local_deb_repo = <<SCRIPT
XYMONVER=$1
apt-get update
apt-get install -y dpkg-dev
debbuild=`readlink -f ~vagrant/xymon-${XYMONVER}/debbuild`
cd ~vagrant/xymon-${XYMONVER}/debbuild
dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
echo "deb file:${debbuild} /" > /etc/apt/sources.list.d/xymon.list
apt-get update
SCRIPT
  # the actual provisions of machines
  config.vm.define "centos6" do |centos6|
    centos6.vm.provision :shell, :inline => "hostname centos6", run: "always"
    centos6.vm.provision :shell, :inline => $etc_hosts
    centos6.vm.provision :shell, :inline => $epel6
    centos6.vm.provision :shell, :inline => $service_iptables_stop, run: "always"
    centos6.vm.provision "shell" do |s|
      s.inline = $xymon_build_rpms
      s.args   = XYMONVER
    end
    centos6.vm.provision "shell" do |s|
      s.inline = $xymon_configure_local_rpms_repo
      s.args   = XYMONVER
    end
    centos6.vm.provision :shell, :inline => "yum install -y xymon-client"
    centos6.vm.provision :shell, :inline => "chkconfig --add xymon-client"
    centos6.vm.provision :shell, :inline => "sed -i 's/XYMONSERVERS=.*/XYMONSERVERS=\"bbserver\"/' /etc/default/xymon-client"
    centos6.vm.provision :shell, :inline => "service xymon-client start"
    centos6.vm.provision :shell, :inline => "chkconfig xymon-client on"
  end
  config.vm.define "centos7" do |centos7|
    centos7.vm.provision :shell, :inline => "hostname centos7", run: "always"
    centos7.vm.provision :shell, :inline => $etc_hosts
    centos7.vm.provision :shell, :inline => $epel7
    centos7.vm.provision :shell, :inline => $systemctl_stop_firewalld, run: "always"
    centos7.vm.provision "shell" do |s|
      s.inline = $xymon_build_rpms
      s.args   = XYMONVER
    end
    centos7.vm.provision "shell" do |s|
      s.inline = $xymon_configure_local_rpms_repo
      s.args   = XYMONVER
    end
    centos7.vm.provision :shell, :inline => "yum install -y xymon-client"
    centos7.vm.provision :shell, :inline => "chkconfig --add xymon-client"
    centos7.vm.provision :shell, :inline => "sed -i 's/XYMONSERVERS=.*/XYMONSERVERS=\"bbserver\"/' /etc/default/xymon-client"
    centos7.vm.provision :shell, :inline => "service xymon-client start"
    centos7.vm.provision :shell, :inline => "chkconfig xymon-client on"
  end
  config.vm.define "ubuntu14" do |ubuntu14|
    ubuntu14.vm.provision :shell, :inline => "hostname ubuntu14", run: "always"
    ubuntu14.vm.provision :shell, :inline => $etc_hosts
    ubuntu14.vm.provision "shell" do |s|
      s.inline = $xymon_build_deb
      s.args   = XYMONVER
    end
    ubuntu14.vm.provision "shell" do |s|
      s.inline = $xymon_configure_local_deb_repo
      s.args   = XYMONVER
    end
    ubuntu14.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get install -y --force-yes xymon-client"
    # FIXME: http://comments.gmane.org/gmane.comp.monitoring.hobbit/30534
    ubuntu14.vm.provision :shell, :inline => "wget 'http://sourceforge.net/p/xymon/code/HEAD/tree/trunk/debian/xymon-client.init?format=raw' -O /etc/init.d/xymon-client"
    ubuntu14.vm.provision :shell, :inline => "chmod 755 /etc/init.d/xymon-client"
    ubuntu14.vm.provision :shell, :inline => "sed -i 's/XYMONSERVERS=.*/XYMONSERVERS=\"bbserver\"/' /etc/default/xymon-client"
    ubuntu14.vm.provision :shell, :inline => "sed -i 's/XYMSRV=.*/XYMSRV=\"bbserver\"/' /usr/lib/xymon/client/etc/xymonclient.cfg"
    ubuntu14.vm.provision :shell, :inline => "service xymon-client start"
    ubuntu14.vm.provision :shell, :inline => "update-rc.d xymon-client defaults"
  end
  # last provision bbserver
  config.vm.define "bbserver" do |bbserver|
    bbserver.vm.provision :shell, :inline => "hostname bbserver", run: "always"
    bbserver.vm.provision :shell, :inline => $etc_hosts
    bbserver.vm.provision :shell, :inline => $epel6
    bbserver.vm.provision :shell, :inline => $service_iptables_stop, run: "always"
    bbserver.vm.provision "shell" do |s|
      s.inline = $xymon_build_rpms
      s.args   = XYMONVER
    end
    bbserver.vm.provision "shell" do |s|
      s.inline = $xymon_configure_local_rpms_repo
      s.args   = XYMONVER
    end
    bbserver.vm.provision :shell, :inline => "yum install -y httpd"
    bbserver.vm.provision :shell, :inline => "yum install -y xymon"
    bbserver.vm.provision :shell, :inline => "htpasswd -b -c /etc/xymon/xymonpasswd xymon xymon"
    bbserver.vm.provision :shell, :inline => "chown apache.apache /etc/xymon/xymonpasswd"
    bbserver.vm.provision :shell, :inline => "chmod 700 /etc/xymon/xymonpasswd"
    bbserver.vm.provision :shell, :inline => "chkconfig --add xymon"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.10   centos6      # ssh http://centos6' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.20   centos7      # ssh' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.30   ubuntu14      # ssh' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "service xymon start"
    bbserver.vm.provision :shell, :inline => "chkconfig xymon on"
    bbserver.vm.provision :shell, :inline => "service httpd start"
    bbserver.vm.provision :shell, :inline => "chkconfig httpd on"
  end
end
