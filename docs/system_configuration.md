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



{% endraw %}

</div>
</div>