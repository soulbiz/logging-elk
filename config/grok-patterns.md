# Grok Parsing & Patterns

## About Grok:

According to the official [Grok Plugin](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-grok.html) reference for Logstash:

>Grok works by combining text patterns into something that matches your logs.
>The syntax for a grok pattern is `%{SYNTAX:SEMANTIC}`
>The SYNTAX is the name of the pattern that will match your text. For example, `3.44` will be matched by the NUMBER pattern and `55.3.244.1` will be matched by the IP pattern. The syntax is how you match.
>The SEMANTIC is the identifier you give to the piece of text being matched. For example, `3.44` could be the duration of an event, so you could call it simply duration. Further, a string `55.3.244.1` might identify the client making a request.
>For the above example, your grok filter would look something like this:
>`%{NUMBER:duration} %{IP:client}`

>Examples: With that idea of a syntax and semantic, we can pull out useful fields from a sample log like this fictional http request log:

	55.3.244.1 GET /index.html 15824 0.043

>The pattern for this could be:

	%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}

>After the grok filter, the event will have a few extra fields in it:

	client: 55.3.244.1
	method: GET
	request: /index.html
	bytes: 15824
	duration: 0.043 

>Grok sits on top of regular expressions, so any regular expressions are valid in grok as well. 

### Custom Patterns

After understanding how it works, we can also make our own custom patterns, which will make it easier for us to fetch our target data.
They can even contain other grok patterns!
We can save them in a text file with the following syntax, one pattern per line:

	PATTERN-SAMPLE [0-9A-F]{10,11}
	TYPE type=%{WORD}
	MSG_ID msg=audit([0-9]+\.[0-9]+:[0-9]+)
	PID pid=[0-9]+
	UID uid=[0-9]+
	UNIT unit=%{WORD}@%{WORD}
	EXE exe="%{UNIXPATH}"
	MSG msg=%{QUOTEDSTRING}
	RES res=%{WORD}
	SMBDATE %{YEAR}\/%{MONTHNUM}\/%{MONTHDAY}%{SPACE}%{TIME}

Then we just have to tell Grok where to find those patterns so we can use them!
To do so, we have to specify where to find them with the `patterns_dir` option inside the Grok parameters:
	
	patterns_dir => ["./patterns"]

Grok will load all the pattern files inside that directory.

For a more detailed use of the grok plugin inside Logstash, please check Elastic's [Grok Plugin](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-grok.html) reference!

Now, let's take a look at the filters we created for the logs we're using.

## Used Grok Patterns

### Audit Patterns

#### Audit-Service Log Line

| LOG LINE - AUDIT SERVICE | FILTER MATCH  |
|:------------------------:|:-------------:|
| type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success' | ^type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |

EXAMPLE:

	type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'

MATCHED BY:

	^type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'

#### Audit-Auth Log Line

| LOG LINE - AUDIT AUTH | FILTER MATCH  |
|:---------------------:|:-------------:|
| type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success' | ^type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |


EXAMPLE:

	type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success'

MATCHED BY:

	^type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'

---

### Samba Patterns

Custom pattern for this grok filter (Samba timestamp format):

	SMBDATE %{YEAR}\/%{MONTHNUM}\/%{MONTHDAY}%{SPACE}%{TIME}

#### Samba Log Line (w/ Message)

| LOG LINE - SAMBA W/MESSAGE | FILTER MATCH  |
|:---------------------:|:-------------:|
| [2017/05/07 03:49:40,  0] lib/util_sock.c:1491(get_peer_addr_internal)\n  getpeername failed. Error was Transport endpoint is not connected\n  Denied connection from 0.0.0.0 (0.0.0.0) | ^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{DATA:samba_class}\n%{SPACE}%{GREEDYDATA:samba_message} |

EXAMPLE:

	[2017/05/07 03:49:40,  0] lib/util_sock.c:1491(get_peer_addr_internal)\n  getpeername failed. Error was Transport endpoint is not connected\n  Denied connection from 0.0.0.0 (0.0.0.0)

MATCHED BY:

	^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{DATA:samba_class}\n%{SPACE}%{GREEDYDATA:samba_message}

#### Samba Log Line (No Message)
	
