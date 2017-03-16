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
    $variable = value
    ...
    keyword = value
    ...
}
```

The minimum requirement for an alert is that it have a `warn` or `crit` expresion. However, the most common case is to define at least: `warn`, `warnNotification`, `crit`, `critNotification`, and `template`.

### Alert Keywords

#### warn
{: .keyword}
The expression to evaluate to set a warn state for an incident that is instantiated from the alert definition. 

The expression must evaluate to a NumberSet or a Scalar (See [Data Types](expressions#data-types)). 0 is false (do not trigger) and any non-zero value is true (will trigger). 

If the crit expression is true, the warn expression will not be evaluated as crit supersedes warn.

No warn notifications will be sent if `warnNotification` is not declared in the alert definition. It will still however appear on the dashboard.

#### crit
{: .keyword}
The expression to evaluate to set a critical state for an incident that is instantiated from the alert definition.

As with warn, the expression must return a Scalar or NumberSet. 

No crit notifications will be sent if `critNotification` is not declared in the alert definition. It will still appear on the dashboard.

#### critNotification
{: .keyword}
Comma-separated list of notifications to trigger on critical a state (when the crit expression is non-zero). This line may appear multiple times and duplicate notifications, which will be merged so only one of each notification is triggered. Lookup tables may be used when `lookup("table", "key")` is an entire `critNotification` value. (TODO: Link to notification lookups).

#### warnNotification
{: .keyword}
Identical to `critNotification` above, but the condition evaluates to warning state.

#### template
{: .keyword}
The name of the template that will be used to send alerts to the specified notifications for the alert.

#### runEvery
{: .keyword}
Multiple of global system configuration value `checkFrequency` at which to run this alert. If unspecified, the global system configuration value `defaultRunEvery` will be used.

#### squelch
{: .keyword}
`squelch` is comma-separated list of `tagk=tagv` pairs. `tagv` is a regex. If the current tag group matches all values, the alert is squelched, and will not trigger as crit or warn. For example, `squelch = host=ny-web.*,tier=prod` will match any group that has at least that host and tier. Note that the group may have other tags assigned to it, but since all elements of the squelch list were met, it is considered a match. Multiple squelch lines may appear; a tag group matches if any of the squelch lines match.

This can also be defined at the global of level of the configuration. 

When using squelch, alerts will be removed even if they are not with-in the scope of the final tagset. The common case of this would be using the `t` (tranpose function) to reduce the number of final results. So when doing this, results will still be remove because they are removed at the expression level for the `warn` and `crit` expressions.

#### unknown
{: .keyword}
`unknown` is the duration (i.e. `unknown = 5m` ) at which to mark an incident [unknown](/usage#severity-states) if it can not be evaluated. It defaults the system configuration global variable `checkFrequency`.

#### ignoreUnknown
{: .keyword}
Setting `ignoreUnknown = true`, will prevent an alert from becoming unknown. This is often used where you expect the tagsets or data for an alert to be sparse and/or you want to ignore things that stop sending information. 

#### unknownIsNormal
{: .keyword}
Setting `unknownIsNormal = true` will convert unknown events for an incident into a normal event.

This is often useful if you are alerting on log messages where the absence of log messages means that the state should go back to normal. Using `ignoreUnknown` with this setting would be unnecessary.

#### unjoinedOk
{: .keyword}
If present, will ignore unjoined expression errors. Unjoins happen when expressions with in an alert use a comparison operator, and there are tagsets in one set but are not in the other set.

#### log
{: .keyword}
Setting `log = true` will make the alert behave as a "log alert". It will never show up on the dashboard, but will execute notifications every check interval where the status is abnormal.

#### maxLogFrequency
{: .keyword}
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

Macros can not be used in templates, however, templates can reference subtemplates.

Note that templates are rendered when the expression is evaluated and it is non-normal. This is to eliminate changes in what is presented in the template because the data has changed in the tsdb since the alert was triggered.

### Template Inclusions
Templates can include other templates that have been defined in the confiuration.

```
template include {
    body = `<p>This gets included!</p>`
}

template includes {
    body = `{{ template "include" . }}`
}

alert include {
    template = includes
    warn = 1
}
```

### Template CSS
HTML templates will "inline" css. Since email doesn't support `<style>` blocks, an inliner ([douceur](https://github.com/aymerick/douceur)) takes a style block, and process the HTML to include those style rules as style attributes. 

Example:

```
template header {
    body = `
    <style>
        td, th {
            padding-right: 10px;
        }
    </style>
`
}

template table {
    body = `
    {{ template "header" . }}
    <table>
        <tr>
            <th>One</th>
            <!-- Will be rendered as:
            <th style="padding-right: 10px;">One</th> -->
            <th>Two</th>
        <tr>
            <td>This will have</td>
            <td>Styling applied when rendered</td>
        </tr>
    </table>
    `
}

