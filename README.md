md_proc
=======

[![Build Status](https://travis-ci.org/dikmax/md_proc.svg?branch=master)](https://travis-ci.org/dikmax/md_proc)
[![Coverage Status](https://coveralls.io/repos/dikmax/md_proc/badge.svg?branch=master)](https://coveralls.io/r/dikmax/md_proc?branch=master)
[![Pub](https://img.shields.io/pub/v/md_proc.svg)](https://pub.dartlang.org/packages/md_proc)
[![CommonMark spec](https://img.shields.io/badge/commonmark-0.22-green.svg)](http://spec.commonmark.org/)

[CommonMark]-compliant Markdown parser.

Basic usage:

```dart
print(markdownToHtml('# Hello world!'));
```

Main difference from other parsers is Markdown-based AST. You can parse Markdown, process resulting AST and write
results back to markdown.

Project main goal is create processing library for Markdown.

Parsing
-------

```dart
import "package:md_proc/md_proc.dart";

void main() {
  Document doc = CommonMarkParser.defaults.parse('Hello world!\n===');
  print(doc); // Document [SetextHeader 1 [Str "Hello", Space, Str "world", Str "!"]]
}
```

Writing html
------------

```dart
import "package:md_proc/md_proc.dart";

void main() {
  Document doc = CommonMarkParser.defaults.parse('Hello world!\n===');
  String res = HtmlWriter.defaults.write(doc);
  print(res); // <h1>Hello world!</h1>
}
```

Writing markdown
----------------

```dart
import "package:md_proc/md_proc.dart";

void main() {
  Document doc = CommonMarkParser.defaults.parse('Hello world!\n===');
  String res = MarkdownWriter.defaults.write(doc);
  print(res); // Hello world!
              // ============
}
```

Extensions
==========

md_proc supports some language extensions. You can specify enabled extensions using options parameter in parser and 
renderer.

```dart
Options options = new Options(superscript: true);
CommonMarkParser parser = new CommonMarkParser(options);
Document doc = parser.parse('Hello world!\n===');
HtmlWriter writer = new HtmlWriter(options);
String res = writer.write(doc);
```

There three predefined versions of parsers/writers:

- `strict`: all extensions are disabled
- `commonmark`: enabled only `smartPunctuation` extension.
- `defaults`: all extensions (`smartPunctuation`, `strikeout`, `subscript`, `superscript`) are enabled.

To get correspondent parser/writer instance use static getter on class:

```dart
CommonMarkParser defaultParser = CommonMarkParser.defaults;
HtmlWriter strictWriter = HtmlWriter.strict;
```

Smart punctuation
-----------------

Smart punctuation is automatic replacement of `...`, `---`, `--`, `"` and `'` to "???", "???", "???" and curly versions of
quote marks accordingly. It's only official extension to date.

**NOTE:** This extension uses Unicode chars. Make sure that your code support it.

Strikeout
---------

Strikeouts text (~~like this~~). Just wrap text with double tildes (`~~`).

```md
Strikeouts text (~~like this~~).
```

Subscript
---------

Support for subscript (H<sub>2</sub>O). Wrap text with tildes (`~`).

```md
H~2~O
```

Subscript couldn't contain spaces. If you need to insert space into subscript, escape space (`\ `).

```md
subscript~with\ spaces~
```

Superscript
-----------

Support for superscript (2<sup>2</sup>=4). Wrap text with caret (`^`).

```md
2^2^=4
```

Superscript couldn't contain spaces. If you need to insert space into superscript, escape space (`\ `).

```md
superscript^with\ spaces^
```

Custom reference resolver
-------------------------

Custom reference resolver may be required when parsing document without implicit defined references, for example 
Dartdoc.

```dart
/**
 * Throws a [StateError] if ...
 * similar to [anotherMethod], but ...
 */
```

In that case you could supply parser with resolver, which should provide all missing links.
  
```dart
String library = "md_proc";
String version = "0.4.0";
Target linkResolver(String normalizedReference, String reference) {
  if (reference.startsWith("new ")) {
    String className = reference.substring(4);
    return new Target("http://www.dartdocs.org/documentation/$library/$version/index.html#$library/$library.$className@id_$className-", null);
  } else {
    return null;
  }
}

CommonMarkParser parser = new CommonMarkParser(new Options(linkResolver: linkResolver));
Document doc = parser.parse('Hello world!\n===');
String res = HtmlWriter.defaults.write(doc);
```

High-level plan for development
===============================

1. Follow CommonMark specification changes.
2. Add MarkDown-extensions from [pandoc], then change them to CommonMark extensions when they will be finally developed
and accepted. (inline math, footnotes, etc.)
3. AST-processing classes. Don't have much time to think about this. But this is definitely required.

[CommonMark]: http://commonmark.org/
[pandoc]: http://johnmacfarlane.net/pandoc/
