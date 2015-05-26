################################
revue: getting Tumbleweed on it.
################################

This is a series of notes and articles about getting OpenSuSE [#opensuse]_ Tumbleweed [#tumbleweed]_ to run on a headless [#headless]_ (i.e. text-only, no GUI) server.

The first machine I installed OpenSuSE Tumbleweed on is ``revue`` [#revue]_. It is a bit of a gimmic, as I already had machines called ``snip`` and ``snap``. Dutch people will know why.

``snip`` and ``snap`` run regular OpenSuSE releases. ``revue`` is my first machine running a `rolling<https://en.wikipedia.org/wiki/Rolling_release>`_ release.

Over time, there will be some documents indicating installation progress and problem solving.

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


----------------------------------------------------------------------------

.. [#opensuse] I keep using the old `SuSE <https://en.wikipedia.org/wiki/SUSE>`_ writing, I'm an old fart.

.. [#tumbleweed] `Tumbleweed <https://en.opensuse.org/Portal:Tumbleweed>`_ is the rolling release of OpenSuSE.

.. [#revue] See `Snip en Snap revue<https://en.wikipedia.org/wiki/Snip_en_Snap>`_.

.. [#headless] `Headless<https://en.wikipedia.org/wiki/Headless_software>`_ as in no GUI, not as in `Embedded System<https://en.wikipedia.org/wiki/Embedded_system>`_. So there is a text `console<https://en.wikipedia.org/wiki/System_console>`_, and remote `ssh<https://en.wikipedia.org/wiki/Secure_Shell>`_.

.. [#patterns-openSUSE-minimal_base-conflicts] The `patterns-openSUSE-minimal_base-conflicts<https://www.google.com/search?q=patterns-openSUSE-minimal_base-conflicts>`_ is there to `prevent recommended packages to blow up a minimal installation<http://unix.stackexchange.com/questions/144438/missing-broken-dependancies-on-opensuse-normal/144583#144583>`_

.. [#removeconflicts] The `actual conflicts package<http://unix.stackexchange.com/questions/73427/cant-install-python-because-of-zypper-conflict>`_ contains the version number of the distribution you use.
