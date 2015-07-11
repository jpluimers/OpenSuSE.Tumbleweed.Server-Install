###############################
revue: getting Tumbleweed on it
###############################

This is a series of notes and articles about getting OpenSuSE [#opensuse_footnote]_ Tumbleweed [#tumbleweed_footnote]_ to run on a headless [#headless_footnote]_ (i.e. text-only, no GUI) server.

The first machine I installed OpenSuSE Tumbleweed on is ``revue`` [#revue_footnote]_. It is a bit of a gimmic, as I already had machines called ``snip`` and ``snap``. Dutch people will know why.

``snip`` and ``snap`` run regular OpenSuSE releases. ``revue`` is my first machine running a `rolling <https://en.wikipedia.org/wiki/Rolling_release>`_ release.

Over time, there will be some documents indicating installation progress and problem solving.

Table of Contents
=================

.. contents::

TODO
====

- harden OpenSSL.
- web site (HTTP-HTTPS)
- pop3 (port 110)
- email (SMTP-SSMTP) 25/587
- rsync (backups) port 873
- modem reboot script (when either ipv4 or ipv6 are down)
- certificates for web and shellinabox
- update root zones through cron
- DNS security <https://www.digitalocean.com/community/tutorials/how-to-setup-dnssec-on-an-authoritative-bind-dns-server--2> and <http://csrc.nist.gov/groups/SMA/fasp/documents/network_security/NISTSecuringDNS/NISTSecuringDNS.htm>
- ensure 10rsync-var-lib-named-master.sh works
- fix syslogd and logrotate
- via `Secure your Apache Server <https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html>`_:

  - `HTTP Strict Transport Security <https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security>`_
  - `SecurityEngineering/Public Key Pinning <https://wiki.mozilla.org/SecurityEngineering/Public_Key_Pinning>`_
  - `OCSP Stapling on Apache <https://raymii.org/s/tutorials/OCSP_Stapling_on_Apache2.html>`_
  - `testssl.sh <https://testssl.sh/>`_
  - `SSLLabs SSL Server Test <https://www.ssllabs.com/ssltest/>`_

NOTES
=====

Things that might need doing.

syslogd
-------

See `this output <http://www.linuxquestions.org/questions/linux-general-1/how-to-completely-remove-service-from-systemd-using-systemctl-opensuse-4175531795/>`_::

    revue:/etc/xinetd.d # systemctl --failed --all
      UNIT              LOAD   ACTIVE SUB    DESCRIPTION
    ● logrotate.service loaded failed failed Rotate log files
    ● syslogd.service   loaded failed failed System Logging Service

    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    2 loaded units listed.
    To show all installed unit files use 'systemctl list-unit-files'.

Needs investigation.

Rest
----

Wget/curl are the best solution to update the ``root.hint``. See:

- <http://lists.opensuse.org/opensuse/2008-05/msg01589.html> - use dig, maybe not good
- <http://lists.opensuse.org/opensuse/2008-05/msg01746.html>
- <http://lists.opensuse.org/opensuse/2008-05/msg01755.html> - how to post it so Security picks it up
- <http://lists.opensuse.org/opensuse/2008-05/msg01658.html> - use ftp

The change in root servers resulted in a `security bug fix <https://bugzilla.novell.com/show_bug.cgi?id=392173>`_, but that took a while.

`This script <http://www.tldp.org/HOWTO/DNS-HOWTO-8.html>`_ gets it through dig too, but not the best solution.

Neither ftp, nor http are really secure to get these files from <http://ftp.internic.net/domain/>:

- <ftp://ftp.internic.net/domain/db.cache>
- <ftp://ftp.internic.net/domain/named.cache>
- <ftp://ftp.internic.net/domain/named.root>
- <http://www.internic.net/domain/db.cache>
- <http://www.internic.net/domain/named.cache>
- <http://www.internic.net/domain/named.root>

An alternative might be to get the ``.sig`` there in in a secure way, then `use gpg to verify the signatures <http://www.linuxquestions.org/questions/linux-newbie-8/md5-and-sig-537564/>`_ (as `gpg seems more secure than md5 signatures <http://stackoverflow.com/questions/15194779/md5-vs-gpg-signature/15195785#15195785>`_).

This is more difficult than it looks like, as you need their GPG public key with ID ``0BD07395``.

Some notes:

    ## http://codenimbus.com/2010/08/02/override-robots-txt-with-wget/
    wget -e robots=off --wait 1 http://your.site.here

    ## http://data.iana.org/root-anchors/draft-icann-dnssec-trust-anchor.html
    wget -e robots=off -m -np http://data.iana.org/root-anchors

    wget -m -np http://www.internic.net/zones

    ## http://www.pgpi.org/doc/pgpintro/#p12
    gpg --verify named.root.sig named.root

    ## http://www.links.org/?p=542
    ## https://www.google.com/search?q=key+0BD07395
    ## http://xenotrope.blogspot.nl/2015/04/on-dnssec-part-2-i-actually-used-dnssec.html

    ## http://ivan.kanis.fr/verifying-a-gpg-signed-file.html
    ## https://www.gnupg.org/documentation/manuals/gnupg/GPG-Configuration-Options.html

    ## https://www.gnupg.org/gph/en/manual/x457.html
    ## http://superuser.com/questions/227991/where-to-upload-pgp-public-key-are-keyservers-still-surviving
    gpg --keyserver keys.gnupg.net --recv-key 0BD07395
    gpg --verify named.root.sig named.root

    ## http://security.stackexchange.com/questions/6841/ways-to-sign-gpg-public-key-so-it-is-trusted
    ## http://stackoverflow.com/questions/26217766/download-key-with-gpg-recv-key-and-simultaneously-check-fingerprint-in-a-scr

Some more::

    snap:/tmp/www.internic.net/zones # gpg --verify named.root.sig named.root
    gpg: Signature made Sat May 23 14:50:54 2015 CEST using DSA key ID 0BD07395
    gpg: Can't check signature: No public key

    gpg --keyserver keys.gnupg.net --recv-key 0BD07395

    gpg --verify named.root.sig named.root
    gpg: Signature made Sat May 23 14:50:54 2015 CEST using DSA key ID 0BD07395
    gpg: Good signature from "Registry Administrator <nstld@verisign-grs.com>"
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 81F6 6E4A 1CE4 4531 08DB  6811 84FA 869E 0BD0 7395



I had this in ``named_root_hint.cron``::

    #! /bin/sh
    #

    RootHint=root.hint
    NamedCache=named.cache
    NamedCacheDownloadPath=ftp.internic.net/domain/$NamedCache
    FtpNamedCacheDownloadPath=ftp://$NamedCacheDownloadPath
    VarLibNamed=/var/lib/named/
    VarLibNamedNamedCache=$VarLibNamed$NamedCache
    VarLibNamedRootHint=$VarLibNamed$RootHint
    VarLibNamedNamedCacheNew=$VarLibNamed$NamedCache.new

    #echo "$RootHint"
    #echo "$NamedCacheDownloadPath"
    #echo "ftp://ftp.internic.net/domain"
    #echo "$FtpNamedCacheDownloadPath"
    #echo "$VarLibNamedNamedCache"
    #echo "$VarLibNamedNamedCacheNew"

    cd $VarLibNamed
    wget -q -N $FtpNamedCacheDownloadPath

    if (test -e $VarLibNamedNamedCache) ; then

      diff $VarLibNamedNamedCache $VarLibNamedNamedCacheNew

      if [ "$?" -ne "0" ] ; then
      # if $VarLibNamedNamedCacheNew does not exist, or $VarLibNamedNamedCache is different from $VarLibNamedNamedCacheNew

        cp -f $VarLibNamedNamedCache $VarLibNamedNamedCacheNew
        echo "There is a fresh $VarLibNamedNamedCacheNew file that you might want to update into $VarLibNamedRootHint"
      fi

      diff $VarLibNamedRootHint $VarLibNamedNamedCacheNew

      if [ "$?" -ne "0" ] ; then
      # if $VarLibNamedNamedCacheNew does not exist, or $VarLibNamedRootHint is different from $VarLibNamedNamedCacheNew

    #    rcnamed restart
        echo "$VarLibNamedRootHint is different from $VarLibNamedNamedCacheNew, you might need to update $VarLibNamedRootHint, then perform rcnamed restart "
      fi

      rm -f $VarLibNamedNamedCache
    fi


headless install
================

text-mode console will break line drawing after first boot
----------------------------------------------------------

A long standing bug, and I'm amazed not more people complain about this.

I've queued a `blog entry <https://wiert.wordpress.com/?p=27755">`_ about this titled "TUMBLEWEED: local console yast linedrawing characters garbage after first reboot".

The workaround is simple: Call ``/bin/unicode_start`` on the command line
before starting ``yast``. It looks you need this only once per machine.

Start with "Minimal server selection (text mode)"
-------------------------------------------------

The OpenSuSE way of a headless install starts with "Minimal server selection (text mode)". On top of that you extend the installation.

In about 20 gigabyte disk space, you can "Minimal server selection (text mode)" extended by a limited set of packages.

These are the **patterns** I extended with:

- `Enhanced Base System <https://software.opensuse.org/package/patterns-openSUSE-enhanced_base>`_
- `Console Tools <https://www.google.com/search?q="Console+Tools"+site%3Aopensuse.org>`_
- `File Server <https://www.google.com/search?q="File+Server"+site%3Aopensuse.org>`_
- `Network Administration <https://www.google.com/search?q="Network+Administration"+site%3Aopensuse.org>`_
- `Mail and News Server <https://www.google.com/search?q="Mail+and+News+Server"+site%3Aopensuse.org>`_
- `Web and LAMP Server <https://www.google.com/search?q="Web+and+LAMP+Server"+site%3Aopensuse.org>`_
- `Internet Gateway <hhttps://www.google.com/search?q="Internet+Gateway"+site%3Aopensuse.org>`_
- `DHCP and DNS Server <https://www.google.com/search?q="DHCP+and+DNS+Server"+site%3Aopensuse.org>`_

As **LAMP** installs mariadb, and as of somewhere around July 2015 mariadb bugs about it being installed with default non-password database root credentials::

    revue:/etc # zypper rm mariadb
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...

    The following package is going to be REMOVED:
      mariadb

    1 package to remove.
    After the operation, 78.7 MiB will be freed.
    Continue? [y/n/? shows all options] (y): y
    (1/1) Removing mariadb-10.0.17-1.3 .....................................................................................................................................................................................................[done]

    revue:/etc # zypper search "LAMP"
    Loading repository data...
    Reading installed packages...

    S | Name                          | Summary             | Type
    --+-------------------------------+---------------------+--------
    i | lamp_server                   | Web and LAMP Server | pattern
    i | patterns-openSUSE-lamp_server | Web and LAMP Server | package

    revue:/etc # zypper remove patterns-openSUSE-lamp_server
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...

    The following package is going to be REMOVED:
      patterns-openSUSE-lamp_server

    The following pattern is going to be REMOVED:
      lamp_server

    1 package to remove.
    After the operation, 57.0 B will be freed.
    Continue? [y/n/? shows all options] (y): y
    (1/1) Removing patterns-openSUSE-lamp_server-20150603-4.1 ..............................................................................................................................................................................[done]
    revue:/etc #

    revue:/etc # zypper remove mariadb-client mariadb-errormessages
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...

    The following 2 packages are going to be REMOVED:
      mariadb-client mariadb-errormessages

    2 packages to remove.
    After the operation, 21.8 MiB will be freed.
    Continue? [y/n/? shows all options] (y): y
    (1/2) Removing mariadb-client-10.0.17-1.3 ..............................................................................................................................................................................................[done]
    (2/2) Removing mariadb-errormessages-10.0.17-1.3 .......................................................................................................................................................................................[done]
    revue:/etc #

If I ever need MySQL or MariaDB, I will get it again and solve the root rights.

Finally time for some manual adding of **packages**:

.. note::

  Note that some of these won't install just yet, see the `text-mode installation and conflicts <text-mode-installation-and-conflicts>`_ section.

- `etckeeper <https://software.opensuse.org/package/etckeeper>`_
- `syslogd <https://software.opensuse.org/package/syslogd>`_
- `emacs <https://software.opensuse.org/package/emacs>`_
- `joe <https://software.opensuse.org/package/joe>`_
- `nano <https://software.opensuse.org/package/nano>`_
- `pico <https://software.opensuse.org/package/pico>`_
- `vim <https://software.opensuse.org/package/vim>`_
- `dovecot <https://software.opensuse.org/package/dovecot>`_
- `mutt <https://software.opensuse.org/package/mutt>`_
- `par <https://software.opensuse.org/package/par>`_
- `make <https://software.opensuse.org/package/make>`_
- `monit <https://software.opensuse.org/package/monit>`_
- `mc <https://software.opensuse.org/package/mc>`_
- `mirror <https://software.opensuse.org/package/mirror>`_
- `p7zip <https://software.opensuse.org/package/p7zip>`_
- `zip <https://software.opensuse.org/package/zip>`_
- `zsync <https://software.opensuse.org/package/zsync>`_
- `git <https://software.opensuse.org/package/git>`_
- `mercurial <hhttps://software.opensuse.org/package/mercurial>`_\*
- `perl <hhttps://software.opensuse.org/package/perl>`_
- `php <https://software.opensuse.org/package/php>`_\*
- `apache2-mod_php5 <https://software.opensuse.org/package/apache2-mod_php5>`_\*
- `python <https://software.opensuse.org/package/python>`_\*
- `dropbox <https://software.opensuse.org/package/dropbox>`_\*
- `ca-certificates-cacert <https://software.opensuse.org/package/ca-certificates-cacert>`_
- `bridge-utils <https://software.opensuse.org/package/bridge-utils>`_
- `fping <https://software.opensuse.org/package/fping>`_
- `ftp <https://software.opensuse.org/package/ftp>`_
- `gftp <https://software.opensuse.org/package/gftp>`_
- `icecast <https://software.opensuse.org/package/icecast>`_
- `links <https://software.opensuse.org/package/links>`_
- `iptraf-ng <https://software.opensuse.org/package/iptraf-ng>`_
- `shellinabox <https://software.opensuse.org/package/shellinabox>`_
- `kvirustotal <hhttps://software.opensuse.org/package/kvirustotal>`_
- `monit <https://software.opensuse.org/package/monit>`_

These packages were already installed:

- `info <https://software.opensuse.org/package/info>`_
- `man <https://software.opensuse.org/package/man>`_
- `man-pages <https://software.opensuse.org/package/man-pages>`_
- `mc <https://software.opensuse.org/package/mc>`_
- `w3m <https://software.opensuse.org/package/w3m>`_

Didn't yet install:

- `bash-doc <https://software.opensuse.org/package/bash-doc>`_\*
- `samba-doc <https://software.opensuse.org/package/samba-doc>`_\*

.. note::

  If you want to know `which package provides a certain file <http://unix.stackexchange.com/questions/158041/how-do-i-find-a-package-that-provides-a-given-file-in-opensuse>`_, then use this command::

      zypper search --provides --match-exact hg

  Where ``hg`` is the file you are looking for.

``halt`` will fail, use ``halt -p`` to halt under ESXi 5.1
----------------------------------------------------------

A long time ago, `I wrote that <http://wiert.me/2012/12/30/opensuse-12-x-a-plain-halt-will-not-shutdown-the-system-properly/>`_ ``halt`` fails, but ``halt -p`` succeeds when running under VMware ESXi 5.1 (I don't run physical boxes any more).

This still fails under OpenSuSE Tumbleweed 13.2.

text-mode installation and conflicts
------------------------------------

The easiest way to start a headless install is picking "Minimal server selection (text mode)" during installation.

The problem however is that this indeed minimal. It is enforced by the ``patterns-openSUSE-minimal_base-conflicts`` [#patterns-openSUSE-minimal_base-conflicts_footnote]_ pattern which is part of the minimal install.

It prevents some packages to install like ``mercurial``, ``php`` and ``python``.

To prevent that, remove the ``patterns-openSUSE-minimal_base-conflicts`` package specific for the OpenSuSE version you use [#removeconflicts_footnote]_.

Do this **after** you've selected the patterns you want to install. Otherwise recommended packages can be installed potentially blowing your size.

add git-extras
--------------

See the `git-extras Install documentation <https://github.com/tj/git-extras/blob/master/Installation.md>`_ for why/how.

Just run this command::

    (cd /tmp && git clone https://github.com/tj/git-extras.git && cd git-extras && git checkout $(git describe --tags $(git rev-list --tags --max-count=1)) && sudo make install)

configuration
=============

getting started with etckeeper
------------------------------

A while ago ``etckeeper`` (which is `open source on GitHub <https://github.com/joeyh/etckeeper>`_) was `requested <http://joeyh.name/code/etckeeper/>`_ to be put into the factory repository, and now `is <https://software.opensuse.org/package/etckeeper>`_.

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

.. note::

  `etckeeper <http://etckeeper.branchable.com/>`__ is a collection of tools to let ``/etc`` be stored in a git, mercurial, bazaar or darcs repository. This lets you use git to review or revert changes that were made to ``/etc``. Or even push the repository elsewhere for backups or cherry-picking configuration changes.

  It hooks into package managers like apt to automatically commit changes made to ``/etc`` during package upgrades. It tracks file metadata that git does not normally support, but that is important for /etc, such as the permissions of ``/etc/shadow``.

  It's quite modular and configurable, while also being simple to use if you understand the basics of working with version control.

  Three important ``etckeeper`` gotchas with powerful scripts like `pre-commit <https://github.com/joeyh/etckeeper/tree/master/pre-commit.d>`_ ``/etc/etckeeper/pre-commit.d``:

  1. ensure you give them executable permissions like `chmod 755 <http://www.networkredux.com/answers/linux-in-general/users-and-permissions/how-do-i-use-the-chmod-command-in-linux>`_.
  2. ensure they are valid `sh <https://en.wikipedia.org/wiki/Bourne_shell>`_ scripts.
  3. do not give them the .sh extension:

    - fails: ``/etc/etckeeper/pre-commit.d/10rsync-var-lib-named-master``
    - works: ``/etc/etckeeper/pre-commit.d/10rsync-var-lib-named-master.sh``

removing hardlinks from the ``etckeeper`` repository
----------------------------------------------------

Inspired by `this answer <http://unix.stackexchange.com/questions/63627/excluding-files-in-etckeeper-with-gitignore-doesnt-work/63628#63628>`_ to get rid of these messages during `etckeeper commit <https://github.com/joeyh/etckeeper#what-etckeeper-does>`_ to delete many `hardlinked bootsplash files <http://lists.opensuse.org/opensuse-factory/2014-06/msg00115.html>`_::

    etckeeper warning: hardlinked files could cause problems with git:
    bootsplash/themes/openSUSE/bootloader/af.tr
    ...
    bootsplash/themes/openSUSE/bootloader/pt.tr
    bootsplash/themes/openSUSE/bootloader/pt_BR.tr
    bootsplash/themes/openSUSE/bootloader/ro.tr
    ...
    bootsplash/themes/openSUSE/bootloader/xh.tr
    bootsplash/themes/openSUSE/bootloader/zh_CN.tr
    bootsplash/themes/openSUSE/bootloader/zh_TW.tr
    bootsplash/themes/openSUSE/bootloader/zu.tr
    bootsplash/themes/openSUSE/cdrom/af.tr
    ...
    bootsplash/themes/openSUSE/cdrom/pt.tr
    bootsplash/themes/openSUSE/cdrom/pt_BR.tr
    bootsplash/themes/openSUSE/cdrom/ro.tr
    ...
    bootsplash/themes/openSUSE/cdrom/xh.tr
    bootsplash/themes/openSUSE/cdrom/zh_CN.tr
    bootsplash/themes/openSUSE/cdrom/zh_TW.tr
    bootsplash/themes/openSUSE/cdrom/zu.tr

Add these two lines to ``/etc/.gitignore``

    bootsplash/themes/openSUSE/bootloader/*.tr
    bootsplash/themes/openSUSE/cdrom/*.tr

Note the ``--cache`` part in the command to delete, as then the files will not be deleted locally, only in the repository::

    git add .gitignore
    git rm --cached bootsplash/themes/openSUSE/bootloader/*.tr
    git rm --cached bootsplash/themes/openSUSE/cdrom/*.tr
    git commit -m "git rm --cached bootsplash/themes/openSUSE/bootloader/*.tr and bootsplash/themes/openSUSE/cdrom/*.tr"


Adding user ``jeroenp`` to ``SUDOERS`` so it can perform ``sudo``
-----------------------------------------------------------------

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

.. note::

  Note that `each ALL entry has a different meaning <http://superuser.com/questions/357467/what-do-the-alls-in-the-line-admin-all-all-all-in-ubuntus-etc-sudoers>`_.

configuring ssh
---------------

Up until OpenSuSE 12.x, there was yast2-sshd. It is `still in the documentation <https://www.suse.com/documentation/opensuse114/book_security/data/sec_ssh_yast.html>`_, but it `has been orphaned <http://lists.opensuse.org/opensuse/2013-11/msg00751.html>`_ so you need to configure it manually. It isn't hard: below is the diff of the ``/etc/sshd_config`` file.

Note that when manually changing sshd configuration options, you can test (``-t``) or test-extended (``-T``) `like this <https://www.ixsystems.com/whats-new/how-secure-can-secure-shell-ssh-be-basic-configuration-of-openssh/>`_::

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

.. note::

  In the diff are steps from `SSH Server Configuration rhel-lockdown <http://people.redhat.com/swells/mea/SECSCAN-FirstRun/sshd_config.htm>`_, `Hardening your SSH server (opensshd_config) <http://wp.kjro.se/2013/09/06/hardening-your-ssh-server-opensshd_config/>`_ and the script behind  `http://kacper.blog.redpill-linpro.com/archives/702 <http://kacper.blog.redpill-linpro.com/archives/702>`_ from `gone/ssh at github · comotion/gone <https://github.com/comotion/gone/blob/github/modules/ssh>`_. Note that the ``sandbox`` value for ``UsePrivilegeSeparation`` is even `more secure <http://www.openbsd.org/cgi-bin/man.cgi/OpenBSD-current/man5/sshd_config.5?query=sshd_config&sec=5>`_ than the ``yes`` value.

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

install and configure `noip` dynamic DNS update script
------------------------------------------------------

The script is based on <https://github.com/mdmower/bash-no-ip-updater.git>.

Create the below ``/etc/noip.com.install.sh`` script with ``chmod 700``, then run it to install.

One of the things it does is move the config file outside the repository (`I've made a pull-request for that <https://github.com/mdmower/bash-no-ip-updater/pull/2>`_) as it contains credentials.

Full source is at <https://gist.github.com/jpluimers/3f8c9c024446f6c6dab3>::

    #! /bin/sh
    #
    # creates /etc/NoIP directory
    # clones https://github.com/mdmower/bash-no-ip-updater.git
    # copies configuration file so it is outside of the git sub-repository (and can be versioned with etckeeper)
    # modifies the script to use the copied configuration file

    ETC_TARGET=/etc/noip.com
    LOG_TARGET=/var/log/noip.com
    CONFIG_BASE=bash-no-ip-updater
    CONFIG_TARGET=$CONFIG_BASE.config
    SCRIPT_TARGET=noipupdater.sh
    CRON_HOURLY_TARGET=/etc/cron.hourly/$SCRIPT_TARGET

    mkdir $ETC_TARGET
    pushd $ETC_TARGET
    git clone https://github.com/mdmower/$CONFIG_BASE.git
    cp $CONFIG_BASE/config $CONFIG_TARGET

    mkdir -p $LOG_TARGET

    # replace
    ## LOGDIR="$HOME/logs"
    # with
    ## LOGDIR="/var/logs/noip.com"
    # use double quotes to allow for variable expansion: http://stackoverflow.com/questions/17477890/expand-variables-in-sed/17477911#17477911
    # escape slashes in arguments: http://www.grymoire.com/Unix/Sed.html#uh-62
    echo old:
    sed -n "/^LOGDIR=\"\$HOME\/logs\"$/ p" $CONFIG_TARGET
    LOG_TARGET_EXPANDED=`echo "$LOG_TARGET" | sed 's:[]\[\^\$\.\*\/]:\\\\&:g'`
    #echo "/^LOGDIR=\"\$HOME\/logs\"$/ s/\"\$HOME\/logs\"$/\"${LOG_TARGET}\"/"
    #echo "/^LOGDIR=\"\$HOME\/logs\"$/ s/\"\$HOME\/logs\"$/\"${LOG_TARGET_EXPANDED}\"/"
    sed -e "/^LOGDIR=\"\$HOME\/logs\"$/ s/\"\$HOME\/logs\"$/\"${LOG_TARGET_EXPANDED}\"/" $CONFIG_TARGET > $CONFIG_TARGET.tmp && mv $CONFIG_TARGET.tmp $CONFIG_TARGET
    echo new:
    sed -n "/^LOGDIR=\".*\"$/ p" $CONFIG_TARGET

    pushd $CONFIG_BASE
    # in ``noip.com/bash-no-ip-updater/noipupdater.sh``  replace
    ## CONFIGFILE="$( cd "$( dirname "$0" )" && pwd )/config"
    # by
    ## CONFIGFILE="$( cd "$( dirname "$0" )" && pwd ).config"
    # in-place sed: http://stackoverflow.com/questions/5171901/sed-command-find-and-replace-in-file-and-overwrite-file-doesnt-work-it-empties/5174368#5174368
    # set tips: http://www.grymoire.com/Unix/Sed.html
    ## sed -e 'script script' index.html > index.html.tmp && mv index.html.tmp index.html
    echo "old:"
    sed -n '/^CONFIGFILE\=.*\/config"$/ p' $SCRIPT_TARGET
    sed -e '/^CONFIGFILE\=.*\/config"$/ s/\/config"$/.config"/' $SCRIPT_TARGET > $SCRIPT_TARGET.tmp && mv $SCRIPT_TARGET.tmp $SCRIPT_TARGET
    echo "new:"
    sed -n '/^CONFIGFILE\=.*\.config"$/ p' $SCRIPT_TARGET
    chmod 755 $SCRIPT_TARGET
    popd

    popd
    echo files:
    find noip.com* | grep -v \.git

    # http://stackoverflow.com/questions/7875540/how-do-you-write-multiple-line-configuration-file-using-bash-and-use-variables/7875614#7875614
    #!/bin/bash
    cat >$CRON_HOURLY_TARGET <<EOL
    #! /bin/sh
    #
    # Hourly job to ensure the noip.com information for this host is up-to-date.
    #
    $ETC_TARGET/$CONFIG_BASE/$SCRIPT_TARGET
    EOL

    echo Hourly crontab entry in $CRON_HOURLY_TARGET
    chmod 755 $CRON_HOURLY_TARGET
    cat $CRON_HOURLY_TARGET

Now modify the ``/etc/noip.com/bash-no-ip-updater.config`` file; alter these entries::

    USERNAME="email@address.com"
    PASSWORD="password"
    HOST="host.domain.com"

.. sidebar:: no-ip login note

  I could just use my account name (email was not needed). Other people seem to need their email. Try both.

Finally run ``/etc/noip.com/bash-no-ip-updater/noipupdater.sh`` ones and look at the log file ``/var/log/noip.com/noip.log`` to see the result and check ``/var/log/noip.com/last_ip`` if the IP-address is indeed correct.

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

An `ntpq <http://doc.ntp.org/4.2.8/ntpq.html>`_ verification shows the client portion works fine (you `could do this in the past from rcntpd status <http://linux.derkeiler.com/Mailing-Lists/SuSE/2013-02/msg00442.html>`_, see below)::

    revue:/etc # ntpq -p
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +vps.vdven.org   193.79.237.14    2 u  132  128  377    3.839    0.102   0.130
    *metronoom.dmz.c .PPS.            1 u   64  128  377    4.520   -0.079   0.096
    +arethusa.tweake 193.190.230.65   2 u  131  128  377    2.795    0.047   0.066
    -srv.nl.margash. 113.133.43.202   3 u   58  128  377    3.371    0.919   0.390

But it won't run as a server just yet, as the deprecated `ntpdc <http://doc.ntp.org/4.2.8/ntpdc.html>`_ shows::

    revue:/etc # ntpdc -p
    localhost: timed out, nothing received
    ***Request timed out

This is also shown when running `rcntpd status` where you get message containing `"localhost: timed out, nothing received" <https://www.google.com/search?q="localhost%3A+timed+out%2C+nothing+received">`_::

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

It took me quite a while to figure out why these two show failures. It's because ``ntpdc`` is deprecated, and it is `used by conf.start-ntpd <https://build.opensuse.org/package/view_file/openSUSE:Factory/ntp/conf.start-ntpd?expand=1>`_. Too bad it is so hard to get the actual source DVCS of OpenSuSE so I don't know the history of that file.

.. note::

  For the tests, I got inspired by `How to Install and Configure Linux NTP Server and Client <http://www.thegeekstuff.com/2014/06/linux-ntp-server-client/>`_


Configuring ``samba``
---------------------

1. Start ``yast``
2. Open ``Network Services``, then ``Samba Server``
3. Fill in the ``Workgroup or Domain Name`` (I kept it at ``WORKGROUP`` as my domain-less Windows machines are configured like that)
4. Press ``Next``
5. Choose the ``Server type`` (I kept it at ``Not a Domain Controller`` as don't run a domain)
6. Press ``Next``
7. In the ``Samba Configuration`` screen:

  1. Ensure ``Service Start`` is set to ``During Boot``.
  2. Ensure ``Open Port in Firewall`` is checked.
  3. Press ``OK``

8. Quit ``yast``

This will modify these files:

- ``/etc/apparmor.d/local/usr.sbin.smbd-shares`` (upon Samba start)
- ``/etc/samba/smb.conf``
- ``/etc/sysconfig/SuSEfirewall2``

and add these configuration files:

- ``/etc/printcap`` (which will be auto-generated from ``/etc/cups/printers.conf`` if it exists)
- ``/etc/systemd/system/multi-user.target.wants/nmb.service``
- ``/etc/systemd/system/multi-user.target.wants/smb.service``

Run these commands to `test if the basic configuration was successful <hhttps://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/install.html#id2553312>`_ with `testclient <https://www.samba.org/samba/docs/man/manpages/testparm.1.html>`_ and `https://www.samba.org/samba/docs/man/manpages/smbclient.1.html <h>`_::

    testparm /etc/samba/smb.conf
    smbclient -L `hostname`

.. note::

  During ``smbclient`` you will have to type your unix password.

Testing and fixing so clients can talk to our Samba server
----------------------------------------------------------

Now it is time to test the smb connectivity as well::

  smbclient //`hostname`/profiles -U jeroenp
  Enter jeroenp's password:
  Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.1-3406-SUSE-oS13.2-x86_64]
  tree connect failed: NT_STATUS_ACCESS_DENIED

.. note::

  Do **not** try to solve the `NT_STATUS_ACCESS_DENIED issue <https://forum.manjaro.org/index.php?topic=19252.0>`_ by enabling ``client lanman auth`` as this makes your system less secure (`LANMAN authentication can be cracked quite easily <hhttps://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html#idp59214864>`_).

The first think to check is the samba password database, as samba uses different authentication database than the standard linux one (hence the linux password above).
Check it with `pdbedit <https://www.samba.org/samba/docs/man/manpages/pdbedit.8.html>`_ like this::

    pdbedit --list --verbose jeroenp

If it shows ``Username not found!`` then you need to add the user:

    revue:/etc # pdbedit --create --user jeroenp
    new password:
    retype new password:
    Unix username:        jeroenp
    NT username:
    Account Flags:        [U          ]
    User SID:             S-1-5-21-539969646-619626457-384116915-1000
    Primary Group SID:    S-1-5-21-539969646-619626457-384116915-513
    Full Name:            Jeroen Pluimers
    Home Directory:       \\revue\jeroenp\.9xprofile
    HomeDir Drive:        P:
    Logon Script:
    Profile Path:         \\revue\profiles\.msprofile
    Domain:               REVUE
    Account desc:
    Workstations:
    Munged dial:
    Logon time:           0
    Logoff time:          Wed, 06 Feb 2036 16:06:39 CET
    Kickoff time:         Wed, 06 Feb 2036 16:06:39 CET
    Password last set:    Wed, 27 May 2015 20:51:21 CEST
    Password can change:  Wed, 27 May 2015 20:51:21 CEST
    Password must change: never
    Last bad password   : 0
    Bad password count  : 0
    Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF

.. note::

  Do **not** use `smbpasswd <https://www.samba.org/samba/docs/man/manpages/smbpasswd.8.html>`_ to add the user as that only supports the ``smbpasswd`` database format, `whereas ``pdbedit`` supports any password backend <http://unix.stackexchange.com/questions/107032/deleting-a-samba-user-pbdedit-vs-smbpasswd-whats-the-difference/107033#107033>`_.

Now do final checks::

    smbclient --list `hostname` --user jeroenp
    smbclient //`hostname`/jeroenp -U jeroenp

One day: `syncing between the Samba password and system password storage <https://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/pam.html#id2667418>`_ is setup
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

See `Use SMB Information for Linux Authentication <https://www.google.com/search?q="Use+SMB+Information+for+Linux+Authentication">`_.

Fixing password synchronisation?
--------------------------------

.. note::

  Background reading (web-archive link as the site itself is down): `Samba Server and Suse / openSUSE: HowTo Configure a Professional File Server on a SOHO LAN, covering Name Resolution, Authentication, Security and Shares <http://web.archive.org/web/20130801222534/http://swerdna.dyndns.org/susesambaserver.html>`_.

configuring named/BIND
----------------------

1. Start ``yast``
2. Open ``System``, then ``/etc/sysconfig Editor``
3. In ``Configuration Options``, open these tree nodes: ``Network``; ``DNS``; ``Name Server``
4. Ensure the below entries have the correct values:

  1. ``NAMED_RUN_CHROOTED`` has no value
  2. ``NAMED_ARGS`` has no value
  3. ``NAMED_CONF_INCLUDE_FILES`` has value ``options logging master slaves rnd-access.conf``
  4. ``NAMED_INITIALIZE_SCRIPTS`` has value ``createNamedConfInclude`` (this is the default value)

5. If any value needed to be changed, then press ``Finish`` and confirm the changes.
6. Open ``Security and Users``, then ``Firewall``
7. Go to ``Allowed Services``
8. Ensure ``bind DNS server`` is in the list, when not:

  1. Add ``bind DNS server`` to the list
  2. Press ``Next`` followed by ``Finish`` to apply the changes

9. Quit ``yast``

Add an empty ``/etc/named.d/forwarders.conf``.

Add ``/etc/named.d/master``::

    zone "4delphi.com" {
            type master;
            file "master/4delphi.com";
    };

    zone "pluimers.com" {
            type master;
            file "master/pluimers.com";
    };

    zone "pluimers.localnet" {
            type master;
            file "master/pluimers.localnet";
            notify no;
            allow-query     { internals; };
            allow-transfer  { internals; };
    };

    zone "71.168.192.IN-ADDR.ARPA" {
            type master;
            file "master/192.168.71";
            notify no;
            allow-query     { internals; };
            allow-transfer  { internals; };
    };

    zone "171.168.192.IN-ADDR.ARPA" {
            type master;
            file "master/192.168.171";
            notify no;
            allow-query     { internals; };
            allow-transfer  { internals; };
    };

Add ``/etc/named.d/options``::

    acl internals {
    		127.0.0.1/24;
                    192.168.71.0/16;
                    192.168.171.0/16;
    	      };

    acl externals {
    		82.161.131.169; // jeroen - ADSL xs4all
    		80.100.143.119; // jeroen - fiber xs4all
    		37.153.243.241; // jeroen - fiber helden van nu 1 - router
    		37.153.243.242; // jeroen - fiber helden van nu 2 - server DNS 1
    		37.153.243.243; // jeroen - fiber helden van nu 3 - server
    		37.153.243.244; // jeroen - fiber helden van nu 4 - server
    		37.153.243.245; // jeroen - fiber helden van nu 5 - server
    		37.153.243.246; // jeroen - fiber helden van nu 6 - server DNS 2
    		  62.195.34.14; // jeroen - Cable UPC (tijdelijk)
    		 136.243.21.95; // remco/cor - Hetzner host - ziggy.domainnetwerk.info
    		 83.163.69.172; // martijn - mwpg.xs4all.nl
                       109.70.6.22; // jaco - Dynasol
            };

Ensure these files exist:

``/var/lib/named/master/192.168.171``::

    $TTL 1H
    @               IN      SOA     ns.pluimers.localnet.   root.4delphi.com. (
                            2005011803 ; serial
                            1H         ; refresh
                            900        ; retry
                            3W         ; expire
                            2H         ; default_ttl
                            )
    @               IN      NS      ns.pluimers.localnet.
    80             IN      PTR     jp1.pluimers.localnet.
    80             IN      PTR     snap.pluimers.localnet.
    80             IN      PTR     ns.pluimers.localnet.
    70             IN      PTR     snip.pluimers.localnet.

``/var/lib/named/master/192.168.71``::

    $TTL 1H
    @               IN      SOA     ns.pluimers.localnet.   root.4delphi.com. (
                            2005011803 ; serial
                            1H         ; refresh
                            900        ; retry
                            3W         ; expire
                            2H         ; default_ttl
                            )
    @               IN      NS      ns.pluimers.localnet.
    80             IN      PTR     jp1.pluimers.localnet.
    80             IN      PTR     snap.pluimers.localnet.
    80             IN      PTR     ns.pluimers.localnet.
    70             IN      PTR     snip.pluimers.localnet.

``/var/lib/named/master/named.local``::

    $TTL 2H
    @               IN      SOA     localhost.      root.localhost. (
                            2004111611 ; serial
                            1H         ; refresh
                            900        ; retry
                            3W         ; expire
                            2H         ; default_ttl
                            )
    1               IN      PTR     localhost.
    @               IN      NS      localhost.

``/var/lib/named/master/pluimers.localnet``::

    $TTL 2H
    @               IN      SOA     ns.pluimers.localnet.    root.4delphi.com. (
                            2004111615 ; serial
                            1H         ; refresh
                            900        ; retry
                            3W         ; expire
                            2H         ; default_ttl
                            )
    @                       IN      MX      5       mail.pluimers.com.
    @                       IN      NS      ns.pluimers.localnet.
    @                       IN      A       192.168.71.80
    localhost               IN      A       127.0.0.1
    jp1                     IN      A       192.168.71.80
    ns                      IN      A       192.168.71.80
    snap                    IN      A       192.168.71.80
    snip                    IN      A       192.168.71.70

``/var/lib/named/master/pluimers.com``::

    to fill in later

``/var/lib/named/master/4delphi.com``::

    to fill in later

Finally stop/start the named service::

    rcnamed stop
    rcnamed start
    rcnamed status

.. note::

    Check if your zone files are correct by executing `named-checkzone <http://www.cyberciti.biz/faq/howto-linux-unix-zone-file-validity-checking/>`_::

        named-checkzone 4delphi.com /var/lib/named/master/4delphi.com
        named-checkzone pluimers.com /var/lib/named/master/pluimers.com

    Check if your named configuration is correct by executing `named-checkconf <http://www.cyberciti.biz/tips/howto-linux-unix-check-dns-file-errors.html>`_::

        named-checkconf /etc/named.conf

    Check if ``named`` delivers the correct zone::

        dig @localhost axfr 4delphi.com
        dig @localhost axfr pluimers.com

    See:

    - `Check BIND – DNS Server configuration file for errors with named-checkconf tools <http://www.cyberciti.biz/tips/howto-linux-unix-check-dns-file-errors.html>`_
    - `Troubleshoot Linux / UNIX bind dns server zone problems with named-checkzone tool <hhttp://www.cyberciti.biz/faq/howto-linux-unix-zone-file-validity-checking/>`_

    `Check the named configuration <https://ask.fedoraproject.org/en/question/24288/how-to-debug-bind-conf-and-zone-files/>`_::

        named-checkconf /etc/named.conf

    `Check each named zone <http://www.ewhathow.com/2013/09/how-do-i-check-bind-zone-file-for-errors/>`_::

        named-checkzone localhost /var/lib/named/master/4delphi.com
        named-checkzone localhost /var/lib/named/master/pluimers.com


Ensure that ``/var/lib/named/master`` gets synced to ``/etc/named/master``
--------------------------------------------------------------------------

Based on these links, I've added a sync script.

- `etckeeper configuration documentation <https://github.com/joeyh/etckeeper#configuration>`_
- `unix: using variables <http://www.tutorialspoint.com/unix/unix-using-variables.htm>`_

I stored it in ``/etc/etckeeper/pre-commit.d/10rsync-var-lib-named-master``::

    #! /bin/sh
    ## http://www.tutorialspoint.com/unix/unix-using-variables.htm
    TARGET=/etc/named/master
    mkdir -p $TARGET
    rsync -avloz /var/lib/named/master/ $TARGET/

You need to give this script the right permissons, otherwise ``etckeeper`` wil skip it::

    chmod 755 /etckeeper/pre-commit.d/10rsync-var-lib-named-master.sh

Adding aliases for commands removed in ``net-tools``
----------------------------------------------------

Add this to ``/etc/bash.bashrc.local``::

    # stuff removed from net-tools
    # see https://features.opensuse.org/317197 and https://build.opensuse.org/package/view_file/network:utilities/net-tools/net-tools.changes
    ## Because of changes on Thu Apr 10 12:33:41 UTC 2014
    alias "arp=echo 'use \"ip neigh\" or \"ip -r neight\"' && ip neigh"
    alias "ifconfig=echo 'use \"ip a\"' && ip a"
    alias "netstat= echo 'use \"ss\" or \"ss -r\"' && ss"
    alias "route=echo 'use \"ip r\"' && ip r"
    ## Because of changes on Sun Mar 29 00:41:21 UTC 2015
    alias "ipmaddr=echo 'use \"ip maddr\"' && ip maddr"
    alias "iptunnel=echo 'use \"ip tunnel\"' && ip tunnel"


Configuring ``monit`` monitoring service
----------------------------------------

At first, `monit <https://software.opensuse.org/package/monit>`_ won't run::

    revue:~ # rcmonit restart
    redirecting to systemctl restart monit.service
    Failed to restart monit.service: Unit monit.service failed to load: No such file or directory.

Even though it is an offical package, it is missing the `.service file <http://www.freedesktop.org/software/systemd/man/systemd.service.html>`_.

That is easy to fix by downloading and modifying the ``monit.service`` template https://bitbucket.org/tildeslash/monit/raw/master/system/startup/monit.service.in::

    #! /bin/sh
    #
    # Fixes this error:
    #    revue:~ # rcmonit restart
    #    redirecting to systemctl restart monit.service
    #    Failed to restart monit.service: Unit monit.service failed to load: No such file or directory.
    SERVICE_TARGET=monit.service
    pushd /etc/systemd/system/
    # http://stackoverflow.com/questions/13735051/curl-and-capturing-output-to-a-file
    curl https://bitbucket.org/tildeslash/monit/raw/master/system/startup/monit.service.in -o $SERVICE_TARGET
    # escape slashes in arguments: http://www.grymoire.com/Unix/Sed.html#uh-62
    ## might need to get rid of the backtick and replace \\ by \, see:
    # http://unix.stackexchange.com/questions/5778/whats-the-difference-between-stuff-and-stuff/5782#5782
    MONIT_EXPANDED=`echo "$(which monit)" | sed 's:[]\[\^\$\.\*\/]:\\\\&:g'`
    echo SERVICE_TARGET=$SERVICE_TARGET
    echo MONIT_EXPANDED=$MONIT_EXPANDED

    echo old:
    sed -n "/@prefix@\/bin\/monit/ p" $SERVICE_TARGET
    # replace @prefix@ with the directory where monit resides
    # replace @sysconfigdir@ with etc
    sed -e "/@prefix@\/bin\/monit/ s/@prefix@\/bin\/monit/${MONIT_EXPANDED}/" $SERVICE_TARGET > $SERVICE_TARGET.tmp && mv $SERVICE_TARGET.tmp $SERVICE_TARGET
    sed -e "/@sysconfdir@/ s/@sysconfdir@/etc/" $SERVICE_TARGET > $SERVICE_TARGET.tmp && mv $SERVICE_TARGET.tmp $SERVICE_TARGET
    echo new:
    sed -n "/monitrc/ p" $SERVICE_TARGET

    chmod 755 $SERVICE_TARGET
    popd
    systemctl enable monit.service
    systemctl status monit.service
    systemctl start monit.service

But it still doesn't start, as `journalctl <https://www.google.com/search?q=journalctl>`_ (the logging part part of `systemd <https://en.wikipedia.org/wiki/Systemd>`_) shows::

    revue:/etc # journalctl _COMM=monit
    -- Logs begin at Sat 2015-06-06 10:05:54 CEST, end at Sat 2015-06-06 15:00:01 CEST. --
    Jun 06 10:01:24 revue monit[1496]: Error opening the idfile '/run/monit/.monit.id' -- No such file or directory
    Jun 06 10:01:24 revue monit[1496]: Error opening the idfile '/run/monit/.monit.id' -- No such file or directory
    Jun 06 10:01:24 revue monit[1496]: Starting Monit 5.10 daemon with http interface at [localhost:2812]
    Jun 06 10:01:24 revue monit[1496]: Error opening pidfile '@@PIDDIR@@/monit.pid' for writing -- No such file or directory
    Jun 06 10:01:24 revue monit[1496]: Monit daemon died
    Jun 06 10:01:24 revue monit[1496]: Starting Monit 5.10 daemon with http interface at [localhost:2812]
    Jun 06 10:01:24 revue monit[1551]: Error opening the idfile '/run/monit/.monit.id' -- No such file or directory
    Jun 06 10:01:24 revue monit[1551]: Error opening the idfile '/run/monit/.monit.id' -- No such file or directory
    Jun 06 10:01:24 revue monit[1551]: No daemon process found
    Jun 06 10:01:24 revue monit[1551]: No daemon process found

.. note::

  Note that `journalctl <https://www.google.com/search?q=journalctl>`_ can feel a bit complex for casual users, so to get ``/var/log/messages`` back you might want to install ``rsyslog`` as explained by `Whither /var/log/messages? <https://forums.opensuse.org/showthread.php/505084-Whither-var-log-messages>`_.

  For a comparison, read:

  - `Why journalctl is cool and syslog will survive for another decade « Luc de Louw's Blog <http://blog.delouw.ch/2013/07/24/why-journalctl-is-cool-and-syslog-will-survive-for-another-decade/>`_.
  - `3.8 Configuring and Using System Logging <https://docs.oracle.com/cd/E52668_01/E54670/html/ol7-log-sec.html>`_.

A quick look into ``/etc/monitrc`` reveals the initialisation of the ``monit`` package forgot to create ``/run/monit/.monit.id``::

    revue:/etc # grep "\.monit\.id" /etc/monitrc
    ## default the file is placed in $HOME/.monit.id.
    set idfile /run/monit/.monit.id

The cause is that the ``idfile`` must both exist (see the error message) `and have a unique id in it <https://mmonit.com/wiki/MMonit/FAQ#monitid>`_.

If it exists but does not have a valid id, then you get this error::

    Jun 06 15:34:13 revue systemd[1]: Starting Pro-active monitoring utility for unix systems...
    Jun 06 15:34:13 revue monit[4404]: Error reading id from file '/run/monit/.monit.id'
    Jun 06 15:34:13 revue monit[4404]: Error reading id from file '/run/monit/.monit.id'
    Jun 06 15:34:13 revue monit[4404]: Starting Monit 5.10 daemon with http interface at [localhost:2812]
    Jun 06 15:34:13 revue systemd[1]: monit.service: main process exited, code=exited, status=1/FAILURE

Both issues is easily fixed by creating and running this ``/etc/monit-create-idfile.sh`` script::

    #! /bin/sh
    #
    # creates idfile from configuration in in /etc/monitrc
    ETC_TARGET=/etc/monitrc
    # http://unix.stackexchange.com/questions/84922/extract-a-part-of-one-line-from-a-file-with-sed/84957#84957
    # trick: search and edit and print at the same time
    ID_FILE=`sed -n -e "/^set idfile .*monit.id$/ s/^set idfile // p" $ETC_TARGET`
    echo id file: $ID_FILE
    # http://stackoverflow.com/questions/6121091/get-file-directory-path-from-filepath/6121114#6121114
    ID_FILE_DIRECTORY=$(dirname "${ID_FILE}")
    echo id file directory: $ID_FILE_DIRECTORY
    ls -al $ID_FILE
    mkdir -p $ID_FILE_DIRECTORY
    touch $ID_FILE
    ls -al $ID_FILE
    echo y | monit --resetid
    cat $ID_FILE && echo

Finally we caome to the last error: some more replacement needs to take place to prevent this error because ``monit`` cannot find its `pid <http://stackoverflow.com/questions/8296170/what-is-a-pid-file-and-what-does-it-contain/8296204#8296204>`_ file::

    Jun 06 16:20:48 revue monit[4760]: Error opening pidfile '@@PIDDIR@@/monit.pid' for writing -- No such file or directory
    Jun 06 16:20:48 revue monit[4760]: Monit daemon died

This is then fixed by creating and running this ``/etc/monitrc-fix.sh``::

    #! /bin/sh
    #
    # fixes the pidfile in /etc/monitrc
    ETC_TARGET=/etc/monitrc
    # use double quotes to allow for variable expansion: http://stackoverflow.com/questions/17477890/expand-variables-in-sed/17477911#17477911
    # escape slashes in arguments: http://www.grymoire.com/Unix/Sed.html#uh-62
    echo old:
    sed -n "/^# set pidfile \/var\/run\/monit.pid$/ p" $ETC_TARGET
    sed -e "/^# set pidfile \/var\/run\/monit.pid$/ s/^# //" $ETC_TARGET > $ETC_TARGET.tmp && mv $ETC_TARGET.tmp $ETC_TARGET
    echo new:
    sed -n "/^set pidfile \/var\/run\/monit.pid$/ p" $ETC_TARGET
    systemctl status monit.service
    systemctl start monit.service

More ``monit`` configuration tips (including setting up `HTTPs <https://en.wikipedia.org/wiki/HTTPS>`_ with a `self-signed certificate <https://en.wikipedia.org/wiki/Self-signed_certificate>`_ - imporant as ``monit`` uses plain username/password `http basic authentication <https://en.wikipedia.org/wiki/Basic_access_authentication>`_) are at:

- `How to set up server monitoring system with Monit - Xmodulo <http://xmodulo.com/server-monitoring-system-monit.html>`_.
- `Install Monit on openSUSE 13.2 <http://www.itzgeek.com/how-tos/linux/opensuse/install-monit-on-opensuse-13-2.html>`_.

Configuring apache2 for the first time
--------------------------------------

To display the *Apache* version::

    # httpd2 -v
    Server version: Apache/2.4.12 (Linux/SUSE)
    Server built:   2015-06-09 09:24:07.000000000 +0000

Or since both ``httpd`` and ``httpd2`` point to the same file::

    # httpd2 -v
    Server version: Apache/2.4.12 (Linux/SUSE)
    Server built:   2015-06-09 09:24:07.000000000 +0000

    # ls -al `which httpd2` `which httpd`
    lrwxrwxrwx 1 root root 23 Jun 20 15:26 /usr/sbin/httpd -> /usr/sbin/httpd-prefork
    lrwxrwxrwx 1 root root 23 Jun 20 15:26 /usr/sbin/httpd2 -> /usr/sbin/httpd-prefork

To verify your configuration files are correct, use this command before restarting the apache2 httpd2 server::

    httpd2 -S

Apart from replacing ``combined`` by ``vhost_combined``, you might want to ensure logging is done for each vhost in a separate file: it makes checkout out vhost issues a lot easier.

You can either use the default ``vhost.template`` for that, or the `apache wiki <http://wiki.apache.org>`_ examples: https://wiki.apache.org/httpd/ExampleVhosts

This is my diff between the default ``vhost.template `` and ``pluimers.com.conf`` in ``/etc/apache2/vhosts.d``::

    /etc/apache2/vhosts.d # diff pluimers.com.conf vhost.template
    1c1
    < # pluimers.com apache2 vhost configuration based on
    ---
    > #
    15,19c15,17
    < # <VirtualHost *:80>
    < <VirtualHost *>
    <     ServerAlias *.pluimers.com
    <     ServerAdmin jeroen.pluimers.com+pluimers.com@gmail.com
    <     ServerName revue
    ---
    > <VirtualHost *:80>
    >     ServerAdmin webmaster@dummy-host.example.com
    >     ServerName dummy-host.example.com
    24c22
    <     DocumentRoot /srv/www/vhosts/pluimers.com
    ---
    >     DocumentRoot /srv/www/vhosts/dummy-host.example.com
    27,30c25,26
    <     # ErrorLog /var/log/apache2/pluimers.com/error_log
    <     ErrorLog /var/log/apache2/pluimers.com-error_log
    <     # CustomLog /var/log/apache2/pluimers.com/access_log vhost_combined
    <     CustomLog /var/log/apache2/pluimers.com-access_log vhost_combined
    ---
    >     ErrorLog /var/log/apache2/dummy-host.example.com-error_log
    >     CustomLog /var/log/apache2/dummy-host.example.com-access_log combined
    59c55
    <     ScriptAlias /cgi-bin/ "/srv/www/vhosts/pluimers.com/cgi-bin/"
    ---
    >     ScriptAlias /cgi-bin/ "/srv/www/vhosts/dummy-host.example.com/cgi-bin/"
    64c60
    <     <Directory "/srv/www/vhosts/pluimers.com/cgi-bin">
    ---
    >     <Directory "/srv/www/vhosts/dummy-host.example.com/cgi-bin">
    102c98
    <     <Directory "/srv/www/vhosts/pluimers.com">
    ---
    >     <Directory "/srv/www/vhosts/dummy-host.example.com">
    137d132
    < </VirtualHost>
    138a134
    > </VirtualHost>

Note that I experimented with a log directory per domain like ``/var/log/apache2/pluimers.com/``, but these won't be auto-created, like ``httpd2 -S`` shows::

    /etc/apache2/vhosts.d # httpd2 -S
    VirtualHost configuration:
    ...
    (2)No such file or directory: AH02291: Cannot access directory '/var/log/apache2/pluimers.com/' for error log of vhost defined at /etc/apache2/vhosts.d/pluimers.com.conf:16
    AH00014: Configuration check failed

Note that ``httpd2 -S`` by default does not execute ``
This means that if you have ``SSL`` configured, ``httpd2 -S`` will not take that into account the ``APACHE_SERVER_FLAGS`` setting in ``/etc/sysconfig/apache2``

`Workaround from another frustrated user <http://web.ornl.gov/~jar/Apache/SSL_in_Apache_2.html>`_ which `one day I will make easier to use <http://serverfault.com/questions/702155/why-doesnt-httpd2-s-catch-ssl-certificate-issues-whereas-systemctl-restart/702156#comment868640_702156>`_::

    httpd2 -D SSL -S

The same frustrated user also suggested to make this small change in ``/etc/sysconfig/apache2``, from

    APACHE_LOGLEVEL="warn"

to::

    APACHE_LOGLEVEL="debug"




.. sidebar:: Notes when updating (vhosts) configuration from Apache 2.2 to Apache 2.4:

  Instead of using `mod_access_compat <http://httpd.apache.org/docs/2.4/mod/mod_access_compat.html>`_ modify the configuration files to use the directives in `mod_authz_host <http://httpd.apache.org/docs/2.4/mod/mod_authz_host.html>`_.

  See `Upgrading to 2.4 from 2.2 <http://httpd.apache.org/docs/2.4/upgrading.html>`_

  Replace the lines::

      Order deny,allow
      Deny from all

  with::

      Require all denied

  Replace the lines::

      Order allow,deny
      Allow from all

  with::

      Require all granted

  More background info:

  - Denying: `Apache 2.4 Upgrade and the "Invalid Command 'Order'" Error <https://systembash.com/apache-2-4-upgrade-and-the-invalid-command-order-error/>`_
  - Granting: `Upgrading to Apache 2.4 from Apache HTTP Server 2.2.x <http://brianflove.com/2014/04/23/upgrading-to-apache-2-4-from-apache-http-server-2-2-x/>`_


  **Common problems when upgrading**

  - Startup errors:

    - ``Invalid command 'User', perhaps misspelled or defined by a module not included in the server configuration`` - load module `mod_unixd <http://httpd.apache.org/docs/2.4/mod/mod_unixd.html>`_
    - ``Invalid command 'Require', perhaps misspelled or defined by a module not included in the server configuration``, or ``Invalid command 'Order', perhaps misspelled or defined by a module not included in the server configuration`` - load module `mod_access_compat <http://httpd.apache.org/docs/2.4/mod/mod_access_compat.html>`_, or update configuration to 2.4 authorization directives.
    - ``Ignoring deprecated use of DefaultType in line NN of /path/to/httpd.conf`` - remove `DefaultType <http://httpd.apache.org/docs/2.4/mod/core.html#defaulttype>`_ and replace with other configuration settings.
    - ``Invalid command 'AddOutputFilterByType', perhaps misspelled or defined by a module not included in the server configuration`` - `AddOutputFilterByType <http://httpd.apache.org/docs/2.4/mod/mod_filter.html#addoutputfilterbytype>`_ has moved from the core to mod_filter, which must be loaded.

  - Errors serving requests:

    - ``configuration error: couldn't check user: /path`` - load module `mod_authn_core <http://httpd.apache.org/docs/2.4/mod/mod_authn_core.html>`_.
    - ``.htaccess files aren't being processed`` - Check for an appropriate `AllowOverride <http://httpd.apache.org/docs/2.4/mod/core.html#allowoverride>`_ directive; the default changed to ``None`` in 2.4.

Getting and running ``testssl.sh``
----------------------------------

Simple steps from a non-root account::

    git clone https://github.com/drwetter/testssl.sh.git
    cd testssl.sh

    OPENSSL=./openssl-bins/openssl-1.0.2-chacha.pm/openssl32-1.0.2pm-krb5.chacha+poly ./testssl.sh beginend.net
    OPENSSL=./testssl.sh beginend.net
    OPENSSL=./openssl-bins/openssl-1.0.2-chacha.pm/openssl32-1.0.2pm-krb5.chacha+poly ./testssl.sh www.beginend.net
    OPENSSL=./testssl.sh www.beginend.net

Login / Reboot Fritz!Box
------------------------

There is a bash script for Fritz!Box access at <https://github.com/jpluimers/bash-fritzclient>.

Installation is simple:

1. Go to ``/etc``
2. ``git clone https://github.com/jpluimers/bash-fritzclient.git``
3. Copy ``bash-fritzclient\bash-fritzclient.config.template`` to ``/etc/bash-fritzclient.template``
4. Configure ``/etc/bash-fritzclient.template``.

configuring and running shellinabox with ssh
--------------------------------------------

A great tool for configuring your machine over a connection not allowing ssh is `shellinabox <http://shellinabox.com>`__.

Certificates are in ``/etc/shellinabox/certs``.

After installation, it isn't running::

    revue:/etc/shellinabox/certs # systemctl status shellinabox.service
    ● shellinabox.service - LSB: shellinabox
       Loaded: loaded (/etc/init.d/shellinabox)
       Active: inactive (dead)
         Docs: man:systemd-sysv-generator(8)

    revue:/etc/shellinabox/certs # systemctl enable shellinabox.service
    shellinabox.service is not a native service, redirecting to /sbin/chkconfig.
    Executing /sbin/chkconfig shellinabox on
    revue:/etc/shellinabox/certs # systemctl start shellinabox.service
    revue:/etc/shellinabox/certs # systemctl status shellinabox.service
    ● shellinabox.service - LSB: shellinabox
       Loaded: loaded (/etc/init.d/shellinabox)
       Active: active (running) since Tue 2015-06-09 19:56:21 CEST; 20s ago
         Docs: man:systemd-sysv-generator(8)
      Process: 4997 ExecStart=/etc/init.d/shellinabox start (code=exited, status=0/SUCCESS)
       CGroup: /system.slice/shellinabox.service
               ├─5030 /usr/bin/shellinaboxd --background=/var/run/shellinaboxd.pid -u shellinabox -s /:SSH -c /etc/shellinabox/certs
               └─5031 /usr/bin/shellinaboxd --background=/var/run/shellinaboxd.pid -u shellinabox -s /:SSH -c /etc/shellinabox/certs

    Jun 09 19:56:20 revue shellinabox[4997]: No shellinabox certificate found, creating one now...
    Jun 09 19:56:20 revue shellinabox[4997]: Generating a 2048 bit RSA private key
    Jun 09 19:56:20 revue shellinabox[4997]: .................................+++
    Jun 09 19:56:21 revue shellinabox[4997]: .............................................................+++
    Jun 09 19:56:21 revue shellinabox[4997]: unable to write 'random state'
    Jun 09 19:56:21 revue shellinabox[4997]: writing new private key to '/tmp/create-ssl-key-7GwyL'
    Jun 09 19:56:21 revue shellinabox[4997]: -----
    Jun 09 19:56:21 revue shellinabox[4997]: Created certificate: SHA1 Fingerprint=1B:AE:9D:C3:57:37:34:BB:64:79:0D:3D:D4:B9:50:54:9F:FE:FC:82
    Jun 09 19:56:21 revue shellinabox[4997]: Starting shellinabox ..done
    Jun 09 19:56:21 revue systemd[1]: Started LSB: shellinabox.
    revue:/etc/shellinabox/certs # ls -al
    total 12
    drwxr-xr-x 1 root        root          70 Jun  9 19:56 .
    drwxr-xr-x 1 root        root          10 May 17 10:41 ..
    lrwxrwxrwx 1 root        root          15 Jun  9 19:56 064fecbc.0 -> certificate.pem
    lrwxrwxrwx 1 root        root          15 Jun  9 19:56 b3543706.0 -> certificate.pem
    -rw------- 1 shellinabox shellinabox 2916 Jun  9 19:56 certificate.pem
    revue:/etc/shellinabox/certs # nmap -sV -p 4200 localhost

    Starting Nmap 6.47 ( http://nmap.org ) at 2015-06-09 19:58 CEST
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.00013s latency).
    PORT     STATE SERVICE VERSION
    4200/tcp open  http    ShellInABox httpd

    Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 11.49 seconds

This will work locally::

    lynx http://localhost:4200

Remotely, it needs the firewall to be enabled for it:

1. Start ``yast``
2. Go to ``Security and Users``, ``Firewall``
3. Go to ``Allowed Services``
4. Ensure ``Shellinabox`` is in the list, when not:

  1. Add ``Shellinabox`` to the list
  2. Press ``Next`` followed by ``Finish`` to apply the changes

5. Quit ``yast``

Now it works, but note that the https isn't really secure. Chrome will show ``ERR_SSL_VERSION_OR_CIPHER_MISMATCH``, and this will show far more details::

    jeroenp@revue:~/testssl.sh> OPENSSL=./openssl-bins/openssl-1.0.2-chacha.pm/openssl32-1.0.2pm-krb5.chacha+poly ./testssl.sh localhost:4200

So the best is setting up an Apache redirect as shown in in the `shellinabox configuration page <https://code.google.com/p/shellinabox/wiki/shellinaboxd_man#CONFIGURATION>`_::

    <Location /shell>
      ProxyPass  http://localhost:4200/
      Order      allow,deny
      Allow      from all
    </Location>

For Apache 2.4 we need to slightly change that as we saw when configuring ``apache2`` above. So add these lines to ``/etc/apache2/vhosts.d/00-default.snap.conf``::

    <Location /shell>
      ProxyPass  http://localhost:4200/
      Require all granted
    </Location>

Now test your vhost configuration by running this command::

    htttpd2 -S

If you get the below error, then you need tht http proxy modue to be installed in apache2::

    Invalid command 'ProxyPass', perhaps misspelled or defined by a module not included in the server configuration

In that case, in ``/etc/sysconfig/apache2`` find the line starting with ``APACHE_MODULES`` and add ``proxy_http_module`` to the lis of modules, then perform the ``httpd2 -S `` check again. Finally, restart ``apache2`` with this command::

    systemctl status apache2.service

.. note::

  We need to change ``/etc/sysconfig/apache2`` because ``yast`` will nuke the vhost configs into IP-based-vhosts. See `Configuring Apache <https://www.suse.com/documentation/sles11/book_sle_admin/data/sec_apache2_configuration.html>`_ for more details about manually configuring these files without using ``yast``.

If apache doesn't restart, then use ``journalctl -xe`` fo find out what went wrong. In my case ``proxy_http_module`` wasn't installed it's because that's the name in the ``/etc/apache2/sysconfig.d`` which is generated by ``/etc/sysconfig/apache2``. In ``/etc/sysconfig/apache2`` you need two entries appended to ``APACHE_MODULES``::

     mod_proxy mod_proxy_http

I found out about this by rereading `Apache Module mod_proxy_http <https://httpd.apache.org/docs/2.4/mod/mod_proxy_http.html>`_ three times.

If this works, then you should see ``shellinabox`` when going to <http://localhost/shell>> but not yet for <https://localhost/shell>. For the latter we need to enable ``https`` in ```apache2``.

Viewing the long journal
------------------------

Like mentioned before, ``journalctl`` views the log journal.

An even handier command is this::

    journalctl -xe

This is shorthand for::

    journalctl --catalog --pager-end

It uses these options::

    -e --pager-end           Immediately jump to the end in the pager
    -x --catalog             Add message explanations where available

This is exactly why I like it over log files:

- it has explanations that can come in very handy
- it directly goes to the pager (on my system ``less``)

Hardening apache2 SSL cyphers
-----------------------------

Via `Hardening Your Web Server’s SSL Ciphers — Hynek Schlawack <https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/>`_:

Edit ``/etc/apache2/ssl-global.conf``, then modify/add these lines::

    --- a/apache2/ssl-global.conf
    +++ b/apache2/ssl-global.conf
    @@ -77,7 +77,17 @@
            #   SSL Cipher Suite:
            #   List the ciphers that the client is permitted to negotiate.
            #   See the mod_ssl documentation for a complete list.
    -       SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
    +       # SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
    +        # https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    +        SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
    +        # normally this is in copies of default-vhost-ssl.conf, but it needs to be default:
    +        # https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    +        SSLHonorCipherOrder On
    +
    +        ##  SSL compression:
    +        # https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
    +        # as of Apache2 2.4.4 the default is Off; this is in case you ever run on a lower version.
    +        SSLCompression Off

            #   Server Certificate:
            #   Point SSLCertificateFile at a PEM encoded certificate.  If

After that, test the config, then restart ``apache2``::

    revue:~ # apache2ctl -t
    Syntax OK
    revue:~ # systemctl restart apache2.service
    revue:~ # systemctl status apache2.service
    ● apache2.service - The Apache Webserver
       Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; vendor preset: disabled)
       Active: active (running) since Thu 2015-06-11 20:58:12 CEST; 8s ago
     Main PID: 23760 (httpd-prefork)
       Status: "Processing requests..."
       CGroup: /system.slice/apache2.service
               ├─23760 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start
               ├─23780 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start
               ├─23781 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start
               ├─23782 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start
               ├─23783 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start
               └─23784 /usr/sbin/httpd-prefork -f /etc/apache2/httpd.conf -DSSL -D SYSTEMD -DFOREGROUND -k start

    Jun 11 20:58:11 revue systemd[1]: Starting The Apache Webserver...
    Jun 11 20:58:12 revue systemd[1]: Started The Apache Webserver.
    revue:~ #

Additional information at `Secure your Apache Server <https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html>`_.

Enabling https for apache2
--------------------------

First get and install a certificate.

Then enable `mod_ssl <http://httpd.apache.org/docs/2.4/mod/mod_ssl.html>`_, in ``/etc/sysconfig/apache2`` you need one entrie appended to ``APACHE_MODULES``::

     mod_ssl

.. note::

   TODO figure out how to use this together with `mod_proxy` and `mod_proxy_http`; see `how to make Apache proxy http requests to https <http://stackoverflow.com/questions/9977215/how-to-make-apache-proxy-http-requests-to-https>`.


Generating the private key protected with a strong password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: strong passwords

  `Avoid weak passwords <https://en.wikipedia.org/wiki/Password_strength#Examples_of_weak_passwords>`_.

  One way of creating a single strong password is by estimating `required bits of entropy <https://en.wikipedia.org/wiki/Password_strength#Required_Bits_of_Entropy>`_, then choosing a `set of characters` and `password length` and generate a `random password with at least that length <https://en.wikipedia.org/wiki/Password_strength#Random_passwords>`_).

  Here 16 characters gets you about 80 bits of precision.

  But remember that often you need multiple strong passwords, so be sure to read some `guidelines around strong passwords <https://en.wikipedia.org/wiki/Password_strength#Guidelines_for_strong_passwords>`_.



First a few notes:

1. `There is no AES-512 <http://crypto.stackexchange.com/questions/20253/why-we-cant-implement-aes-512-key-size>`_, so the best to use is `AES-256 <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard#Description_of_the_cipher>`_ (AES wiht a 256-bit key).
2. `RSA keys should be at least 2048 bits long <https://en.wikipedia.org/wiki/RSA_(cryptosystem)#cite_note-20>`_, but 4096 provide even more security (`the factoring of a 4096-bit RSA key was a faulty copy <https://blog.hboeck.de/archives/872-About-the-supposed-factoring-of-a-4096-bit-RSA-key.html>`_).
3. `DSA keys <https://en.wikipedia.org/wiki/Digital_Signature_Algorithm>`_ are limited to 1024 bits. Don't use them.

The `genrsa <https://www.openssl.org/docs/apps/genrsa.html>`_ command of openssl generates RSA keys.

Generate a 4096 bit RSA private key (keep it in a safe place!) encrypted using the 256-bit AES algorithm (be sure to give it a `strong password <https://en.wikipedia.org/wiki/Password_strength>`_!::

    openssl genrsa -out 4096-bit-rsa-key-encrypted-using-256-bit-aes.private.key -aes256 4096

You can also use `ssh-keygen <https://en.wikipedia.org/wiki/Ssh-keygen>`_, but the `default setup is not that secure <https://wiki.osuosl.org/howtos/ssh_key_tutorial.html>`_ and `making it more secure requires openssl <https://github.com/nickchappell/personal-projects-docs/blob/master/services_notes/ssh_notes.md>`_.

Generate the CSR (Certificate Signing Request)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar:: `tmpfs` on a Mac

  On a Mac you have `these nice scripts <https://gist.github.com/koshigoe/822455>`_ by `koshigoe <https://gist.github.com/koshigoe>`_ to help you creating a `tmpfs`.

  They use `hdid <https://www.google.com/search?q=hdid>`_,
  `newfs_hfs <https://www.google.com/search?q=newfs_hfs>`_,
  `mount <https://www.google.com/search?q=mount>`_,
  `umount <https://www.google.com/search?q=umount>`_ and
  `hdiutil <https://www.google.com/search?q=hdiutil>`_.

  `mount-ram.sh <https://gist.githubusercontent.com/koshigoe/822455/raw/dd33aeeb0267040743a4f4272cab371e0880770f/mount-ram.sh>`_::

      #!/bin/sh

      # This program has two feature.
      #
      # 1. Create a disk image on RAM.
      # 2. Mount that disk image.
      #
      # Usage:
      #   $0 <dir> <size>
      #
      #   size:
      #     The `size' is a size of disk image (MB).
      #
      #   dir:
      #     The `dir' is a directory, the dir is used to mount the disk image.
      #
      # See also:
      #   - hdid(8)
      #

      mount_point=${1}
      size=${2:-64}

      mkdir -p $mount_point
      if [ $? -ne 0 ]; then
          echo "The mount point didn't available." >&2
          exit $?
      fi

      sector=$(expr $size \* 1024 \* 1024 / 512)
      device_name=$(hdid -nomount "ram://${sector}" | awk '{print $1}')
      if [ $? -ne 0 ]; then
          echo "Could not create disk image." >&2
          exit $?
      fi

      newfs_hfs $device_name > /dev/null
      if [ $? -ne 0 ]; then
          echo "Could not format disk image." >&2
          exit $?
      fi

      mount -t hfs $device_name $mount_point
      if [ $? -ne 0 ]; then
          echo "Could not mount disk image." >&2
          exit $?
      fi

  `umount-ram.sh <https://gist.githubusercontent.com/koshigoe/822455/raw/802073f8f0791ba050180986dfe9091f1bca9abf/umount-ram.sh>`_::

      #!/bin/sh

      # This program has two features.
      #
      # 1. Unmount a disk image.
      # 2. Detach the disk image from RAM.
      #
      # Usage:
      #   $0 <dir>
      #
      #   dir:
      #     The `dir' is a directory, the dir is mounting a disk image.
      #
      # See also:
      #   - hdid(8)
      #

      mount_point=$1
      if [ ! -d "${mount_point}" ]; then
          echo "The mount point didn't available." >&2
          exit 1
      fi
      mount_point=$(cd $mount_point && pwd)

      device_name=$(df "${mount_point}" 2>/dev/null | tail -1 | grep "${mount_point}" | cut -d' ' -f1)
      if [ -z "${device_name}" ]; then
          echo "The mount point didn't mount disk image." >&2
          exit 1
      fi

      umount "${mount_point}"
      if [ $? -ne 0 ]; then
          echo "Could not unmount." >&2
          exit $?
      fi

      hdiutil detach -quiet $device_name

Here we will generate a `CSR <https://en.wikipedia.org/wiki/Certificate_signing_request>`_ using SHA-256 (which is a `secure hashing <https://en.wikipedia.org/wiki/Secure_Hash_Algorithm>`_ function that is part of the `SHA-2 <https://en.wikipedia.org/wiki/SHA-2#Comparison_of_SHA_functions>`_ family of hashing functions `secure enough for the forseeable future <http://crypto.stackexchange.com/questions/3153/sha-256-vs-any-256-bits-of-sha-512-which-is-more-secure>`_).

1. Copy your encrypted private key to a temporary directory (important: you **have to clean that directory later on**) preferably in a `tmpfs <https://en.wikipedia.org/wiki/Tmpfs>`_ temporary file system.

2. Decrypt your key (enter your password when openssl asks for it)::

    openssl rsa -in 4096-bit-rsa-key-encrypted-using-256-bit-aes.private.key -out 4096-bit-rsa-key.private.key

3. Create a signing request using the decrypted key for your domain (in this case for the `pluimers.com` domain) with some sensible attributes::

    # openssl req -new -sha256 -key 4096-bit-rsa-key.private.key -out pluimers.com.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:NL
    State or Province Name (full name) [Some-State]:Noord-Holland
    Locality Name (eg, city) []:Amsterdam
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Pluimers Software Ontwikkeling B.V.
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:pluimers.com
    Email Address []:webmaster@pluimers.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:
    # openssl req -noout -text -in pluimers.com.csr
        Data:
            Version: 0 (0x0)
            Subject: C=NL, ST=Noord-Holland, L=Amsterdam, O=Pluimers Software Ontwikkeling B.V., CN=pluimers.com/emailAddress=webmaster@pluimers.com
            Subject Public Key Info:

4. Follow the `StartSSL` steps at `Generating the Certificate <https://konklone.com/post/switch-to-https-now-for-free#generating-the-certificate>`_.

    - note you can only have 1 specific subdomain when your StartSSL identity is class 1.
    - the upload and processing of the CRS takes a few minutes
    - the generation of the certificate can take like 5 minutes
    - copy the resulting certificate to ``pluimers.com.crt``
    - on your OpenSuSE server, save these files:
        - Certificate: ``/etc/apache2/ssl.crt/pluimers.com.crt``
        - Decrypted private key: ``/etc/apache2/ssl.key/pluimers.com.key``

5. Fix this error by changing ``/etc/apache2/vhosts.d/pluimers.com-ssl.conf`` from ``ServerName revue`` into ``ServerName www.pluimers.com``::

    [Sun Jun 28 16:43:40.342067 2015] [ssl:debug] [pid 5251] ssl_util_ssl.c(356): AH02412: [revue:443] Cert does not match for name 'revue' [subject: emailAddress=webmaster@pluimers.com,CN=www.pluimers.com,C=NL / issuer: CN=StartCom Class 1 Primary Intermediate Server CA,OU=Secure Digital Certificate Signing,O=StartCom Ltd.,C=IL / serial: 05D759EC0CF620 / notbefore: Jun 14 08:24:00 2015 GMT / notafter: Jun 14 14:46:36 2016 GMT]


<http://ndg-security.ceda.ac.uk/wiki/ndg_security/Apache2/SUSE#SSL>

SSLLabs test will most likely give this downgrade:

    This server's certificate chain is incomplete. Grade capped to B.

So make sure you chain your certificates when using startSSL::

    pushd /etc/apache2/ssl.crt
    wget -m -np https://www.startssl.com/certs/class1/sha2/pem/sub.class1.server.sha2.ca.pem
    cat pluimers.com.crt www.startssl.com/certs/class1/sha2/pem/sub.class1.server.sha2.ca.pem > pluimers.com-chain.crt
    popd

Now ensure these two lines in ``/etc/apache2/vhosts.d/pluimers.com-ssl.conf`` are as follows::

    #SSLCertificateFile /etc/apache2/ssl.crt/pluimers.com.crt
    SSLCertificateKeyFile /etc/apache2/ssl.key/pluimers.com.key
    SSLCertificateChainFile /etc/apache2/ssl.crt/pluimers.com-chain.crt

Finally, restart the apache2 service::

    rcapache2 stop
    rcapache2 start

If it fails, then look through errors in ``/var/log/apache2/pluimers.com-ssl-error_log``, as ``journalctl -xe`` will not show details.

.. sidebar:: Important apache restart note

  ``rcapache2 restart`` will not fully unload the apache configuration. Use this in stead::

      rcapache2 stop
      rcapache2 start

  Without it, you will get spurious errors (like https://www.pluimers.com re-using part of the virtual directory configuration for http://www.pluimers.com thereby generating spurious 403-errors) in log files like the ``403 1032`` and ``403 1018`` error codes and ``authorization result of <RequireAny>: denied`` in below logs.

  BTW:

    Note the differences in time-stamp logging. Don't you hate that? What happened to ISO-8601?

  pluimers.com-ssl_request_log::

      [29/Jun/2015:20:16:40 +0200] 80.100.143.119 TLSv1.2 ECDHE-RSA-AES128-GCM-SHA256 "GET / HTTP/1.1" 1032 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Safari/537.36"
      [29/Jun/2015:20:16:40 +0200] 80.100.143.119 TLSv1.2 ECDHE-RSA-AES128-GCM-SHA256 "GET /favicon.ico HTTP/1.1" 1018 "https://www.pluimers.com/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Safari/537.36"

  pluimers.com-access_log::

      revue 80.100.143.119 - - [29/Jun/2015:20:16:40 +0200] "GET / HTTP/1.1" 403 1032 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Safari/537.36"
      revue 80.100.143.119 - - [29/Jun/2015:20:16:40 +0200] "GET /favicon.ico HTTP/1.1" 403 1018 "https://www.pluimers.com/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.130 Safari/537.36"

  pluimers.com-ssl-error_log::

      [Mon Jun 29 20:16:40.266336 2015] [ssl:info] [pid 5588] [client 80.100.143.119:63297] AH01964: Connection to child 5 established (server revue:443)
      [Mon Jun 29 20:16:40.267090 2015] [ssl:debug] [pid 5588] ssl_engine_kernel.c(1908): [client 80.100.143.119:63297] AH02043: SSL virtual host for servername www.pluimers.com found
      [Mon Jun 29 20:16:40.267832 2015] [ssl:info] [pid 5589] [client 80.100.143.119:63298] AH01964: Connection to child 6 established (server revue:443)
      [Mon Jun 29 20:16:40.268728 2015] [ssl:debug] [pid 5589] ssl_engine_kernel.c(1908): [client 80.100.143.119:63298] AH02043: SSL virtual host for servername www.pluimers.com found
      [Mon Jun 29 20:16:40.298407 2015] [ssl:debug] [pid 5588] ssl_engine_kernel.c(1841): [client 80.100.143.119:63297] AH02041: Protocol: TLSv1.2, Cipher: ECDHE-RSA-AES128-GCM-SHA256 (128/128 bits)
      [Mon Jun 29 20:16:40.303995 2015] [ssl:debug] [pid 5589] ssl_engine_kernel.c(1841): [client 80.100.143.119:63298] AH02041: Protocol: TLSv1.2, Cipher: ECDHE-RSA-AES128-GCM-SHA256 (128/128 bits)
      [Mon Jun 29 20:16:40.646809 2015] [ssl:debug] [pid 5588] ssl_engine_kernel.c(243): [client 80.100.143.119:63297] AH02034: Initial (No.1) HTTPS request received for child 5 (server revue:443)
      [Mon Jun 29 20:16:40.647091 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of Require all denied: denied
      [Mon Jun 29 20:16:40.647104 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of <RequireAny>: denied
      [Mon Jun 29 20:16:40.647112 2015] [authz_core:error] [pid 5588] [client 80.100.143.119:63297] AH01630: client denied by server configuration: /srv/www/vhosts/pluimers.com/
      [Mon Jun 29 20:16:40.647239 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of Require all granted: granted
      [Mon Jun 29 20:16:40.647250 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of <RequireAny>: granted
      [Mon Jun 29 20:16:40.792584 2015] [ssl:debug] [pid 5588] ssl_engine_kernel.c(243): [client 80.100.143.119:63297] AH02034: Subsequent (No.2) HTTPS request received for child 5 (server revue:443), referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:40.792637 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of Require all denied: denied, referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:40.792667 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of <RequireAny>: denied, referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:40.792676 2015] [authz_core:error] [pid 5588] [client 80.100.143.119:63297] AH01630: client denied by server configuration: /srv/www/vhosts/pluimers.com/favicon.ico, referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:40.792711 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of Require all granted: granted, referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:40.792722 2015] [authz_core:debug] [pid 5588] mod_authz_core.c(809): [client 80.100.143.119:63297] AH01626: authorization result of <RequireAny>: granted, referer: https://www.pluimers.com/
      [Mon Jun 29 20:16:50.646230 2015] [ssl:info] [pid 5589] (70014)End of file found: [client 80.100.143.119:63298] AH01991: SSL input filter read failed.
      [Mon Jun 29 20:16:50.646659 2015] [ssl:debug] [pid 5589] ssl_engine_io.c(1003): [client 80.100.143.119:63298] AH02001: Connection closed to child 6 with standard shutdown (server revue:443)
      [Mon Jun 29 20:16:55.808702 2015] [ssl:info] [pid 5588] (70007)The timeout specified has expired: [client 80.100.143.119:63297] AH01991: SSL input filter read failed.
      [Mon Jun 29 20:16:55.808989 2015] [ssl:debug] [pid 5588] ssl_engine_io.c(1003): [client 80.100.143.119:63297] AH02001: Connection closed to child 5 with standard shutdown (server revue:443)


------------------

0. Remove the temporary directory (preferably, delete the whole ``tmpfs`` volume it is on).


<https://blog.websenat.de/2014/05/03/sysadmin-openssl-befehle-und-tipps/>

<https://twitter.com/konklone/status/493748256933810178>

<https://konklone.com/post/switch-to-https-now-for-free#generating-the-certificate>

<https://gist.github.com/konklone/98f48a90bfd9cfd076dc>
Full log::

    # openssl req -new -sha256 -key 4096-bit-rsa-key.private.key -out pluimers.com.csr
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:NL
    State or Province Name (full name) [Some-State]:Noord-Holland
    Locality Name (eg, city) []:Amsterdam
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:Pluimers Software Ontwikkeling B.V.
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:pluimers.com
    Email Address []:webmaster@pluimers.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:



Notes
~~~~~

<https://www.suse.com/documentation/sles11/book_sle_admin/data/sec_apache2_ssl.html#sec_apache2_ssl_configuration_name-based>


<https://192.168.71.62/shell>

fails: <https://revue.pluimers.com/shell>

<https://80.100.143.119/shell>

<https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/>

<http://httpd.apache.org/docs/2.4/ssl/ssl_howto.html>

<https://raymii.org/s/tutorials/Strong_SSL_Security_On_Apache2.html>

<http://www.techytalk.info/remote-cli-access-to-ubuntu-pc-using-web-browser-through-authenticated-https/comment-page-1/>

<https://konklone.com/post/switch-to-https-now-for-free>

<https://letsencrypt.org>

<http://www.unixmen.com/create-ssl-cetificates-in-opensuse-12-3/>

<https://en.opensuse.org/SDB:Apache_installation>

<https://www.suse.com/documentation/sles11/book_sle_admin/data/sec_apache2_ssl.html>

<http://stackoverflow.com/questions/24888407/setting-up-apache-w-ssl-using-opensuse-yast>

<http://blog.remibergsma.com/2013/03/15/always-available-linux-terminal-shell-in-a-box-on-raspberry-pi/>

.. Note:: test with testssl.sh

<https://weakdh.org/>

Ligt complexer; het SHA-256 verhaal gaat over de certificate signature; niet over de versleuteling van de verbinding zelf. Zie ook: http://googleonlinesecuri...lly-sunsetting-sha-1.html

Om 'modern cryptography / groene balk / geen warnings / errors ' te krijgen in de nieuwste Chrome, IE en Firefox moet je dus certs hebben die op RSA 2048 with SHA-256 of langer gebaseerd zijn of langer, dus geen RSA-512/1024 of SHA-1 of MD5 meer in de certificaat-chain (op de root CA na) en tevens dient er moderne, veilige ciphers met Forward Secrecy gebruikt te worden; waarbij AES-GCM de meest gangbare is. Ook b.v. AES-256-CBC is dus niet goed, omdat CBC een niet-authenticated blockcipher is en GCM wel authenticated is (en stukken sneller !).

Voor meer info zie b.v: http://googleonlinesecuri...lly-sunsetting-sha-1.html Of de SslLabs blog: https://community.qualys.com/blogs/securitylabs


Zypper updating
---------------

The only way to update Tumbleweed is through the distribution update::

    zypper dup

If it doesn't update anything: find when more repostories are added:

<https://forums.opensuse.org/showthread.php/506273-Distribution-Upgrade-on-Tumbleweed-Gnome-(sudo-zypper-dup)>

<http://linux.derkeiler.com/Mailing-Lists/SuSE/2014-11/msg01554.html>

Show all the details of your configured repositories::

    zypper repos --details

mariadb dependency
------------------

20150706 - somehow mariadb got installed (MySQL)::

    (Use the Enter or Space key to scroll the text by lines or pages.)

    Message from package mariadb:


    You just installed MySQL server for the first time.

    You can start it using:
     rcmysql start

    During first start empty database will be created for your automatically.

    PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
    To do so, start the server, then issue the following commands:

    '/usr/bin/mysqladmin' -u root password 'new-password'
    '/usr/bin/mysqladmin' -u root -h misibook password 'new-password'

    Alternatively you can run:
    '/usr/bin/mysql_secure_installation'

    which will also give you the option of removing the test
    databases and anonymous user created by default. This is
    strongly recommended for production servers.


    -----------------------------------------------------------------------------


    (Press 'q' to exit the pager.)
    /var/tmp/TmpFile.A0c0jv lines 1-30/30 (END)

----------------------------------------------------------------------------

.. [#opensuse_footnote] I keep using the old `SuSE <https://en.wikipedia.org/wiki/SUSE>`_ writing, I'm an old fart.

.. [#tumbleweed_footnote] `Tumbleweed <https://en.opensuse.org/Portal:Tumbleweed>`_ is the rolling release of OpenSuSE.

.. [#revue_footnote] See `Snip en Snap revue <https://en.wikipedia.org/wiki/Snip_en_Snap>`_.

.. [#headless_footnote] `Headless <https://en.wikipedia.org/wiki/Headless_software>`_ as in no GUI, not as in `Embedded System <https://en.wikipedia.org/wiki/Embedded_system>`_. So there is a text `console <https://en.wikipedia.org/wiki/System_console>`_, and remote `ssh <https://en.wikipedia.org/wiki/Secure_Shell>`_.

.. [#patterns-openSUSE-minimal_base-conflicts_footnote] The `patterns-openSUSE-minimal_base-conflicts <https://www.google.com/search?q=patterns-openSUSE-minimal_base-conflicts>`_ is there to `prevent recommended packages to blow up a minimal installation <http://unix.stackexchange.com/questions/144438/missing-broken-dependancies-on-opensuse-normal/144583#144583>`_

.. [#removeconflicts_footnote] The `actual conflicts package <http://unix.stackexchange.com/questions/73427/cant-install-python-because-of-zypper-conflict>`_ contains the version number of the distribution you use.
