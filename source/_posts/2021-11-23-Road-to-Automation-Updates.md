---
title: 'Road to Automation: Updates'
date: 2021-11-26 15:17:15
tags:
  - programming
  - software development
  - automation
---

It is crucial for any work involving team collaboration to get a sense of each others’ whereabouts as quickly and accurately as possible. Usually, we resort to frequent meetings to get updates from each other.

But meetings are not always the best way for this scenario:
* Meetings are synchronous, so everyone has to wait for the others to finish before they can continue.
* Especially when communicating verbally, it is difficult not to misinterpret each others' words.
* It is challenging to scale up the team size because that would amount to more meetings and more probable misinterpretation of the updates.
* It is challenging to work as a distributed team if the team scatters around the world.

Building on top of the experience to {% post_link Road-to-Automation-Release-Notes 'automate our product release process' %}, we can now automate the process to [get daily and weekly updates](https://github.com/EQWorks/updates) from each other from exactly where work happens and consume them asynchronously at our own pace.
<!-- more -->
## Source from where work happens

Instead of listening to what each other has to say about the work, we can look at exactly what the work is and how it is done.

## Source from journals

To supplement concrete work progress from GitHub, we can also look at the journals that team members opt to record throughout their workdays.

## AI-assisted curation

As team grows in size and work grows in complexity, the amount of updates can be overwhelming. So rational curation is needed to help each individual to sift through and find relevant information for themselves.

## Future work

We already use this automated tool beyond the scope of daily and weekly team updates. One such example is that we have been using it to generate quarterly and yearly reports to capture all the work done by the team for audit and [SR&ED](https://www.canada.ca/en/revenue-agency/services/scientific-research-experimental-development-tax-incentive-program.html) (Scientific Research and Experimental Development, a Canadian federal government tax incentive program) purposes.