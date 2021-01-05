---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Sailing Autonavigate: Kickoff"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-01-04T21:48:24-07:00
lastmod: 2021-01-04T21:48:24-07:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

If you don't have a special interest in sailing, you probably haven't heard of [Vendee Globe](https://en.wikipedia.org/wiki/Vendée_Globe). It is an round-the-world sailing race that takes place every 4 years the last of which is happening as I am writing the blog post. The racers are not allowed to set foot on the ground for months. I left the most interesting detail to last. The race is single-handed, meaning only one person is allowed on the boat. Therefore it requires an insane amount of sailing skills and mental toughness. This blog post will introduce you the next side-project I'm working on with a few other sailor/engineer friends of mine, for which the idea was influenced from this race.

Vendee Globe racers have to solve tons of problems on a daily basis. One of those problems is to plot the optimum route on the ocean that will take them fastest to the checkpoints set by the rules of the race and hopefully finish the race in front of other competitors. To the untrained eye, this would seem like an easy one. It's plain water, no city roads or traffic. Why not go straight to each of the checkpoints, right? *Wrong*. There are two major reason why the problem is more complicated than it looks: (1) the varying boat speed with respect to wind direction and (2) the wind itself.

Let's start from the latter. The wind patterns are complex and predictions are not exact. Play the widget below, embedded from a popular wind prediction and visualization tool, Windy. Generally speaking, the racers want to know the wind because they want to avoid extremely light or strong wind conditions.

<iframe width="650" height="450" src="https://embed.windy.com/embed2.html?lat=30.145&lon=-31.641&detailLat=33.466&detailLon=-111.996&width=650&height=450&zoom=3&level=surface&overlay=wind&product=ecmwf&menu=&message=&marker=&calendar=now&pressure=&type=map&location=coordinates&detail=&metricWind=default&metricTemp=default&radarRange=-1" frameborder="0"></iframe>

They also care about the angle of the wind. Sailing boats don't go the same on every speed and every angle. They typically tend to go the fastest, when the wind is coming from the sides of the boat. Combined with the everchanging speed and direction of the wind, it becomes a challenging task for the racer to plot the course that would get her fastest from point A to point B on the ocean.

{{< figure src="https://www.researchgate.net/profile/Sio-Hoi_Ieng/publication/265139871/figure/fig4/AS:613919513141249@1523381226064/Speed-polar-diagram-of-the-sailboat.png" title="Speed polar diagram of a boat, taken from [Toward an Autonomous Sailing Boat](https://www.researchgate.net/figure/Speed-polar-diagram-of-the-sailboat_fig4_265139871)." >}}

Looking at the official tracking tool of the race, we see that no racer goes the same exact course as another. Basic manouvers and early positioning can lead to days of separation between boats given the race takes months.
<iframe width="650" height="450" src="https://tracking2020.vendeeglobe.org/en/" frameborder="0"></iframe>

## The Project Idea
The question we are after is simple: can we build a bot to chart us the optimal route from the point A to point B? It will be an interesting engineering project in it's own way but I also see it as a great way to keep connected with my friends during this work-from-home situation given the pandemic. At the time of writing this post, we already had two meetings and sketched up with a rough gameplan. This post is only to announce the project but I will update with further details as I have more time. Until now, thank you for reading this far and feel free to hit me up with questions if you have any.
