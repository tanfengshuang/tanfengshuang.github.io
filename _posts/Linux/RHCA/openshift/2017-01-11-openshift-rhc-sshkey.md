
---
layout: post
title:  "Remote Connection (SSH) via rhc sshkey"
categories: Linux
tags: openshift2
---

[Setting up SSH Keys](https://developers.openshift.com/managing-your-applications/remote-connection.html)

```
$ rhc sshkey add ftan_key /home/ftan/.ssh/id_rsa.pub_1
/usr/local/share/gems/gems/commander-4.2.1/lib/commander/user_interaction.rb:328: warning: constant ::TimeoutError is deprecated
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:746:in `connect': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
RESULT:
SSH key /home/ftan/.ssh/id_rsa.pub_1 has been added as 'ftan_key'

$ rhc sshkey show
/usr/local/share/gems/gems/commander-4.2.1/lib/commander/user_interaction.rb:328: warning: constant ::TimeoutError is deprecated
Missing required argument 'name'.
Usage: rhc sshkey-show <name>

$ rhc sshkey show ftan_key
/usr/local/share/gems/gems/commander-4.2.1/lib/commander/user_interaction.rb:328: warning: constant ::TimeoutError is deprecated
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:746:in `connect': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:616:in `query': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:872:in `parse_header': Object#timeout is deprecated, use Timeout.timeout instead.
/usr/local/share/gems/gems/httpclient-2.6.0.1/lib/httpclient/session.rb:950:in `read_body_length': Object#timeout is deprecated, use Timeout.timeout instead.
ftan_key (type: ssh-rsa)
------------------------
  Fingerprint: 49:5a:cb:24:fc:8c:88:0c:5a:f0:38:1c:a6:06:ef:cc
```
