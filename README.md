-----------
Description
-----------

Vagrantfile that builds from source,
installs and configures Xymon server and clients.
The server runs on RHEL6, clients can run on Debian(Ubuntu), RHEL(Fedora).

Tested on: Ubuntu 14.04, RHEL 6/7.

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

You should see the following Xymon setup:

![Xymon](https://raw.github.com/marcindulak/vagrant-xymon-tutorial/master/screenshots/xymon.png)

Be patient, first only the contents of /usr/lib/xymon/server/www/ is listed.
Note the red devil on at the Apache service (http) on **rhel6**

Configure Apache on the **rhel6** machine::

        $ vagrant ssh rhel6 -c "sudo su -c 'yum install -y httpd'"
        $ vagrant ssh rhel6 -c "sudo su -c 'service httpd start'"

The Xymon web interface should change its status to green for the Apache service (http) on **rhel6**.
The default monitoring interval of repeating failed checks is 1 minute,
and of passed checks 5 minutes (see /etc/xymon/tasks.cfg).
Note that the http status, though changed into green, is still "HTTP/1.1 403 Forbidden". In order
to obtain the "HTTP/1.1 200 OK" status, do::

        $ vagrant ssh rhel6 -c "sudo su -c 'touch /var/www/html/index.html'"
        $ vagrant ssh rhel6 -c "sudo su -c 'chown apache.apache /var/www/html/index.html'"
        $ vagrant ssh rhel6 -c "sudo su -c 'service httpd reload'"

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

1. use a "real" configuration management software for Xymon configuration instead of sed ...
2. setup Apache password protection
