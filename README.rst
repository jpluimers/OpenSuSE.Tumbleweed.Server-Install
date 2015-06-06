################################
revue: getting Tumbleweed on it.
################################

This is a series of notes and articles about getting OpenSuSE [#opensuse]_ Tumbleweed [#tumbleweed]_ to run on a headless [#headless]_ (i.e. text-only, no GUI) server.

The first machine I installed OpenSuSE Tumbleweed on is ``revue`` [#revue]_. It is a bit of a gimmic, as I already had machines called ``snip`` and ``snap``. Dutch people will know why.

``snip`` and ``snap`` run regular OpenSuSE releases. ``revue`` is my first machine running a `rolling<https://en.wikipedia.org/wiki/Rolling_release>`_ release.

Over time, there will be some documents indicating installation progress and problem solving.

TODO
====

- web site (HTTP-HTTPS)
- pop3 (port 110)
- email (SMTP-SSMTP) 25/587
- rsync (backups) port 873
- modem reboot script (when either ipv4 or ipv6 are down)
- certificates for web and shellinabox
- update root zones through cron
- DNS security <https://www.digitalocean.com/community/tutorials/how-to-setup-dnssec-on-an-authoritative-bind-dns-server--2> and <http://csrc.nist.gov/groups/SMA/fasp/documents/network_security/NISTSecuringDNS/NISTSecuringDNS.htm>
- ensure 10rsync-var-lib-named-master.sh works
- reboot / reconnect fritz!box http://www.gtkdb.de/index_7_1302.html
- fix syslogd and logrotate

NOTES
=====

syslogd
~~~~~~~

See `this output<http://www.linuxquestions.org/questions/linux-general-1/how-to-completely-remove-service-from-systemd-using-systemctl-opensuse-4175531795/>`_::

    revue:/etc/xinetd.d # systemctl --failed --all
      UNIT              LOAD   ACTIVE SUB    DESCRIPTION
    ● logrotate.service loaded failed failed Rotate log files
    ● syslogd.service   loaded failed failed System Logging Service

    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    2 loaded units listed.
    To show all installed unit files use 'systemctl list-unit-files'.

Login / Reboot Fritz!Box
~~~~~~~~~~~~~~~~~~~~~~~~

- https://home.debian-hell.org/blog/2013/05/13/update-konfiguration-der-avm-fritzbox-7390-per-wgetcurl-script-sichern/
- https://debianforum.de/forum/viewtopic.php?f=15&t=149338
- https://www.symcon.de/forum/threads/20405-Funktionierende-Scripts-f%C3%BCr-FRITZ!OS-05-50-7390/page3
- http://www.ip-phone-forum.de/showthread.php?t=196309&page=5
- https://github.com/XIDA/windowstools/blob/master/tools/callsTray/callsTray.ahk
- http://www.fritzmod.net/en/tools/curl/
- https://packetstormsecurity.com/files/129834/AVM-Fritz-box-Auto-Exploiter.html
- https://debianforum.de/forum/viewtopic.php?f=26&t=136977
- http://board.gulli.com/thread/1754005-hilfe-fritzbox-ohne-internet-neustarten/
- https://github.com/carlos22/fritzbox_api_php/blob/master/bin/fritzbox_reboot.php
- http://www.delphipraxis.net/91808-fritzbox-reconnector-update-fritzbox-control-2-0-videos.html
- http://www.winfuture-forum.de/index.php?showtopic=165894
- http://www.wehavemorefun.de/fritzbox/Anrufliste_von_der_Box_holen

- https://www.64k-tec.de/2010/01/fritzbox-tuning-part-1-enable-remote-access-over-ssh/
- http://sourceforge.net/p/openpli/mailman/openpli-git-commits/?viewmonth=201304&viewday=28

- http://192.168.71.1/system/reboot.lua?sid=5abed5e90f9c7e99&

- Python library: https://github.com/valpo/fritzbox
- sh sid md5 login: http://www.ip-phone-forum.de/showthread.php?t=264639
- http://homematic-forum.de/forum/viewtopic.php?f=26&t=11645
- sh sid md5 login download config: https://home.debian-hell.org/blog/2013/03/21/konfiguration-der-avm-fritzbox-7390-per-wgetcurl-script-sichern/
- https://home.debian-hell.org/blog/2013/05/13/update-konfiguration-der-avm-fritzbox-7390-per-wgetcurl-script-sichern/

- deze werkt! https://home.debian-hell.org/dokuwiki/scripts/fritzbox.backup.mit.curl.bash

- http://superuser.com/questions/149329/what-is-the-curl-command-line-syntax-to-do-a-post-request
- http://curl.haxx.se/docs/httpscripting.html#POST

html form code::

    <form action="/system/reboot.lua" method="POST">
    <div id="btn_form_foot">
    <input type="hidden" name="sid" value="5abed5e90f9c7e99">
    <button type="submit" name="reboot">Restart</button>
    </div>
    </form>

Rest
~~~~

``create-shellinabox-self-signed-certificate.sh``::

    cd /var/lib/shellinabox
    openssl genrsa -des3 -out server.key 1024
    openssl req -new -key server.key -out server.csr
    cp server.key server.key.org
    openssl rsa -in server.key.org -out server.key
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
    cat server.crt server.key > certificate.pem

Wget/curl are the beste solution to update the ``root.hint``. See:

- <http://lists.opensuse.org/opensuse/2008-05/msg01589.html> - use dig, maybe not good
- <http://lists.opensuse.org/opensuse/2008-05/msg01746.html>
- <http://lists.opensuse.org/opensuse/2008-05/msg01755.html> - how to post it so Security picks it up
- <http://lists.opensuse.org/opensuse/2008-05/msg01658.html> - use ftp

The change in root servers resulted in a `security bug fix<https://bugzilla.novell.com/show_bug.cgi?id=392173>`_, but that took a while.

`This script<http://www.tldp.org/HOWTO/DNS-HOWTO-8.html>`_ gets it through dig too, but not the best solution.

Neither ftp, nor http are really secure to get these files from <http://ftp.internic.net/domain/>:

- <ftp://ftp.internic.net/domain/db.cache>
- <ftp://ftp.internic.net/domain/named.cache>
- <ftp://ftp.internic.net/domain/named.root>
- <http://www.internic.net/domain/db.cache>
- <http://www.internic.net/domain/named.cache>
- <http://www.internic.net/domain/named.root>

An alternative might be to get the ``.sig`` there in in a secure way, then `use gpg to verify the signatures<http://www.linuxquestions.org/questions/linux-newbie-8/md5-and-sig-537564/>`_ (as `gpg seems more secure than md5 signatures<http://stackoverflow.com/questions/15194779/md5-vs-gpg-signature/15195785#15195785>`_).

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


Table of Contents
=================

.. contents::

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

- `Enhanced Base System<https://software.opensuse.org/package/patterns-openSUSE-enhanced_base>`_
- `Console Tools<https://www.google.com/search?q="Console+Tools"+site%3Aopensuse.org>`_
- `File Server<https://www.google.com/search?q="File+Server"+site%3Aopensuse.org>`_
- `Network Administration<https://www.google.com/search?q="Network+Administration"+site%3Aopensuse.org>`_
- `Mail and News Server<https://www.google.com/search?q="Mail+and+News+Server"+site%3Aopensuse.org>`_
- `Web and LAMP Server<https://www.google.com/search?q="Web+and+LAMP+Server"+site%3Aopensuse.org>`_
- `Internet Gateway<https://www.google.com/search?q="Internet+Gateway"+site%3Aopensuse.org>`_
- `DHCP and DNS Server<https://www.google.com/search?q="DHCP+and+DNS+Server"+site%3Aopensuse.org>`_

After that I added some **packages** too:

.. sidebar::

  Note that some of these won't install just yet, see the `text-mode installation and conflicts<text-mode-installation-and-conflicts>`_ section.

- `etckeeper<https://software.opensuse.org/package/etckeeper>`_
- `syslogd<https://software.opensuse.org/package/syslogd>`_
- `emacs<https://software.opensuse.org/package/emacs>`_
- `joe<https://software.opensuse.org/package/joe>`_
- `nano<https://software.opensuse.org/package/nano>`_
- `pico<https://software.opensuse.org/package/pico>`_
- `vim<https://software.opensuse.org/package/vim>`_
- `dovecot<https://software.opensuse.org/package/dovecot>`_
- `mutt<https://software.opensuse.org/package/mutt>`_
- `par<https://software.opensuse.org/package/par>`_
- `make<https://software.opensuse.org/package/make>`_
- `monit<https://software.opensuse.org/package/monit>`_
- `mc<https://software.opensuse.org/package/mc>`_
- `mirror<https://software.opensuse.org/package/mirror>`_
- `p7zip<https://software.opensuse.org/package/p7zip>`_
- `zip<https://software.opensuse.org/package/zip>`_
- `zsync<https://software.opensuse.org/package/zsync>`_
- `git<https://software.opensuse.org/package/git>`_
- `mercurial<https://software.opensuse.org/package/mercurial>`_*
- `perl<https://software.opensuse.org/package/perl>`_
- `php<https://software.opensuse.org/package/php>`_*
- `apache2-mod_php5<https://software.opensuse.org/package/apache2-mod_php5>`_*
- `python<https://software.opensuse.org/package/python>`_*
- `dropbox<https://software.opensuse.org/package/dropbox>`_*
- `ca-certificates-cacert<https://software.opensuse.org/package/ca-certificates-cacert>`_
- `bridge-utils<https://software.opensuse.org/package/bridge-utils>`_
- `fping<https://software.opensuse.org/package/fping>`_
- `ftp<https://software.opensuse.org/package/ftp>`_
- `gftp<https://software.opensuse.org/package/gftp>`_
- `icecast<https://software.opensuse.org/package/icecast>`_
- `links<https://software.opensuse.org/package/links>`_
- `iptraf-ng<https://software.opensuse.org/package/iptraf-ng>`_
- `shellinabox<https://software.opensuse.org/package/shellinabox>`_
- `kvirustotal<https://software.opensuse.org/package/kvirustotal>`_
- `monit<https://software.opensuse.org/package/monit>`_

These packages were already installed:

- `info<https://software.opensuse.org/package/info>`_
- `man<https://software.opensuse.org/package/man>`_
- `man-pages<https://software.opensuse.org/package/man-pages>`_
- `mc<https://software.opensuse.org/package/mc>`_
- `w3m<https://software.opensuse.org/package/w3m>`_

Didn't yet install:

- `bash-doc<https://software.opensuse.org/package/bash-doc>`_*
- `samba-doc<https://software.opensuse.org/package/samba-doc>`_*

.. sidebar::

  If you want to know `which package provides a certain file<http://unix.stackexchange.com/questions/158041/how-do-i-find-a-package-that-provides-a-given-file-in-opensuse>`_, then use this command::

      zypper search --provides --match-exact hg

  Where ``hg`` is the file you are looking for.

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

add git-extras
--------------

See the `git-extras Install documentation<>https://github.com/tj/git-extras/blob/master/Installation.md`_ for why/how.

Just run this command::

    (cd /tmp && git clone https://github.com/tj/git-extras.git && cd git-extras && git checkout $(git describe --tags $(git rev-list --tags --max-count=1)) && sudo make install)

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

removing hardlinks from the ``etckeeper`` repository
----------------------------------------------------

Inspired by `this answer<http://unix.stackexchange.com/questions/63627/excluding-files-in-etckeeper-with-gitignore-doesnt-work/63628#63628>`_ to get rid of these messages during `etckeeper commit<https://github.com/joeyh/etckeeper#what-etckeeper-does>`_ to delete many `hardlinked bootsplash files<http://lists.opensuse.org/opensuse-factory/2014-06/msg00115.html>`_::

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

.. sidebar::

  Note that `each ALL entry has a different meaning<http://superuser.com/questions/357467/what-do-the-alls-in-the-line-admin-all-all-all-in-ubuntus-etc-sudoers>`_.

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

install and configure `noip` dynamic DNS update script
------------------------------------------------------

The script is based on <https://github.com/mdmower/bash-no-ip-updater.git>.

Create the below ``/etc/noip.com.install.sh`` script with ``chmod 700``, then run it to install.

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

.. sidebar:

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

Run these commands to `test if the basic configuration was successful<https://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/install.html#id2553312>`_ with `testclient<https://www.samba.org/samba/docs/man/manpages/testparm.1.html>`_ and `https://www.samba.org/samba/docs/man/manpages/smbclient.1.html<>`_::

    testparm /etc/samba/smb.conf
    smbclient -L `hostname`

.. sidebar::

  During ``smbclient`` you will have to type your unix password.

Testing and fixing so clients can talk to our Samba server
----------------------------------------------------------

Now it is time to test the smb connectivity as well::

  smbclient //`hostname`/profiles -U jeroenp
  Enter jeroenp's password:
  Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.1-3406-SUSE-oS13.2-x86_64]
  tree connect failed: NT_STATUS_ACCESS_DENIED

.. sidebar::

  Do **not** try to solve the `NT_STATUS_ACCESS_DENIED issue<https://forum.manjaro.org/index.php?topic=19252.0>`_ by enabling ``client lanman auth`` as this makes your system less secure (`LANMAN authentication can be cracked quite easily<https://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html#idp59214864>`_).

The first think to check is the samba password database, as samba uses different authentication database than the standard linux one (hence the linux password above).
Check it with `pdbedit<https://www.samba.org/samba/docs/man/manpages/pdbedit.8.html>`_ like this::

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

.. sidebar::

  Do **not** use `smbpasswd<https://www.samba.org/samba/docs/man/manpages/smbpasswd.8.html>`_ to add the user as that only supports the ``smbpasswd`` database format, `whereas ``pdbedit`` supports any password backend<http://unix.stackexchange.com/questions/107032/deleting-a-samba-user-pbdedit-vs-smbpasswd-whats-the-difference/107033#107033>`_.

Now do final checks::

    smbclient --list `hostname` --user jeroenp
    smbclient //`hostname`/jeroenp -U jeroenp

One day: `syncing between the Samba password and system password storage<https://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/pam.html#id2667418>`_ is setup
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

See `Use SMB Information for Linux Authentication<https://www.google.com/search?q="Use+SMB+Information+for+Linux+Authentication">`_`

Fixing password synchronisation?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. sidebar::

  Background reading (web-archive link as the site itself is down): `Samba Server and Suse / openSUSE: HowTo Configure a Professional File Server on a SOHO LAN, covering Name Resolution, Authentication, Security and Shares.<http://web.archive.org/web/20130801222534/http://swerdna.dyndns.org/susesambaserver.html>`_.

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

.. sidebar::

    Check if your zone files are correct by executing ``named-checkzone``.

    Check if your named configuration is correct by executing ``named-checkconfig``.

    Check if ``named`` delivers the correct zone::

        dig @localhost axfr pluimers.com

    See:

    - `Check BIND – DNS Server configuration file for errors with named-checkconf tools<http://www.cyberciti.biz/tips/howto-linux-unix-check-dns-file-errors.html>`_
    - `Troubleshoot Linux / UNIX bind dns server zone problems with named-checkzone tool<http://www.cyberciti.biz/faq/howto-linux-unix-zone-file-validity-checking/>`_

Ensure that ``/var/lib/named/master`` gets synced to ``/etc/named/master``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Based on these links, I've added a sync script.

- `etckeeper configuration documentation<https://github.com/joeyh/etckeeper#configuration>`_
- `unix: using variables<http://www.tutorialspoint.com/unix/unix-using-variables.htm>`_

I stored it in ``/etc/etckeeper/pre-commit.d/10rsync-var-lib-named-master.sh``::

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

At first, `monit<https://software.opensuse.org/package/monit>`_ won't run::

    revue:~ # rcmonit restart
    redirecting to systemctl restart monit.service
    Failed to restart monit.service: Unit monit.service failed to load: No such file or directory.

Even though it is an offical package, it is missing the `.service file<http://www.freedesktop.org/software/systemd/man/systemd.service.html>`_.

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

But it still doesn't start, as ``


Fooo


    This is a `known bug<https://bugzilla.novell.com/show_bug.cgi?id=497574>`_ explained
    by both `Running two or more Monit instances on the same machine<https://mmonit.com/wiki/Monit/FAQ#instances>`_
    and `Install Monit on openSUSE 13.2<http://www.itzgeek.com/how-tos/linux/opensuse/install-monit-on-opensuse-13-2.html>`_ but in different wording:

    .. sidebar::

      If you get any error like below,::

        Status not available -- the monit daemon is not running

      Edit /etc/monitrc and uncomment the following pid entry.::

        set pidfile /var/run/monit.pid

    The file ``/usr/sbin/rcmonit`` indicates there the `pid<http://stackoverflow.com/questions/8296170/what-is-a-pid-file-and-what-does-it-contain/8296204#8296204>`_ file is::

        revue:~ # grep -w pid `which rcmonit`
        MONIT_PID_FILE="/run/monit/monit.pid"
        				echo "Warning: No pid file, ${MONIT_PID_FILE} found.  Do not know which process to stop.  Calling stop and start instead."
        revue:~ # grep -w pid /etc/monitrc
        revue:~ # ls -al /run/monit/monit.pid
        ls: cannot access /run/monit/monit.pid: No such file or directory

    #! /bin/sh
    #
    # fixes the pidfile in /etc/monitrc
    ETC_TARGET=/etc/monitrc
    # use double quotes to allow for variable expansion: http://stackoverflow.com/questions/17477890/expand-variables-in-sed/17477911#17477911
    # escape slashes in arguments: http://www.grymoire.com/Unix/Sed.html#uh-62
    echo old:
    sed -n "/^# set pidfile \/var\/run\/monit.pid$/ p" $ETC_TARGET
    sed -e "/^# set pidfile \/var\/run\/monit.pid$/ s/^# //" $ETC_TARGET > $ETC_TARGET.tmp && mv $ETC_TARGET.tmp $ETC_TARGET && chmod 600 $ETC_TARGET
    echo new:
    sed -n "/^set pidfile \/var\/run\/monit.pid$/ p" $ETC_TARGET

Run it and the result should be like this:

    revue:/etc # ./monitrc-fix.sh
    old:
    # set pidfile /var/run/monit.pid
    new:
    set pidfile /var/run/monit.pid

If you didn't run ``sed``, then you get this error::

    revue:/etc/systemd/system # systemctl status monit.service
    ● monit.service - Pro-active monitoring utility for unix systems
       Loaded: error (Reason: Invalid argument)
       Active: inactive (dead)

    Jun 04 21:21:26 revue systemd[1]: [/etc/systemd/system/monit.service:23] Executable path is not absolute, ignoring: @prefix@/bin/monit -I -c @sysco...r@/monitrc
    Jun 04 21:21:26 revue systemd[1]: [/etc/systemd/system/monit.service:24] Executable path is not absolute, ignoring: @prefix@/bin/monit -c @sysconfd...nitrc quit
    Jun 04 21:21:26 revue systemd[1]: [/etc/systemd/system/monit.service:25] Executable path is not absolute, ignoring: @prefix@/bin/monit -c @sysconfd...trc reload
    Jun 04 21:21:26 revue systemd[1]: monit.service lacks both ExecStart= and ExecStop= setting. Refusing.
    Hint: Some lines were ellipsized, use -l to show in full.


Finally run this:

    revue:/etc # systemctl enable monit.service
    monit.service is not a native service, redirecting to /sbin/chkconfig.
    Executing /sbin/chkconfig monit on

Configuring apache2 for the first time
--------------------------------------

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

.. sidebar:: Notes when updating (vhosts) configuration from Apache 2.2 to Apache 2.4:

  Instead of using `mod_access_compat<http://httpd.apache.org/docs/2.4/mod/mod_access_compat.html>`_ modify the configuration files to use the directives in `mod_authz_host<http://httpd.apache.org/docs/2.4/mod/mod_authz_host.html>`_.

  See `Upgrading to 2.4 from 2.2<http://httpd.apache.org/docs/2.4/upgrading.html>`_

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

  - Denying: `Apache 2.4 Upgrade and the “Invalid Command ‘Order'” Error<https://systembash.com/apache-2-4-upgrade-and-the-invalid-command-order-error/#sthash.u4jTuZ7o.dpuf<https://systembash.com/apache-2-4-upgrade-and-the-invalid-command-order-error/>`_
  - Granting: `Upgrading to Apache 2.4 from Apache HTTP Server 2.2.x<http://brianflove.com/2014/04/23/upgrading-to-apache-2-4-from-apache-http-server-2-2-x/>`_


  Common problems when upgrading
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  - Startup errors:

    - ``Invalid command 'User', perhaps misspelled or defined by a module not included in the server configuration`` - load module `mod_unixd<http://httpd.apache.org/docs/2.4/mod/mod_unixd.html>`_
    - ``Invalid command 'Require', perhaps misspelled or defined by a module not included in the server configuration``, or ``Invalid command 'Order', perhaps misspelled or defined by a module not included in the server configuration`` - load module `mod_access_compat<http://httpd.apache.org/docs/2.4/mod/mod_access_compat.html>`_, or update configuration to 2.4 authorization directives.
    - ``Ignoring deprecated use of DefaultType in line NN of /path/to/httpd.conf`` - remove `DefaultType<http://httpd.apache.org/docs/2.4/mod/core.html#defaulttype>`_ and replace with other configuration settings.
    - ``Invalid command 'AddOutputFilterByType', perhaps misspelled or defined by a module not included in the server configuration`` - `AddOutputFilterByType<http://httpd.apache.org/docs/2.4/mod/mod_filter.html#addoutputfilterbytype>`_ has moved from the core to mod_filter, which must be loaded.

  - Errors serving requests:

    - ``configuration error: couldn't check user: /path`` - load module `mod_authn_core<http://httpd.apache.org/docs/2.4/mod/mod_authn_core.html>`_.
    - ``.htaccess files aren't being processed`` - Check for an appropriate `AllowOverride<http://httpd.apache.org/docs/2.4/mod/core.html#allowoverride>`_ directive; the default changed to ``None`` in 2.4.

----------------------------------------------------------------------------

.. [#opensuse] I keep using the old `SuSE <https://en.wikipedia.org/wiki/SUSE>`_ writing, I'm an old fart.

.. [#tumbleweed] `Tumbleweed <https://en.opensuse.org/Portal:Tumbleweed>`_ is the rolling release of OpenSuSE.

.. [#revue] See `Snip en Snap revue<https://en.wikipedia.org/wiki/Snip_en_Snap>`_.

.. [#headless] `Headless<https://en.wikipedia.org/wiki/Headless_software>`_ as in no GUI, not as in `Embedded System<https://en.wikipedia.org/wiki/Embedded_system>`_. So there is a text `console<https://en.wikipedia.org/wiki/System_console>`_, and remote `ssh<https://en.wikipedia.org/wiki/Secure_Shell>`_.

.. [#patterns-openSUSE-minimal_base-conflicts] The `patterns-openSUSE-minimal_base-conflicts<https://www.google.com/search?q=patterns-openSUSE-minimal_base-conflicts>`_ is there to `prevent recommended packages to blow up a minimal installation<http://unix.stackexchange.com/questions/144438/missing-broken-dependancies-on-opensuse-normal/144583#144583>`_

.. [#removeconflicts] The `actual conflicts package<http://unix.stackexchange.com/questions/73427/cant-install-python-because-of-zypper-conflict>`_ contains the version number of the distribution you use.