alert table {
    template = table
    warn = 1
}
```

### Template Keywords

#### body
{: .keyword}
The message body. This is always formated as HTML.

#### subject
{: .keyword}
The subject of the template. This is also the text that will be used in the dashboard for triggered incidents. The format of the subject is plaintext.

### Template Variables
Template variables hold information specific to the instance of an alert. They are bound to the template's root context. That means that when you reference them in a block they need to be referenced differently just like context bound functions ([see template function types](/definitions#template-function-types))

#### Example of template variables
This example shows examples of the template variables documented below that are simple enough to be in a table:

```
alert vars {
    template = vars
    warn = avg(q("avg:rate:os.cpu{host=*bosun*}", "5m", ""))
}

template vars {
    body = `
    <!-- Examples of Variables -->
    <table>
        <tr>
            <th>Variable</th>
            <th>Example Value</th>
        </tr>
        <tr>
            <!-- Incident Id -->
            <td>Id</td>
            <td>{{ .Id }}</td>
        </tr>
        <tr>
            <!-- Start Time of Incident -->
            <td>Start</td>
            <td>{{ .Start }}</td>
        </tr>
        <tr>
            <!-- Alert Key -->
            <td>AlertKey</td>
            <td>{{.AlertKey}}</td>
        </tr>
        <tr>
            <!-- The Tags for the Alert instance -->
            <td>Tags</td>
            <td>{{.Tags}}</td>
        </tr>
        <tr>
            <!-- The rendered subject field of the template. -->
            <td>Subject</td>
            <td>{{.Subject}}</td>
        </tr>
        <tr>
            <!-- Boolean that is true if the alert has not been acknowledged -->
            <td>NeedAck</td>
            <td>{{.NeedAck}}</td>
        </tr>
        <tr>
            <!-- Boolean that is true if the alert is unevaluated (due to a dependency) -->
            <td>Unevaluated</td>
            <td>{{.Unevaluated}}</td>
        </tr>
        <tr>
            <!-- Status object representing current severity -->
            <td>CurrentStatus</td>
            <td>{{.CurrentStatus}}</td>
        </tr>
        <tr>
            <!-- Status object representing the highest severity -->
            <td>WorstStatus</td>
            <td>{{.WorstStatus}}</td>
        </tr>
        <tr>
            <!-- Status object representing the the most recent non-normal severity -->
            <td>LastAbnormalStatus</td>
            <td>{{.LastAbnormalStatus}}</td>
        </tr>
        <tr>
            <!-- Unix epoch (as int64) representing the time of LastAbnormalStatus -->
            <td>LastAbnormalTime</td>
            <td>{{.LastAbnormalTime}}</td>
        </tr>
        <tr>
            <!-- The name of the alert -->
            <td>Alert.Name</td>
            <td>{{.Alert.Name}}</td>
        </tr>
        <tr>
            <!-- Get Override Uknown Duration -->
            <td>Alert.Unknown</td>
            <td>{{.Alert.Unknown}}</td>
        </tr>
        <tr>
            <!-- Get Ignore Unknown setting for the alert (bool) -->
            <td>Alert.IgnoreUnknown</td>
            <td>{{.Alert.IgnoreUnknown}}</td>
        </tr>
        <tr>
            <!-- Get UnknownsNormal setting for the alert (bool) -->
            <td>Alert.UnknownsNormal</td>
            <td>{{.Alert.UnknownsNormal}}</td>
        </tr>
        <tr>
            <!-- Get UnjoinedOk setting for the alert (bool) -->
            <td>Alert.UnjoinedOK</td>
            <td>{{.Alert.UnjoinedOK}}</td>
        </tr>
        <tr>
            <!-- Get the Log setting for the alert (bool) -->
            <td>Alert.Log</td>
            <td>{{.Alert.Log}}</td>
        </tr>
        <tr>
            <!-- Get the MaxLogFrequency setting for the alert -->
            <td>Alert.MaxLogFrequency</td>
            <td>{{.Alert.MaxLogFrequency}}</td>
        </tr>
        <tr>
            <!-- Get the root template name -->
            <td>Alert.TemplateName</td>
            <td>{{.Alert.TemplateName}}</td>
        </tr>
        
    </table>
    `
    subject = `This is the subject`
}
```

#### .Attachments
{: .var}
When the graph functions that generate images are used they are added to `.Attachments`. Although it is available, you should *not* need to access this variable from templates. It is a slice of pointers to Attachment objects. An attachment has three fields, Data (a byte slice), Filename (string), and ContentType string. 

#### .Id
{: .var}
`.Id` is a unique number that identifies an incident in Bosun. It is an int64, see the documentation on the [lifetime of an incident](/usage#the-lifetime-of-an-incident) to understand when new incidents are created.

#### .Start
{: .var}
`.Start` is the the time the incident started and a Golang time.Time object. This means you can work with the time object if you need to, but a simple `{{ .Start }}` will print the time in 

#### .AlertKey
{: .var}
`.AlertKey` is a string representation of the alert key. The alert key is in the format alertname{tagset}. For example `diskused{host=ny-bosun01,disk=/}`.

#### .Tags
{: .var}
`.Tags` is a string representation of the tags for the alert. It is in the format of tagkey=tagvalue,tag=tagvalue. For example `host=ny-bosun01,disk=/`

#### .Result
{: .var}
`.Result` is a pointer to a "result object". This is *not* the same as the [result object](/definitions#result-1) i.e. the object return by `.Eval`. Rather is has the following properties:

    * Value: The number returned by the expression (technically a float64)
    * Expr: The expression evaluated as a string

If the severity status is Warn that it will be the result of the Warn expression, or if it crit than it will be a pointer to the result of the crit expressoin.

Example:

```
template result {
    body = `
    {{ if notNil .Result }}
        {{ .Result.Expr }}
        {{ .Result.Value }}
    {{ end }}
    `
}

