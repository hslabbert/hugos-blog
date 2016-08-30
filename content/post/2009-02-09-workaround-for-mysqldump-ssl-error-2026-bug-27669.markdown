---
date: 2009-02-09 16:36:55+00:00
slug: workaround-for-mysqldump-ssl-error-2026-bug-27669
title: 'Workaround for mysqldump SSL error 2026 bug #27669'
categories:
- linux
- mysql
- ssl
- web
---

I had not previously played aroung much with MySQL replication, but got the chance to do so recently. I'm doing some testing with a new mail filter setup composed of [amavisd-new](http://www.ijs.si/software/amavisd/), [SpamAssassin](http://spamassassin.apache.org/), and some other SA modules running through [Postfix](http://www.postfix.org/) on [Debian](http://www.debian.org/) Etch. The setup uses [Maia Mailguard](http://www.maiamailguard.com/) as a web front-end and management system, including per-user settings and quarantines. We're using MySQL for the database backend for Maia for storing quarantines, per-user settings and the like, but wanted to have a dual-MX setup with a secondary MX sitting in a remote site.

<!-- more -->So, we needed some way to keep the MySQL data between the two servers in sync. I had looked at [DRBD](http://www.drbd.org/) and [linux-ha](http://www.linux-ha.org/), but in the end decided to go with a dual-master [MySQL replication](http://dev.mysql.com/doc/refman/5.0/en/replication.html) setup ([howto1](http://www.neocodesoftware.com/replication/), [howto2](http://lug.wsu.edu/node/545/print)), using the built-in SSL capabilities in MySQL to encrypt the replication traffic over the WAN. After a bit of tweaking, I got that going, but then noticed that I could no longer perform mysqldump commands on the databases when running the command locally. I got the following error message whenever I tried a mysqldump command:



`Got error: 2026: SSL connection error when trying to connect`

I wasn't specifying any SSL parameters in the mysqldump command, though, and while I did configure the replication users to require SSL in their user setup, the mysqldump user did not have an SSL requirement. I got to the official MySQL bug report for this error(#[27669](http://bugs.mysql.com/bug.php?id=27669)) as well as the patch link. But to be honest, that didn't really get me anywhere. The bug report indicates that this behaviour has been corrected in MySQL release 5.0.42, and I was running release 5.0.32.

I updated the mysql-client package on a test machine to 5.0.75 and tried the mysqldump again; the same error still occurred. I didn't want to upgrade the actual mysql-server package as well, so I looked elsewhere for a fix. It occurred to me that, in order for the MySQL replication to use SSL, I had specified the SSL-related options (ssl-ca and so forth) in the [client] section of the global configuration file at /etc/mysql/my.cnf. As mysqldump starts a MySQL client connection, it would stand to reason that it uses the [client] configuration settings that it would find in the global config file, and hence would try to use SSL.

I wanted to get mysqldump to _not_ use SSL connections, but still wanted my replication traffic to use SSL. Since the mysqld daemon runs under the mysql user account, I moved the ssl-related config options out of the [client] section of the global config file of /etc/mysql/my.cnf, created a new .my.cnf config file in the mysql user's $HOME directory (/var/lib/mysql on my system), and entered just the ssl-related options in a [client] section of that .my.cnf file. I reloaded the mysqld daemon and confirmed that the slave connection to the remote server came up properly. It did; the configuration options for the mysql user includes both the global config file of /etc/mysql/my.cnf and the per-user .my.cnf file I had just created, so it is able to pick up the ssl-related options in the [client] section of the per-user settings.

NOTE: The ssl-related options I am talking about here are only those configured in the [client] section. The ssl-related options in the [mysqld] section of the global config file can stay where they are. It should work to have those present in the local mysql user's .my.cnf config file, but I did not test this.

Since no ssl-related options were specified in the global [client] configuration anymore, and no per-user settings were present for the user I was using for the mysqldump command, the command connected without SSL, the SSL error 2026 messages went away, and I was again able to successfully perform MySQL backups from the local machine.

Bitcoin tip address for this post: 15ZhSjpB8iDRHvzgXphpJ9AhkJUPJw5uE5
