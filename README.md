# Mage.sovereign-common

Splitted and slightly modified common task from the [Sovereign project](https://github.com/sovereign/sovereign), a set of [Ansible](http://ansible.com) playbooks that you can use to build and maintain your own 
[personal cloud](http://www.urbandictionary.com/define.php?term=clown%20computing) based entirely on open source software, so you’re in control.

Services Provided by the Mage split-fork
----------------------------------------

What do you get if you point Sovereign at a server? All kinds of good stuff!

-   [IMAP](https://en.wikipedia.org/wiki/Internet_Message_Access_Protocol) over SSL via [Dovecot](http://dovecot.org/), complete with full text search provided by [Solr](https://lucene.apache.org/solr/).
-   [POP3](https://en.wikipedia.org/wiki/Post_Office_Protocol) over SSL, also via Dovecot
-   [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) over SSL via Postfix, including a nice set of [DNSBLs](https://en.wikipedia.org/wiki/DNSBL) to discard spam before it ever hits your filters.
-   Webmail via [Roundcube](http://www.roundcube.net/).
-   Mobile push notifications via [Z-Push](http://z-push.sourceforge.net/soswp/index.php?pages_id=1&t=home).
-   Email client [automatic configuration](https://developer.mozilla.org/en-US/docs/Mozilla/Thunderbird/Autoconfiguration).
-   Virtual domains for your email, backed by [PostgreSQL](http://www.postgresql.org/).
-   Spam fighting via [Rspamd](https://www.rspamd.com/) and [Postgrey](http://postgrey.schweikert.ch/).
-   Mail server verification via [OpenDKIM](http://www.opendkim.org/) and [OpenDMARC](http://www.trusteddomain.org/opendmarc/) so the Internet knows your mailserver is legit.
-   [Monit](http://mmonit.com/monit/) to keep everything running smoothly (and alert you when it’s not).
-   [collectd](http://collectd.org/) to collect system statistics.
-   Web hosting (ex: for your blog) via [Apache](https://www.apache.org/).
-   Firewall management via [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall).
-   Intrusion prevention via [fail2ban](http://www.fail2ban.org/) and rootkit detection via [rkhunter](http://rkhunter.sourceforge.net).
-   SSH configuration preventing root login and insecure password authentication
-   [RFC6238](http://tools.ietf.org/html/rfc6238) two-factor authentication compatible with [Google Authenticator](http://en.wikipedia.org/wiki/Google_Authenticator) and various hardware tokens
-   SSL cetificates obtained from [Let's Encrypt](https://letsencrypt.org/).

What the Mage fork doesn't provide but the original SOvereign does:

-   Jabber/[XMPP](http://xmpp.org/) instant messaging via [Prosody](http://prosody.im/).
-   An RSS Reader via [Selfoss](http://selfoss.aditu.de/).
-   Secure on-disk storage for email and more via [EncFS](http://www.arg0.net/encfs).
-   [CalDAV](https://en.wikipedia.org/wiki/CalDAV) and [CardDAV](https://en.wikipedia.org/wiki/CardDAV) to keep your calendars and contacts in sync, via [ownCloud](http://owncloud.org/).
-   Your own private storage cloud via [ownCloud](http://owncloud.org/).
-   Your own VPN server via [OpenVPN](http://openvpn.net/index.php/open-source.html).
-   An IRC bouncer via [ZNC](http://wiki.znc.in/ZNC).
-   Nightly backups to [Tarsnap](https://www.tarsnap.com/).
-   Git hosting via [cgit](http://git.zx2c4.com/cgit/about/) and [gitolite](https://github.com/sitaramc/gitolite).
-   Read-it-later via [Wallabag](https://www.wallabag.org/)
-   A bunch of nice-to-have tools like [mosh](http://mosh.mit.edu) and [htop](http://htop.sourceforge.net) that make life with a server a little easier.

BIG FAT WARNING
===============

Either disable two factor authentication or make sure to setup google authenticator correctly IMMEDIATELLY AFTER SETUP. I locked myself twice out of a VM during testing this role.


Usage
=====

What You’ll Need
----------------

1.  A VPS (or bare-metal server if you wanna ball hard). My VPS is hosted at [Linode](http://www.linode.com/?r=45405878277aa04ee1f1d21394285da6b43f963b). You’ll probably want at least 512 MB of RAM between Apache, Solr, and PostgreSQL. Mine has 1024.
2.  [64-bit Debian 8.3](http://www.debian.org/) or an equivalent Linux distribution.


Remote server preparations
--------------------------


### 1. Install required packages

```sh
apt-get install sudo
```

### 2. If this is a new machine, set the root password and create a user account for Ansible

```sh
passwd

useradd deploy
passwd deploy
mkdir /home/deploy
```

### 3. Optionally authorize your ssh key if you want passwordless ssh login:

```sh
    mkdir /home/deploy/.ssh
    chmod 700 /home/deploy/.ssh
    nano /home/deploy/.ssh/authorized_keys
    chmod 400 /home/deploy/.ssh/authorized_keys
    chown deploy:deploy /home/deploy -R
    echo 'deploy ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/deploy
```


Installing mage-sovereign (from local machine)
----------------------------------------------

### 1. Get the roles:

```sh
git clone https://github.com/Vaizard/mage.sovereign-common.git
git clone https://github.com/Vaizard/mage.sovereign-mailstack.git
git clone https://github.com/Vaizard/mage.sovereign-webmail.git
git clone https://github.com/Vaizard/mage.sovereign-monitoring.git
```

and set up your password storage

```sh
sudo mkdir /etc/ansible/secret
sudo chown user_you_use_to_run_ansible:user_you_use_to_run_ansible /etc/ansible/secret
```

### 2. Configure your installation

Modify the settings in the `group_vars/sovereign` folder to your liking. If you want to see how they’re used in context, just search for the corresponding string.
All of the variables in `group_vars/sovereign` must be set for sovereign to function. To set up the `password_hash` for your mail users in group_vars,

    python3 -c 'import crypt; print(crypt.crypt("YOUR_EMAIL_PASSWORD", salt=crypt.METHOD_SHA512))'

### 3. Set up DNS

Suggested `A` or `CNAME` records from the Sovereign project are:

* `example.com`
* `mail.example.com`
* `www.example.com` (for Web hosting)
* `autoconfig.example.com` (for email client automatic configuration)
* `read.example.com` (for Wallabag)
* `news.example.com` (for Selfoss)
* `cloud.example.com` (for ownCloud)
* `git.example.com` (for cgit)

What we actually use in production is something like:

```
example.com 	TXT 	v=spf1 mx a ptr include:example.net include:example.org 	0 	600
example.com 	A 	1.2.3.4 	0 	600
example.com 	MX 	mail.example.com 	10 	900
mail.example.com 	A 	1.2.3.4
*.example.com 	CNAME 	example.com 	100 	600
default._domainkey.example.com 	TXT 	v=DKIM1; k=rsa; s=email; p=4GNADCBiQKMIGfMA0GCSqGSIb3DQEBAQUAA/areallylongrandomstring/BgQDETFzXLgzfHVOw3YDAQAB 	0 	600
_dmarc.example.com 	TXT 	v=DMARC1; p=none
```

The DKIM key can be found in `/etc/opendkim/keys/EXAMPLE.COM/default.txt`

### 4. Testing the setup

* Send an email to <a href="mailto:check-auth@verifier.port25.com">check-auth@verifier.port25.com</a> and reviewing the report that will be emailed back to you.
* http://dkimvalidator.com/
* https://pingability.com/zoneinfo.jsp
* http://www.dnsstuff.com/tools#dnsReport|type=domain

### 5. Example playbook 

TODO

