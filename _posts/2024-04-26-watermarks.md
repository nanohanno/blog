---
layout: post
title: "Avro"
subtitle: "Events with a schema"
date: 2024-04-19
background: '/images/default_post.jpg'
---
In a world of distributed systems, serialization of messages is an important topic to exchange data. The data is often serialized as plain xml or json which does not need a defined schema. On the other hand, there are serialization formats that require a defined schema like protobuf, thrift and Avro to use binary encodings.

In my previous job, the main language was Go and messages were serialized as protobuf which is deeply integrated into the ecosystem. Investigating Apache tools now, it seems that the main language is still Java and Hadoop had a big influence in many components. Therefore, Avro can be used in many places like Kafka, Flink and parquet.

## Schema

Avro, uses a defined schema which is typically written in JSON like in the following example from the [official docs](https://avro.apache.org/docs/1.11.1/getting-started-java/):

```json
{"namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": ["int", "null"]},
     {"name": "favorite_color", "type": ["string", "null"]}
 ]
}
```

Here, the fields and their type are defined in the `fields` array. From this schema definition, code can be generated (for Java). That means classes can be generated that have the same structure as the schema and can be used to serialize an object to the avro format or the other way, de-serializing a message from avro to a Java object:

```Java
User user1 = new User();
user1.setName("Alyssa");
```

## Benefits

As discussed by Kleppmann[^1], such binary encodings based on schemas are more compact than the plain text counterparts using less network bandwidth. Also the schema is a "valuable form of documentation" which is always up-to-date and can be used to check compatibility of APIs before deployments.


## References

[^1]: Designing Data-Intensive Applications, Martin Kleppman, O'Reilly, 2017