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

The minimum requirement for an alert is that it have a `warn` or `crit` expresion. However, the most common case is to define at least: `warn`, `warnNotification`, `crit`, `critNotification`, and `template`.

### Alert Keywords

#### warn
The expression to evaluate to set a warn state for an incident that is instantiated from the alert definition. 

The expression must evaluate to a NumberSet or a Scalar. 0 is false (do not trigger) and any non-zero value is true (will trigger). 

If the crit expression is true, the warn expression will not be evaluated as crit supersedes warn.

No warn notifications will be sent if `warnNotification` is not declared in the alert definition. It will still however appear on the dashboard.

#### crit
The expression to evaluate to set a critical state for an incident that is instantiated from the alert definition.

As with warn, the expression must return a Scalar or NumberSet. 

No crit notifications will be sent if `critNotification` is not declared in the alert definition. It will still appear on the dashboard.

#### critNotification
Comma-separated list of notifications to trigger on critical a state (when the crit expression is non-zero). This line may appear multiple times and duplicate notifications, which will be merged so only one of each notification is triggered. Lookup tables may be used when `lookup("table", "key")` is an entire `critNotification` value. See example below.

#### warnNotification
Identical to critNotification (TODO: Link to above), but for warning states.

#### template
The name of the template that will be used to send alerts to the specified notifications for the alert.

#### runEvery
Multiple of global system configuration value `checkFrequency` at which to run this alert. If unspecified, the global system configuration value `defaultRunEvery` will be used.

#### squelch
Squelch (TODO Link to squelch detail) is comma-separated list of `tagk=tagv` pairs. `tagv` is a regex. If the current tag group matches all values, the alert is squelched, and will not trigger as crit or warn. For example, `squelch = host=ny-web.*,tier=prod` will match any group that has at least that host and tier. Note that the group may have other tags assigned to it, but since all elements of the squelch list were met, it is considered a match. Multiple squelch lines may appear; a tag group matches if any of the squelch lines match.

This can also be defined at the global of level of the configuration. 

When using squelch, alerts will be removed even if they are not with-in the scope of the final tagset. The common case of this would be using the `t` (tranpose function) to reduce the number of final results. So when doing this, results will still be remove because they are removed at the expression level for the `warn` and `crit` expressions.

#### unknown
unknown is the time at which to mark an incident unknown (TODO: Link to details of unknown) if it can not be evaluated. It defaults the system configuration global variable `checkFrequency`.

#### ignoreUnknown
Setting `ignoreUnknown = true`, will prevent an alert from becoming unknown. This is often used where you expect the tagsets or data for an alert to be sparse and/or you want to ignore things that stop sending information. 

#### unknownIsNormal
Setting `unknownIsNormal = true` will convert unknown events for an incident into a normal event.

This is often useful if you are alerting on log messages where the absence of log messages means that the state should go back to normal. Using `ignoreUnknown` with this setting would be unnecessary.

#### unjoinedOk
If present, will ignore unjoined expression errors. Unjoins happen when expressions with in an alert use a comparison operator, and there are tagsets in one set but are not in the other set.

#### log
Setting `log = true` will make the alert behave as a "log alert". It will never show up on the dashboard, but will execute notifications every check interval where the status is abnormal.

#### maxLogFrequency
Setting `maxLogFrequency = true` will throttle log notifications to the specified duration. `maxLogFrequency = 5m` will ensure that notifications only fire once every 5 minutes for any given alert key. Only valid on alerts that have `log = true`.

## Variables
Variables are in the form of `$foo = someText` where someText continues until the end of the line. These are not variables in the sense that they hold a value, rather they are simply text replacement done by the the parsers.

## Global Variables

## Templates 
Templates are the message body for emails that are sent when an alert is triggered. Syntax is the golang [text/template](http://golang.org/pkg/text/template/) package. Variable expansion is not performed on templates because `$` is used in the template language, but a `V()` function is provided instead. Email bodies are HTML, subjects are plaintext. Macro support is currently disabled for the same reason due to implementation details.

They are pointed to in the definition of an alert (TODO: Link to alert def template keyword).

Note that templates are rendered when the expression is evaluated and it is non-normal. This is to eliminate changes in what is presented in the template because the data has changed in the tsdb since the alert was triggered.

### Template Keywords

#### body
The message body. This is formated in HTML (TODO: Unless What? I think always currently, maybe I should link to issue)

#### subject
The subject of the template. This is also the text that will be used in the dashboard for triggered incidents. 

### Template Variables

#### Ack
The value of Ack is the URL for the alert acknowledgement. This is generated using the (TODO: link to system configuration variable that sets the hostname).

#### Expr 
The value of Expr is the warn or crit expression that was used to evaluate the alert in the format of a string

#### Group
A map of tags and their corresponding values for the alert. (TODO: Add Example)

#### History
The value is an array of events. An Event has a `Status` field (an integer) with a textual string representation; and a `Time` field. Most recent last. The status fields have identification methods: `IsNormal()`, `IsWarning()`, `IsCritical()`, `IsUnknown()`, `IsError()`.

(TODO: Example)

(TODO: Document Event Type)

#### Incident 
The value a URL to the incident view for the incident. The url is based on (TODO: link to systemconf setting )

#### IsEmail
The value is true if template is being rendered for an email. Needed because email clients often modify HTML.

(TODO: Example)

#### Last
The last Event in the `History` array. (TODO: Link to Event type)

#### Subject
A string representation of the (TODO: Verify Rendered?) subject. 

#### Touched
The time this alert was last updated

(TODO: link to time object in Go documentation)

#### Alert
Dictionary of rule data (TODO: Document the object properly). Current values don't have documentation

### Alert Functions

#### Eval(string|Expression|ResultSlice) (resultValue, error)

(TODO: Understand the Resultslice argument. I think it might be so that Eval can be passed as arguments to other eval (TODO: include an example of this))

Executes the given expression and returns the first result with identical tags to the alert instance. In other words, it evaluates the alert within the context of the alert.


If the result is empty, than `NaN` is returned. Otherwise the value of the first matching result is returned. That result can be any type of value that Bosun can return (TODO: link to a document that shows the types that can be returned, and their structure as they appear to templates)

(TODO: Example)

#### EvalAll(string|Expression|ResultSlice) (result, error)
(TODO: For the above, going to need to explain the way a result differs from the resultValue (`result[0].Value` technically))





## Notifications

## Lookups 




{% endraw %}

</div>
</div>