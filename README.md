[![Build Status](https://github.com/hukkin/tomli-w/actions/workflows/tests.yaml/badge.svg?branch=master)](https://github.com/hukkin/tomli-w/actions?query=workflow%3ATests+branch%3Amaster+event%3Apush)
[![codecov.io](https://codecov.io/gh/hukkin/tomli-w/branch/master/graph/badge.svg)](https://codecov.io/gh/hukkin/tomli-w)
[![PyPI version](https://img.shields.io/pypi/v/tomli-w)](https://pypi.org/project/tomli-w)

# Tomli-W

> A lil' TOML writer

**Table of Contents** *generated with [mdformat-toc](https://github.com/hukkin/mdformat-toc)*

<!-- mdformat-toc start --slug=github --maxlevel=6 --minlevel=2 -->

- [Intro](#intro)
- [Installation](#installation)
- [Usage](#usage)
  - [Write to string](#write-to-string)
  - [Write to file](#write-to-file)
- [FAQ](#faq)
  - [Does Tomli-W sort the document?](#does-tomli-w-sort-the-document)
  - [Does Tomli-W support writing documents with comments?](#does-tomli-w-support-writing-documents-with-comments)
  - [Can I customize insignificant whitespace?](#can-i-customize-insignificant-whitespace)
  - [Why does Tomli-W not write a multi-line string if the string value contains newlines?](#why-does-tomli-w-not-write-a-multi-line-string-if-the-string-value-contains-newlines)
  - [Is Tomli-W output guaranteed to be valid TOML?](#is-tomli-w-output-guaranteed-to-be-valid-toml)

<!-- mdformat-toc end -->

## Intro<a name="intro"></a>

Tomli-W is a Python library for writing [TOML](https://toml.io).
It is a write-only counterpart to [Tomli](https://github.com/hukkin/tomli),
which is a read-only TOML parser.
Tomli-W is fully compatible with [TOML v1.0.0](https://toml.io/en/v1.0.0).

## Installation<a name="installation"></a>

```bash
pip install tomli-w
```

## Usage<a name="usage"></a>

### Write to string<a name="write-to-string"></a>

```python
import tomli_w

doc = {"table": {"nested": {}, "val3": 3}, "val2": 2, "val1": 1}
expected_toml = """\
val2 = 2
val1 = 1

[table]
val3 = 3

[table.nested]
"""
assert tomli_w.dumps(doc) == expected_toml
```

### Write to file<a name="write-to-file"></a>

```python
import tomli_w

doc = {"one": 1, "two": 2, "pi": 3}
with open("path_to_file/conf.toml", "wb") as f:
    tomli_w.dump(doc, f)
```

## FAQ<a name="faq"></a>

### Does Tomli-W sort the document?<a name="does-tomli-w-sort-the-document"></a>

No, but it respects sort order of the input data,
so one could sort the content of the `dict` (recursively) before calling `tomli_w.dumps`.

### Does Tomli-W support writing documents with comments?<a name="does-tomli-w-support-writing-documents-with-comments"></a>

No.

### Can I customize insignificant whitespace?<a name="can-i-customize-insignificant-whitespace"></a>

Indent width of array content can be configured via the `indent` keyword argument.
`indent` takes a non-negative integer, defaulting to 4.

```python
import tomli_w

doc = {"fruits": ["orange", "kiwi", "papaya"]}
expected_toml = """\
fruits = [
 "orange",
 "kiwi",
 "papaya",
]
"""
assert tomli_w.dumps(doc, indent=1) == expected_toml
```

### Why does Tomli-W not write a multi-line string if the string value contains newlines?<a name="why-does-tomli-w-not-write-a-multi-line-string-if-the-string-value-contains-newlines"></a>

This default was chosen to achieve lossless parse/write round-trips.

TOML strings can contain newlines where exact bytes matter, e.g.

```toml
s = "here's a newline\r\n"
```

TOML strings also can contain newlines where exact byte representation is not relevant, e.g.

```toml
s = """here's a newline
"""
```

A parse/write round-trip that converts the former example to the latter does not preserve the original newline byte sequence.
This is why Tomli-W avoids writing multi-line strings.

A keyword argument is provided for users who do not need newline bytes to be preserved:

```python
import tomli_w

doc = {"s": "here's a newline\r\n"}
expected_toml = '''\
s = """
here's a newline
"""
'''
assert tomli_w.dumps(doc, multiline_strings=True) == expected_toml
```

### Is Tomli-W output guaranteed to be valid TOML?<a name="is-tomli-w-output-guaranteed-to-be-valid-toml"></a>

No.
If there's a chance that your input data is bad and you need output validation,
parse the output string once with `tomli.loads`.
If the parse is successful (does not raise `tomli.TOMLDecodeError`) then the string is valid TOML.

Examples of bad input data that can lead to writing invalid TOML without an error being raised include:

- A mapping where keys behave very much like strings, but aren't. E.g. a tuple of strings of length 1.
- A mapping where a value is a subclass of a supported type, but which overrides the `__str__` method.

Given proper input (a mapping consisting of non-subclassed [types returned by Tomli](https://github.com/hukkin/tomli?tab=readme-ov-file#how-do-toml-types-map-into-python-types))
the output should be valid TOML.
