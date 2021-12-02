---
title: 'Road to Automation: Updates'
date: 2021-11-26 15:17:15
tags:
  - programming
  - software development
  - automation
---

Team collaboration must get a sense of each others' whereabouts as quickly and accurately as possible. Usually, teams resort to recurrent meetings to get updates from each other.

But meetings are not always the best way for this scenario:
* Meetings are synchronous, so everyone has to wait before continuing for the others to finish.
* Especially when communicating verbally, it is difficult not to misinterpret each others' words.
* It is challenging to scale up the team size because that would amount to more meetings and more probable misinterpretation of the updates.
* It is challenging to work as a distributed team where members can reside in different time zones.

Building on top of the experience to {% post_link Road-to-Automation-Release-Notes 'automate our product release process' %}, we now automate the process to [get daily and weekly updates](https://github.com/EQWorks/updates) from each other from exactly where work happens and consume them asynchronously at our own pace.
<!-- more -->
## 👀 where work happens

Instead of listening to what each other has to say about the work, we can look at the actual execution. Since we already use GitHub to collaborate, we can consume the various signals around PRs (pull requests) and issues.

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

A member can scan through the updates and go to the PRs or issues they are interested in for more details. The updates allow us to gather insights and ask more informed questions to the relevant contributors.

## Assisted curation

To help us sift through noises, we incorporate status symbols and what we have done in our [release CLI](https://github.com/EQWorks/release) by utilizing NLP models to label the PRs.

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

To supplement concrete work progress and issues from GitHub, we also look at elective journals that some of our team members opt to record throughout their workdays (see our [`journal-bot`](https://github.com/EQWorks/journal-bot/) project for more details.) The journals are an excellent way to understand each team member's focus throughout the day.

On top of the journals, we also source information on who is on or to have upcoming vacations, so teams can proactively work around potential blocks without some members.

> **Vacations**
> - redacted1@eqworks.com - Mon, Nov 15 to Fri, Nov 19, 2021
> - redacted2@eqworks.com - Mon, Nov 29 to Fri, Dec 3, 2021
>
> **Journals**
>
> KC
>
> Did:
> - connection-hub - side connection dropdown selection
>
> Doing:
> - lumen-table - test filtering options with virtualized table
> - connection-hub - new connection UI modal design
> - connection-hub - test each connection & fix small bugs
> - connection-hub - optimize state management
>
> Erika
>
> Did:
> - react-maps/LocusMap - only show MVT tile polygons that have data if possible #106
> - flood-maps - implement select locations #2
> - flood-maps open #1 & #2 PRs for review
> - kepler.gl - review [BUILD] v1.1.17 - custom build to avoid dependency conflicts #1
> - snoke - review [G2M] Deps/kepler sidegrade #1030
> - react-maps/LocusMap - create [WIP] LocusMap - fix cursor and set visible and pickable for Select layer; allow setting MVT polygon strokes to transparent for no data #106
flood-maps - capitalize locations and close #2
>
> Doing:
> - react-maps/LocusMap - implement onClick - later
> - react-maps/LocusMap - implement select data process - later
> - widget-studio/map-widget - create controller for MVT layer
> - widget-studio/map-widget - create controller for polygon GeoJSON layer in case we use province coordinates for GroupBy
>
> _(more)_

## Future roadmap

We already use this automated tool beyond the scope of daily and weekly team updates:
- We have been using the updates tool to generate quarterly reports to capture all the work done by the team for audit purposes.
- Similarly, we started to utilize the tool to generate yearly reports for our [SR&ED](https://www.canada.ca/en/revenue-agency/services/scientific-research-experimental-development-tax-incentive-program.html) (Scientific Research and Experimental Development) documentation and filing reference. As a result, we significantly cut down the cost to interview individual contributors who may or may not recollect what they have done during the past year.

As for the roadmap:
- We want to add more sources, such as updates from Zeplin, Figma, Google Calendars, and many more, to balance between personal preferences of tools and channel relevant information back to a more common destination for everyone to ingest.
- We are looking to enhance our NLP assistance to deduplicate between various sources to [reduce similar journal and GitHub items](https://github.com/EQWorks/updates/blob/nlp/similarity/nlp/similarity.ipynb). ![sim](https://user-images.githubusercontent.com/2837532/144511628-ebc59647-0f42-4f9e-9c7f-77522ef08936.png)
