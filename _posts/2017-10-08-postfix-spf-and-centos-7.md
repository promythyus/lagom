---
layout: post
title: Postfix, pypolicyd-spf and CentOS 7
categories:
- blog
- unix
---

If you happened to be configuring postfix on a publicly accessible server, you might want to enable [SPF](http://www.openspf.org/Introduction). To do this, one would typically google the terms "postfix spf centos 7". Several of the results of this google search will direct you to install `pypolicyd-spf`.

Unfortunately what they don't tell you is that the version of `pypolicyd-spf` found on the [EPEL](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F) yum repository doesn't play nice with python2 on CentOS 7. Combined with the right SMTP recipient restrictions this will result in your mailserver rejecting all messages. When attempting to verify an incoming connection, you might see something like this in the postfix log:

	Oct 08 16:29:46 morbo postfix/smtpd[4923]: connect from mail-pg0-f43.google.com[74.125.83.43]
	Oct 08 16:29:46 morbo postfix/smtpd[4923]: Anonymous TLS connection established from mail-pg0-f43.google.com[74.125.83.43]: TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits)
	Oct 08 16:29:47 morbo policyd-spf[4932]: ERROR: 127.0.0.0/8 in skip_addresses not IP network.  Message: '74.125.83.43' does not appear to be an IPv4 or IPv6 address. Did you pass in a bytes (str in Python 2) instead of a unicode object?. Aborting whitelist processing.
	Oct 08 16:29:47 morbo policyd-spf[4932]: Traceback (most recent call last):
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/libexec/postfix/policyd-spf", line 700, in <module>
	Oct 08 16:29:47 morbo policyd-spf[4932]:     instance_dict, configData, peruser)
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/libexec/postfix/policyd-spf", line 412, in _spfcheck
	Oct 08 16:29:47 morbo policyd-spf[4932]:     res = spf.check2(ip, helo_fake_sender, helo, querytime=configData.get('Lookup_Time'))
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/lib/python2.7/site-packages/spf.py", line 297, in check2
	Oct 08 16:29:47 morbo policyd-spf[4932]:     receiver=receiver,timeout=timeout,verbose=verbose,querytime=querytime).check()
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/lib/python2.7/site-packages/spf.py", line 378, in __init__
	Oct 08 16:29:47 morbo policyd-spf[4932]:     self.set_ip(i)
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/lib/python2.7/site-packages/spf.py", line 405, in set_ip
	Oct 08 16:29:47 morbo policyd-spf[4932]:     self.ipaddr = ipaddress.ip_address(i)
	Oct 08 16:29:47 morbo policyd-spf[4932]:   File "/usr/lib/python2.7/site-packages/ipaddress.py", line 163, in ip_address
	Oct 08 16:29:47 morbo policyd-spf[4932]:     ' a unicode object?' % address)
	Oct 08 16:29:47 morbo policyd-spf[4932]: AddressValueError: '74.125.83.43' does not appear to be an IPv4 or IPv6 address. Did you pass in a bytes (str in Python 2) instead of a unicode object?
	Oct 08 16:29:47 morbo postfix/spawn[4931]: warning: command /usr/libexec/postfix/policyd-spf exit status 1
	Oct 08 16:29:47 morbo postfix/smtpd[4923]: warning: premature end-of-input on private/policyd-spf while reading input attribute name
	Oct 08 16:29:48 morbo policyd-spf[4933]: ERROR: 127.0.0.0/8 in skip_addresses not IP network.  Message: '74.125.83.43' does not appear to be an IPv4 or IPv6 address. Did you pass in a bytes (str in Python 2) instead of a unicode object?. Aborting whitelist processing.
	Oct 08 16:29:48 morbo policyd-spf[4933]: Traceback (most recent call last):
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/libexec/postfix/policyd-spf", line 700, in <module>
	Oct 08 16:29:48 morbo policyd-spf[4933]:     instance_dict, configData, peruser)
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/libexec/postfix/policyd-spf", line 412, in _spfcheck
	Oct 08 16:29:48 morbo policyd-spf[4933]:     res = spf.check2(ip, helo_fake_sender, helo, querytime=configData.get('Lookup_Time'))
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/lib/python2.7/site-packages/spf.py", line 297, in check2
	Oct 08 16:29:48 morbo policyd-spf[4933]:     receiver=receiver,timeout=timeout,verbose=verbose,querytime=querytime).check()
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/lib/python2.7/site-packages/spf.py", line 378, in __init__
	Oct 08 16:29:48 morbo policyd-spf[4933]:     self.set_ip(i)
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/lib/python2.7/site-packages/spf.py", line 405, in set_ip
	Oct 08 16:29:48 morbo policyd-spf[4933]:     self.ipaddr = ipaddress.ip_address(i)
	Oct 08 16:29:48 morbo policyd-spf[4933]:   File "/usr/lib/python2.7/site-packages/ipaddress.py", line 163, in ip_address
	Oct 08 16:29:48 morbo policyd-spf[4933]:     ' a unicode object?' % address)
	Oct 08 16:29:48 morbo policyd-spf[4933]: AddressValueError: '74.125.83.43' does not appear to be an IPv4 or IPv6 address. Did you pass in a bytes (str in Python 2) instead of a unicode object?
	Oct 08 16:29:48 morbo postfix/spawn[4931]: warning: command /usr/libexec/postfix/policyd-spf exit status 1
	Oct 08 16:29:48 morbo postfix/smtpd[4923]: warning: premature end-of-input on private/policyd-spf while reading input attribute name
	Oct 08 16:29:48 morbo postfix/smtpd[4923]: warning: problem talking to server private/policyd-spf: Success
	Oct 08 16:29:48 morbo postfix/smtpd[4923]: NOQUEUE: reject: RCPT from mail-pg0-f43.google.com[74.125.83.43]: 451 4.3.5 Server configuration problem; from=<brooksaar@gmail.com> to=<aaron@aaronbrooks.com.au> proto=ESMTP helo=<mail-pg0-f43.google.com>
	Oct 08 16:29:48 morbo postfix/smtpd[4923]: disconnect from mail-pg0-f43.google.com[74.125.83.43]


