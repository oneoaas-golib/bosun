---
layout: default
title: Definitions
order: 3
---

<div class="row">
<div class="col-sm-3" >
  <div class="sidebar" data-spy="affix" data-offset-top="0" data-offset-bottom="0" markdown="1">
 
 * Some TOC
 {:toc}
 
  </div>
</div>

<div class="doc-body col-sm-9" markdown="1">

<p class="title h1">{{page.title}}</p>

{% raw %}

## Changes Since 0.5.0
This configuration has been split into two different files. This page documents the various bosun sections thats can be defined. For example alerts, templates, notifications, macros, global variables, and lookups.

## Definition Configuration File
All definitions are in a single file that is pointed to in the system configuration (TODO: Make link here to that def). The file is UTF-8 encoded.

### Syntax
Syntax is sectional, with each section having a type and a name, followed by `{` and ending with `}`. Each section is a definition (for example, and alert definition or a notification definition. Key/value pairs follow of the form `key = value`. Key names are non-whitespace characters before the `=`. The value goes until end of line and is a string. Multi-line strings are supported using backticks to delimit start and end of string. Comments go from a `#` to end of line (unless the `#` appears in a backtick string). Whitespace is trimmed at ends of values and keys.

## Alert Definitions
An alert is defined with the following syntax:

```
alert uniqueAlertName {
    variable = value
    ...
    keyword = value
    ...
}
```

## Variables
Variables are in the form of `$foo = someText` where someText continues until the end of the line. These are not variables in the sense that they hold a value, rather they are simply text replacement done by the the parsers.



{% endraw %}

</div>
</div>