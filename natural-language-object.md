# **NOBJ** (Natural Object) - Natural Language Object Notation

## Introduction

`NOBJ` aims to provide a structured data format that is closer to natural language while retaining the simplicity and ease of parsing of JSON. The goal is to allow LLMs to output higher-quality results that can be directly parsed by programs without the need for secondary conversion to JSON.

### Advantages:

- **Readability:** Uses words and structures closer to everyday language, making it easier for LLMs to read and understand.
- **Maintainability:** More intuitive design, making data structures easier to understand and modify.
- **Cross-Language Usability:** Natural language elements can more easily bridge language barriers for better understanding.

### Disadvantages:

Current limitations:

- Supports only flat simple objects or arrays.
- Lacks hierarchical structure, unable to represent nested objects or arrays.
- Does not support optional fields.
- Requires prompt engineering to ensure LLM outputs in the correct order and format.

## Examples

**NOBJ Representation of a Simple Object**:

```yaml
name: John Doe
age: 30
city: New York
# Array values
tags: developer, engineer
```

**NOBJ Representation of a Simple Array**:

```yaml
name: John Doe
age: 30
city: New York

name: Mike Doe
age: 20
city: New York
```

**Notes**:

* `NOBJ` is still under development, and specific syntax rules and parsing methods will be further refined in the future.
* String values do not include newline characters.
* Array values are comma-separated.
* If a `key` contains spaces, the spaces in the returned JSON object's `key` will be replaced with underscores (`_`).
