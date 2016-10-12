---
layout: default
title: Configuration
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
This configuration has been split into two different files. One file is for definitions of various Bosun sections such as alerts, templates, etc. The documentation for this is (TODO: link to new definitions documentation). This file, can be edited and saved via Bosun's UI when (TODO: link to setting, also link to how the other setting enables the other one implicitly) has been enabled. 

This was done because the definitions can now be reloaded without restarting the Bosun process. This also means that users can edit alerts directly in the UI.

System configuration has been moved into a new file. Settings in this file require that Bosun be restarted. The new file format is in toml (TODO: link to toml). The page documents this new system configuration. 

There is also an example file that can be looked at. It should be noted that this file does not follow the tradition of commenting out all defaults. This is because the file is used for testing as well. For the time being, the value of the example being tested is has been valued over following that tradition for until we have the bandwidth to duplicate the two files in a way where this tradition can be maintained. 

(TODO: Find a way to utilize some sort of technical note mockup for the documentation. I would also like a way to a warning style note for current pitfalls. Need to document more of the pitfalls)

## Configuration Keys

(TODO: Note that in toml key value pairs must be provided before any sections are defined)

### Hostname
The `Hostname` sets the hostname that bosun will use to construct all its links. The common use case would be in any template functions (TODO: Link to template functions) that construct links.

Example:
`Hostname = "bosun.example.com"`

### HTTPListen
`HTTPListen` sets the HTTP IP and Port to Listen on. (TODO: Document what this will look like when we have SSL merged). The default if not specified is to listen on `:8070`

Example:
`HTTPListen = ":8080"`

### CheckFrequency
`CheckFrequency` specifies the minimum interval that alert checks will run at on a schedule. The format of the value is the same as [Go's duration format](https://golang.org/pkg/time/#Duration.String). By default, alert checks are run at every `CheckFrequency` multipied by the `DefaultRunEvery` value. This defaults to "5m".

Example:
`CheckFrequency = "1m"`

### DefaultRunEvery
By default, alert checks are run at every `CheckFrequency` multipied by the `DefaultRunEvery` value. This can be overridden in an alert definition with `runEvery` (TODO: link to runEvery alert key). This defaults to 1.
 
So for example if you have a `CheckFrequency` of "1m" and a `DefaultRunEvery` of 5, alerts by default will run every 5 minutes. But you could have some run as frequent as every "1m", and others that run less often (any multiple of "1m").

Example:
`DefaultRunEvery = 5`

### RuleFilePath
This is the path to the file that contains all the definitions of alerts, macros, lookups, templates, notifications, and global variables. This configuration file is documented in (TODO: Link to other configuration page). If saving is enabled, this file can be written to by Bosun via the API or UI.

Example: `RuleFilePath = "dev.sample.conf"`

### TimeAndDate
`TimeAndDate` is used to configure time zones that will be linked to in Bosun's dashboard. It is an array of timeanddate.com zones (the page that gets linked to from Bosun's UI.) It has no impact on what time zone Bosun operates in. Bosun is expected to use UTC and does support other timezones.

Example:
`TimeAndDate = [ 202, 75, 179, 136 ]`

### ShortURLKey
Bosun's UI can generate shortlinks using Google's goo.gl URL Shortener service. If you are hitting their API limits, you can get an API key and specify here, and that key will get used.

Example:
`ShortURLKey = "aKey"`

### MinGroupSize
Bosun's dashboard will use grouping logic on the dashboard if there are many similar alerts (in name, or tag/key pairs). `MinGroupSize` sets the minimum number of alerts needed to create a group in the UI. (TODO: See what the default is here)

If you have a lot of grouping, it often means you should refactor the alert to have a less granular scope. Therefore, it is recommended that this is used as a "protection mechanism" from flooding the dashboard with too many alerts.

Example: `MinGroupSize = 5`

### Unknown Threshold 
Bosun will group all unknowns in a single check cycle (alerts on the same `CheckFrequency` and `runEvery`) into a single email. This sets how many unknowns would be sent in a single check cycle before a group is created. The default value is 5.

This is done because unknowns are generally caused by the data "disappearing". So if your TSDB Provider is broken or behind, it can generate a lot of unknowns. This alleviates flooding in the specific case of unknowns.

Example: `UnknownThreshold = 5`

### Ping
If set to `true`, Bosun will ping every value of the host tag that it has indexed (TODO: link to an explanation of indexing, and probably create that section as well .. sigh) and record that value to your TSDB. It currently only support OpenTSDB style data input, which is means you must use either OpenTSDB or Influx with the OpenTSDB endpoint on Influx configured. 

Example: 
`Ping = true`

## Configuration Sections
All your key value pairs must be defined before any sections are defined. Sections are used for things that have multiple values to configure them.






{% endraw %}

</div>
</div>