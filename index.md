---
title: A JSON Schema for JSON Pointer and Relative JSON Pointer
---

# A JSON Schema Vocabulary for JSON Pointer and Relative JSON Pointer

_[**WARNING:** This project is in development and not yet suitable for
anything but demonstrating the uses of vocabularies._

_It is not necessarily intended for standardization, although all are welcome
to use it, including as the basis for standardization work.]_

## Introduction

This vocabulary is an alternative to `"format": "json-pointer"` and
`"format": "relative-json-pointer"`
with more targeted assertions and annotations.

All assertions apply only to string instances, and all annotations
are only meaningful against string instances that could pass
at least one assertion from this vocabulary.  However, there is
no requirement that an assertion from this vocabulary be present
in order to use the annotations.

The URI for this vocabulary is:

[`https://handrews.github.io/jsonpointer-jsonschema-vocabulary`](https://handrews.github.io/jsonpointer-jsonschema-vocabulary)

The URI for the vocabulary's meta-schema is:

[`https://handrews.github.io/jsonpointer-jsonschema-vocabulary/schema`](https://handrews.github.io/jsonpointer-jsonschema-vocabulary/schema)

Regrettably, due to limitations of GitHub pages, the meta-schema will not have
the proper Content-Type setting.  For proper content type, [add ".json" to the URI](https://handrews.github.io/jsonpointer-jsonschema-vocabulary/schema.json).

## References

* JSON Pointer is defined by [RFC 6901](https://datatracker.ietf.org/doc/html/rfc6901)
* Relative JSON Pointer is an [Internet Draft](https://datatracker.ietf.org/doc/html/draft-bhutton-relative-json-pointer-00)
* Capitalized requirement words such as MUST are to be interpreted
  in this document according to [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119)

In this document, RFC 6901 JSON Pointers are often referred to as
"absolute JSON Pointers" to distinguish them from Relative JSON Pointers.

## Assertion Keywords

All assertions in this vocabulary apply only to strings.  Any
other JSON type MUST be considered valid against these assertions.

### `jsonPointer`

The value of `jsonPointer` MUST be a string, and MUST
be either `"absolute"`, `"relative`", or `"any"`.

JSON Pointer and Relative JSON Pointer are closely related
yet distinct specifications, with mutually exclusive syntax
requirements.

If the instance is a Relative JSON Pointer, this prefix
applies only to the pointer section, without regard
to the initial numeric section.  A Relative JSON Pointer
that ends with `#` rather than a JSON Pointer is not
valid against this keyword.

### Relative JSON Pointer assertions

The following assertions are provided to simplify
and clarify the intent of constraints that would
otherwise be handled through regular expressions.
In particular, a numeric range is often awkward
to express as a regular expression.

These keywords function independently from each
other and from `"jsonPointer"`.
String instances that are not syntactically valid
Relative JSON Pointers MUST be considered valid
against these assertions.  This allows them to be
used effectively with `"jsonPointer": "any"`,
and with values that may or may not be JSON Pointers
at all.

### `relJsonPointerMinUp`

The value of this keyword MUST be a non-negative integer.

An instance that is a valid Relative JSON Pointer is
valid against this keyword if its initial numeric
component is mathematically greater than or equal
to this value.

### `relJsonPointerMaxUp`

The value of this keyword MUST be a non-negative integer.

An instance that is a valid Relative JSON Pointer is
valid against this keyword if its initial numeric
component is mathematically less than or equal
to this value.

### `relJsonPointerMinOver`

The value of this keyword MUST be an integer.

An instance that is a valid Relative JSON Pointer
is valid against this keyword if an index adjustment
follows the initial numeric component, and if the
adjustment (evaluated as an integer using the `+` or `-`
sign present) is greater than or equal to this value.

If an adjustment of 0 is allowed by this keyword,
then valid Relative JSON Pointers that do not have
an explicit index adjustment also MUST be considered
valid against this keyword.

### `relJsonPointerMaxOver`

The value of this keyword MUST be an integer.

An instance that is a valid Relative JSON Pointer
is valid against this keyword if an index adjustment
follows the initial numeric component, and if the
adjustment (evaluated as an integer using the `+` or `-`
sign present) is less than or equal to this value.

If an adjustment of 0 is allowed by this keyword,
then valid Relative JSON Pointers that do not have
an explicit index adjustment also MUST be considered
valid against this keyword.

### `relJsonPointerGetNameOrIndex`

The value of this keyword MUST be a boolean.

If the value is `true`, an instance that is a valid
Relative JSON Pointer is valid against this keyword
if it consists only of the initial numeric component,
an optional index adjustment, and the `#` character,
indicating that it evaluates to the name of the
property or index of the array location identified
by the pointer up to the `#` character.

If the value is `false`, an instance that is a valid
Relative JSON Pointer is valid against this keyword
if the initial numeric component and optional
index adjustment are not followed by the `#` character,
meaning that there are either no further characters
or the remainder of the string is a valid JSON Pointer.

## Annotation keywords

### `jsonPointerTarget`

The value of this keyword MUST be a string, which is
returned as the annotation value for the keyword.
There are no restrictions on string value.

The semantics of the value for this keyword are
application-defined.


## Examples

### Constraining a relative pointer

This constrains the relative pointer to only look at previous
entries (or children of previous entries) in the array containing
the starting point:

```JSON
{
    "type": "string",
    "jsonPointer": "relative",
    "relJsonPointerUpMax": 0,
    "relJsonPointerOverMax": -1,
    "relJsonPointerGetNameOrIndex": false
}
```

Note that this sets a maximum (not minimum) "over" value of -1.

Valid instances:

* `"0-1/foo"`
* `"0-2/bar/12/whatever#"`
* `"0-100"`

Invalid instances:

* `"0-1#`
* `"0+1`
* `0/foo`

Recall that `#` does _not_ have any special meaning at the end of a Relative JSON Pointer that includes a JSON Pointer component, only when it immediately follows the up (and optional over) numbers.  This is why the second valid instance is valid, while the first invalid instance is not.

### JSON Schema's output schema

JSON Schema's output format makes use of JSON Pointers both when referencing schema keywords (specifically the dynamic scope during schema evaluation), and when referencing the instance.

Here is an excerpt of those parts of the output schema, from the 2020-12 draft:

```JSON
"$defs": {
  "outputUnit":{
    "properties": {
      "keywordLocation": {
        "type": "string",
        "format": "json-pointer"
      },    
      "absoluteKeywordLocation": {
        "type": "string",
        "format": "uri" 
      },    
      "instanceLocation": {
        "type": "string",
        "format": "json-pointer"
      }
    }
  }
}
```

Here is what it could look like using this vocabulary:

```JSON
"$defs": {
  "outputUnit":{
    "properties": {
      "keywordLocation": {
        "type": "string",
        "jsonPointer": "absolute",
        "jsonPointerTarget": "schema dynamic scope"
      },    
      "absoluteKeywordLocation": {
        "type": "string",
        "format": "uri"
      },    
      "instanceLocation": {
        "type": "string",
        "jsonPointer": "absolute",
        "jsonPointerTarget": "instance"
      }
    }
  }
}
```

In addition to replacing the unreliable `"format"` keyword with `"jsonPointer"`, this uses `"jsonPointerTarget"` to clarify the usage of the pointers.  While it's probably pretty obvious that `"instanceLocation"` applies to the instance, the meaning of `"keywordLocation"` is less clear.

Notably, while `"absoluteKeywordLocation"` also uses JSON Pointer, it does so in the form of a URI fragment.  Therefore the JSON Pointer vocabulary is not relevant, and even if it were, it would be redundant as only absolute JSON Pointers can be used in URIs, and the target of the pointer is by definition the resource identified by the non-fragment part of the URI.

Of course, the actual JSON Schema specification makes the targets of all of these perfectly clear, but it is convenient as extra documentation, and in some applications an implementation could do automatic processing of pointers based on this sort of use of `"jsonSchemaTarget"`.


### Hyper-Schema's Meta-Schema

Let's look at the meta-schema for JSON Hyper-Schema's
`"anchorPointer"` property within the link object (it is not
necessary to understand Hyper-Schema to understand this example).

Here is the meta-schema for that property as it appeared in `links.json`
starting with draft-07:

```JSON
{
    "type": "string",
    "anyOf": [
        { "format": "json-pointer" },
        { "format": "relative-json-pointer" }
    ]
}
```

It uses both JSON Pointer-related `format` values, which are not
reliably implemented across all validators, combined with an `"anyOf"`,
a general-purpose keyword that can be somewhat challenging to support
for applications like documentation or code generation.  It's also
a bit odd as either `"anyOf"` or `"oneOf"` could work, as no string
is both an absolute JSON Pointer and a Relative JSON Pointer.

There is also no indication of how the pointer is used.  The text of
the specification explains that it is applied to the instance
(meaning the instance to which a hyper-schema is applied) in order
to generate part of the Hyper-Schema output.

Using the keywords in this vocabulary, we get something a little
simpler and a little more clear:

```JSON
{
    "type": "string",
    "jsonPointer": "any",
    "jsonPointerTarget": "instance"
}
```

Of course this still requires a documentation or code generator to
support both absolute and relative pointers, but the semantics are
directly explicit, and a documentation generator in particular can
make use of that to produce more clear documentation.

This example also exposes the challenge of specifying values for
`"jsonPointerTarget"`, in that `"instance"` here refers to the instance
being evaluated by the schema to which this meta-schema applies, while
in the previous example `"instance"` referred to the instance that
was evaluated in order to produce the output.

In neither case does `"instance"` refer to the direct instance to
which the schema containing the `"jsonPointerTarget"` is applied!
That would be the schema (in the case of the meta-schema) or the
output itself (in the case of the output schema).  So here we have
at least three reasonable meanings of the word `"instance"`, which
is why the semantics of `"jsonPointerTarget"` values are
purely application-defined.


