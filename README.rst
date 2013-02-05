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

1. Clone ssh-dyndns into a sensible location, e.g. ``/usr/local/src/ssh-dyndns``::

    $ cd /usr/local/src/ && git clone git://git.cweiske.de/ssh-dyndns.git

2. Create a user with ``ssh-dyndns`` as login shell::

    $ useradd -g nogroup -m -N -s /usr/local/src/ssh-dyndns dyndns

3. Setup password-less ssh keys for the dyndns user::

    $ su - dyndns -s /bin/bash
    $ mkdir ~/.ssh
    $ cat /path/to/key.pub >> ~/.ssh/authorized_keys

4. Prevent showing login messages::

    $ su - dyndns -s /bin/bash
    $ touch ~/.hushlogin

5. Configure ssh-dyndns as root::

    $ cp /usr/local/src/ssh-dyndns/ssh-dyndns.sh.config-dist /etc/ssh-dyndns.sh
    $ nano /etc/ssh-dyndns.sh


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
