# Data Flow

## Radius Raw Output:

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

## After Filebeat:

	{
	  "@timestamp": "2017-05-23T08:22:02.752Z",
	  "beat": {
		"hostname": "i03",
		"name": "i03",
		"version": "5.4.0"
	  },
	  "input_type": "log",
	  "message": "Tue Apr 11 11:10:42 2017\n        Acct-Session-Id = \"00000028-0000013C\"\n        Acct-Status-Type = Stop\n        Acct-Authentic = RADIUS\n        User-Name = \"oit47325278\"\n        NAS-IP-Address = 10.200.199.12\n        NAS-Identifier = \"002722fc0683\"\n        NAS-Port = 0\n        Called-Station-Id = \"0E-27-22-FD-06-83:edt_alumnes\"\n        Calling-Station-Id = \"84-B5-41-D7-45-BE\"\n        NAS-Port-Type = Wireless-802.11\n        Connect-Info = \"CONNECT 0Mbps 802.11b\"\n        Acct-Session-Time = 310\n        Acct-Input-Packets = 23\n        Acct-Output-Packets = 22\n        Acct-Input-Octets = 3004\n        Acct-Output-Octets = 8432\n        Event-Timestamp = \"Apr 11 2017 11:10:41 CEST\"\n        Acct-Terminate-Cause = User-Request\n        Acct-Unique-Session-Id = \"8c01c85615651a70\"\n        Timestamp = 1491901842",
	  "offset": 1360,
	  "source": "/var/tmp/logs/radius/detail-sample-5.log",
	  "type": "radius-detail"
	}

## After Logstash Filters + Indexed in Elasticsearch

	{
	  "_index": "filebeat-2017.05.23",
	  "_type": "radius-detail",
	  "_id": "AVw0ZiLYgZ0KaAKuAf_5",
	  "_score": null,
	  "_source": {
		"AcctSessionId": "00000028-0000013C",
		"AcctOutputPackets": "22",
		"source": "/var/tmp/logs/radius/detail-sample-5.log",
		"type": "radius-detail",
		"radius-timestamp": "Tue Apr 11 11:10:42 2017",
		"Timestamp": "1491901842",
		"NASIdentifier": "002722fc0683",
		"CallingStationId": "84-B5-41-D7-45-BE",
		"AcctInputPackets": "23",
		"@version": "1",
		"beat": {
		  "hostname": "i03",
		  "name": "i03",
		  "version": "5.4.0"
		},
		"host": "i03",
		"UserName": "oit47325278",
		"offset": 1360,
		"input_type": "log",
		"AcctSessionTime": "310",
		"NASPort": "0",
		"AcctInputOctets": "3004",
		"message": "Tue Apr 11 11:10:42 2017\n        Acct-Session-Id = \"00000028-0000013C\"\n        Acct-Status-Type = Stop\n        Acct-Authentic = RADIUS\n        User-Name = \"oit47325278\"\n        NAS-IP-Address = 10.200.199.12\n        NAS-Identifier = \"002722fc0683\"\n        NAS-Port = 0\n        Called-Station-Id = \"0E-27-22-FD-06-83:edt_alumnes\"\n        Calling-Station-Id = \"84-B5-41-D7-45-BE\"\n        NAS-Port-Type = Wireless-802.11\n        Connect-Info = \"CONNECT 0Mbps 802.11b\"\n        Acct-Session-Time = 310\n        Acct-Input-Packets = 23\n        Acct-Output-Packets = 22\n        Acct-Input-Octets = 3004\n        Acct-Output-Octets = 8432\n        Event-Timestamp = \"Apr 11 2017 11:10:41 CEST\"\n        Acct-Terminate-Cause = User-Request\n        Acct-Unique-Session-Id = \"8c01c85615651a70\"\n        Timestamp = 1491901842",
		"AcctAuthentic": "RADIUS",
		"CalledStationId": "0E-27-22-FD-06-83:edt_alumnes",
		"tags": [
		  "beats_input_codec_plain_applied"
		],
		"NASPortType": "Wireless-802.11",
		"@timestamp": "2017-05-23T08:22:02.752Z",
		"AcctOutputOctets": "8432",
		"AcctStatusType": "Stop",
		"EventTimestamp": "Apr 11 2017 11:10:41 CEST",
		"AcctTerminateCause": "User-Request",
		"AcctUniqueSessionId": "8c01c85615651a70",
		"NASIPAddress": "10.200.199.12",
		"ConnectInfo": "CONNECT 0Mbps 802.11b"
	  },
	  "fields": {
		"@timestamp": [
		  1495527722752
		]
	  },
	  "highlight": {
		"type": [
		  "@kibana-highlighted-field@radius@/kibana-highlighted-field@-@kibana-highlighted-field@detail@/kibana-highlighted-field@"
		]
	  },
	  "sort": [
		1495527722752
	  ]
	}
