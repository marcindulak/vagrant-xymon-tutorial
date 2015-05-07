-----------
Description
-----------

Vagrantfile that builds from source,
installs and configures Xymon server and clients.
The server runs on CentOS6, clients can run on Debian(Ubuntu), CentOS(Fedora),
Windows (http://bbwin.sourceforge.net/).

Tested on: Ubuntu 14.04, CentOS 6/7, Windows Server 2012 R2.

------------
Sample Usage
------------

Assuming you have VirtualBox and Vagrant installed
https://www.virtualbox.org/ https://www.vagrantup.com/downloads.html
test the module with::

        $ git clone https://github.com/marcindulak/vagrant-xymon-tutorial.git
        $ cd vagrant-xymon-tutorial
        $ vagrant up
        $ firefox http://localhost:8080/xymon/  # Note the final slash!

You should see a Xymon setup similar to this one:

![Xymon](https://raw.github.com/marcindulak/vagrant-xymon-tutorial/master/screenshots/xymon.png)

Be patient, first only the contents of /usr/lib/xymon/server/www/ is listed.
Note the red devil on at the Apache service (http) on **centos6**

Configure Apache on the **centos6** machine::

        $ vagrant ssh centos6 -c "sudo su -c 'yum install -y httpd'"
        $ vagrant ssh centos6 -c "sudo su -c 'service httpd start'"

The Xymon web interface should change its status to green for the Apache service (http) on **centos6**.
The default monitoring interval of repeating failed checks is 1 minute,
and of passed checks 5 minutes (see /etc/xymon/tasks.cfg).
Note that the http status, though changed into green, is still "HTTP/1.1 403 Forbidden". In order
to obtain the "HTTP/1.1 200 OK" status, do::

        $ vagrant ssh centos6 -c "sudo su -c 'touch /var/www/html/index.html'"
        $ vagrant ssh centos6 -c "sudo su -c 'chown apache.apache /var/www/html/index.html'"
        $ vagrant ssh centos6 -c "sudo su -c 'service httpd reload'"

When done, destroy the test machines with::

        $ vagrant destroy -f


------------
Dependencies
------------

None


-------
License
-------

BSD 2-clause


----
Todo
----

1. setup Apache password protection on the Xymon server