| LOG LINE - SAMBA SIMPLE | FILTER MATCH  |
|:---------------------:|:-------------:|
| [2017/05/08 11:38:01,  0] lib/util_sock.c:738(write_data) | ^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{DATA:samba_class}\n%{SPACE}%{GREEDYDATA:samba_message} |

EXAMPLE:

	[2017/05/08 11:38:01,  0] lib/util_sock.c:738(write_data)

MATCHED BY:

	^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{GREEDYDATA:samba_class}

---

### Radius Patterns

Custom pattern for this grok filter (Radius timestamp format):

	RADIUSTIMESTAMP %{DAY}%{SPACE}%{MONTH}%{SPACE}%{MONTHDAY}%{SPACE}%{TIME}%{SPACE}%{YEAR}

#### Radius Detail-Start Log Line

EXAMPLE:

	Tue Apr 11 11:05:32 2017
			Acct-Session-Id = "00000028-0000013C"
			Acct-Status-Type = Start
			Acct-Authentic = RADIUS
			User-Name = "oit47325278"
			NAS-IP-Address = 10.200.199.12
			NAS-Identifier = "002722fc0683"
			NAS-Port = 0
			Called-Station-Id = "0E-27-22-FD-06-83:edt_alumnes"
			Calling-Station-Id = "84-B5-41-D7-45-BE"
			NAS-Port-Type = Wireless-802.11
			Connect-Info = "CONNECT 0Mbps 802.11b"
			Acct-Unique-Session-Id = "8c01c85615651a70"
			Timestamp = 1491901532

MATCHED BY:

	^%{RADIUSTIMESTAMP:radius-timestamp}\n%{SPACE}Acct-Session-Id%{SPACE}=%{SPACE}\"%{DATA:AcctSessionId}\"\n%{SPACE}Acct-Status-Type%{SPACE}=%{SPACE}%{WORD:AcctStatusType}\n%{SPACE}Acct-Authentic%{SPACE}=%{SPACE}%{WORD:AcctAuthentic}\n%{SPACE}User-Name%{SPACE}=%{SPACE}\"%{DATA:UserName}\"\n%{SPACE}NAS-IP-Address%{SPACE}=%{SPACE}%{IP:NASIPAddress}\n%{SPACE}NAS-Identifier%{SPACE}=%{SPACE}\"%{DATA:NASIdentifier}\"\n%{SPACE}NAS-Port%{SPACE}=%{SPACE}%{NUMBER:NASPort}\n%{SPACE}Called-Station-Id%{SPACE}=%{SPACE}\"%{DATA:CalledStationId}\"\n%{SPACE}Calling-Station-Id%{SPACE}=%{SPACE}\"%{MAC:CallingStationId}\"\n%{SPACE}NAS-Port-Type%{SPACE}=%{SPACE}%{DATA:NASPortType}\n%{SPACE}Connect-Info%{SPACE}=%{SPACE}\"%{DATA:ConnectInfo}\"\n%{SPACE}Acct-Unique-Session-Id%{SPACE}=%{SPACE}\"%{DATA:AcctUniqueSessionId}\"\n%{SPACE}Timestamp%{SPACE}=%{SPACE}%{NUMBER:Timestamp}

#### Radius Detail-Stop Log Line

EXAMPLE:

	Tue Apr 11 11:10:42 2017
			Acct-Session-Id = "00000028-0000013C"
			Acct-Status-Type = Stop
			Acct-Authentic = RADIUS
			User-Name = "oit47325278"
			NAS-IP-Address = 10.200.199.12
			NAS-Identifier = "002722fc0683"
			NAS-Port = 0
			Called-Station-Id = "0E-27-22-FD-06-83:edt_alumnes"
			Calling-Station-Id = "84-B5-41-D7-45-BE"
			NAS-Port-Type = Wireless-802.11
			Connect-Info = "CONNECT 0Mbps 802.11b"
			Acct-Session-Time = 310
			Acct-Input-Packets = 23
			Acct-Output-Packets = 22
			Acct-Input-Octets = 3004
			Acct-Output-Octets = 8432
			Event-Timestamp = "Apr 11 2017 11:10:41 CEST"
			Acct-Terminate-Cause = User-Request
			Acct-Unique-Session-Id = "8c01c85615651a70"
			Timestamp = 1491901842

