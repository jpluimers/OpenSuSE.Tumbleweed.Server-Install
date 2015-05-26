################################
revue: getting Tumbleweed on it.
################################

This is a series of notes and articles about getting OpenSuSE [#opensuse]_ Tumbleweed [#tumbleweed]_ to run on a headless [#headless]_ (i.e. text-only, no GUI) server.

The first machine I installed OpenSuSE Tumbleweed on is ``revue`` [#revue]_. It is a bit of a gimmic, as I already had machines called ``snip`` and ``snap``. Dutch people will know why.

``snip`` and ``snap`` run regular OpenSuSE releases. ``revue`` is my first machine running a `rolling<https://en.wikipedia.org/wiki/Rolling_release>`_ release.

Over time, there will be some documents indicating installation progress and problem solving.

Table of Contents
=================

.. contents:: Table of Contents

headless install
================

text-mode console will break line drawing after first boot
----------------------------------------------------------

A long standing bug, and I'm amazed not more people complain about this.

I've queued a `blog entry <https://wiert.wordpress.com/?p=27755&amp">`_ about this titled "TUMBLEWEED: local console yast linedrawing characters garbage after first reboot".

The workaround is simple: Call ``/bin/unicode_start`` on the command line
before starting ``yast``. It looks you need this only once per machine.

Start with "Minimal server selection (text mode)"
-------------------------------------------------

The OpenSuSE way of a headless install starts with "Minimal server selection (text mode)". On top of that you extend the installation.

In about 20 gigabyte disk space, you can "Minimal server selection (text mode)" extended by a limited set of packages.

These are the **patterns** I extended with:

- Enhanced Base System
- Console Tools
- File Server
- Network Administration
- Mail and News Server
- Web and LAMP Server
- Internet Gateway
- DHCP and DNS Server

After that I added some **packages** too:

.. sidebar::

  Note that some of these won't install just yet, see the `text-mode installation and conflicts<text-mode-installation-and-conflicts>`_ section.

- `etckeeper<https://software.opensuse.org/package/etckeeper>`_
- `syslogd<https://software.opensuse.org/package/syslogd>`_
- emacs
- joe
- nano
- pico
- vi
- dovecot
- mutt
- par
- mc
- mirror
- p7zip
- zip
- zsync
- git
- mercurial*
- perl
- php*
- apache2-mod_php5*
- python*
- dropbox*
- cacert
- bridge-utils
- fping
- ftp
- gftp
- icecast
- links
- iptraf-ng
- shellinabox
- kvirustotal

These packages were already installed:

- info
- man
- man-pages
- mc
- top
- w3m

Didn't yet install:

- bash-doc*
- samba-doc*

``halt`` will fail, use ``halt -p`` to halt under ESXi 5.1
----------------------------------------------------------

A long time ago, `I wrote that<http://wiert.me/2012/12/30/opensuse-12-x-a-plain-halt-will-not-shutdown-the-system-properly/>`_ ``halt`` fails, but ``halt -p`` succeeds when running under VMware ESXi 5.1 (I don't run physical boxes any more).

This still fails under OpenSuSE Tumbleweed 13.2.

text-mode installation and conflicts
------------------------------------

The easiest way to start a headless install is picking "Minimal server selection (text mode)" during installation.

The problem however is that this indeed minimal. It is enforced by the  ``patterns-openSUSE-minimal_base-conflicts`` [#patterns-openSUSE-minimal_base-conflicts]_ pattern which is part of the minimal install.

It prevents some packages to install like ``mercurial``, ``php`` and ``python``.

To prevent that, remove the ``patterns-openSUSE-minimal_base-conflicts`` package specific for the OpenSuSE version you use [#removeconflicts]_.

Do this **after** you've selected the patterns you want to install. Otherwise recommended packages can be installed potentially blowing your size.

configuration
=============

getting started with etckeeper
------------------------------

A while ago ``etckeeper`` (which is `open source on GitHub<https://github.com/joeyh/etckeeper>`_) was `requested<http://joeyh.name/code/etckeeper/>`_ to be put into the factory repository, and now `is<https://software.opensuse.org/package/etckeeper>`_.

This is how I got started:

1. I created a new private repository on bitbucket called https://bitbucket.org/jeroenp/etckeeper.revue

2. I ran these commands locally::

    etckeeper init
    cd /etc
    git status
    git commit -m "initial checkin"
    git gc # pack git repo to save a lot of space

    cd /path/to/my/repo
    git remote add origin https://jeroenp@bitbucket.org/jeroenp/etckeeper.revue.git
    git push -u origin --all # pushes up the repo and its refs for the first time
    git push -u origin --tags # pushes up any tags

.. sidebar::

  `etckeeper<http://etckeeper.branchable.com/>`_ is a collection of tools to let ``/etc`` be stored in a git, mercurial, bazaar or darcs repository. This lets you use git to review or revert changes that were made to ``/etc``. Or even push the repository elsewhere for backups or cherry-picking configuration changes.

  It hooks into package managers like apt to automatically commit changes made to ``/etc`` during package upgrades. It tracks file metadata that git does not normally support, but that is important for /etc, such as the permissions of ``/etc/shadow``.

  It's quite modular and configurable, while also being simple to use if you understand the basics of working with version control.

configuring sudo
----------------

1. Start ``yast``
2. Open ``Security and Users``, then ``Sudo``
3. Click ``Add``

  1. Select a ``User`` (in my case ``jeroenp``)
  2. Select a ``Host`` (in my case ``ALL``)
  3. At ``RunAs`` type ``ALL`` (this will get translated to ``(ALL)``)
  4. Ensure that ``No Password`` has a checkmark
  5. Click ``Add``

    1. Select a ``Command`` (in my case ``ALL``)
    2. Press ``OK``

  5. Press ``OK``

4. Press ``OK``
5. Quit ``yast``

This will generate ``/etc/sudoers.YaST2.save`` add a line to ``/etc/sudoers``::

    jeroenp	ALL = (ALL) NOPASSWD:ALL

configuring ssh
---------------

Up until OpenSuSE 12.x, there was yast2-sshd. It is `still in the documentation<https://www.suse.com/documentation/opensuse114/book_security/data/sec_ssh_yast.html>`_, but it `has been orphaned<http://lists.opensuse.org/opensuse/2013-11/msg00751.html>`_ so you need to configure it manually. It isn't hard: below is the diff of the ``/etc/sshd_config`` file.

Note that when manually changing sshd configuration options, you can test (``-t``) or test-extended (``-T``) `like this<https://www.ixsystems.com/whats-new/how-secure-can-secure-shell-ssh-be-basic-configuration-of-openssh/>`_::

    sshd –t
    sshd -T

Part of the hardening is executing this from ``/etc/ssh``::

    wget https://github.com/comotion/gone/blob/github/modules/ssh
    chmod 700 ssh
    ./ssh
    rm ./ssh

I finally saved the changes using ``etckeeper``::

    etckeeper commit -m "sshd and hardening"
    git push

This is what the diff looks like::

    --- a/ssh/sshd_config
    +++ b/ssh/sshd_config
    @@ -10,7 +10,13 @@
     # possible, but leave them commented.  Uncommented options override the
     # default value.

    -#Port 22
    +Port 22
    +Port 10022
    +Port 20022
    +Port 30022
    +Port 40022
    +Port 50022
    +Port 60022
     #AddressFamily any
     #ListenAddress 0.0.0.0
     #ListenAddress ::
    @@ -35,15 +41,15 @@

     # Logging
     # obsoletes QuietMode and FascistLogging
    -#SyslogFacility AUTH
    -#LogLevel INFO
    +SyslogFacility AUTH
    +LogLevel INFO

     # Authentication:

     #LoginGraceTime 2m
    -#PermitRootLogin yes
    -#StrictModes yes
    -#MaxAuthTries 6
    +PermitRootLogin no
    +StrictModes yes
    +MaxAuthTries 1
     #MaxSessions 10

     #RSAAuthentication yes
    @@ -61,28 +67,28 @@ AuthorizedKeysFile	.ssh/authorized_keys
     # For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
     #RhostsRSAAuthentication no
     # similar for protocol version 2
    -#HostbasedAuthentication no
    +HostbasedAuthentication no
     # Change to yes if you don't trust ~/.ssh/known_hosts for
     # RhostsRSAAuthentication and HostbasedAuthentication
     #IgnoreUserKnownHosts no
     # Don't read the user's ~/.rhosts and ~/.shosts files
    -#IgnoreRhosts yes
    +IgnoreRhosts yes

     # To disable tunneled clear text passwords, change to no here!
     PasswordAuthentication no
    -#PermitEmptyPasswords no
    +PermitEmptyPasswords no

     # Change to no to disable s/key passwords
    -#ChallengeResponseAuthentication yes
    +ChallengeResponseAuthentication yes

     # Kerberos options
    -#KerberosAuthentication no
    +KerberosAuthentication no
     #KerberosOrLocalPasswd yes
     #KerberosTicketCleanup yes
     #KerberosGetAFSToken no

     # GSSAPI options
    -#GSSAPIAuthentication no
    +GSSAPIAuthentication no
     #GSSAPICleanupCredentials yes
     #GSSAPIStrictAcceptorCheck yes
     #GSSAPIKeyExchange no
    @@ -107,17 +113,17 @@ UsePAM yes

     #AllowAgentForwarding yes
     #AllowTcpForwarding yes
    -#GatewayPorts no
    -X11Forwarding yes
    +GatewayPorts no
    +X11Forwarding no
     #X11DisplayOffset 10
     #X11UseLocalhost yes
     #PermitTTY yes
    -#PrintMotd yes
    -#PrintLastLog yes
    -#TCPKeepAlive yes
    +PrintMotd no
    +PrintLastLog yes
    +TCPKeepAlive yes
     #UseLogin no
     UsePrivilegeSeparation sandbox		# Default for new installations.
    -#PermitUserEnvironment no
    +PermitUserEnvironment no
     #Compression delayed
     #ClientAliveInterval 0
     #ClientAliveCountMax 3
    @@ -129,7 +135,7 @@ UsePrivilegeSeparation sandbox		# Default for new installations.
     #VersionAddendum none

     # no default banner path
    -#Banner none
    +Banner /etc/issue

     # override default of no subsystems
     Subsystem	sftp	/usr/lib/ssh/sftp-server
    @@ -145,3 +151,6 @@ AcceptEnv LC_IDENTIFICATION LC_ALL
     #	AllowTcpForwarding no
     #	PermitTTY no
     #	ForceCommand cvs server
    +KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    +Ciphers chacha20-poly1305@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
    +MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-ripemd160-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,hmac-ripemd160,umac-128@openssh.com

.. sidebar::

  In the diff are steps from `SSH Server Configuration rhel-lockdown<http://people.redhat.com/swells/mea/SECSCAN-FirstRun/sshd_config.htm>`_, `Hardening your SSH server (opensshd_config)<http://wp.kjro.se/2013/09/06/hardening-your-ssh-server-opensshd_config/>`_ and the script behind  `http://kacper.blog.redpill-linpro.com/archives/702<http://kacper.blog.redpill-linpro.com/archives/702>`_ from `gone/ssh at github · comotion/gone<https://github.com/comotion/gone/blob/github/modules/ssh>`_. Note that the ``sandbox`` value for ``UsePrivilegeSeparation`` is even `more secure<http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/sshd_config.5?query=sshd_config&sec=5>`_ than the ``yes`` value.

Now ensure that the firewall allows for ssh:

1. Start ``yast``
2. Go to ``Security and Users``, ``Firewall``
3. Go to ``Allowed Services``
4. Ensure ``Secure Shell Server`` is in the list, when not:

  1. Add ``Secure Shell Server`` to the list
  2. Press ``Next`` followed by ``Finish`` to apply the changes

5. Quit ``yast``

Finally start ``sshd``::

    rcsshd start
    rcsshd status

configuring ntpd, firewall and jail for it
------------------------------------------

By default, OpenSuSE Tumbleweed 13.2 has ``ntdp`` enabled and configured as client and server, even though some of the tools mislead into thinking the server is not working correctly.

But first the firewall portion:

1. Start ``yast``
2. Open ``Security and Users``, then ``Firewall``
3. Go to ``Allowed Services``
4. Ensure ``xntp Server`` is in the list, when not:

  1. Add ``xntp Server`` to the list
  2. Press ``Next`` followed by ``Finish`` to apply the changes

5. Quit ``yast``

1. Start ``yast``
2. Open ``Network Services``, then ``NTP Configuration``
3. Go to ``Security Settings``
4. Ensure ``Run NTP Daemon in Chroot Jail`` is in the checked, when not:

  1. Check ``Run NTP Daemon in Chroot Jail``
  2. Press ``OK``

5. Quit ``yast``

An `ntpq<http://doc.ntp.org/4.2.8/ntpq.html>`_ verification shows the client portion works fine (you `could do this in the past from rcntpd status<http://linux.derkeiler.com/Mailing-Lists/SuSE/2013-02/msg00442.html>`_, see below)::

    revue:/etc # ntpq -p
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +vps.vdven.org   193.79.237.14    2 u  132  128  377    3.839    0.102   0.130
    *metronoom.dmz.c .PPS.            1 u   64  128  377    4.520   -0.079   0.096
    +arethusa.tweake 193.190.230.65   2 u  131  128  377    2.795    0.047   0.066
    -srv.nl.margash. 113.133.43.202   3 u   58  128  377    3.371    0.919   0.390

But it won't run as a server just yet, as the deprecated `ntpdc<http://doc.ntp.org/4.2.8/ntpdc.html>`_ shows::

    revue:/etc # ntpdc -p
    localhost: timed out, nothing received
    ***Request timed out

This is also shown when running `rcntpd status` where you get message containing `"localhost: timed out, nothing received"<https://www.google.com/search?q="localhost%3A+timed+out%2C+nothing+received">`_::

    revue:/etc # rcntpd status
    ● ntpd.service - NTP Server Daemon
       Loaded: loaded (/usr/lib/systemd/system/ntpd.service; enabled; vendor preset: disabled)
       Active: active (running) since Tue 2015-05-26 20:45:59 CEST; 44min ago
         Docs: man:ntpd(1)
      Process: 2371 ExecStart=/usr/sbin/start-ntpd start (code=exited, status=0/SUCCESS)
     Main PID: 2383 (ntpd)
       CGroup: /system.slice/ntpd.service
               └─2383 /usr/sbin/ntpd -p /var/run/ntp/ntpd.pid -g -u ntp:ntp -i /var/lib/ntp -c /etc/ntp.conf

    May 26 20:45:54 revue start-ntpd[2371]: Starting network time protocol daemon (NTPD)sntp 4.2.8p2@1.3265-o Wed Apr 22 00:47:12 UTC 2015 (1)
    May 26 20:45:54 revue start-ntpd[2371]: kod_init_kod_db(): Cannot open KoD db file /var/db/ntp-kod: No such file or directory
    May 26 20:45:54 revue sntp[2384]: 2015-05-26 20:45:54.222429 (-0100) -0.00246 +/- 0.012134 192.168.71.1 s2 no-leap
    May 26 20:45:54 revue start-ntpd[2371]: 2015-05-26 20:45:54.222429 (-0100) -0.00246 +/- 0.012134 192.168.71.1 s2 no-leap
    May 26 20:45:54 revue ntpd[2383]: Listening on routing socket on fd #22 for interface updates
    May 26 20:45:54 revue ntpd[2383]: switching logging to file /var/log/ntp
    May 26 20:45:59 revue start-ntpd[2371]: localhost: timed out, nothing received
    May 26 20:45:59 revue start-ntpd[2371]: ***Request timed out
    May 26 20:45:59 revue /usr/sbin/start-ntpd[2390]: runtime configuration: keyid 1
                                                      passwd 3a84bf3
                                                      addserver 192.168.71.1
                                                      quit
    May 26 20:45:59 revue systemd[1]: Started NTP Server Daemon.

It took me quite a while to figure out why these two show failures. It's because ``ntpdc`` is deprecated, and it is `used by conf.start-ntpd<https://build.opensuse.org/package/view_file/openSUSE:Factory/ntp/conf.start-ntpd?expand=1>`_. Too bad it is so hard to get the actual source DVCS of OpenSuSE so I don't know the history of that file.

.. sidebar::

  For the tests, I got inspired by `How to Install and Configure Linux NTP Server and Client.<http://www.thegeekstuff.com/2014/06/linux-ntp-server-client/>`_

----------------------------------------------------------------------------

.. [#opensuse] I keep using the old `SuSE <https://en.wikipedia.org/wiki/SUSE>`_ writing, I'm an old fart.

.. [#tumbleweed] `Tumbleweed <https://en.opensuse.org/Portal:Tumbleweed>`_ is the rolling release of OpenSuSE.

.. [#revue] See `Snip en Snap revue<https://en.wikipedia.org/wiki/Snip_en_Snap>`_.

.. [#headless] `Headless<https://en.wikipedia.org/wiki/Headless_software>`_ as in no GUI, not as in `Embedded System<https://en.wikipedia.org/wiki/Embedded_system>`_. So there is a text `console<https://en.wikipedia.org/wiki/System_console>`_, and remote `ssh<https://en.wikipedia.org/wiki/Secure_Shell>`_.

.. [#patterns-openSUSE-minimal_base-conflicts] The `patterns-openSUSE-minimal_base-conflicts<https://www.google.com/search?q=patterns-openSUSE-minimal_base-conflicts>`_ is there to `prevent recommended packages to blow up a minimal installation<http://unix.stackexchange.com/questions/144438/missing-broken-dependancies-on-opensuse-normal/144583#144583>`_

.. [#removeconflicts] The `actual conflicts package<http://unix.stackexchange.com/questions/73427/cant-install-python-because-of-zypper-conflict>`_ contains the version number of the distribution you use.
