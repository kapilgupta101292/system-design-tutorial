---
id: system-design-calculations
title: Back of the Envelope Calculations
sidebar_label: Back of the Envelope Calculations
---

![Back of the Envelope Calculations](/img/calc.png)

**Back of the Envelope Calculations** or **Ball Park Estimations** are used to roughly estimate a ballpark figure quickly using approximataion and rounding off. This method gives good results within a error range of exact results using precise calculations.

When it comes to System design it is a useful technique to come with such estimations, to understand whether your design fulfills the requirement of the system or not. If you have certain alternative ideas, using estimations we can choose the one which provides best value.

In System Design interview we do back of the envelope calculations to answer questions like

1. How many servers are need to be able to fulfill all the concurrent requests at peak traffic ?
2. How much disk space is required to store the data and backups ?
3. How much Bandwidth is required to serve x request/second ?
4. How much RAM is required to cache hot data ?
5. How much latency will be there for each request at peak traffic ?

In general for any System Design we need to do Traffic estimates, Storage and Memory estimates, Bandwidth estimates and Latency estimates. As cost is directly proportional to amount of resources used, Most often, the choice of architecture is limited by the most critical resource that can become a bottle neck. Hence we need to identify it.

In an actual interview you would get around 5 minutes to come up with your estimates and prove that your design fulfills the goal of required system. Hence the key is to make correct assumptions and then approximate and round off calculations to be able to get to the result faster.

Here are useful resources to keep in mind, while doing such calculations.

1. https://gist.github.com/jboner/2841832
2. https://servebolt.com/articles/calculate-how-many-simultaneous-website-visitors/
3. https://html.developreference.com/article/11537120/Why+is+the+estimation+of+system+constraints+needed+in+system+design+interviews%3F
4. https://wso2.com/whitepapers/capacity-planning-for-application-design-part-1/