alert result {
    template = result
    warn = 1
    crit = 2
}
```

#### .Actions
{: .var}
`.Actions` is a slice of of [action objects](/definitions#action) of actions taken on the incident. They are ordered by time from past to recent. This list will be empty when using Bosun's testing UI.

Example:

```
<table>
    <tr>
        <th>User</th>
        <th>Action Type</th>
        <th>Time</th>
        <th>Message</th>
    <tr>
    {{ range $action := .Actions }}
        <tr>
            <td>{{.User}}</th>
            <td>{{.Type}}</th>
            <td>{{.Time}}</th>
            <td>{{.Message}}</th>
        </tr>
    {{ end }}
</table>
```

#### .Subject
{: .var}
`.Subject` is the rendered subject field of the template as a string. It is only available in the body, and does not show up via Bosun's testing UI.

#### .NeedAck
{: .var}
`.NeedAck` is a boolean value that is true if the alert has not been acknowledged yet.

#### .Unevaulated
{: .var}
`.Unevaluated` is a boolean value that is true if the alert did not trigger because of a dependency. This field would only show true when viewed on the dashboard.

#### .CurrentStatus
{: .var}
`.CurrentStatus` is a [status object](/definitions#status) representing the current severity state of the incident. This will be "none" when using Bosun's testing UI.

#### .WorstStatus
{: .var}
`.WorstStatus` is a [status object](/definitions#status) representing the highest severity reached in the lifetime of the incident. This will be "none" when using Bosun's testing UI.

#### .LastAbnormalStatus
{: .var}
`.LastAbnormalStatus` is a [status object](/definitions#status) representing the the most recent non-normal severity for the incident. This will be "none" when using Bosun's testing UI.

#### .LastAbnormalTime
{: .var}
`.LastAbnormalTime` is an int64 representing the time of `.LastAbnormalStatus`. This is not a time.Time object, but rather a unix epoch. This will be 0 when using Bosun's testing UI.

#### .Alert.Text
{: .var}
`.Alert.Text` is the raw text of the alert definition as a string. It includes comments:

```
template text {
    body = `<pre>{{.Alert.Text}}</pre>`
}

alert text {
    # A comment
    template = text
    warn = 1
}
```

#### .Alert.Vars
{: .var}
`.Alert.Vars` is a map of string to string. Any variables declared in the alert definition get an entry in the map. The key is name of the variable without the dollar sign prefix, and the value is the text that the variable maps to (Variables in Bosun don't story values, and are just simple text replacement.) It the variable does not exist than an empty string will be returned. Global variables are only accessible via a mapping in the alert definition as show in the example below.

Example:

```
$aGlobalVar = "Hiya"

template alert.vars {
    body = `
        {{.Alert.Vars.foo}}
        
        <!-- baz returns an empty string since it is not defined -->
        {{.Alert.Vars.baz}}
        
        <!-- Global vars don't work -->
        {{.Alert.Vars.aGlobalVar }}
        
        <!-- Workaround for Global vars -->
        {{.Alert.Vars.fakeGlobal }}
    `
}

alert alert.vars {
    template = alert.vars
    $foo = 1 + 1
    $fakeGlobal = $aGlobalVar
    warn = $foo
}
```

#### .Alert.Name
{: .var}
`.Alert.Name` holds the the name of the alert. For example for an alert defined `alert myAlert { ... }` the value would be myAlert.

#### .Alert.Crit
{: .var}
`.Alert.Crit` is a [bosun expression object](/definitions#expr) that maps to the crit expression in the alert. It is only meant to be used to display the expression, or run the expression by passing it to functions like `.Eval`.

Example:

```
template expr {
    body = `
        Expr: {{.Alert.Warn}}</br>
        <!-- note that an error from eval is not checked in this example, 
        see other examples for error checking -->
        Result: {{.Eval .Alert.Warn}}
        
    `
}

