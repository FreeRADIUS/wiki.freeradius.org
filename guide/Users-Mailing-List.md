# Posting questions to the User's mailing list

Many people ask questions on the FreeRADIUS users mailing list.
Good questions get good answers. Bad questions or those lacking
information just waste the time of the people who are trying to
help.

The most important things are to **clearly state your problem**
(not the problem with your solution) and to **include full debug
output** from the server.

It's really hard to try and help when vague questions are asked
and no debug output is included, and you are likely to be told
this.

Your post should include the following information:

* What you are trying to do
* why you are trying to do it
* what you expect the server to do
* what the server does instead (i.e. debug output).

When you give this information, it means that we can help you quickly and efficiently.  When you don't give this information, the usual response to your post will be a request for this information.  You can get answers more quickly by starting off with good questions.

## Full server debug output

This is really important. The server produces comprehensive
information on what it is doing, for a reason.  In 99% of the situations, the debug output contains the information needed to solve the problem.

We understand that the debug output is complex and full of what seems like [magical text](radiusd-X).  You don't need to understand all of it.  But the people on the mailing list *do* understand it, and can use it to help you.

When you post the debug output, the **whole** debug
output needs to be included.

**Run `radiusd -X` only** (`freeradius -X` on Debian based
systems). Don't run `-xx` or `-Xxx` or any other combinations,
unless you are specifically asked to. Timestamps and other debug
output just clutter up the e-mail and make it hard for people to
help.

**Capture right from the starting banner**, not just the few lines
you think are useful.

**Include a packet** being processed. If you stop capturing before
any packets have been received, nobody can tell what happens to
the packets.

Don't post the debug output where the last line is **Ready to process requests.**  This means that the server hasn't received any packets.  If you post that, the only thing that will happen is that people will ask you to post the debug output, **again**, but this time where it receives packets.

**Include the right packet**, in other words, include the one with
the problem. If you've got some authentication that works, and
another that doesn't, then include an authentication attempt for
both.

**Don't include unimportant packets**  If you have a problem with Access-Request packets, don't post a debug output which contains Accounting-Request packets, or packets for users who authenticate successfully.

**Don't send configuration files to the list**.  All of the relevant information is in the debug output. 

## Getting debug output

The most basic method of getting the required debug output is to
start the server as follows:

    radiusd -X 2>&1 | tee debugfile

wait until the server says

    Ready to process requests

then perform an authentication. Once this has taken place, press
`Ctrl-C` to stop the server, and include the contents of the
`debugfile` file in your e-mail, in its entirety. The list is used
to long debug outputs, don't worry about it being too big.

**Please** post the debug output in-line in the email.  **Do not** add it as an attachment.  The more steps people have to take to read your message, the less likely it is that they will help you.

**Please** do not post PNG screen captures of a terminal window showing the debug output.

A good debug output will look something like the following, which is explained [here](radiusd-X)