MATCHED BY:

	^%{RADIUSTIMESTAMP:radius-timestamp}\n%{SPACE}Acct-Session-Id%{SPACE}=%{SPACE}\"%{DATA:AcctSessionId}\"\n%{SPACE}Acct-Status-Type%{SPACE}=%{SPACE}%{WORD:AcctStatusType}\n%{SPACE}Acct-Authentic%{SPACE}=%{SPACE}%{WORD:AcctAuthentic}\n%{SPACE}User-Name%{SPACE}=%{SPACE}\"%{DATA:UserName}\"\n%{SPACE}NAS-IP-Address%{SPACE}=%{SPACE}%{IP:NASIPAddress}\n%{SPACE}NAS-Identifier%{SPACE}=%{SPACE}\"%{DATA:NASIdentifier}\"\n%{SPACE}NAS-Port%{SPACE}=%{SPACE}%{NUMBER:NASPort}\n%{SPACE}Called-Station-Id%{SPACE}=%{SPACE}\"%{DATA:CalledStationId}\"\n%{SPACE}Calling-Station-Id%{SPACE}=%{SPACE}\"%{MAC:CallingStationId}\"\n%{SPACE}NAS-Port-Type%{SPACE}=%{SPACE}%{DATA:NASPortType}\n%{SPACE}Connect-Info%{SPACE}=%{SPACE}\"%{DATA:ConnectInfo}\"\n%{SPACE}Acct-Session-Time%{SPACE}=%{SPACE}%{NUMBER:AcctSessionTime}\n%{SPACE}Acct-Input-Packets%{SPACE}=%{SPACE}%{NUMBER:AcctInputPackets}\n%{SPACE}Acct-Output-Packets%{SPACE}=%{SPACE}%{NUMBER:AcctOutputPackets}\n%{SPACE}Acct-Input-Octets%{SPACE}=%{SPACE}%{NUMBER:AcctInputOctets}\n%{SPACE}Acct-Output-Octets%{SPACE}=%{SPACE}%{NUMBER:AcctOutputOctets}\n%{SPACE}Event-Timestamp%{SPACE}=%{SPACE}\"%{DATA:EventTimestamp}\"\n%{SPACE}Acct-Terminate-Cause%{SPACE}=%{SPACE}%{DATA:AcctTerminateCause}\n%{SPACE}Acct-Unique-Session-Id%{SPACE}=%{SPACE}\"%{DATA:AcctUniqueSessionId}\"\n%{SPACE}Timestamp%{SPACE}=%{SPACE}%{NUMBER:Timestamp}


#### Radius Error/Info Long-Log Line

EXAMPLE:

	Wed May 10 12:51:54 2017 : Error: rlm_radutmp: Logout entry for NAS ap-biblio port 0 has wrong ID

MATCHED BY:

	^%{RADIUSTIMESTAMP}%{SPACE}:%{SPACE}%{WORD:radiusmessagetype}:%{SPACE}%{DATA:radiusmessageclass}:%{SPACE}%{GREEDYDATA:radiusmessage}


#### Radius Info Short-Log Line

EXAMPLE:

	Wed May 10 12:52:30 2017 : Info:   [ldap] Attempting reconnect

MATCHED BY:

	^%{RADIUSTIMESTAMP}%{SPACE}:%{SPACE}%{WORD:radiusmessagetype}:%{SPACE}%{GREEDYDATA:radiusmessage}

---

## References

* [**Logsz.io - Ran Ramati**: Guide to Logstash Grok](https://logz.io/blog/logstash-grok/)
* [**Grok Debugger**](https://grokdebug.herokuapp.com/)
* [**Elastic/Filebeat** - Managing Multiline Messages](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)
* [**Elastic/Logstash** - Grok Plugin Reference](https://www.elastic.co/guide/en/logstash/5.4/plugins-filters-grok.html)
* [**Digital Ocean - Mitchell Anicas**: Adding Logstash Filters To Improve Centralized Logging](https://www.digitalocean.com/community/tutorials/adding-logstash-filters-to-improve-centralized-logging)

