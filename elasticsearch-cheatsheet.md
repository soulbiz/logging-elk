# Elasticsearch Cheatsheet - Draft

## Query to Elastic:

	curl -sXGET 'http://localhost:9200/filebeat-*/_search' | jq .
	curl -sXGET 'localhost:9200/filebeat-2017.05.15/_search?q=_id:AVwLEWovA8gqDUZC_cY5' | jq .
	curl -XDELETE 'localhost:9200/filebeat-2017.05.15'

Recommended to use "jq" to highlight syntax for a more user-friendly preview!
