# Lexical Structure

The lexical structure of Inkling includes these lexical elements:

* [`keyword`][1]: words that Inkling reserves for its own use
* [`operator`][2]: mathematical, logical, and grouping operators
* [`identifier`][3]: user-defined names in Inkling
* [`literal`][4]: numeric and string literal values
* [`comment`][5]: user-specified comments


## Keywords

Keywords are reserved words in the grammar. They all consist of lowercase ASCII strings like `concept`, `input`, and `type`.

## Operators

Operators are symbols that represent mathematical, logical or grouping operations like +, %, (), or [].

**Operators Table**

            |             |            |            |            |            |           
----------- | ----------- | ---------- | ---------- | ---------- | ---------- | ---------- 
<= | < | == | != | >= | >
{ | } | [ | ] | ( | )
: | + | - | / | % | ,
** | * | . | ..


## Identifiers

```inkling2--code
    velocity
    _control_axis_1
    Rotate90Degrees
    `Search & Rescue`
    `input`
    `1 Microsoft Way`
    `\`A \\ B\``
```

Identifiers are user-defined names of an object, such as a concept or a type. Identifiers start with an alphabetical ASCII character or underscore and contain only alphanumeric characters and underscores.

If an identifier doesn't follow these rules (e.g. contains spaces or non-ASCII characters, starts with a number) or is a keyword, it must be surrounded by backticks. Backslash can be used as an escape for back quotes and backslashes.

The use of capitalized identifiers is encouraged to avoid possible collisions with future keywords, which are guaranteed to be lowercase.

## Literals

Inkling supports string and numeric literals (floating point and integer). 

* **String Literals**

```inkling2--code
    "Foo"
    "My \"String\""
    ""
```

String literals are surrounded by double quote characters. A backslash is used as an escape character.

* **Integer Literals**

```inkling2--code
    +99
    -45
    9999
```

Integer literals are a string of digits with an optional sign and no decimal point.

* **Floating Point Literals**

```inkling2--code
    12.0
    .5
    13.0f7
    .3f+2
```

Floating-point literal values use decimal points and optional exponents.

* **Array literals**

```inkling2--code
    [0, 2, 37]
```

Array literals are surrounded by square brackets and contain a comma-separated list of values. Each value can be specified as an expression. All elements within an array literal must be the same type. Empty (zero-length) array literals are not supported.

* **Structure literals**

```inkling2--code
    { X: 2, Y: 3, Name: "Bob" }
```

Struct literals are surrounded by curly brackets and contain a comma-separated list of field name/value pairs separated by a colon. The value can be specified as an expression and can even be another struct literal.

## Comments

```inkling2--code
  # This is a comment
```

* An Inkling comment begins after the character **#**. Comments must be on a line by themselves.

# Inkling Version

```inkling2--code
    inkling "2.0"
```

All Inkling programs must start with an `inkling` statement that specifies the current language version. This allows the language to evolve without breaking existing programs.



[1]: #keywords
[2]: #identifiers
[3]: #literals
[4]: #operators
[5]: #comments