```text
$ radiusd -X 2>&1 | tee debugfile
FreeRADIUS Version 3.0.17
Copyright (C) 1999-2017 The FreeRADIUS server project and contributors
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE
You may redistribute copies of FreeRADIUS under the terms of the
GNU General Public License
For more information about these matters, see the file named COPYRIGHT
Getting debug state failed: ptrace capability not set.  If debugger detection is required run as root or: setcap cap_sys_ptrace+ep <path_to_radiusd>
Starting - reading configuration files ...
including dictionary file /usr/share/freeradius/dictionary
including dictionary file /usr/share/freeradius/dictionary.dhcp
including dictionary file /usr/share/freeradius/dictionary.vqp
including dictionary file /etc/raddb/dictionary
including configuration file /etc/raddb/radiusd.conf
including configuration file /etc/raddb/proxy.conf
including configuration file /etc/raddb/clients.conf
including files in directory /etc/raddb/mods-enabled/
including configuration file /etc/raddb/mods-enabled/pap
including configuration file /etc/raddb/mods-enabled/expiration
including configuration file /etc/raddb/mods-enabled/files
including configuration file /etc/raddb/mods-enabled/linelog
including configuration file /etc/raddb/mods-enabled/soh
including configuration file /etc/raddb/mods-enabled/attr_filter
including configuration file /etc/raddb/mods-enabled/ntlm_auth
including configuration file /etc/raddb/mods-enabled/exec
including configuration file /etc/raddb/mods-enabled/preprocess
including configuration file /etc/raddb/mods-enabled/sradutmp
including configuration file /etc/raddb/mods-enabled/chap
including configuration file /etc/raddb/mods-enabled/digest
including configuration file /etc/raddb/mods-enabled/expr
including configuration file /etc/raddb/mods-enabled/echo
including configuration file /etc/raddb/mods-enabled/unpack
including configuration file /etc/raddb/mods-enabled/detail
including configuration file /etc/raddb/mods-enabled/always
including configuration file /etc/raddb/mods-enabled/eap
including configuration file /etc/raddb/mods-enabled/mschap
including configuration file /etc/raddb/mods-enabled/unix
including configuration file /etc/raddb/mods-enabled/detail.log
including configuration file /etc/raddb/mods-enabled/passwd
including configuration file /etc/raddb/mods-enabled/date
including configuration file /etc/raddb/mods-enabled/logintime
including configuration file /etc/raddb/mods-enabled/utf8
including configuration file /etc/raddb/mods-enabled/dynamic_clients
including configuration file /etc/raddb/mods-enabled/radutmp
including configuration file /etc/raddb/mods-enabled/realm
including configuration file /etc/raddb/mods-enabled/cache_eap
including configuration file /etc/raddb/mods-enabled/replicate
including files in directory /etc/raddb/policy.d/
including configuration file /etc/raddb/policy.d/control
including configuration file /etc/raddb/policy.d/cui
including configuration file /etc/raddb/policy.d/debug
including configuration file /etc/raddb/policy.d/moonshot-targeted-ids
including configuration file /etc/raddb/policy.d/eap
including configuration file /etc/raddb/policy.d/filter
including configuration file /etc/raddb/policy.d/canonicalization
including configuration file /etc/raddb/policy.d/abfab-tr
including configuration file /etc/raddb/policy.d/operator-name
including configuration file /etc/raddb/policy.d/dhcp
including configuration file /etc/raddb/policy.d/accounting
including files in directory /etc/raddb/sites-enabled/
including configuration file /etc/raddb/sites-enabled/default
including configuration file /etc/raddb/sites-enabled/inner-tunnel
main {
	name = "radiusd"
	prefix = "/"
	localstatedir = "/var"
	sbindir = "/usr/sbin"
	logdir = "/var/log/radius"
	run_dir = "/var/run/radiusd"
	libdir = "/usr/lib"
	radacctdir = "/var/log/radius/radacct"
	hostname_lookups = no
	max_request_time = 30
	cleanup_delay = 5
	max_requests = 16384
	pidfile = "/var/run/radiusd/radiusd.pid"
	checkrad = "checkrad"
	debug_level = 0
	proxy_requests = yes
 log {
 	stripped_names = no
 	auth = no
 	auth_badpass = no
 	auth_goodpass = no
 	colourise = yes
 	msg_denied = "You are already logged in - access denied"
 }
 resources {
 }
 security {
 	max_attributes = 200
 	reject_delay = 1.000000
 	status_server = yes
 	allow_vulnerable_openssl = "yes"
 }
}
radiusd: #### Loading Realms and Home Servers ####
 proxy server {
 	retry_delay = 5
 	retry_count = 3
 	default_fallback = no
 	dead_time = 120
 	wake_all_if_all_dead = no
 }
 home_server localhost {
 	ipaddr = 127.0.0.1
 	port = 1812
 	type = "auth"
 	secret = <<< secret >>>
 	response_window = 20.000000
 	response_timeouts = 1
 	max_outstanding = 65536
 	zombie_period = 40
 	status_check = "status-server"
 	ping_interval = 30
 	check_interval = 30
 	check_timeout = 4
 	num_answers_to_alive = 3
 	revive_interval = 120
  limit {
  	max_connections = 16
  	max_requests = 0
  	lifetime = 0
  	idle_timeout = 0
  }
  coa {
  	irt = 2
  	mrt = 16
  	mrc = 5
  	mrd = 30
  }
 }
 home_server_pool my_auth_failover {
	type = fail-over
	home_server = localhost
 }
 realm example.com {
	auth_pool = my_auth_failover
 }
 realm LOCAL {
 }
 realm int {
	virtual_server = inner-tunnel
 }
radiusd: #### Loading Clients ####
 client localhost {
 	ipaddr = 127.0.0.1
 	require_message_authenticator = no
 	secret = <<< secret >>>
 	nas_type = "other"
 	proto = "*"
  limit {
  	max_connections = 16
  	lifetime = 0
  	idle_timeout = 30
  }
 }
 client localhost_ipv6 {
 	ipv6addr = ::1
 	require_message_authenticator = no
 	secret = <<< secret >>>
  limit {
  	max_connections = 16
  	lifetime = 0
  	idle_timeout = 30
  }
 }
Debug state unknown (cap_sys_ptrace capability not set)
 # Creating Auth-Type = mschap
 # Creating Auth-Type = digest
 # Creating Auth-Type = eap
 # Creating Auth-Type = PAP
 # Creating Auth-Type = CHAP
 # Creating Auth-Type = MS-CHAP
radiusd: #### Instantiating modules ####
 modules {
  # Loaded module rlm_pap
  # Loading module "pap" from file /etc/raddb/mods-enabled/pap
  pap {
  	normalise = yes
  }
  # Loaded module rlm_expiration
  # Loading module "expiration" from file /etc/raddb/mods-enabled/expiration
  # Loaded module rlm_files
  # Loading module "files" from file /etc/raddb/mods-enabled/files
  files {
  	filename = "/etc/raddb/mods-config/files/authorize"
  	acctusersfile = "/etc/raddb/mods-config/files/accounting"
  	preproxy_usersfile = "/etc/raddb/mods-config/files/pre-proxy"
  }
  # Loaded module rlm_linelog
  # Loading module "linelog" from file /etc/raddb/mods-enabled/linelog
  linelog {
  	filename = "/var/log/radius/linelog"
  	escape_filenames = no
  	syslog_severity = "info"
  	permissions = 384
  	format = "This is a log message for %{User-Name}"
  	reference = "messages.%{%{reply:Packet-Type}:-default}"
  }
  # Loading module "log_accounting" from file /etc/raddb/mods-enabled/linelog
  linelog log_accounting {
  	filename = "/var/log/radius/linelog-accounting"
  	escape_filenames = no
  	syslog_severity = "info"
  	permissions = 384
  	format = ""
  	reference = "Accounting-Request.%{%{Acct-Status-Type}:-unknown}"
  }
  # Loaded module rlm_soh
  # Loading module "soh" from file /etc/raddb/mods-enabled/soh
  soh {
  	dhcp = yes
  }
  # Loaded module rlm_attr_filter
  # Loading module "attr_filter.post-proxy" from file /etc/raddb/mods-enabled/attr_filter
  attr_filter attr_filter.post-proxy {
  	filename = "/etc/raddb/mods-config/attr_filter/post-proxy"
  	key = "%{Realm}"
  	relaxed = no
  }
  # Loading module "attr_filter.pre-proxy" from file /etc/raddb/mods-enabled/attr_filter
  attr_filter attr_filter.pre-proxy {
  	filename = "/etc/raddb/mods-config/attr_filter/pre-proxy"
  	key = "%{Realm}"
  	relaxed = no
  }
  # Loading module "attr_filter.access_reject" from file /etc/raddb/mods-enabled/attr_filter
  attr_filter attr_filter.access_reject {
  	filename = "/etc/raddb/mods-config/attr_filter/access_reject"
  	key = "%{User-Name}"
  	relaxed = no
  }
  # Loading module "attr_filter.access_challenge" from file /etc/raddb/mods-enabled/attr_filter
  attr_filter attr_filter.access_challenge {
  	filename = "/etc/raddb/mods-config/attr_filter/access_challenge"
  	key = "%{User-Name}"
  	relaxed = no
  }
  # Loading module "attr_filter.accounting_response" from file /etc/raddb/mods-enabled/attr_filter
  attr_filter attr_filter.accounting_response {
  	filename = "/etc/raddb/mods-config/attr_filter/accounting_response"
  	key = "%{User-Name}"
  	relaxed = no
  }
  # Loaded module rlm_exec
  # Loading module "ntlm_auth" from file /etc/raddb/mods-enabled/ntlm_auth
  exec ntlm_auth {
  	wait = yes
  	program = "/path/to/ntlm_auth --request-nt-key --domain=MYDOMAIN --username=%{mschap:User-Name} --password=%{User-Password}"
  	shell_escape = yes
  }
  # Loading module "exec" from file /etc/raddb/mods-enabled/exec
  exec {
  	wait = no
  	input_pairs = "request"
  	shell_escape = yes
  	timeout = 10
  }
  # Loaded module rlm_preprocess
  # Loading module "preprocess" from file /etc/raddb/mods-enabled/preprocess
  preprocess {
  	huntgroups = "/etc/raddb/mods-config/preprocess/huntgroups"
  	hints = "/etc/raddb/mods-config/preprocess/hints"
  	with_ascend_hack = no
  	ascend_channels_per_line = 23
  	with_ntdomain_hack = no
  	with_specialix_jetstream_hack = no
  	with_cisco_vsa_hack = no
  	with_alvarion_vsa_hack = no
  }
  # Loaded module rlm_radutmp
  # Loading module "sradutmp" from file /etc/raddb/mods-enabled/sradutmp
  radutmp sradutmp {
  	filename = "/var/log/radius/sradutmp"
  	username = "%{User-Name}"
  	case_sensitive = yes
  	check_with_nas = yes
  	permissions = 420
  	caller_id = no
  }
  # Loaded module rlm_chap
  # Loading module "chap" from file /etc/raddb/mods-enabled/chap
  # Loaded module rlm_digest
  # Loading module "digest" from file /etc/raddb/mods-enabled/digest
  # Loaded module rlm_expr
  # Loading module "expr" from file /etc/raddb/mods-enabled/expr
  expr {
  	safe_characters = "@abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789.-_: /äéöüàâæçèéêëîïôœùûüaÿÄÉÖÜßÀÂÆÇÈÉÊËÎÏÔŒÙÛÜŸ"
  }
  # Loading module "echo" from file /etc/raddb/mods-enabled/echo
  exec echo {
  	wait = yes
  	program = "/bin/echo %{User-Name}"
  	input_pairs = "request"
  	output_pairs = "reply"
  	shell_escape = yes
  }
  # Loaded module rlm_unpack
  # Loading module "unpack" from file /etc/raddb/mods-enabled/unpack
  # Loaded module rlm_detail
  # Loading module "detail" from file /etc/raddb/mods-enabled/detail
  detail {
  	filename = "/var/log/radius/radacct/%{%{Packet-Src-IP-Address}:-%{Packet-Src-IPv6-Address}}/detail-%Y%m%d"
  	header = "%t"
  	permissions = 384
  	locking = no
  	escape_filenames = no
  	log_packet_header = no
  }
  # Loaded module rlm_always
  # Loading module "reject" from file /etc/raddb/mods-enabled/always
  always reject {
  	rcode = "reject"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "fail" from file /etc/raddb/mods-enabled/always
  always fail {
  	rcode = "fail"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "ok" from file /etc/raddb/mods-enabled/always
  always ok {
  	rcode = "ok"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "handled" from file /etc/raddb/mods-enabled/always
  always handled {
  	rcode = "handled"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "invalid" from file /etc/raddb/mods-enabled/always
  always invalid {
  	rcode = "invalid"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "userlock" from file /etc/raddb/mods-enabled/always
  always userlock {
  	rcode = "userlock"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "notfound" from file /etc/raddb/mods-enabled/always
  always notfound {
  	rcode = "notfound"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "noop" from file /etc/raddb/mods-enabled/always
  always noop {
  	rcode = "noop"
  	simulcount = 0
  	mpp = no
  }
  # Loading module "updated" from file /etc/raddb/mods-enabled/always
  always updated {
  	rcode = "updated"
  	simulcount = 0
  	mpp = no
  }
  # Loaded module rlm_eap
  # Loading module "eap" from file /etc/raddb/mods-enabled/eap
  eap {
  	default_eap_type = "md5"
  	timer_expire = 60
  	ignore_unknown_eap_types = no
  	cisco_accounting_username_bug = no
  	max_sessions = 16384
  }
  # Loaded module rlm_mschap
  # Loading module "mschap" from file /etc/raddb/mods-enabled/mschap
  mschap {
  	use_mppe = yes
  	require_encryption = no
  	require_strong = no
  	with_ntdomain_hack = yes
   passchange {
   }
  	allow_retry = yes
  	winbind_retry_with_normalised_username = no
  }
  # Loaded module rlm_unix
  # Loading module "unix" from file /etc/raddb/mods-enabled/unix
  unix {
  	radwtmp = "/var/log/radius/radwtmp"
  }
Creating attribute Unix-Group
  # Loading module "auth_log" from file /etc/raddb/mods-enabled/detail.log
  detail auth_log {
  	filename = "/var/log/radius/radacct/%{%{Packet-Src-IP-Address}:-%{Packet-Src-IPv6-Address}}/auth-detail-%Y%m%d"
  	header = "%t"
  	permissions = 384
  	locking = no
  	escape_filenames = no
  	log_packet_header = no
  }
  # Loading module "reply_log" from file /etc/raddb/mods-enabled/detail.log
  detail reply_log {
  	filename = "/var/log/radius/radacct/%{%{Packet-Src-IP-Address}:-%{Packet-Src-IPv6-Address}}/reply-detail-%Y%m%d"
  	header = "%t"
  	permissions = 384
  	locking = no
  	escape_filenames = no
  	log_packet_header = no
  }
  # Loading module "pre_proxy_log" from file /etc/raddb/mods-enabled/detail.log
  detail pre_proxy_log {
  	filename = "/var/log/radius/radacct/%{%{Packet-Src-IP-Address}:-%{Packet-Src-IPv6-Address}}/pre-proxy-detail-%Y%m%d"
  	header = "%t"
  	permissions = 384
  	locking = no
  	escape_filenames = no
  	log_packet_header = no
  }
  # Loading module "post_proxy_log" from file /etc/raddb/mods-enabled/detail.log
  detail post_proxy_log {
  	filename = "/var/log/radius/radacct/%{%{Packet-Src-IP-Address}:-%{Packet-Src-IPv6-Address}}/post-proxy-detail-%Y%m%d"
  	header = "%t"
  	permissions = 384
  	locking = no
  	escape_filenames = no
  	log_packet_header = no
  }
  # Loaded module rlm_passwd
  # Loading module "etc_passwd" from file /etc/raddb/mods-enabled/passwd
  passwd etc_passwd {
  	filename = "/etc/passwd"
  	format = "*User-Name:Crypt-Password:"
  	delimiter = ":"
  	ignore_nislike = no
  	ignore_empty = yes
  	allow_multiple_keys = no
  	hash_size = 100
  }
  # Loaded module rlm_date
  # Loading module "date" from file /etc/raddb/mods-enabled/date
  date {
  	format = "%b %e %Y %H:%M:%S %Z"
  	utc = no
  }
  # Loaded module rlm_logintime
  # Loading module "logintime" from file /etc/raddb/mods-enabled/logintime
  logintime {
  	minimum_timeout = 60
  }
  # Loaded module rlm_utf8
  # Loading module "utf8" from file /etc/raddb/mods-enabled/utf8
  # Loaded module rlm_dynamic_clients
  # Loading module "dynamic_clients" from file /etc/raddb/mods-enabled/dynamic_clients
  # Loading module "radutmp" from file /etc/raddb/mods-enabled/radutmp
  radutmp {
  	filename = "/var/log/radius/radutmp"
  	username = "%{User-Name}"
  	case_sensitive = yes
  	check_with_nas = yes
  	permissions = 384
  	caller_id = yes
  }
  # Loaded module rlm_realm
  # Loading module "IPASS" from file /etc/raddb/mods-enabled/realm
  realm IPASS {
  	format = "prefix"
  	delimiter = "/"
  	ignore_default = no
  	ignore_null = no
  }
  # Loading module "suffix" from file /etc/raddb/mods-enabled/realm
  realm suffix {
  	format = "suffix"
  	delimiter = "@"
  	ignore_default = no
  	ignore_null = no
  }
  # Loading module "realmpercent" from file /etc/raddb/mods-enabled/realm
  realm realmpercent {
  	format = "suffix"
  	delimiter = "%"
  	ignore_default = no
  	ignore_null = no
  }
  # Loading module "ntdomain" from file /etc/raddb/mods-enabled/realm
  realm ntdomain {
  	format = "prefix"
  	delimiter = "\\"
  	ignore_default = no
  	ignore_null = no
  }
  # Loaded module rlm_cache
  # Loading module "cache_eap" from file /etc/raddb/mods-enabled/cache_eap
  cache cache_eap {
  	driver = "rlm_cache_rbtree"
  	key = "%{%{control:State}:-%{%{reply:State}:-%{State}}}"
  	ttl = 15
  	max_entries = 0
  	epoch = 0
  	add_stats = no
  }
  # Loaded module rlm_replicate
  # Loading module "replicate" from file /etc/raddb/mods-enabled/replicate
  instantiate {
  }
  # Instantiating module "pap" from file /etc/raddb/mods-enabled/pap
  # Instantiating module "expiration" from file /etc/raddb/mods-enabled/expiration
  # Instantiating module "files" from file /etc/raddb/mods-enabled/files
reading pairlist file /etc/raddb/mods-config/files/authorize
reading pairlist file /etc/raddb/mods-config/files/accounting
reading pairlist file /etc/raddb/mods-config/files/pre-proxy
  # Instantiating module "linelog" from file /etc/raddb/mods-enabled/linelog
  # Instantiating module "log_accounting" from file /etc/raddb/mods-enabled/linelog
  # Instantiating module "attr_filter.post-proxy" from file /etc/raddb/mods-enabled/attr_filter
reading pairlist file /etc/raddb/mods-config/attr_filter/post-proxy
  # Instantiating module "attr_filter.pre-proxy" from file /etc/raddb/mods-enabled/attr_filter
reading pairlist file /etc/raddb/mods-config/attr_filter/pre-proxy
  # Instantiating module "attr_filter.access_reject" from file /etc/raddb/mods-enabled/attr_filter
reading pairlist file /etc/raddb/mods-config/attr_filter/access_reject
[/etc/raddb/mods-config/attr_filter/access_reject]:11 Check item "FreeRADIUS-Response-Delay" 	found in filter list for realm "DEFAULT". 
[/etc/raddb/mods-config/attr_filter/access_reject]:11 Check item "FreeRADIUS-Response-Delay-USec" 	found in filter list for realm "DEFAULT". 
  # Instantiating module "attr_filter.access_challenge" from file /etc/raddb/mods-enabled/attr_filter
reading pairlist file /etc/raddb/mods-config/attr_filter/access_challenge
  # Instantiating module "attr_filter.accounting_response" from file /etc/raddb/mods-enabled/attr_filter
reading pairlist file /etc/raddb/mods-config/attr_filter/accounting_response
  # Instantiating module "preprocess" from file /etc/raddb/mods-enabled/preprocess
reading pairlist file /etc/raddb/mods-config/preprocess/huntgroups
reading pairlist file /etc/raddb/mods-config/preprocess/hints
  # Instantiating module "detail" from file /etc/raddb/mods-enabled/detail
  # Instantiating module "reject" from file /etc/raddb/mods-enabled/always
  # Instantiating module "fail" from file /etc/raddb/mods-enabled/always
  # Instantiating module "ok" from file /etc/raddb/mods-enabled/always
  # Instantiating module "handled" from file /etc/raddb/mods-enabled/always
  # Instantiating module "invalid" from file /etc/raddb/mods-enabled/always
  # Instantiating module "userlock" from file /etc/raddb/mods-enabled/always
  # Instantiating module "notfound" from file /etc/raddb/mods-enabled/always
  # Instantiating module "noop" from file /etc/raddb/mods-enabled/always
  # Instantiating module "updated" from file /etc/raddb/mods-enabled/always
  # Instantiating module "eap" from file /etc/raddb/mods-enabled/eap
   # Linked to sub-module rlm_eap_md5
   # Linked to sub-module rlm_eap_leap
   # Linked to sub-module rlm_eap_gtc
   gtc {
   	challenge = "Password: "
   	auth_type = "PAP"
   }
   # Linked to sub-module rlm_eap_tls
   tls {
   	tls = "tls-common"
   }
   tls-config tls-common {
   	verify_depth = 0
   	ca_path = "/etc/raddb/certs"
   	pem_file_type = yes
   	private_key_file = "/etc/raddb/certs/server.pem"
   	certificate_file = "/etc/raddb/certs/server.pem"
   	ca_file = "/etc/raddb/certs/ca.pem"
   	private_key_password = <<< secret >>>
   	dh_file = "/etc/raddb/certs/dh"
   	fragment_size = 1024
   	include_length = yes
   	auto_chain = yes
   	check_crl = no
   	check_all_crl = no
   	cipher_list = "DEFAULT"
   	cipher_server_preference = no
   	ecdh_curve = "prime256v1"
   	tls_max_version = ""
   	tls_min_version = "1.0"
    cache {
    	enable = no
    	lifetime = 24
    	max_entries = 255
    }
    verify {
    	skip_if_ocsp_ok = no
    }
    ocsp {
    	enable = no
    	override_cert_url = yes
    	url = "http://127.0.0.1/ocsp/"
    	use_nonce = yes
    	timeout = 0
    	softfail = no
    }
   }
   # Linked to sub-module rlm_eap_ttls
   ttls {
   	tls = "tls-common"
   	default_eap_type = "md5"
   	copy_request_to_tunnel = no
   	use_tunneled_reply = no
   	virtual_server = "inner-tunnel"
   	include_length = yes
   	require_client_cert = no
   }
tls: Using cached TLS configuration from previous invocation
   # Linked to sub-module rlm_eap_peap
   peap {
   	tls = "tls-common"
   	default_eap_type = "mschapv2"
   	copy_request_to_tunnel = no
   	use_tunneled_reply = no
   	proxy_tunneled_request_as_eap = yes
   	virtual_server = "inner-tunnel"
   	soh = no
   	require_client_cert = no
   }
tls: Using cached TLS configuration from previous invocation
   # Linked to sub-module rlm_eap_mschapv2
   mschapv2 {
   	with_ntdomain_hack = no
   	send_error = no
   }
  # Instantiating module "mschap" from file /etc/raddb/mods-enabled/mschap
rlm_mschap (mschap): using internal authentication
  # Instantiating module "auth_log" from file /etc/raddb/mods-enabled/detail.log
rlm_detail (auth_log): 'User-Password' suppressed, will not appear in detail output
  # Instantiating module "reply_log" from file /etc/raddb/mods-enabled/detail.log
  # Instantiating module "pre_proxy_log" from file /etc/raddb/mods-enabled/detail.log
  # Instantiating module "post_proxy_log" from file /etc/raddb/mods-enabled/detail.log
  # Instantiating module "etc_passwd" from file /etc/raddb/mods-enabled/passwd
rlm_passwd: nfields: 3 keyfield 0(User-Name) listable: no
  # Instantiating module "logintime" from file /etc/raddb/mods-enabled/logintime
  # Instantiating module "IPASS" from file /etc/raddb/mods-enabled/realm
  # Instantiating module "suffix" from file /etc/raddb/mods-enabled/realm
  # Instantiating module "realmpercent" from file /etc/raddb/mods-enabled/realm
  # Instantiating module "ntdomain" from file /etc/raddb/mods-enabled/realm
  # Instantiating module "cache_eap" from file /etc/raddb/mods-enabled/cache_eap
rlm_cache (cache_eap): Driver rlm_cache_rbtree (module rlm_cache_rbtree) loaded and linked
 } # modules
radiusd: #### Loading Virtual Servers ####
server { # from file /etc/raddb/radiusd.conf
} # server
server default { # from file /etc/raddb/sites-enabled/default
 # Loading authenticate {...}
 # Loading authorize {...}
 # Loading preacct {...}
 # Loading accounting {...}
Ignoring "sql" (see raddb/mods-available/README.rst)
 # Loading post-proxy {...}
 # Loading post-auth {...}
} # server default
server inner-tunnel { # from file /etc/raddb/sites-enabled/inner-tunnel
 # Loading authenticate {...}
 # Loading authorize {...}
 # Loading session {...}
 # Loading post-auth {...}
 # Skipping contents of 'if' as it is always 'false' -- /etc/raddb/sites-enabled/inner-tunnel:335
} # server inner-tunnel
radiusd: #### Opening IP addresses and Ports ####
listen {
  	type = "auth"
  	ipaddr = *
  	port = 0
   limit {
   	max_connections = 16
   	lifetime = 0
   	idle_timeout = 30
   }
}
listen {
  	type = "acct"
  	ipaddr = *
  	port = 0
   limit {
   	max_connections = 16
   	lifetime = 0
   	idle_timeout = 30
   }
}
listen {
  	type = "auth"
  	ipv6addr = ::
  	port = 0
   limit {
   	max_connections = 16
   	lifetime = 0
   	idle_timeout = 30
   }
}
listen {
  	type = "acct"
  	ipv6addr = ::
  	port = 0
   limit {
   	max_connections = 16
   	lifetime = 0
   	idle_timeout = 30
   }
}
listen {
  	type = "auth"
  	ipaddr = 127.0.0.1
  	port = 18120
}
Listening on auth address * port 1812 bound to server default
Listening on acct address * port 1813 bound to server default
Listening on auth address :: port 1812 bound to server default
Listening on acct address :: port 1813 bound to server default
Listening on auth address 127.0.0.1 port 18120 bound to server inner-tunnel
Listening on proxy address * port 39556
Listening on proxy address :: port 52609
Ready to process requests
(0) Received Access-Request Id 104 from 127.0.0.1:33278 to 127.0.0.1:1812 length 73
(0)   User-Name = "bob"
(0)   User-Password = "wrongpassword"
(0)   NAS-IP-Address = 127.0.1.1
(0)   NAS-Port = 0
(0)   Message-Authenticator = 0x3d27116b37323e4f629b4e8217fc25c8
(0) # Executing section authorize from file /etc/raddb/sites-enabled/default
(0)   authorize {
(0) suffix: Checking for suffix after "@"
(0) suffix: No '@' in User-Name = "bob", looking up realm NULL
(0) suffix: No such realm "NULL"
(0)     [suffix] = noop
(0)     [files] = noop
(0)   } # authorize = noop
(0) ERROR: No Auth-Type found: rejecting the user via Post-Auth-Type = Reject
(0) Failed to authenticate the user
(0) Using Post-Auth-Type Reject
(0) # Executing group from file /etc/raddb/sites-enabled/default
(0)   Post-Auth-Type REJECT {
(0) attr_filter.access_reject: EXPAND %{User-Name}
(0) attr_filter.access_reject:    --> bob
(0) attr_filter.access_reject: Matched entry DEFAULT at line 11
(0)     [attr_filter.access_reject] = updated
(0)     [eap] = noop
(0)     policy remove_reply_message_if_eap {
(0)       if (&reply:EAP-Message && &reply:Reply-Message) {
(0)       if (&reply:EAP-Message && &reply:Reply-Message)  -> FALSE
(0)       else {
(0)         [noop] = noop
(0)       } # else = noop
(0)     } # policy remove_reply_message_if_eap = noop
(0)   } # Post-Auth-Type REJECT = updated
(0) Delaying response for 1.000000 seconds
Waking up in 0.3 seconds.
Waking up in 0.6 seconds.
(0) Sending delayed response
(0) Sent Access-Reject Id 104 from 127.0.0.1:1812 to 127.0.0.1:33278 length 20
Waking up in 3.9 seconds.
(0) Cleaning up request packet ID 104 with timestamp +23
Ready to process requests
(1) Received Access-Request Id 146 from 127.0.0.1:40967 to 127.0.0.1:1812 length 73
(1)   User-Name = "bob"
(1)   User-Password = "wrongagain"
(1)   NAS-IP-Address = 127.0.1.1
(1)   NAS-Port = 0
(1)   Message-Authenticator = 0xa5390786792b1de136fa519bc0c2421c
(1) # Executing section authorize from file /etc/raddb/sites-enabled/default
(1)   authorize {
(1) suffix: Checking for suffix after "@"
(1) suffix: No '@' in User-Name = "bob", looking up realm NULL
(1) suffix: No such realm "NULL"
(1)     [suffix] = noop
(1)     [files] = noop
(1)   } # authorize = noop
(1) ERROR: No Auth-Type found: rejecting the user via Post-Auth-Type = Reject
(1) Failed to authenticate the user
(1) Using Post-Auth-Type Reject
(1) # Executing group from file /etc/raddb/sites-enabled/default
(1)   Post-Auth-Type REJECT {
(1) attr_filter.access_reject: EXPAND %{User-Name}
(1) attr_filter.access_reject:    --> bob
(1) attr_filter.access_reject: Matched entry DEFAULT at line 11
(1)     [attr_filter.access_reject] = updated
(1)     [eap] = noop
(1)     policy remove_reply_message_if_eap {
(1)       if (&reply:EAP-Message && &reply:Reply-Message) {
(1)       if (&reply:EAP-Message && &reply:Reply-Message)  -> FALSE
(1)       else {
(1)         [noop] = noop
(1)       } # else = noop
(1)     } # policy remove_reply_message_if_eap = noop
(1)   } # Post-Auth-Type REJECT = updated
(1) Delaying response for 1.000000 seconds
Waking up in 0.3 seconds.
Waking up in 0.6 seconds.
(1) Sending delayed response
(1) Sent Access-Reject Id 146 from 127.0.0.1:1812 to 127.0.0.1:40967 length 20
Waking up in 3.9 seconds.
(1) Cleaning up request packet ID 146 with timestamp +37
Ready to process requests
(2) Received Access-Request Id 135 from 127.0.0.1:40344 to 127.0.0.1:1812 length 77
(2)   User-Name = "bob@int"
(2)   User-Password = "test"
(2)   NAS-IP-Address = 127.0.1.1
(2)   NAS-Port = 0
(2)   Message-Authenticator = 0x3b3f4cf11005dcccfe78bb4a5830dd52
(2) # Executing section authorize from file /etc/raddb/sites-enabled/default
(2)   authorize {
(2) suffix: Checking for suffix after "@"
(2) suffix: Looking up realm "int" for User-Name = "bob@int"
(2) suffix: Found realm "int"
(2) suffix: Adding Stripped-User-Name = "bob"
(2) suffix: Adding Realm = "int"
(2) suffix: Proxying request from user bob to realm int
(2) suffix: Preparing to proxy authentication request to realm "int" 
(2)     [suffix] = updated
(2)     [files] = noop
(2)   } # authorize = updated
(2) Starting proxy to home server (null) port 1812
Proxying to virtual server inner-tunnel
(2) # Executing section authorize from file /etc/raddb/sites-enabled/inner-tunnel
(2)   authorize {
(2) files: users: Matched entry bob@int at line 1
(2)     [files] = ok
(2)     [pap] = updated
(2)   } # authorize = updated
(2) Found Auth-Type = PAP
(2) # Executing group from file /etc/raddb/sites-enabled/inner-tunnel
(2)   Auth-Type PAP {
(2) pap: Login attempt with password
(2) pap: Comparing with "known good" Cleartext-Password
(2) pap: User authenticated successfully
(2)     [pap] = ok
(2)   } # Auth-Type PAP = ok
(2) # Executing section post-auth from file /etc/raddb/sites-enabled/inner-tunnel
(2)   post-auth {
(2)     update reply {
(2)       Reply-Message := "hello"
(2)     } # update reply = noop
(2)     if (0) {
(2)     if (0)  -> FALSE
(2)   } # post-auth = noop
(2) Finished internally proxied request.
(2) Clearing existing &reply: attributes
(2) # Executing section post-proxy from file /etc/raddb/sites-enabled/default
(2)   post-proxy {
(2)     policy debug_reply {
(2)       if ("%{debug_attr:reply:}" == '') {
(2)       Attributes matching "reply:"
(2)         EXPAND %{debug_attr:reply:}
(2)            --> 
(2)         if ("%{debug_attr:reply:}" == '')  -> TRUE
(2)         if ("%{debug_attr:reply:}" == '')  {
(2)           [noop] = noop
(2)         } # if ("%{debug_attr:reply:}" == '')  = noop
(2)       } # policy debug_reply = noop
(2)     } # post-proxy = noop
(2)   Found Auth-Type = Accept
(2)   Auth-Type = Accept, accepting the user
(2)   # Executing section post-auth from file /etc/raddb/sites-enabled/default
(2)     post-auth {
(2)       update {
(2)         No attributes updated
(2)       } # update = noop
(2)       [exec] = noop
(2)       policy remove_reply_message_if_eap {
(2)         if (&reply:EAP-Message && &reply:Reply-Message) {
(2)         if (&reply:EAP-Message && &reply:Reply-Message)  -> FALSE
(2)         else {
(2)           [noop] = noop
(2)         } # else = noop
(2)       } # policy remove_reply_message_if_eap = noop
(2)     } # post-auth = noop
(2)   Sent Access-Accept Id 135 from 127.0.0.1:1812 to 127.0.0.1:40344 length 0
(2)     Reply-Message := "hello"
(2)   Finished request
Waking up in 4.9 seconds.
(2)   Cleaning up request packet ID 135 with timestamp +74
Ready to process requests
```