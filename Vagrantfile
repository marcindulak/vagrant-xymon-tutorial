# -*- mode: ruby -*-
# vi: set ft=ruby :

XYMONVER="4.3.19"

Vagrant.configure(2) do |config|
  # rhel6 (32-bit) client
  config.vm.define "rhel6" do |rhel6|
    rhel6.vm.box = "puppetlabs/centos-6.6-32-nocm"
    rhel6.vm.box_url = 'puppetlabs/centos-6.6-32-nocm'
    rhel6.vm.network "private_network", ip: "192.168.0.10"
    rhel6.vm.provider "virtualbox" do |v|
      v.memory = 128
      v.cpus = 1
    end
  end
  # rhel7 client
  config.vm.define "rhel7" do |rhel7|
    rhel7.vm.box = "puppetlabs/centos-7.0-64-nocm"
    rhel7.vm.box_url = 'puppetlabs/centos-7.0-64-nocm'
    rhel7.vm.network "private_network", ip: "192.168.0.20"
    rhel7.vm.provider "virtualbox" do |v|
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
192.168.0.10 rhel6
192.168.0.20 rhel7
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
yum install -y yp-tools  # FIXME - provides ypmatch
yum install -y c-ares-devel  # FIXME - provides libcares.so
wget http://sourceforge.net/projects/xymon/files/Xymon/${XYMONVER}/xymon-${XYMONVER}.tar.gz
tar zxf xymon-${XYMONVER}.tar.gz
cd xymon-${XYMONVER}/rpm
sed -i "s/@VER@/${XYMONVER}/" xymon.spec
ln -s ../../xymon-${XYMONVER}.tar.gz .
rpmbuild -bb --define "_sourcedir $PWD" xymon.spec
SCRIPT
  $xymon_configure_local_rpms_repo = <<SCRIPT
yum clean all
cat <<'END' > /etc/yum.repos.d/xymon.repo
[xymon]
name=CentOS-$releasever - Xymon locally built RPMS
baseurl=file:///root/rpmbuild/RPMS
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
  config.vm.define "rhel6" do |rhel6|
    rhel6.vm.provision :shell, :inline => "hostname rhel6", run: "always"
    rhel6.vm.provision :shell, :inline => $etc_hosts
    rhel6.vm.provision :shell, :inline => $epel6
    rhel6.vm.provision :shell, :inline => $service_iptables_stop, run: "always"
    rhel6.vm.provision "shell" do |s|
      s.inline = $xymon_build_rpms
      s.args   = XYMONVER
    end
    rhel6.vm.provision :shell, :inline => "yum -y install createrepo"
    rhel6.vm.provision :shell, :inline => "cd /root/rpmbuild/RPMS; createrepo `pwd`"
    rhel6.vm.provision :shell, :inline => $xymon_configure_local_rpms_repo
    rhel6.vm.provision :shell, :inline => "yum install -y xymon-client"
    rhel6.vm.provision :shell, :inline => "chkconfig --add xymon-client"
    rhel6.vm.provision :shell, :inline => "sed -i 's/XYMONSERVERS=.*/XYMONSERVERS=\"bbserver\"/' /etc/default/xymon-client"
    rhel6.vm.provision :shell, :inline => "service xymon-client start"
    rhel6.vm.provision :shell, :inline => "chkconfig xymon-client on"
  end
  config.vm.define "rhel7" do |rhel7|
    rhel7.vm.provision :shell, :inline => "hostname rhel7", run: "always"
    rhel7.vm.provision :shell, :inline => $etc_hosts
    rhel7.vm.provision :shell, :inline => $epel7
    rhel7.vm.provision :shell, :inline => $systemctl_stop_firewalld, run: "always"
    rhel7.vm.provision "shell" do |s|
      s.inline = $xymon_build_rpms
      s.args   = XYMONVER
    end
    rhel7.vm.provision :shell, :inline => "yum -y install createrepo"
    rhel7.vm.provision :shell, :inline => "cd /root/rpmbuild/RPMS; createrepo `pwd`"
    rhel7.vm.provision :shell, :inline => $xymon_configure_local_rpms_repo
    rhel7.vm.provision :shell, :inline => "yum install -y xymon-client"
    rhel7.vm.provision :shell, :inline => "chkconfig --add xymon-client"
    rhel7.vm.provision :shell, :inline => "sed -i 's/XYMONSERVERS=.*/XYMONSERVERS=\"bbserver\"/' /etc/default/xymon-client"
    rhel7.vm.provision :shell, :inline => "service xymon-client start"
    rhel7.vm.provision :shell, :inline => "chkconfig xymon-client on"
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
    bbserver.vm.provision :shell, :inline => "yum -y install createrepo"
    bbserver.vm.provision :shell, :inline => "cd /root/rpmbuild/RPMS; createrepo `pwd`"
    bbserver.vm.provision :shell, :inline => $xymon_configure_local_rpms_repo
    bbserver.vm.provision :shell, :inline => "yum install -y httpd"
    bbserver.vm.provision :shell, :inline => "yum install -y xymon"
    bbserver.vm.provision :shell, :inline => "htpasswd -b -c /etc/xymon/xymonpasswd xymon xymon"
    bbserver.vm.provision :shell, :inline => "chown apache.apache /etc/xymon/xymonpasswd"
    bbserver.vm.provision :shell, :inline => "chmod 700 /etc/xymon/xymonpasswd"
    bbserver.vm.provision :shell, :inline => "chkconfig --add xymon"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.10   rhel6      # ssh http://rhel6' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.20   rhel7      # ssh' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "echo '192.168.0.30   ubuntu14      # ssh' >> /etc/xymon/hosts.cfg"
    bbserver.vm.provision :shell, :inline => "service xymon start"
    bbserver.vm.provision :shell, :inline => "chkconfig xymon on"
    bbserver.vm.provision :shell, :inline => "service httpd start"
    bbserver.vm.provision :shell, :inline => "chkconfig httpd on"
  end
end
