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
All definitions are in a single file that is pointed to by [the system configuration's RuleFilePath](/system_configuration#rulefilepath). The file is UTF-8 encoded.

### Syntax
Syntax is sectional, with each section having a type and a name, followed by `{` and ending with `}`. Each section is a definition (for example, and alert definition or a notification definition). Key/value pairs follow of the form `key = value`. Key names are non-whitespace characters before the `=`. The value goes until end of line and is a string. Multi-line strings are supported using backticks (\`) to delimit start and end of string. Comments go from a `#` to end of line (unless the `#` appears in a backtick string). Whitespace is trimmed at ends of values and keys.

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

The expression must evaluate to a NumberSet or a Scalar (See [Data Types](expressions#data-types)). 0 is false (do not trigger) and any non-zero value is true (will trigger). 

If the crit expression is true, the warn expression will not be evaluated as crit supersedes warn.

No warn notifications will be sent if `warnNotification` is not declared in the alert definition. It will still however appear on the dashboard.

#### crit
The expression to evaluate to set a critical state for an incident that is instantiated from the alert definition.

As with warn, the expression must return a Scalar or NumberSet. 

No crit notifications will be sent if `critNotification` is not declared in the alert definition. It will still appear on the dashboard.

#### critNotification
Comma-separated list of notifications to trigger on critical a state (when the crit expression is non-zero). This line may appear multiple times and duplicate notifications, which will be merged so only one of each notification is triggered. Lookup tables may be used when `lookup("table", "key")` is an entire `critNotification` value. (TODO: Link to notification lookups).

#### warnNotification
Identical to `critNotification` above, but the condition evaluates to warning state.

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

They can be referenced by `$foo` or by `${foo}`, the later being useful if you want to use the variable in a context where whitespace does not immedialty follow the value.

### Global Variables

Global Variables exist outside of any section and should be defined before they are used.

Global variables can be overridden in sections defining a variable within the scope of the section that has the same name.

## Templates 
Templates are used to construct what alerts will look like when they are sent. They are pointed to in the defintion of an alert by the [template keyword](/definitions#template). They are like "views" in web frameworks.

Templates in Bosun are built on top of go's templating. The subject is rendered using the golang [text/template](http://golang.org/pkg/text/template/) package and the body is rendered using the golang [html/template](https://golang.org/pkg/html/template/) package.

Variable expansion is not performed on templates because `$` is used in the template language, but a `V()` function is provided instead. Email bodies are HTML, subjects are plaintext.

Macros can not be used in templates, however, templates can reference subtemplates. (TODO: Example of this).

Note that templates are rendered when the expression is evaluated and it is non-normal. This is to eliminate changes in what is presented in the template because the data has changed in the tsdb since the alert was triggered.

### Template Keywords

#### body
The message body. This is formated in HTML (TODO: Unless What? I think always currently, maybe I should link to issue)

#### subject
The subject of the template. This is also the text that will be used in the dashboard for triggered incidents. The format of the subject is plaintext.

### Template Variables
Template variables hold information specific to the instance of an alert. They are bound to the template's root context. That means that when you reference them in a block they need to be referenced differently just like context bound functions (TODO: link to function section explaining this).

#### Ack
The value of Ack is the URL for the alert acknowledgement. This is generated using the [system configuration's Hostname](/system_configuration#hostname) value as the root of the link.

#### Expr 
The value of `Expr` is the warn or crit expression that was used to evaluate the alert in the format of a string

#### Group
A map of tags and their corresponding values for the alert. (TODO: Add Example)

#### Events
The value of `Events` is a slice of [Event](http://localhost:4000/definitions#event) objects.

Example:  

~~~
template test {
    body = `
    <table>
    
        <tr>
            <th>Time</th>
            <th>Status</th>
            <th>Warn Value</th>
            <th>Crit Value</th>
        </tr>
        
        {{ range $event := .Events}}
            <tr>
                <td>{{ $event.Time }}</td>
                <td>{{ $event.Status }}</td>
                <!-- Ensure Warn or Crit are not nil since they are pointers -->
                <td>{{ if $event.Warn }} {{$event.Warn.Value }} {{ else }} {{ "none" }} {{ end }}</td>
                <td>{{ if $event.Crit }} {{$event.Crit.Value }} {{ else }} {{ "none" }} {{ end }}</td>
            </tr>
        {{ end }}
    </table>
    `
}
~~~


#### Incident 
The value a URL to the incident view for the incident. The url is based on (TODO: link to systemconf setting )

#### IsEmail
The value of is `IsEmail` is true if the template is being rendered for an email. This allows you to use the same template for different types of notifications conditionally within the template.

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

#### Errors
A slice of strings that gets appended to when a context bound function (TODO: Link to section that explains this) returns an error. 

### Template Function Types
Template functions come in two types. Functions that are global, and context-bound functions. Unfortunately, it is important to understand the difference because they are called differently in functions and have different behavior in regards to error handling (TODO: link to error handling section)

#### Global
Calling global functions is simple. The syntax is just the function name and arguments. I can be used in regular format or a chained pipe format.

~~~
template global_type_example {
    body = `
        <!-- Regular Format -->
        {{ bytes 12312313 }}
        <!-- Pipe Format -->
        {{ 12312313 | bytes }}
    `
}
~~~

#### Context Bound
Context bound functions, like Global functions, can be called in regular or pipe format. What makes them different is the syntax used to call them. Context bound functions have a period before them, such as `{{ .Eval }}`. Essentially they are methods on that act on the instance of an alert/template combination when it is rendered and perform queries.

They are bound to the parent context. The "parent context" is essentially the top level namespace within a template. This is because they access data associated with the instance of the template when it sent. 

What this practically means, is that when you are inside a block within a template (for example, inside a range look) context-bound functions need to be called with `{{ $.Func }}` to reference the parent context.

~~~
template context_bound_type_example {
    body = `
        <!-- Context Bound at top level -->
        {{ .Eval .Alert.Vars.avg_cpu }}
        
        <!-- Context Bound in Block -->
        {{ range $x := .Events }}
            {{ $.Eval $.Alert.Vars.avg_cpu }}
        {{ end }}
    `
}
~~~

### Template Error handling
Templates can throw errors at runtime (i.e. when a notification is sent). Although the configuration check makes sure that templates are valid, you can still do things like try to dereference objects that are nil pointers.

When a template fails to render, a generic notification will be emailed to the people that would have recieved the alert. (TODO: Look up what happens for non-email alerts in this case and explain it there.)

In order to prevent the template from completelying failing and resulting in the generic notification, errors can be handled inside the application. 

Errors are handled differently depending on the type of the function (Context Bound vs Global). When context bound functions have errors the `.Errors` variable attached to the template context is appended to. This is not the case for global functions. 

Global functions always returns strings except for parseDuration. When global functions error than `.Errors` is *not* appended to, but the string that would have been returned with an error is show in the template. parseDuration returns nil when it errors, and in this one exception you can't see what the error is.

If the func returns a string or an image (technically an interface) the error message will be displayed in the template. If an object is returned (i.e. a result sets, a slice, etc) nil is returned and the user can check for that in the template. In both cases `.Errors` will be appended to if it is a context bound functions.

(TODO: Example of template using error handling.)

### Template Functions
(TODO: Important, update the template function signatures not have two return values, since that isn't how go templates work and the function signatures changed in https://github.com/bosun-monitor/bosun/pull/1950).

(TODO: Make sure all functions specifiy built-in or context bound)

(TODO: Sort these when done)

#### Eval(string|Expression|ResultSlice) (resultValue)

Type: Context-Bound

(TODO: Understand the Resultslice argument. I think it might be so that Eval can be passed as arguments to other eval (TODO: include an example of this))

Executes the given expression and returns the first result that includes the tag key/value pairs of the alert instance. In other words, it evaluates the expression within the context of the alert. So if you have an alert that could trigger multiple incidents (i.e. `host=*`) then the expression will return data specific to the host for this alert.

If the expression results in an error nil will be returned and `.Errors` will be appended to. If the result set is empty, than `NaN` is returned. Otherwise the value of the first matching result in the set is returned. That result can be any type of value that Bosun can return (TODO: link to a document that shows the types that can be returned, and their structure as they appear to templates) since the type returned by Eval is dependent on what the return type of the expression it evaluates. Mostly commonly it will be float64 when used to evaluate an expression that is enclosed in a reduction like in the following example:

```
alert eval {
    template = eval
    $series = merge(series("host=a", 0, .2), series("host=b", 0, .5))
    $r = avg($series)
    crit = $r
}

template eval {
    body = `
    {{$v := .Eval .Alert.Vars.r }}
    <!-- If $v is not nil (which is what .Eval returns on errors) -->
    {{ if notNil $v }}
        {{ $v }}
    {{ else }}
        {{ .LastError }}
    {{ end }}
`
}
```

The above would display "0.2" for host "a". More simply, you template could just be `{{.Eval .Alert.Vars.r}}` and it would display 0.2 assuming there are no errors.

#### EvalAll(string|Expression|ResultSlice) (result)
(TODO: For the above, going to need to explain the way a `result` differs from the `resultValue` (`result[0].Value` technically))

Execute the given expression and returns the result.

(Todo: Document error returns by linking to the error behavior of eval as I believe they are the same)

(TODO: Need to link to possible return types and their structure)

#### LeftJoin(expression|string|ResultSlice?...) ([][]Result)

Type: Context-Bound

`LeftJoin` allows you to construct tables from the results of multiple expressions. `LeftJoin` takes two or more expressions that return numberSets as arguments. The function evaluates each expression. It then joins the results of other expressions to the first expression. The join is based on the tag sets of the results. If the tagset is a subset or equal the results of the first expression, it will be joined. 

The output can be though of as a table that is structured as an array of rows, where each row is an array. More technically it is a slice of slices that point to [Result](/definitions#result) objects where each result will be a numberSet type.

If the expression results in an error nil will be returned and `.Errors` will be appended to.

Example:

```
alert leftjoin {
    template = leftjoin
    # Host Based
    $osDisksMinPercentFree = last(q("min:os.disk.fs.percent_free{host=*}", "5m", ""))
    
    # Host and Disk Based
    $osDiskPercentFree = last(q("sum:os.disk.fs.percent_free{disk=*,host=*}", "5m", ""))
    $osDiskUsed = last(q("sum:os.disk.fs.space_used{disk=*,host=*}", "5m", ""))
    $osDiskTotal = last(q("sum:os.disk.fs.space_total{disk=*,host=*}", "5m", ""))
    $osDiskFree = $osDiskTotal - $osDiskUsed
    
    #Host Based Alert
    warn = $osDiskPercentFree > 5
}

template leftjoin {
    body = `    
    <h3>Disk Space Utilization</h3>
    {{ $joinResult := .LeftJoin .Alert.Vars.osDiskPercentFree .Alert.Vars.osDiskUsed .Alert.Vars.osDiskTotal .Alert.Vars.osDiskFree }}
    <!-- $joinResult will be nill if there is an error from .LeftJoin -->
    {{ if notNil $joinResult }}    
        <table>
        <tr>
            <th>Mountpoint</th>
            <th>Percent Free</th>
            <th>Space Used</th>
            <th>Space Free</th>
            <th>Space Total</th>
        </tr>
        <!-- loop over each row of the result. In this case, each host/disk -->
        {{ range $x := $joinResult }}
            <!-- Each column in the row is the results in the same order as they 
            were passed to .LeftJoin. The index function is built-in to Go's template
            language and gets the nth element of a slice (in this case, each column of 
            the row -->
            {{ $pf :=  index $x 0}}
            {{ $du :=  index $x 1}}
            {{ $dt :=  index $x 2}}
            {{ $df :=  index $x 3}}
            <!-- .LeftJoin is like EvalAll and GraphAll in that it does not filter
            results to the tags of the alert instance, but rather returns all results.
            So we compare the result's host to that of the host for the alert to only
            show disks related to the host that the alert is about. -->
            {{ if eq $pf.Group.host $.Group.host }}
                <tr>
                    <td>{{$pf.Group.disk}}</td>
                    <td>{{$pf.Value | pct}}</td>
                    <td>{{$du.Value | bytes }}</td>
                    <td>{{$df.Value | bytes }}</td>
                    <td>{{$dt.Value | bytes}}</td>
                </tr>
            {{end}}
        {{end}}
    {{ else }}
        Error Creating Table: {{ .LastError }}
    {{end}}`
}
```

#### Ack() string

#### Graph(string|Expression|ResultSlice(TODO: Not sure this can take a result slice?))
Creates a graph of the expression. It will error if the return type of the expression is not a `seriesSet`. 

(TODO: Document auto down sampling behavior)

(TODO: Document SVG vs PNG depending on interface vs email)

#### notNil(value) bool

Type: Global

notNil returns true if the value is nil. This is only meant to be used with error checking on Context-Bound functions 

#### .LastError

Type: Context-Bound

Returns the string representation of the last Error, or an empty string if there are no errors. 

### Types available in Templates
Since templating is based on Go's template language, certain types will be returned. Understanding these types can help you construct richer alert notifications.

#### Event
An Event represent a change in the [severity state](/usage#severity-states) within the [duration of an incident](/usage#the-lifetime-of-an-incident). When an incident triggers, it will have at least one event.  An Event contains the following fields

 * **Warn**: A pointer to an [Event Result](definitions#event-result) that the warn expression generated if the event has a warning status.
 * **Crit**: A pointer to an [Event Result](definitions#event-result) if the event has a critical status.
 * **Status**: An integer representing the current severity (normal, warning, critical, unknown). As long as it is printed as a string, one will get the textual representation. The status field has identification methods: `IsNormal()`, `IsWarning()`, `IsCritical()`, `IsUnknown()`, `IsError()` which return a boolean.
 * **Time**: A [Go time.Time object](https://golang.org/pkg/time/#Time) representing the time of the event. All the methods you find in Go's documentation attached to time.Time are available in the template
 * **Unevaluated**: A boolean value if the alert was unevaluated. Alerts on unevaluated when the current was using `depends` to depend on another alert, and that other alert was non-normal. 

It is important to note that the `Warn` and `Crit` fields are pointers. So if there was no `Warn` result and you attempt to access a property of `Warn` then you would get a template error at runtime. Therefore when referecing any fields of `Crit` or `Warn` such as `.Crit.Value`, it is vital that you ensure the `Warn` or `Crit` property of the Event is not a nil pointer first.

 See the example under the [Events template variable](/definitions#events) to see how to use events inside a template.

#### Event Result 
An Event Result (note: in the code this is actually a models.Result) has two properties:

* **Expr**: A string representation of the full expression used to generate the value of the Result's Value.
* **Value**: A float representing the result of the parent expression.

There is a third property **Computations**. But it is not recommended that you access it even though it is available and it will not be documented.

#### ResultSlice
A `ResultSlice` is returned by using the `.EvalAll` function. It is a slice of pointers to `Result` objects. Each result represents the an item in the set when the type is something like a NumberSet or a SeriesSet.

#### Result
A `Result` as two fields:

 1. **Group**: The Group is the TagSet of the result. 
 2. **Value**: The Value of the Result

A tagset is a map of of string to string (`map[string]string` in Go). The keys represent the tag key and their corresponding values are the tag values.

The Value can be of different types. Technically, it is a go `interface{}` with two methods, `Type()` which returns the type of the `Value()` which returns an `interface{}` as well, but will be of the type returned by the `Type()` method.

The most common case of dealing with Results in a ResultSlice is to use the `.EvalAll` func on an expression that would return a NumberSet. The following example shows how you would do that to construct a table.

(TODO: Update this example with error handling)

~~~
alert example_result_slice {
    template = example_result_slice
    # the type returned by the avg() func is a numberSet
    $cpu_avg = avg(q("sum:rate{counter,,1}:os.cpu{host=ny-bosun*}", "5m", ""))
    warn = 1
}

template example_result_slice {
    body = `
    <table>
        <tr>
            <th>Host</th>
            <th>Avg CPU Value</th>
        <tr>
        {{ range $result := (.EvalAll .Alert.Vars.cpu_avg) }}
            <tr>
                <!-- Show the value of the host tag key -->
                <td>{{$result.Group.host}}</td>
                <!-- Since we know the value of cpu_avg is a numberSet which contains floats.
                we pipe the float to printf to make it so it only print to two decimal places -->
                <td>{{$result.Value | printf "%.2f"}}</td>
            </tr>
        {{end}}
    <table>
    `
}
~~~



## Notifications

## Lookups 




{% endraw %}

</div>
</div>