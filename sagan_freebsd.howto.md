Building & Installing SAGAN For FreeBSD
===============================

Configure Sagan to log to unified2 snort logging format. This is best way to decouple the processor and allow for the fastest logging. Use barnyard2 for output plugins.

## Install from Ports Tree:

Build These: 

pcre-8.30_2         Perl Compatible Regular Expressions library
perl-5.12.4_4       Practical Extraction and Report Language
libdnet-1.11_3      A simple interface to low level networking routines
libee-0.3.2         An event expression library inspired by CEE
libestr-0.1.2       A library for some string essentials
autoconf-2.68       Automatically configure source code on many Un*x platforms 
automake-1.11.1     GNU Standards-compliant Makefile generator (1.11)
barnyard2-1.9_2     An output system for Snort or Suricata that parses unified2
pulledpork-0.6.1_2  Script to update snort-2.8+ rules
syslog-ng-3.3.5     A powerful syslogd replacement

Example: [user@sensor /usr/ports/devel/libee]# sudo make clean install 

Barnyard2 Output Plugin:

Example: mysql-client-5.5.23 Multithreaded SQL database (client) (can be added for barnyard2 sql logging)

## Switch FreeBSD syslog to syslog-ng using FIFO

Modify your '/etc/rc.conf' 

```shell 

syslog_ng_enable="YES"
syslogd_enable="NO"
syslog_ng_config="-u root"
syslog_ng_pid="/var/run/syslog-ng.pid"
```

Add New syslog-ng outputs to `/usr/local/etc/syslog-ng.conf`

```shell
destination sagan {

	pipe(
	    "/var/run/sagan.fifo"
            template("$SOURCEIP|$FACILITY|$PRIORITY|$LEVEL|$TAG|$YEAR-$MONTH-$DAY|$HOUR:$MIN:$SEC|$PROGRAM| $MSG\n") 
            template-escape(no)
	); 

};

log {
	source(s_local);
	# uncomment this line to open port 514 to receive messages
	#source(s_network);

	destination(d_local);
	destination(sagan);
};
```

Note: FreeBSD imports in the /etc/syslog.conf as a module to syslog-ng

Stop old Syslog & Start syslog-ng

```shell
[user@sensor ~/sagan-0.2.1]# sudo mkfifo /var/run/sagan.fifo

[user@sensor ~/]# sudo /etc/rc.d/syslog stop
[user@sensor ~/]# sudo /usr/local/etc/rc.d/syslog-ng start
```
Installing the rest From Source: 
(At this time Sagan and liblognorm are not in the FreeBSD ports tree.)

## Liblognorm

View GIT REPO Status for liblognorm
http://git.adiscon.com/?p=liblognorm.git;a=summary

fetch a tag/snapshot 
liblognorm.0.3.4.tar.gz
http://git.adiscon.com/?p=liblognorm.git;a=snapshot;h=f4b985047cd23be087aa93632acdd7ef7ea8ec70;sf=tgz

```shell
[user@sensor ~/]# wget -O liblognorm.0.3.4.tar.gz "http://git.adiscon.com/?p=liblognorm.git;a=snapshot;h=f4b985047cd23be087aa93632acdd7ef7ea8ec70;sf=tgz"

- or -

[user@sensor ~/]# fetch http://www.liblognorm.com/files/download/liblognorm-0.3.4.tar.gz
```
Decompress and compile liblognorm

```shell
[user@sensor ~/]# tar -zxvf liblognorm-*

[user@sensor ~/]# cd liblognorm*

[user@sensor ~/]# LDFLAGS=-L/usr/local/lib CFLAGS=-I/usr/local/include ./configure
[user@sensor ~/]# make 
[user@sensor ~/]# sudo make install
```

You should see
```shell
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/local/lib
```

## Sagan

Download and Decompress Sagan

```shell
[user@sensor ~/]# fetch http://sagan.softwink.com/download/sagan-0.2.1.tar.gz

[user@sensor ~/]# tar zxvf sagan-*

[user@sensor ~/]# cd sagan *
```

Configure Sagan to log to unified2 snort logging format. This is best way to decouple the processor and allow for the fastest logging. Use barnyard2 for output plugins.

```shell
[user@sensor ~/sagan-0.2.1] LDFLAGS=-L/usr/local/lib CFLAGS=-I/usr/local/include ./configure --disable-mysql --disable-postgresql --disable-esmtp --disable-prelude --enable-lognorm --enable-libdnet --disable-snortsam
[user@sensor ~/sagan-0.2.1]# make 
[user@sensor ~/sagan-0.2.1]# sudo make install
```

