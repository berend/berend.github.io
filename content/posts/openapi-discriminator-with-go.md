---
title: "Openapi Discriminator With Go"
date: 2023-01-04T21:58:12+01:00
draft: false
---

I just recently solved a problem I had for a while. After winter holidays I decided to come back to it and solve it.

I have a openapi schema, that documents and validates a small web app. It made use of the `oneOf` operator to allow different sub documents for some requests.

However - when the request is slightly off for one of those sub document schema - *none* of those sub documents matches. In complex schema and during development this can be tedious. The validation lib I am using can not display _any_ error message and just show an empty string as error in older version. With the current version it displays all sub schemas and their error. Both not really helpful.

But there is a better way! The openapi spec allow to add a discriminator that gives the hint, what sub document schema should be uses for the given object.

Let's look at an example (complete code below in gists):

Here are the two sub documents:

```yaml
oneOf:
- $ref: "#/components/schemas/foo"
- $ref: "#/components/schemas/bar"
discriminator:
propertyName: thing_type
mapping:
    foo: "#/components/schemas/foo"
    bar: "#/components/schemas/bar"
```

and the two sub document schemas:

```yaml
foo:
    properties:
    thing_type:
        type: string
    a_value:
        type: string
bar:
    properties:
    thing_type:
        type: string
    a_value:
        type: number
```

Both have the `thing_type` in common. This serves as discriminator and gives the hint for the schema validator. The openapi has a mapping, where we define the discriminator propertyName and a mapping from `thing_type` to the corresponding schema part.

So - in short - foo wants a string value, bar wants a number.

When we now send foo a boolean and not a string
```json
{"thing_type":"foo", "a_value":false}
```

we get a detailed error message, just about what is wrong with the foo type, since the discriminator hint is working

```text
doesn't match schema: 
Error at "/a_value": field must be set to string or not be present
```

Without this discriminator hint, we get the (lengthy) error message

```text
doesn't match schema: doesn't match schema due to: 
Error at "/a_value": field must be set to string or not be present
Schema:
  {
    "type": "string"
  }

Value:
  "boolean"
 Or Error at "/a_value": field must be set to number or not be present
Schema:
  {
    "type": "number"
  }

Value:
  "boolean"

```

If your schema is more complex and has more than two sub document, that are allowed, the hinted error message is way better.

Complete code

{{< gist berend 0ce5cf2dfb513f7acca93016b74eb735 >}}


