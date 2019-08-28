version 6.7

create mapping of aws elasticsearch


```
PUT domain/news
{
  "mappings": {
    "_doc": {
      "properties": {
        "createdAt": {
          "type": "date"
        },
        "slug": {
          "type": "text"
        },
        "title": {
          "type": "object",
          "properties": {
            "en-us": {
              "type": "text"
            },
            "zh-hans": {
              "type": "text",
              "analyzer": "smartcn"
            }
          }
        }
      }
    }
  }
}
```

But in verison 7.1
Error: The mapping definition cannot be nested under a type [_doc] unless include_type_name is set to true

Note that in 7.0, _doc is a permanent part of the path, and represents the endpoint name rather than the document type.

[elastic doc](https://www.elastic.co/guide/en/elasticsearch/reference/master/removal-of-types.html#removal-of-types)

[elastic mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)

```
PUT domain/news
{
  "mappings": {
    "properties": {
      "createdAt": {
        "type": "date"
      },
      "slug": {
          "type": "text"
      },
      "title": {
        "type": "object",
        "properties": {
          "en-us": {
            "type": "text"
          },
          "zh-hans": {
            "type": "text",
            "analyzer": "smartcn"
          }
        }
      }
    }
  }
}
```