At the end of the install you should see

```shell
------------------------------------------------------------------------------

/usr/bin/install -c -d "/usr/local/share/man/man8"
/usr/bin/install -c -m 644 etc/sagan.8 "/usr/local/share/man/man8"
/usr/bin/install -c -m 755 src/sagan "/usr/local/sbin/sagan"
/usr/bin/install -c -d "/var/log/sagan"
/usr/bin/install -c -d "/var/run/sagan"

------------------------------------------------------------------------------
Sagan has been installed! You still need to do a few more things before your
up and running. See https://wiki.quadrantsec.com/bin/view/Main/SaganHOWTO for
more information.
------------------------------------------------------------------------------
```

Ensure the binary is properly linked and will run without segfault
 - LDD shows that libee, libestr, liblognorm, libpcap, libdnet, threading, pcre are all enabled and compiled in.

```shell
[user@sensor ~/sagan-0.2.1]# sudo ldd /usr/local/sbin/sagan 
/usr/local/sbin/sagan:
	libdnet.so => /usr/local/lib/libdnet.so (0x80085e000)
	libpcap.so.8 => /lib/libpcap.so.8 (0x800a6c000)
	liblognorm.so.0 => /usr/local/lib/liblognorm.so.0 (0x800c9f000)
	libee.so.0 => /usr/local/lib/libee.so.0 (0x800ea7000)
	libestr.so.0 => /usr/local/lib/libestr.so.0 (0x8010ae000)
	libm.so.5 => /lib/libm.so.5 (0x8012b0000)
	libthr.so.3 => /lib/libthr.so.3 (0x8014d1000)
	libpcre.so.1 => /usr/local/lib/libpcre.so.1 (0x8016f4000)
	libc.so.7 => /lib/libc.so.7 (0x80194a000)

```

Create a FreeBSD Sagan Service Script

```shell
[user@sensor ~/sagan-0.2.1]# fetch https://raw.github.com/gist/2595417/71a2cad676a242c7dbaaf9b65b379097d0c6c266/sagan -o /usr/local/etc/rc.d/sagan

[user@sensor ~/sagan-0.2.1]# sudo chmod a+x /usr/local/etc/rc.d/sagan 
```

Modify your '/etc/rc.conf' and this new sagan rc.d startup script.

```shell
sagan_enable="YES"
sagan_user="root"
```

## Pulledpork

Download rules via Pulledpork (rule set manager) 
Note: pulledpork does not at this time support the classification.config, reference.config, or any *.rulebase files  

```shell
[user@sensor ~/sagan-0.2.1]# fetch https://raw.github.com/gist/2596939/7d2c9d1c53f8ab69622609c71244b45a45de178e/pulledpork.sagan.conf -o /usr/local/etc/pulledpork/pulledpork.sagan.conf

[user@sensor ~/sagan-0.2.1]# fetch https://raw.github.com/beave/sagan-rules/master/classification.config -o /usr/local/etc/sagan-rules/classification.config

[user@sensor ~/sagan-0.2.1]# fetch https://raw.github.com/beave/sagan-rules/master/reference.config -o /usr/local/etc/sagan-rules/reference.config

[user@sensor ~/sagan-0.2.1]# pulledpork.pl -d -T -vv -c /usr/local/etc/pulledpork/pulledpork.sagan.conf
```

You should see pulled pork run.

```shell
----------------------------------
Writing /var/log/sid_changes.log....
	Done
Rule Stats....
	New:-------0
	Deleted:---0
	Enabled Rules:----1538
	Dropped Rules:----6
	Disabled Rules:---1
	Total Rules:------1545
	Done
Please review /var/log/sid_changes.log for additional details
Fly Piggy Fly!
```

Modify the Sagan Config '/usr/local/etc/sagan.conf' to # all rules file names and only use

```shell
include $RULE_PATH/sagan.rules
```
## FetchCarl

Download and install 'fetchcarl' 

