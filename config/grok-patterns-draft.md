# Parsing Config Draft

## Grok Parsing:

### Audit-Service Log Line

| LOG LINE - AUDIT SERVICE | FILTER MATCH  |
|:------------------------:|:-------------:|
| type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success' | type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |

EXAMPLE:

	type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'

MATCHED BY:

	type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'

### Audit-Auth Log Line

| LOG LINE - AUDIT AUTH | FILTER MATCH  |
|:---------------------:|:-------------:|
| type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success' | type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |


EXAMPLE:

	type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success'

MATCHED BY:

	type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'

### Samba Log Line

Custom pattern for this grok filter (Samba timestamp format):

	SMBDATE %{YEAR}\/%{MONTHNUM}\/%{MONTHDAY}%{SPACE}%{TIME}

| LOG LINE - SAMBA | FILTER MATCH  |
|:---------------------:|:-------------:|
| \[2017/05/07 03:49:40,  0\] lib/util_sock.c:1491(get_peer_addr_internal)\n  getpeername failed. Error was Transport endpoint is not connected\n  Denied connection from 0.0.0.0 (0.0.0.0) | ^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{DATA:samba_class}\n%{SPACE}%{GREEDYDATA:samba_message}", "\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{GREEDYDATA:samba_class} |


EXAMPLE:

	\[2017/05/07 03:49:40,  0\] lib/util_sock.c:1491(get_peer_addr_internal)\n  getpeername failed. Error was Transport endpoint is not connected\n  Denied connection from 0.0.0.0 (0.0.0.0)

MATCHED BY:

	^\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{DATA:samba_class}\n%{SPACE}%{GREEDYDATA:samba_message}", "\[%{SMBDATE:samba_date},%{SPACE}%{NUMBER:samba_severity_code}\]%{SPACE}%{GREEDYDATA:samba_class}

### Radius Detail Log Line

Custom pattern for this grok filter (Radius timestamp format):

	RADIUSTIMESTAMP %{DAY} %{MONTH} %{MONTHDAY} %{TIME} %{YEAR}

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

	%{RADIUSTIMESTAMP}%{SPACE}Acct-Session-Id = %{QUOTEDSTRING:Acct-Session-Id}%{SPACE}Acct-Status-Type = %{WORD:AcctStatusType}%{SPACE}Acct-Authentic = %{WORD:AcctAuthentic}%{SPACE}User-Name = \"%{DATA:UserName}\"%{SPACE}NAS-IP-Address = %{IP:NASIPAddress}%{SPACE}NAS-Identifier = \"%{DATA:NASIdentifier}\"%{SPACE}NAS-Port = %{NUMBER:NASPort}%{SPACE}Called-Station-Id = \"%{DATA:CalledStationId}\"%{SPACE}Calling-Station-Id = \"%{MAC:CallingStationId}\"%{SPACE}NAS-Port-Type = %{DATA:NASPortType}%{SPACE}Connect-Info = \"%{DATA:ConnectInfo}\"%{SPACE}Acct-Unique-Session-Id = \"%{DATA:AcctUniqueSessionId}\"%{SPACE}Timestamp = %{NUMBER:Timestamp}


[//]: ##################################################################
[//]: ##################################################################

---

## Custom patterns file example

```
TYPE type=%{WORD}
MSG_ID msg=audit([0-9]+\.[0-9]+:[0-9]+)
PID pid=[0-9]+
UID uid=[0-9]+
UNIT unit=%{WORD}@%{WORD}
EXE exe="%{UNIXPATH}"
MSG msg=%{QUOTEDSTRING}
RES res=%{WORD}
SMBDATE %{YEAR}\/%{MONTHNUM}\/%{MONTHDAY}%{SPACE}%{TIME}
```

[//]: ##################################################################
[//]: ##################################################################

---

## Query to Elastic:

	curl -sXGET 'http://localhost:9200/filebeat-*/_search' | jq .
	curl -sXGET 'localhost:9200/filebeat-2017.05.15/_search?q=_id:AVwLEWovA8gqDUZC_cY5' | jq .
	curl -XDELETE 'localhost:9200/filebeat-2017.05.15'

Recommended to use "jq" to highlight syntax for a more user-friendly preview.

[//]: ##################################################################
[//]: ##################################################################

---

## References

* https://logz.io/blog/logstash-grok/
* https://grokdebug.herokuapp.com/
* https://discuss.elastic.co/t/grok-multiple-match-logstash/27870