In particular, the message `ERROR: 127.0.0.0/8 in skip_addresses not IP network.  Message: '74.125.83.43' does not appear to be an IPv4 or IPv6 address. Did you pass in a bytes (str in Python 2) instead of a unicode object?. Aborting whitelist processing.` seems to indicate the source of the issue. Some googling reveals that this seems to be caused by [improper usage of python's ipaddress](https://bugzilla.redhat.com/show_bug.cgi?id=1232595). 

Fortunately for us, the Fedora packagers fixed this in Fedora 22. Unfortunately for us, this fix didn't make its way over to the EPEL repository. We could use the fix provided on the bug for Fedora, or we could do away with this old version of `pypolicyd-spf` entirely and use the latest version directly from the source. The latest version of `pypolicyd-spf` (2.0.1 right now) no longer supports `python2`, so we will require `python34`.

Install Python 3 and pip using yum:
```
sudo yum install python34 python34-pip
```

Then use pip to install `pypolicyd-spf` and its dependencies:
```
sudo pip3 install pypolicyd-spf pyspf py3dns
```

Then modify `/etc/postfix/master.cf`, replacing the definition for `policyd-spf` with 
```
policyd-spf unix -	n	n	-	0	spawn		user=nobody argv=/usr/bin/policyd-spf
```

Don't forget to add your `smtpd_recipient_restrictions` in `/etc/postfix/main.cf` if you haven't already. It should have the following restrictions:
```
reject_unauth_destination, check_policy_service unix:private/policyd-spf
```

Once that is done, restart postfix and it should start allowing messages that pass SPF.