```shell
[user@sensor ~/sagan-0.2.1]# fetch https://raw.github.com/gist/2638215/23ee40556620c1a2529f71649fee820d541511cc/fetchcarl.sh -o /usr/local/bin/fetchcarl

[user@sensor ~/sagan-0.2.1]# chmod +x /usr/local/bin/fetchcarl

[user@sensor ~]# fetchcarl --help
usage: fetchcarl options

This command will assist in downloading and updating sagan-rules rulebase, and map files. 

OPTIONS:
   -f, --file		Sagan configuration file location	
		  	  default: /usr/local/etc/sagan.conf  	
   -u, --url		Sagan-rule git repo url 
		  	  default: https://github.com/beave/sagan-rules.git  	

GENERIC:
   -v, --verbose  	Verbose
   -h, --help		Show this message

[user@sensor ~]# fetchcarl --verbose
the folder (/tmp/sagan_rules) you specified does not exist or doesn't contain a git repo.. fetching
/tmp/sagan_rules
Cloning into '/tmp/sagan_rules'...
remote: Counting objects: 549, done.
remote: Compressing objects: 100% (255/255), done.
remote: Total 549 (delta 462), reused 368 (delta 292)
Receiving objects: 100% (549/549), 275.21 KiB, done.
Resolving deltas: 100% (462/462), done.
Finished pulling sagan rules.
Sagan rulebase and config update complete. 
 (Note: Sagan *.rules were not updated. Use pulledpork for this process.)
```

## Running Sagan

Run Sagan for the first time.

```shell
[user@sensor ~]# /usr/local/etc/rc.d/sagan start
```
... wait -- do stuff like fail ssh logins, and sudo cmds ...

```shell
[user@sensor ~]# ls -la /var/log/sagan/sagan*

-rw-r--r--  1 root   sagan   4785 May 10 18:20 sagan.u2.1336685484
```

## Barnyard2

Create barnyard2.conf files 

```shell
[user@sensor ~]# cat /usr/local/etc/barnyard2.conf.cli

# this is not hard, only unified2 is supported ;)
input unified2

# Step 3: setup the output plugins

output alert_fast: stdout
```

Run Barnyard2 to collect the unified2 data and output to double check alert chain is working.

```shell
[user@sensor ~]# sudo mkdir /var/log/barnyard2  # Barnyard complains when this directory doesnt exist, although it is not used.

[user@sensor ~]# barnyard2 -c /usr/local/etc/barnyard2.cli.conf -C /usr/local/etc/sagan-rules/classification.config -S /usr/local/etc/sagan-rules/sagan-sid-msg.map -R /usr/local/etc/sagan-rules/reference.config -f sagan.u2 -d /var/log/sagan/ --nolock-pidfile

[user@sensor ~]# cat alert 

[**] [5000075] [OPENSSH] Authentication success [shadowbq] [**]
[Classification: successful-user] [Priority: 1]
2012-05-10 17:25:39 1.2.5.6:59625 -> 1.2.3.32:22 auth info
Message:  Accepted publickey for shadowbq from 1.2.5.6 port 59625 ssh2
[Xref => http://wiki.quadrantsec.com/bin/view/Main/5000075]

[**] [5000406] [OPENSSH] Accepted publickey [**]
[Classification: successful-user] [Priority: 1]
2012-05-10 17:25:39 1.2.5.3:59625 -> 1.2.5.3:22 auth info
Message:  Accepted publickey for shadowbq from 1.2.5.6 port 59625 ssh2
[Xref => http://wiki.quadrantsec.com/bin/view/Main/5000406]
```

YEA! Working.. Moving ON!

Set up barnyard2 to run in daemon mode, and log to snorby mysql remote database (this can be skipped if not running snorby)

Modify your '/etc/rc.conf' and barnyard rc.d startup script.

```shell
barnyard2_enable="YES"
barnyard2_flags="-D -f sagan.u2 -d /var/log/sagan"
```

## Barnyard2 and Existing Snorby

```shell
[user@sensor ~]# sudo cp /usr/local/etc/barnyard2.sagan.conf /usr/local/etc/barnyard2.conf
[user@sensor ~]# sudo cat /usr/local/etc/barnyard2.conf 

config reference_file:	    /usr/local/etc/sagan-rules/reference.config
config classification_file: /usr/local/etc/sagan-rules/classification.config
config sid_file:	    /usr/local/etc/sagan-rules/sagan-sid-msg.map
config hostname:	    sagan
config interface:	    misc
config waldo_file:          /var/log/sagan/barnyard2.waldo
input unified2
output database: log, mysql, user=snorby password=s3cr3tsauce dbname=snorby host=snorby
```