alert expr {
    template = expr
    warn = 1 + 1
    crit = 3
}
```

#### .Alert.Warn
{: .var}
Like `.Alert.Crit` but the crit expression.

#### .Alert.Depends
{: .var}
Like `.Alert.Crit` but the depends expression.

#### .Alert.CritNotification
{: .var}

#### .Alert.WarnNotification
{: .var}

#### .Alert.Unknown
{: .var}
`.Alert.Unknown` is a time.Duration that is the duration for unknowns if set to override the global duration.  It will be zero if the alert is using the global setting.

#### .Alert.MaxLogFrequency
{: .var}
`.Alert.MaxLogFrequency` is a time.Durtion that shows the alert setting for a Log Alert that determines how often a log alert should send its notification.

#### .Alert.IgnoreUnknown
{: .var}
`.Alert.IgnoreUnknown` is a bool that will be true if ignoreunknown is set on the alert.

#### .Alert.UnknownsNormal
{: .var}
`.Alert.UnknownsNormal` is a bool that is true of the unknowns normal setting is true. This means unknown events will be treated as normal events.

#### .Alert.UnjoinedOK
{: .var}
`.Alert.UnjoinedOk` is a bool that is true of the unjoinedOK is set to true on the alert. This makes it so when doing operations with two sets, if there are items in one set that have no match they will be ignored instead of triggering an error.

#### .Alert.Log
{: .var}
`.Alert.Log` is a bool that is true if this is a log style alert.

#### .Alert.RunEvery
{: .var}
`.Alert.RunEvery` is an int that shows an override for how often the alert should run. It is a multiplper that overrides how many check intervals exist between each run.

#### .Alert.TemplateName
{: .var}
`.Alert.TemplateName` is the name of the template that the alert is configured to use.

#### .Expr 
{: .var}
The value of `.Expr` is the warn or crit expression that was used to evaluate the alert in the format of a string.

#### .Events
{: .var}
The value of `.Events` is a slice of [Event](/definitions#event) objects.

Example:  

```
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
```

#### .IsEmail
{: .var}
The value of is `IsEmail` is true if the template is being rendered for an email. This allows you to use the same template for different types of notifications conditionally within the template.

#### .Errors
{: .var}
A slice of strings that gets appended to when a context bound function returns an error. 

### Template Function Types
Template functions come in two types. Functions that are global, and context-bound functions. Unfortunately, it is important to understand the difference because they are called differently in functions and have different behavior in regards to [error handling](/definitions#template-error-handling).

#### Global
Calling global functions is simple. The syntax is just the function name and arguments. I can be used in regular format or a chained pipe format.

```
template global_type_example {
    body = `
        <!-- Regular Format -->
        {{ bytes 12312313 }}
        <!-- Pipe Format -->
        {{ 12312313 | bytes }}
    `
}
```

#### Context Bound
Context bound functions, like Global functions, can be called in regular or pipe format. What makes them different is the syntax used to call them. Context bound functions have a period before them, such as `{{ .Eval }}`. Essentially they are methods on that act on the instance of an alert/template combination when it is rendered and perform queries.

They are bound to the parent context. The "parent context" is essentially the top level namespace within a template. This is because they access data associated with the instance of the template when it sent. 

What this practically means, is that when you are inside a block within a template (for example, inside a range look) context-bound functions need to be called with `{{ $.Func }}` to reference the parent context.

```
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
```

### Template Error handling
Templates can throw errors at runtime (i.e. when a notification is sent). Although the configuration check makes sure that templates are valid, you can still do things like try to dereference objects that are nil pointers.

When a template fails to render, a generic notification will be emailed to the people that would have recieved the alert. In the case of a post notification (where the subject is used). The following text will be posted `error: template rendering error for alert <alertkey>` where the alert key is something like `os.cpu{host=a}`.

In order to prevent the template from completelying failing and resulting in the generic notification, errors can be handled inside the application. 

Errors are handled differently depending on the type of the function (Context Bound vs Global). When context bound functions have errors the `.Errors` variable attached to the template context is appended to. This is not the case for global functions. 

Global functions always returns strings except for parseDuration. When global functions error than `.Errors` is *not* appended to, but the string that would have been returned with an error is show in the template. parseDuration returns nil when it errors, and in this one exception you can't see what the error is.

If the func returns a string or an image (technically an interface) the error message will be displayed in the template. If an object is returned (i.e. a result sets, a slice, etc) nil is returned and the user can check for that in the template. In both cases `.Errors` will be appended to if it is a context bound functions.

See the examples in the functions that follow to see examples of Error handling. 

### Template Functions
(TODO: Sort these when done)

#### .Eval(string|Expression) (resultValue)
{: .func}
Type: Context-Bound

Executes the given expression and returns the first result that includes the tag key/value pairs of the alert instance. In other words, it evaluates the expression within the context of the alert. So if you have an alert that could trigger multiple incidents (i.e. `host=*`) then the expression will return data specific to the host for this alert.

If the expression results in an error nil will be returned and `.Errors` will be appended to. If the result set is empty, than `NaN` is returned. Otherwise the value of the first matching result in the set is returned. That result can be any type of value that Bosun can return since the type returned by Eval is dependent on what the return type of the expression it evaluates. Mostly commonly it will be float64 when used to evaluate an expression that is enclosed in a reduction like in the following example:

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

#### .EvalAll(string|Expression|ResultSlice) (Result)
{: .func}
Type: Context-Bound

`.EvalAll` executes the given expression and returns a slice of [ResultSlice](/definitions#resultslice). The type of each results depends on the return type of the expresion. Mostly commonly one uses a an expression that returns a numberSet. If there is an error nil is returned and `.Errors` is appended to.

Example:

```
alert evalall {
    template = evalall
    # the type returned by the avg() func is a seriesSet
    $cpu = q("sum:rate{counter,,1}:os.cpu{host=ny-bosun*}", "5m", "")
    # the type returned by the avg() func is a numberSet
    $cpu_avg = avg($cpu)
    warn = 1
}

