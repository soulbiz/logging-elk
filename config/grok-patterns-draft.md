# Examples Draft

## Grok Parsing Examples:

| LOG LINE - AUDIT SERVICE | FILTER MATCH  |
|:------------------------:|:-------------:|
| type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success' | type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |

EXAMPLE:

	type=SERVICE_STOP msg=audit(1493202842.882:2336): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=user@26 comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'

MATCHED BY:

	type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'unit=%{GREEDYDATA:unit} comm=\"%{WORD:command}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'


| LOG LINE - AUDIT AUTH | FILTER MATCH  |
|:---------------------:|:-------------:|
| type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success' | type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\' |


EXAMPLE:

	type=USER_AUTH msg=audit(1493202855.160:2337): pid=16686 uid=100495 auid=100495 ses=3 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 msg='op=PAM:authentication grantors=pam_unix acct="root" exe="/usr/bin/su" hostname=? addr=? terminal=pts/2 res=success'

MATCHED BY:

	type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{UNIXPATH:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'


[//]: ##################################################################
[//]: ##################################################################

---

## Custom patterns file example

TYPE type=%{WORD}
MSG_ID msg=audit([0-9]+\.[0-9]+:[0-9]+)
PID pid=[0-9]+
UID uid=[0-9]+
UNIT unit=%{WORD}@%{WORD}
EXE exe="%{UNIXPATH}"
MSG msg=%{QUOTEDSTRING}
RES res=%{WORD}

[//]: ##################################################################
[//]: ##################################################################

---

## Query to Elastic:

	curl -sXGET 'http://localhost:9200/filebeat-*/_search' | jq .

Recommended to use "jq" to highlight syntax for a more user-friendly preview.

[//]: ##################################################################
[//]: ##################################################################

---

```
filter {
  if [type] == "syslog" {
        grok {
         # patterns_dir => ["./patterns"]
         # example:
                 # type=CRED_DISP msg=audit(1431084081.914:298): pid=1807 uid=0 auid=1000
                 # ses=7 msg='op=PAM:setcred acct="user1" exe="/usr/sbin/sshd" hostname=host1 addr=192.168.160.1 terminal=ssh res=success'

        match => { "message" => "type=%{WORD:audit_type} msg=audit\(%{NUMBER:audit_epoch}:%{NUMBER:audit_counter}\): pid=%{NUMBER:audit_pid} uid=%{NUMBER:audit_uid} auid=%{NUMBER:audit_audid} ses=%{NUMBER:ses} subj=%{GREEDYDATA:subj} msg=\'op=%{WORD:operation}:%{WORD:detail_operation} grantors=%{WORD:grantors} acct=\"%{WORD:acct_user}\" exe=\"%{GREEDYDATA:exec}\" hostname=%{GREEDYDATA:hostname} addr=%{GREEDYDATA:ipaddr} terminal=%{GREEDYDATA:terminal} res=%{WORD:result}\'" }

        add_field => { "audit_type" => "%{audit_type}"}
        add_field => { "audit_epoch" => "%{audit_epoch}"}
        add_field => { "audit_counter" => "%{audit_counter}"}
        add_field => { "audit_pid" => "%{audit_pid}"}
        add_field => { "audit_uid" => "%{audit_uid}"}
        add_field => { "operation" => "%{operation}"}
        add_field => { "detail_operation" => "%{detail_operation}"}
        add_field => { "acct_user" => "%{acct_user}"}
        add_field => { "exec" => "%{exec}"}
        add_field => { "hostname" => "%{hostname}"}
        add_field => { "ipaddr" => "%{ipaddr}"}
        add_field => { "terminal" => "%{terminal}"}
        add_field => { "result" => "%{result}"}

}

        date {
          match => [ "audit_epoch", "UNIX_MS" ]
        }
  }
}
```

[//]: ##################################################################
[//]: ##################################################################

---

```
filter {
  if [type] == "syslog" {
        grok {
          patterns_dir => ["./patterns"]
          # example:
                  # type=CRED_DISP msg=audit(1431084081.914:298): pid=1807 uid=0 auid=1000
                  # ses=7 msg='op=PAM:setcred acct="user1" exe="/usr/sbin/sshd" hostname=host1
                  # addr=192.168.160.1 terminal=ssh res=success'

          match => { "message" => "%{AUDIT}" }
          tag_on_failure => [ "_grokparsefailure_auditlog" ]
          overwrite => [ "message" ]

        }

        if [audit_epoch] {
          date {
            match => [ "audit_epoch", "UNIX" ]
            remove_field => [ "audit_epoch" ]
          }
        }
  }
}
```

[//]: ##################################################################
[//]: ##################################################################

---

## References

* https://logz.io/blog/logstash-grok/
* https://grokdebug.herokuapp.com/
