# Elasticsearch Cheatsheet - Draft

## Query to Elastic:

	curl -sXGET 'http://localhost:9200/filebeat-*/_search' | jq .
	curl -sXGET 'localhost:9200/filebeat-2017.05.15/_search?q=_id:AVwLEWovA8gqDUZC_cY5' | jq .
	curl -XDELETE 'localhost:9200/filebeat-2017.05.15'
	curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "file.json"

Recommended to use "jq" to highlight syntax for a more user-friendly preview!

## Elastic API Samples

The following example inserts the typed JSON document into the "twitter" index, under a type called "tweet" with an id of 1:

Command:

	PUT twitter/tweet/1
	{
		"user" : "kimchy",
		"post_date" : "2009-11-15T14:12:12",
		"message" : "trying out Elasticsearch"
	}

Result:

	{
		"_shards" : {
			"total" : 2,
			"failed" : 0,
			"successful" : 2
		},
		"_index" : "twitter",
		"_type" : "tweet",
		"_id" : "1",
		"_version" : 1,
		"created" : true,
		"result" : created
	}
	
You can also use POST without specifying an ID, so it will be generated automatically.

	POST twitter/tweet/
	{
		"user" : "kimchy",
		"post_date" : "2009-11-15T14:12:12",
		"message" : "trying out Elasticsearch"
	}
	
In both cases, the index and type will be created.

### Searchs:

REST Request URI:

	GET /bank/_search?q=*&sort=account_number:asc&pretty

REST Request body:

	GET /bank/_search
	{
	  "query": { "match_all": {} },
	  "sort": [
		{ "account_number": "asc" }
	  ]
	}