template evalall {
    body = `
    {{ $numberSetResult := .EvalAll .Alert.Vars.cpu_avg }}
    {{ if notNil $numberSetResult }}
    <table>
        <tr>
            <th>Host</th>
            <th>Avg CPU Value</th>
        <tr>
        {{ range $result := $numberSetResult }}
            <tr>
                <!-- Show the value of the host tag key -->
                <td>{{$result.Group.host}}</td>
                <!-- Since we know the value of cpu_avg is a numberSet which contains floats.
                we pipe the float to printf to make it so it only print to two decimal places -->
                <td>{{$result.Value | printf "%.2f"}}</td>
            </tr>
        {{end}}
    <table>
    {{ else }}
        {{ .LastError }}
    {{ end }}
    
    <!-- You could end up with other types, but their usage is not recommended, but to illustrate the point working with a seriesSet is show -->
    {{ $seriesSetResult := .EvalAll .Alert.Vars.cpu }}
    {{ if notNil $seriesSetResult }}
        {{ range $result := $seriesSetResult }}
            <h2>{{ $result.Group }}</h2>
            <table>
                <tr>
                    <th>Time</th>
                    <th>Value</th>
                <tr>
                <!-- these are *not* sorted -->
                {{ range $time, $value := $result.Value }}
                    <tr>
                        <td>{{ $time }}</td>
                        <td>{{ $value }}</td>
                    </tr>
                {{end}}
            <table>
        {{ end }}
    {{ else }}
        {{ .LastError }}
    {{ end }}
    
    `
}
```

#### .LeftJoin(expression|string) ([][]Result)
{: .func}

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

#### .Ack() (string)
{: .func}

Type: Context-Bound

`.Ack` creates a link to Bosun's view for alert acknowledgement. This is generated using the [system configuration's Hostname](/system_configuration#hostname) value as the root of the link.

#### .Incident() (string)
{: .func}

Type: Context-Bound

`.Incident` creates a link to Bosun's incident view. This is generated using the [system configuration's Hostname](/system_configuration#hostname) value as the root of the link.

#### .GraphLink(string) (string)
{: .func}

Type: Context-Bound

`.GraphLink` creates a link to Bosun's rule editor page. This is useful to provide a quick link to the view someone would use to edit the alert. This is generated using the [system configuration's Hostname](/system_configuration#hostname) value as the root of the link. The link will set the the alert, which template should be rendered, and time on the rule editor page. The time that represents "now" will be the time of the alert. The rule editor's alert will be set to point to the alert definition that corresponds to this alert. However, it will always be loading the current definitions, so it is possible that the alert or template definitions will have changed since the template was rendered. 


#### .Graph(string|Expression, unit string) (image)
{: .func}

Type: Context-Bound

Creates a graph of the expression. It will error (That can not be handled) if the return type of the expression is not a `seriesSet`. If the expression is a an OpenTSDB query, it will be auto downsampled so that there are approx no more than 1000 points per series in the graph. Like `.Eval`, it filters the results to only those that include the tag key/value pairs of the alert instance. In other words, in the example, for an alert on `host=a` only the series for host a would be graphed.

If the optional unit argument is provided it will be shown as a label on the y axis.

When the rendered graph is viewed in Bosun's UI (either the config test UI, or the dashboard) than the Graph will be an SVG. For email notifications the graph is rendered into a PNG. This is because most email providers don't allow SVGs embedded in emails.

If there is an error executing `.Graph` than a string showing the error will be returned instead of an image and `.Errors` will be appended to.

Example:

```
alert graph {
    template = graph
    $series = merge(series("host=a", 0, 1, 15, 2, 30, 3), series("host=b", 0, 2, 15, 3, 30, 1))
    $r = avg($series)
    crit = $r
}

template graph {
    body = `
    {{$v := .Graph .Alert.Vars.series "Random Numbers" }}
    <!-- If $v is not nil (which is what .Graph returns on errors) -->
    {{ if notNil $v }}
        {{ $v }}
    {{ else }}
        {{ .LastError }}
    {{ end }}
`
}
```

#### .GraphAll(string|Expression, unit string) (image)
{: .func}

Type: Context-Bound

`.GraphAll` behaves exactly like `.Graph` but does not filter results to match the tagset of the alert. So if you changed the call in the example for `.Graph` to be `.GraphAll`, in an alert about `host=a` the series for both host a and host b would displayed (unlike Graph where only the series for host a would be displayed). 

#### .GetMeta(metric, key string, tags string|TagSet) (object|string)
{: .func}

Type: Context-Bound

`.GetMeta` fetches information from Bosun's metadata store. This function returns two types of metadata: metric metadata and metadata attached to tags. If the metric argument is a non-empty string, them metric metadata is fetched, otherwise tag metadata is fetched. 

For both Metric and Tag metadata, in case where the key is a non-empty string then a string will be returned which will either be the value or an error. In cases where key is an empty string, a slice of objects is returned unless there is an error in which case nil is returned. The example shows these cases.

When a slice of objects are returned, the objects have the following properties:

 * Metric: A string representing the metric name
 * Tags: A map of tag keys to tag values (string[string]) for the metadata
 * Name: The key of the metadata (same as key argument to this function, if provided)
 * Value: A string
 * Time: Last time this Metadata was updated

For Metric metadata the Tags field will be empty, and for tag metadata the metric field is empty. 

For Tag metadata, metadata is returned that includes the key/value pairs provided as an argument. So for example, `host=a` would also return metadata that is tagged `host=a,interface=eth0`.


Example:

```

