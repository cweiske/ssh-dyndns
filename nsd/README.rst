**************************************************
ssh-dyndns for NLnet Labs Name Server Daemon (NSD)
**************************************************
Update DNS records by simply SSH'ing into a server.

The idea is to create a separate "dyndns" user on the DNS server.
It gets the ``ssh-dyndns`` script as login shell, so that no other programs
may be executed.
SSH provides secure, password-less key-based authentication.

Upon login, the remote IP is used to create/update a NSD zone part file with
the DNS record for a hostname given by the SSH client.
In addition to the IP record, a TXT record with the update time will be added.


=====
Setup
=====

Server
======
1. Clone ssh-dyndns into a sensible location, e.g. ``/usr/local/src/ssh-dyndns``::

    $ cd /usr/local/src/ && git clone https://git.cweiske.de/ssh-dyndns.git

2. Create a user with ``ssh-dyndns`` as login shell::

    $ useradd -g nogroup -m -N -s /usr/local/src/ssh-dyndns/nsd/ssh-dyndns dyndns

3. Prepare directories for the dyndns user ssh keys::

    $ su - dyndns -s /bin/bash
    $ mkdir ~/.ssh

4. Prevent showing login messages::

    $ su - dyndns -s /bin/bash
    $ touch ~/.hushlogin

   Alternatively, you may comment out the "motd" lines in ``/etc/pam.d/sshd``

5. Configure ssh-dyndns as root::

    $ cp /usr/local/src/ssh-dyndns/nsd/ssh-dyndns.conf.sh-dist /etc/ssh-dyndns.conf.sh
    $ nano /etc/ssh-dyndns.conf.sh

6. Allow ssh-dyndns to run "sudo nsd-control" and "sudo zsu" without password::

    $ visudo
    dyndns ALL = NOPASSWD: \
      /usr/local/src/ssh-dyndns/nsd/zsu -fn /etc/nsd/zones/cweiske.de.zone,\
      /usr/sbin/nsd-control reload cweiske.de

7. Allow ssh-dyndns to create zone file parts::

     mkdir /etc/nsd/zones/dyndns/
     chown dyndns /etc/nsd/zones/dyndns/

8. Load the zone file parts from the zone files, e.g.
   ``/etc/nsd/zones/example.org.zone``::

     ; Dyndns
     $INCLUDE /etc/nsd/zones/dyndns/home.example.org-v4.zonepart
     $INCLUDE /etc/nsd/zones/dyndns/home.example.org-v6.zonepart


Client
======
On a machine at home, or which other IP you want to dyndns, setup a new ssh key
as one of your users::

    $ mkdir ~/ssh-dyndns
    $ cd ~/ssh-dyndns
    $ ssh-keygen -N "" -C "dyndns@home.example.org" -f ~/ssh-dyndns/ssh-dyndns_rsa

Copy the contents of the public key (``ssh-dyndns_rsa.pub``) into
``/home/dyndns/.ssh/authorized_keys`` on your server.

Run the next command manually to confirm the new ssh key
(4+6 are for IPv4 and IPv6)::

    $ cd ~/ssh-dyndns/ && ssh -4i ssh-dyndns_rsa dyndns@example.org home.example.org example.org
    $ cd ~/ssh-dyndns/ && ssh -6i ssh-dyndns_rsa dyndns@example.org home.example.org example.org

If that worked, and you DNS entry worked, add the command to cron::

    $ crontab -e
    # update dns entry home.example.org every 5 minutes
    */5 * * * *  cd /home/$user/ssh-dyndns/ && ssh -4i ssh-dyndns_rsa dyndns@example.org home.example.org example.org
    */5 * * * *  cd /home/$user/ssh-dyndns/ && ssh -6i ssh-dyndns_rsa dyndns@example.org home.example.org example.org

The first parameter after user+hostname is the DynDNS host name, the second
is the zone to reload after updating.


Configuration
=============
The configuration file template ``ssh-dyndns.conf.sh-dist`` may be copied
to either ``/etc/ssh-dyndns.conf.sh`` or ``~/.config/ssh-dyndns.conf.sh``.

The system-wide config file is loaded first, the user-specific one after that.

The configuration file may define the following variables:

``domain_file_pattern``
    Where zone part files are stored.

    Placeholders are ``%DOMAIN%`` for the domain name, and
    ``%TYPE%`` for "v4" or "v6".

    Default: ``/etc/nsd/zones/dyndns/%DOMAIN%-%TYPE%.zonepart``
``zone_file_pattern``
    Path of zone files that load the ``*.zonepart`` files.
    This file contains the serial number that gets updated by ``zsu``.

    Placeholder ``%ZONE%`` gets replaced with the last ssh parameter.

    Default: ``/etc/nsd/zones/%ZONE%.zone``
``timeout``
    DNS entry TTL (time to live) in seconds

    Default: 300
``domain_patterns``
    Defines patterns for domains that may be dynamically changed.
    If the domain name does not match the pattern, the script aborts.

    You may use several patterns by separating them with a space.
    Shell wildcards are supported (``*`` and ``?``).

    Default: ``home.example.org *.home.example.org``
``zone_patterns``
    Defines patterns for DNS zone names that will be updated.
    If the zone name does not match the pattern, the script aborts.

    You may use several patterns by separating them with a space.
    Shell wildcards are supported (``*`` and ``?``).

    Default: ``example.org *.example.org``


=====
Usage
=====
Simply ssh into the server and pass the hostname as first parameter,
and the zone name as second parameter::

    $ ssh dyndns@example.org home.example.org example.org

This will start the ``ssh-dyndns`` script on the remote server, generate
or update the ``home.example.org-v4.zonepart`` file,
update the serial number in the zone file ``example.org.zone``
and then call ``nsd-control reload example.org``


Check time of last update
=========================
::

    $ dig +short home.example.org ANY
    "Last update 2013-08-21 21.21.28+02.00."
    123.45.67.89


Test
====
You can test it locally:

1. Create config file::

     $ cp ssh-dyndns.conf.sh-dist ~/.config/ssh-dyndns.conf.sh

   And adjust it to use ``/tmp/`` as directory in ``domain_file_pattern``
   and ``zone_file_pattern``.

2. Create dummy zone file::

     $ touch /tmp/example.org.zone

3. Run it::

     $ SSH_CLIENT=192.168.1.4 SSH_CONNECTION=1 ./ssh-dyndns foo home.example.org example.org

4. See generated file::

     $ cat /tmp/home.example.org-v4.zonepart


=======
License
=======
ssh-dyndns is licensed under the `AGPL v3`__ or later.

__ http://www.gnu.org/licenses/agpl.html


======
Author
======
Written by Christian Weiske, cweiske@cweiske.de
