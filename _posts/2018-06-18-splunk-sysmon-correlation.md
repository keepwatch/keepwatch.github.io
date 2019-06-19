## Correlating Sysmon data in Splunk: Part 1

[Splunk](https://www.splunk.com/) is a great tool for log management (assuming you have $$$, licensing gets expensive). It uses functions which can be chained together to accomplish almost anything imaginable. However, understanding how to use these functions well is challenging. This series of posts will expose several Splunk functions, how they interact, and how to correlate data (connecting two different events) in Splunk. 

### Why correlate at all?

[Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) events are amazing when correlated. This open-source host monitoring tool generates `process create`, `network connection`, `file create`, `registry add/modify/remove` event types (to name a few). These events are associated with an `Image` (the program that performs the logged action) and `ProcessGuid` field (identifying the exact instance of the Image that performed the action). Correlating a suspicious network connection with the CommandLine that launched the instance of that Image is invaluable.

Note that many data sources can be correlated - proxy and DNS logs could be correlated on a hostname, Ironport logs can be correlated on a Message ID (`MID`), etc - so the Splunk principles we'll discuss are useful beyond Sysmon data.

### How to correlate manually 

First, let's state some assumptions about your Splunk and Sysmon configuration. Swap these fieldnames with the appropriate values for your environment.
* Sysmon data stored in Splunk at: `index=sysmon`
* Sysmon event ID field name: `EventID`

OK, so how can we correlate?

* Find a non-process Sysmon event you want to investigate further - like a network connection: `index=sysmon EventID=3 | head 1`
* Identify the `ProcessGuid` field value
* Search Splunk for the associated process create event: `index=sysmon EventID=1 "<ProcessGuid field value>" ProcessGuid="<ProcessGuid field value>"`
* That's it! Now you have access to `User`, `CommandLine`, and `ParentCommandLine`, along with every other event (if you remove `EventID=1`) spawned by that process!

Doing this correlation in a single search is quite a bit harder. Our next post will talk about correlation using `| join` and subsearches.