alert meta {
    template = meta
    $metric = os.mem.free
    warn = avg(q("avg:rate:$metric{host=*bosun*}", "5m", ""))
}

template meta {
    body = `
        <h1>Metric Metadata</h1>
        <!-- Metric Metadata as slice -->
        {{ $metricMetadata := .GetMeta .Alert.Vars.metric "" "" }}
        {{ if notNil $metricMetadata }}
            <h2>Metric Metadata as Slice</h2>
            <table>
                <tr>
                    <th>Property</th>
                    <th>Value</th>
                </tr>
                {{ range $prop := $metricMetadata }}
                    <tr>
                        <td>{{ $prop.Name }}</td>
                        <td>{{ $prop.Value }}</td>
                    </tr>
                {{ end }}
            </table>
        {{ else }}
            {{ .LastError }}
        {{ end }}
        
        <h2>Metric Metadata as string values</h2>
        <!-- Metric Metadata as strings (specific keys) -->
        Desc: {{ .GetMeta .Alert.Vars.metric "desc" "" }}</br>
        Unit: {{ .GetMeta .Alert.Vars.metric "unit" "" }}</br>
        RateType: {{ .GetMeta .Alert.Vars.metric "rate" "" }}</br>
        
        <h1>Tag Metadata<h1>
        <h2>Tag Metadata as slice</h2>
        {{ $tagMeta := .GetMeta "" "" "host=ny-web01" }}
        {{ if notNil $tagMeta }}
            <table>
                <tr>
                    <th>Property</th>
                    <th>Tags</th>
                    <th>Value</th>
                    <th>Last Touched Time</th>
                </tr>
                {{ range $metaSend := $tagMeta }}
                    <tr>
                        <td>{{ $metaSend.Name }}</td>
                        <td>{{ $metaSend.Tags }}</td>
                        <td>{{ $metaSend.Value }}</td>
                        <td>{{ $metaSend.Time }}</td>
                    </tr>
                {{ end }}
            </table>
        {{ else }}
            {{ .LastError }}
        {{ end }}
        
        <h2>Keyed Tag Metadata</h2>
        <!-- Will return first match -->
        {{ $singleTagMeta := .GetMeta "" "memory" "host==ny-web01" }}
        {{ if notNil $singleTagMeta }}
            {{ $singleTagMeta }}
        {{ else }}
            {{ .LastError }}
        {{ end }}
    `
}
```

#### .HTTPGet(url string) string
{: .func}

Type: Context-Bound

`.HTTPGet` fetches a url and returns the raw text as a string, unless there is an error or an http response code `>= 300` in which case the response code or error is displayed and `.Errors` is appended to. The client will identify itself as Bosun via the user agent header. Since templates are procedural and this meant to for fetching extra information, the timeout is set to five seconds. Otherwise this function could severly slow down the delivery of notifications.

Example:

```
template httpget {
    body = `
    {{ .HTTPGet "http://localhost:9090"}}
    `
}

alert httpget {
    template = httpget
    warn = 1
}
```

#### .HTTPGetJSON(url string) (*jsonq.JsonQuery)
{: .func}

Type: Context-Bound

(TODO: Document, link to jsonq library and how to work with objects. Note limitation about top level object being an array)

#### .HTTPPost(url, bodyType, data string) (string)
{: .func}

Type: Context-Bound

`.HTTPPost` sends a HTTP POST request to the specified url. The data is provided a string, and bodyType will set the Content-Type HTTP header. It will return the response or error as a string in the same way that `.HTTPGet` behaves. 

Example:

```
template httppost {
    body = `
    {{ .HTTPPost "http://localhost:9090" "application/json" "{ \"Foo\": \"bar\" }" }}
    `
}

