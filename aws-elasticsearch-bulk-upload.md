# AWS elasticsearch bulk upload

version 6.7

curl -XPOST https://<domain>/_bulk --data-binary @data.json -H 'Content-Type: application/json'

format:
{"index":{"_index":"news","_type":"_doc","_id": "123"}}
{"news":[{"id":"123", "title": "foo"}
{"index":{"_index":"news","_type":"_doc","_id": "456"}}
{"news":[{"id":"456", "title": "bar"}
