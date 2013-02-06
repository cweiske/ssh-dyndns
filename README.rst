**********
ssh-dyndns
**********
Update DNS records by simply SSH'ing into a server.

The idea is to create a separate "dyndns" user on the DNS server.
It gets the ``ssh-dyndns`` script as login shell, so that no other programs
may be executed.
SSH provides secure, password-less key-based authentication.

Upon login, the remote IP is used to create/update a tinydns file with the
DNS record for a hostname given by the SSH client.

tinydns is part of the dbjdns/dbndns package.


=====
Setup
=====

Server
======
1. Clone ssh-dyndns into a sensible location, e.g. ``/usr/local/src/ssh-dyndns``::

    $ cd /usr/local/src/ && git clone git://git.cweiske.de/ssh-dyndns.git

2. Create a user with ``ssh-dyndns`` as login shell::

    $ useradd -g nogroup -m -N -s /usr/local/src/ssh-dyndns/ssh-dyndns dyndns

3. Prepare password-less ssh keys for the dyndns user::

    $ su - dyndns -s /bin/bash
    $ mkdir ~/.ssh

4. Prevent showing login messages::

    $ su - dyndns -s /bin/bash
    $ touch ~/.hushlogin

   Alternatively, you may commend out the "motd" lines in ``/etc/pam.d/sshd``
5. Configure ssh-dyndns as root::

    $ cp /usr/local/src/ssh-dyndns/ssh-dyndns.sh.config-dist /etc/ssh-dyndns.sh
    $ nano /etc/ssh-dyndns.sh

6. Allow ssh-dyndns to run "sudo make" without password::

    $ visudo
    dyndns  ALL= NOPASSWD: /usr/bin/make


Client
======
On a machine at home, or which other IP you want to dyndns, setup a new ssh key
as one of your users::

    $ mkdir ~ssh-dyndns
    $ cd ~/ssh-dyndns
    $ ssh-keygen -N "" -C "dyndns@home.example.org" -f ~/ssh-dyndns/ssh-dyndns_rsa

Copy the contents of the public key (``ssh-dyndns_rsa.pub``) into
``/home/dyndns/.ssh/authorized_keys`` on your server.

Run the next command manually to confirm the new ssh key::

    $ cd ~/ssh-dyndns/ && ssh -i ssh-dyndns_rsa dyndns@example.org home.example.org

If that worked, and you DNS entry worked, add the command to cron::

    $ crontab -e
    # update dns entry home.example.org every 5 minutes
    */5 * * * *  cd /home/$user/ssh-dyndns/ && ssh -i ssh-dyndns_rsa dyndns@example.org home.example.org


Configuration
=============
The configuration file template ``ssh-dyndns.sh.config-dist`` may be copied
to either ``/etc/ssh-dyndns.sh`` or ``~/.config/ssh-dyndns.sh``.

The system-wide config file is loaded first, the user-specific one after that.

The configuration file may define the following variables:

``data_dir``
    Path to the tinydns zone files, e.g. ``/etc/tinydns/root/``
``file_pattern``
    File name template. ``%DOMAIN%`` will be replaced with the actual
    domain name.

    Default: ``data-dyndns-%DOMAIN%``
``timeout``
    DNS entry TTL (time to live) in seconds

    Default: 300
``domain_patterns``
    Defines patterns for domains that may be dynamically changed.
    If the domain name does not match the pattern, the script aborts.

    You may use several patterns by separating them with a space.
    Shell wildcards are supported (``*`` and ``?``).

    Default: ``home.example.org *.home.example.org``


=====
Usage
=====
Simply ssh into the server and pass the hostname as parameter::

    $ ssh dyndns@example.org home.example.org

This will start the ``ssh-dyndns`` script on the remote server, generate
the tinydns zone file and run ``make`` in the ``data_dir`` directory to
compile the ``data.cdb`` file.
tinydns will automatically pick up the change.


====
Bugs
====
- IPv6 is not supported yet


=======
License
=======
ssh-dyndns is licensed under the `AGPL v3`__ or later.

__ http://www.gnu.org/licenses/agpl.html


======
Author
======
Written by Christian Weiske, cweiske@cweiske.de