alert httppost {
    template = httppost
    warn = 1
}
```

#### .ESQuery(indexRoot expr.ESIndexer, filter expr.ESQuery, sduration, eduration string, size int) ([]interface{})
{: .func}

Type: Context-Bound

`.ESQuery` returns a slice of elastic documents. The function behaves like the escount and esstat [elastic expression functions](/expressions#elastic-query-functions) but returns documents instead of statistics about those documents. The number of documents is limited to the provided size argument. Each item in the slice is the a document that has been marshaled to a golang interface{}. This means the contents are dynamic like elastic documents. If there is an error, than nil is returned and `.Errors` is appended to. 

The group (aka tags) of the alert is used to further filter the results. This is implemented by taking each key/value pair in the alert, and adding them as an [elastic term query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html).

Example:

```
template esquery {
    body = `
        {{ $filter := (.Eval .Alert.Vars.filter)}}
        {{ $index := (.Eval .Alert.Vars.index)}}
        {{ $esResult := .ESQuery $index $filter "5m" "" 10 }}
        {{ if notNil $esResult }}
            {{ range $row := $esResult }}
                <p>{{ range $key, $value := $row }}
                    <!-- Show the Key, Value, and the Type of the value. Values could be objects
                    but dereferencing their properties if they don't exist could cause the template
                    to fail to render -->
                    {{ $key }}: {{ $value }} ({{ $value | printf "%T" }}),
                {{ end }}<p> 
            {{ end }}
        {{ else }}
            <p>{{ .LastError }}
        {{end}}
    `
}

alert esquery {
    template = esquery
    $index = esls("logstash")
    $filter = esregexp("logsource", "ny-.*")
    crit = avg(escount($index, "logsource", $filter, "2m", "10m", ""))
}
```

#### .ESQueryAll(indexRoot expr.ESIndexer, filter expr.ESQuery, sduration, eduration string, size int) (interface{})
{: .func}

Type: Context-Bound

`.ESQueryAll` behaves just like `.ESQuery`, but the tag filtering to filter results to match the alert instance is *not* applied.

#### .Group() (TagSet)
{: .func}

Type: Context-Bound

A map of tags keys to their corresponding values for the alert.

#### .Last() (string)
{: .func}

Type: Context-Bound

The most recent [Event](/definitions#event) in the `.History` array.

#### .LastError() (string)
{: .func}

Type: Context-Bound

Returns the string representation of the last Error, or an empty string if there are no errors. This only contains errors from context-bound functions. 

#### .Lookup(table string, key string) (string)
{: .func}

Type: Context-Bound

(TODO: Document + Example, also does this use search or is it like lookup series?)

#### .LookupAll(table string, key string) (string)
{: .func}

Type: Context-Bound

(TODO: Document + Example, also does this use search or is it like lookup series?)

#### notNil(value) (bool)
{: .func}

Type: Global

`notNil` returns true if the value is nil. This is only meant to be used with error checking on context-bound functions.

#### bytes(string|int|float) (string)
{: .func}

Type: Global

`bytes` converts a number of bytes into a human readable number with a postfix (such as KB or MB). Conversions are base ten and not base two.

Example:

```
template bytes {
    body = `
        <!-- bytes uses base ten, *not* base two -->
        {{ .Alert.Vars.ten | bytes }},{{ .Alert.Vars.two | bytes }}
        <!-- results are 97.66KB,1000.00KB -->
    `
}

alert bytes {
    template = bytes
    $ten = 100000
    $two = 1024000
    warn = $ten
}
```

#### pct(number) (string)
{: .func}

Type: Global

`pct` takes a number, limits it to two decimal places and adds a "%" suffix. It does *not* multiply the number by 100.

Example:

```
template pct {
    body = `
    <!-- Need to eval to get number type instead of string -->
    {{ .Eval .Alert.Vars.value | pct }}
    <!-- result is: 55.56% -->
    `
}

alert pct {
    template = pct
    $value = 55.55555
    warn = $value
}
```

#### replace(s, old, new string, n int) (string)
{: .func}

Type: Global

`replace` maps to golang's [strings.Replace](http://golang.org/pkg/strings/#Replace) function. Which states:

> Replace returns a copy of the string s with the first n non-overlapping instances of old replaced by new. If old is empty, it matches at the beginning of the string and after each UTF-8 sequence, yielding up to k+1 replacements for a k-rune string. If n < 0, there is no limit on the number of replacements

Example:

```
template replace {
    body = `
    {{ replace "Foo.Bar.Baz" "." " " -1 }}
    <!-- result is: Foo Bar Baz -->
    `
}

alert replace {
    template = replace
    warn = 1
}
```

#### short(string) (string)
{: .func}

Type: Global

`short` Trims the string to everything before the first period. Useful for turning a FQDN into a shortname. For example: `{{short "foo.baz.com"}}` in a template will return `foo`

#### html(string) (htemplate.HTML)
{: .func}

Type: Global

`html` takes a string and makes it part of the template. This allows you to inject HTML from variables set in the alert definition. A use case for this is when you have many alerts that share a template and fields you have standardized. For example, you might have a `$notes` variable that you attach to all alerts. This way when filling out the notes for an alert, you can include things like HTML links.

Example:

```
template htmlFunc {
    body = `All our templates always show the notes: <!-- the following will be rendered as subscript -->
    {{ .Alert.Vars.notes | html }}`
}

