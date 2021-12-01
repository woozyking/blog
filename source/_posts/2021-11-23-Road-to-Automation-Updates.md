---
title: 'Road to Automation: Updates'
date: 2021-11-26 15:17:15
tags:
  - programming
  - software development
  - automation
---

Team collaboration must get a sense of each others’ whereabouts as quickly and accurately as possible. Usually, teams resort to recurrent meetings to get updates from each other.

But meetings are not always the best way for this scenario:
* Meetings are synchronous, so everyone has to wait for the others to finish before continuing.
* Especially when communicating verbally, it is difficult not to misinterpret each others' words.
* It is challenging to scale up the team size because that would amount to more meetings and more probable misinterpretation of the updates.
* It is challenging to work as a distributed team where members can reside in different time zones.

Building on top of the experience to {% post_link Road-to-Automation-Release-Notes 'automate our product release process' %}, we now automate the process to [get daily and weekly updates](https://github.com/EQWorks/updates) from each other from exactly where work happens and consume them asynchronously at our own pace.
<!-- more -->
## 👀 where work happens

Instead of listening to what each other has to say about the work, we can look at the actual issues or implementations. Since we already use GitHub to collaborate, we can consume the various signals around PRs and issues.

An example of our daily update for the entire engineering organization:

> 13 PR/issues updates
> - ✔️ Done: 1
> - ⚠️ WIP: 4
> - 👀 Needs review: 8
>
> **LOCUS**
>
> snoke - (vxsl)
> - ⚠️ <u>PR #1030</u> Deps/kepler sidegrade (vxsl)
>   - <u>1 comment</u> - (netlify[bot])
>   - <u>4 updated commits</u>
>
> locussdk - (amy83762100)
> - 👀 <u>PR #158</u> marketplace_datasets - pass cu, wl to'/market/subscription' (amy83762100)
>   - <u>2 updated commits</u>
> - 👀 <u>PR #157</u> get_aoi_dataset - add run_async option (amy83762100)
>   - <u>1 updated commit</u>
>
> firstorder - (amy83762100)
> - 👀 <u>PR #644</u> marketplace_datasets - get 'cu', 'wl' from query (amy83762100)
>   - <u>1 comment</u> - (woozyking)
>   - <u>5 reviews</u> - (woozyking, amy83762100, kevlabs)
>   - <u>2 updated commits</u>
> - 👀 <u>PR #643</u> get_aoi_dataset - migrate get_aoi function from EnrichData to firstorder (amy83762100)
>   - <u>1 updated commit</u>
>
> _(more)_

A member can scan through the update and go to the PRs or issues they are interested in for more details. The update allows us to gather insights and ask more informed questions to the relevant contributors.

## NLP-assisted curation

To help us sift through the noise, we incorporate icons to indicate the type of update and what we have done in our [release CLI](https://github.com/EQWorks/release) to label the PRs.

An example of our weekly digest (which naturally contains more information than the daily updates), with labelled changes:

> 30 PR/issues updates
> - ✔️ Done: 23
> - ⚠️ WIP: 4
> - 👀 Needs review: 3
>
> **LOCUS**
>
> locussdk - (divyselva)
>
> `OTHERS`
> - 👀 <u>Issue #156</u> get_aoi_dataset() timeout error (divyselva)
>
> data-hub - (vikivanov)
>
> `ADDED`
> - ✔️ <u>PR #82</u> sftp - added paramiko to requirements and new image version (vikivanov)
> - ✔️ <u>PR #81</u> sftp - added missing connector type to dag helper (vikivanov)
>
> firstorder - (amy83762100, DoParkEQ)
>
> `FIXED`
> - ✔️ <u>PR #642</u> convert_geoid - fix bug - pass wrong payload to lambda function (amy83762100)
>
> `ADDED`
> - ✔️ <u>PR #635</u> marketplace search update (DoParkEQ)
>
> snoke - (woozyking, vxsl, DoParkEQ)
>
> `CHANGED`
> - ✔️ <u>PR #1029</u> city - load locus.city demo as an iframe sidecar to the main page (woozyking)
> - ✔️ <u>PR #1028</u> navbar - initialize wl/cu names on page load (vxsl)
> - ✔️ <u>PR #1024</u> marketplace product/detail page update (DoParkEQ)
>
> _(more)_

## Other sources

To supplement work progress from GitHub, we can also look at the journals that team members opt to record throughout their workdays.

We also source vacation times to keep everyone informed.

## Other usages and future work

We already use this automated tool beyond the scope of daily and weekly team updates. One such example is that we have been using it to generate quarterly and yearly reports to capture all the work done by the team for audit and [SR&ED](https://www.canada.ca/en/revenue-agency/services/scientific-research-experimental-development-tax-incentive-program.html) (Scientific Research and Experimental Development, a Canadian federal government tax incentive program) purposes.

We are looking to incorporate more NLP assistance to deduplicate between various sources.