alert htmlFunc {
    template = htmlFunc
    $notes = <sub>I'm ashmaed about the reason for this alert so this is in subscript...</sub>. In truth, the solution to this alert is the solution to all technical problems, go ask on <a href="https://stackoverflow.com" target="blank">StackOverflow</a>
    warn = 1 
}
```

#### parseDuration(string) (*time.Duration)
{: .func}

Type: Global

`parseDuration` maps to Golang's [time.ParseDuration](http://golang.org/pkg/time/#ParseDuration). It returns a pointer to a time.Duration. If there is an error nil will be returned. Unfortunately the error message for this particular can not be seen.

Example:

```
template parseDuration {
    body = `
        <!-- More commonly you would use .Last.Time.Add , but .Last does not function in the testing interface -->
        Doomsday: {{ .Start.Add (parseDuration (.Eval .Alert.Vars.secondsUntilDoom | printf "%fs"))}}
        <!-- result is: Doomsday: 2021-02-11 15:11:06.727332631 +0000 UTC -->
    `
}

alert parseDuration {
    template = parseDuration
    $secondsUntilDoom = 123453245
    warn = $secondsUntilDoom
} 
```

#### V(string) (string)
{: .func}

Type: Global

The `V` func allows you to access [global variables](/definitions#global-variables) from within templates.

Example:

```
$myGlobalVar = Kon'nichiwa

template globalvar {
    body = `{{ V "$myGlobalVar" }}`
}

alert globalvar {
    template = globalvar
    warn = 1
}
```

### Types available in Templates
Since templating is based on Go's template language, certain types will be returned. Understanding these types can help you construct richer alert notifications.

#### Action
{: .type}

An Action is an object that represents actions that people do on an incident. It has the following fields:

 * User: a string of the username for the person that took the action
 * Message: a string of an optional message that a user added to the action when taking it
 * Time: a time.Time object representing the time the action it was taken
 * ActionType: an int representing the type of action. The values and their strings are:
   * 0: "none"
   * 1: "Acknowledged"
   * 2: "Closed"
   * 3: "Forgotten"
   * 4: "ForceClosed"
   * 5: "Purged"
   * 6: "Note"

#### Event
{: .type}

An Event represent a change in the [severity state](/usage#severity-states) within the [duration of an incident](/usage#the-lifetime-of-an-incident). When an incident triggers, it will have at least one event.  An Event contains the following fields

 * **Warn**: A pointer to an [Event Result](definitions#event-result) that the warn expression generated if the event has a warning status.
 * **Crit**: A pointer to an [Event Result](definitions#event-result) if the event has a critical status.
 * **Status**: An integer representing the current severity (normal, warning, critical, unknown). As long as it is printed as a string, one will get the textual representation. The status field has identification methods: `IsNormal()`, `IsWarning()`, `IsCritical()`, `IsUnknown()`, `IsError()` which return a boolean.
 * **Time**: A [Go time.Time object](https://golang.org/pkg/time/#Time) representing the time of the event. All the methods you find in Go's documentation attached to time.Time are available in the template
 * **Unevaluated**: A boolean value if the alert was unevaluated. Alerts on unevaluated when the current was using `depends` to depend on another alert, and that other alert was non-normal. 

It is important to note that the `Warn` and `Crit` fields are pointers. So if there was no `Warn` result and you attempt to access a property of `Warn` then you would get a template error at runtime. Therefore when referecing any fields of `Crit` or `Warn` such as `.Crit.Value`, it is vital that you ensure the `Warn` or `Crit` property of the Event is not a nil pointer first.

 See the example under the [Events template variable](/definitions#events) to see how to use events inside a template.

#### Event Result 
{: .type}

An Event Result (note: in the code this is actually a models.Result) has two properties:

* **Expr**: A string representation of the full expression used to generate the value of the Result's Value.
* **Value**: A float representing the result of the parent expression.

There is a third property **Computations**. But it is not recommended that you access it even though it is available and it will not be documented.

#### Expr
{: .type}

A `.Expr` is a bosun expression. Although various properties and methods are attached to it, it should only be used for printing (to see the underlying text) and for passing it to function that evaluate expressions such as `.Eval` within templates.

#### ResultSlice
{: .type}

A `ResultSlice` is returned by using the `.EvalAll` function. It is a slice of pointers to `Result` objects. Each result represents the an item in the set when the type is something like a NumberSet or a SeriesSet.

#### Result
{: .type}

A `Result` as two fields:

 1. **Group**: The Group is the TagSet of the result. 
 2. **Value**: The Value of the Result

A tagset is a map of of string to string (`map[string]string` in Go). The keys represent the tag key and their corresponding values are the tag values.

The Value can be of different types. Technically, it is a go `interface{}` with two methods, `Type()` which returns the type of the `Value()` which returns an `interface{}` as well, but will be of the type returned by the `Type()` method.

The most common case of dealing with Results in a ResultSlice is to use the `.EvalAll` func on an expression that would return a NumberSet. See the example under [EvalAll](/definitions#evalall).

#### Status 
{: .type}

The `Status` type is an integer that represents the current severity status of the incident with associated string repsentation. The possible values and their string representation is:

 * 0 for "none"
 * 1 for "normal"
 * 2 for "warning"
 * 3 for "critical"
 * 4 for "unknown"


## Notifications

## Lookups 

## Macros


{% endraw %}

</div>
</div>