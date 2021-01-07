---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Sailing Autonavigate: The Problem"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2021-01-06T07:24:24-07:00
lastmod: 2021-01-06T07:24:24-07:00
featured: false
math: true

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
{{% callout note %}}
This is the second post on the series that explains the side project I just started working on: Sailing Autonavigate. If you haven't done so, I suggest reading [the kickoff post]({{< ref "/post/sailing-autonavigate-kickoff/index.md" >}}) to understand the context behind it.
{{% /callout %}}

In this post, my goal is to introduce you to the nitty-gritty details of the problem. To make it more fun, I will assume a scenario where one of the Vendee Globe racers is our potential user. We will try to build the Autonavigate product for her.

## The Winning Formula
Following the Design Thinking process, let's focus on what our user really wants. At the end of the day, our users want to win the race, period. What are the elements of winning? What winning really means is to sail the boat faster than all the other competitors from the Start line to the Finish line. Defining the problem more explicitly helps us breaking it down even further.

{{< figure src="AutoNavigation-DesignThinking.svg" title="Breaking down the winning formula. (Click to enlarge)" >}}

Let me explain some of the terms here for the non-sailor audience:
- **Performance of Competitors:** This simply means how well others race, i.e., how fast they can get their boats from Start to Finish.
- **Trimming Skills:** Even if you have the most optimal strategy to win the race, you still need to trim your sails and trimming elements on your boat optimally with respect to wind speed, angle and wave conditions. In other words, you get the most out of your boat's potential by doing this. 
- **Boat Specs:** Vendee Globe enforces box rules to dictate some specs that affect boat performance. These rules are in the form of constraints and the designers are free to make choices as long as they keep within these constraints. The boat specs of other boats will determine the performance of competitors. By the term "Boat Specs", we specifically refer to the boat specs of the boat our user is going to be racing in.
  - **Polar Diagram:** Polar diagram is a set of measures that is determined by other low-level boat design decisions (boat lenght, sail cuts and materials etc.). It is certainly not the sole consideration when designing a race boat. However, it is going to be the only relevant spec of the boat that we care about when we are designing the navigation logic.
- **Wind Forecast**: We cannot know exactly what the wind is going to be in the future. This is why we use forecasting tools.

This breakdown helps us in two ways: (1) we find our niche market, (2) we make our simplifying assumptions explicit. 

### The Niche Market for the Hypothetical Product
Our niche market is the market for building navigation logic for racing boat. The value we add to our user is to increase navigation capabilities through imrpoved navigation logic, and eventually help her win the race.

### Simplifying Assumptions
Now we can start populating the list for simplifying assumpions:
- *Our logic will be agnostic to the state of other racers*. In other words, we won't have access to where the other racers are, in what direction and how fast they are going or the weather conditions they are experiencing. It is actually a good practice in sailing races to watch what others are doing, especially if you are behind, to look out for opportunities of favorable wind or to avoid dead wind zones. However, for the time being, this would make the minimun viable product unnecessarily complicated, so we leave it out.
- *Trimming skills of our skipper is perfect*. Whatever recipe we give to our skipper, she will execute it perfectly. The implication of this for our navigation logic is that we will take the boat speed optimal no matter which direction we are heading towards or what the weather conditions are. The information about "optimal boat speed given a weather conditions and heading" is embedded in the polar diagram.
- *Wind forecasts are given*. We won't attempt to do our own wind forecast but instead take the forecasting model as given an try to find the best route possible with respect to it. Obviously, the quality of the forecasting model will be a limiting factor for our performance.

The list will grow larger as we move further but for now it seems good enough.

## The Math Behind
We shall get serious by translating the problem at hand into mathematical terms.

We can represent the problem as an optimizitation problem, borrowing some ideas from Optimal Control.

$$
\begin{aligned}
\min_{u(.),\tau} \quad & \tau \\\\\\
\textrm{s.t.} \quad & \dot{x}(t) = v(x(t), u(t), t) \\\\\\
                    & x(0) = x_s    \\\\\\
                    & x(\tau) = x_f    \\\\\\
\end{aligned}
$$

Let $x(t)$ represent the coordinates of the boat at time $t$. The boat starts from the initial coordinates $x_s$ and wants to make it to the checkpoint $x_f$ as fast as possible. When the problem is solves, it shall return *an optimal control* $u(t)$, a function that denotes what heading with respect to world coordinates the boat should be heading to at each time $t$. We minimize $\tau$, the total trip time and thus the endpoint constraint $x(\tau)=x_f$ guarantees that the boat will end up exactly on the checkpoint at the end of total sail time. Finally, $\dot{x}(t)$ represents the instantenous velocity of the boat over ground at time $t$. It is dependent on the location $x(t)$ and time $t$ because that will determine the wind the boat will observe. It also depends on $u(t)$, because combined with the wind direction and speed, that will determine the velocity of the boat. Observe the polar diagram chart from the last post again to understand this.

{{< figure src="https://www.researchgate.net/profile/Sio-Hoi_Ieng/publication/265139871/figure/fig4/AS:613919513141249@1523381226064/Speed-polar-diagram-of-the-sailboat.png" title="Speed polar diagram of a boat, taken from [Toward an Autonomous Sailing Boat](https://www.researchgate.net/figure/Speed-polar-diagram-of-the-sailboat_fig4_265139871)." >}}

Polar diagram is basically a function of wind speed, wind angle and the heading of the boat. It takes these arguments and outputs the optimal speed of the boat under perfect sail conditions and trimming.

{{< figure src="CoordsTimeHeading-To-Velocity.svg" title="From coordinates, time and heading to velocity over ground." >}}

Now that we have a full understanding of the problem, I can get to the bitter news. The optimization problem is not easy to solve and it may not have analytical solutions. I am not aware of a solution method for a problem given in this structure. Besides, wind forecast and polar diagram functions are going to be non-smooth. Not all hope is lost, though. A very common technique to deal with such problems, especially in the realm of Optimal Control, is to come up with numerical alternatives. We are actually working on a few alternatives at this point, which I will introduce in a later blog post. But first, we need to talk about something just as important as the method itself.

### How to Measure Success?
Let us first define what a scenario is. A scenario is simply composed of the following:
- **A scenario field**. The geographical context in which the race is happening. This could be a little pond for RC boats or the entire three oceans in the case of Vendee Globe.
- **The start and finish coordinates**. These could be any two points selected within the scenario field.
- **Real wind conditions**. Defined for the scenario field over the time limit of the race, the wind conditions at any point and any time.
- **Wind forecasting tool**. The tool we use predict the wind into the future. For starters, we can assume this to be perfect but in the future, it might be interesting to dive into the realm of Robust Optimization given we know or can estimate the probabilistic structure of the errors of this tool.
- **Polar Diagram**. This was explained above. A boat specific measure, thus boat itself is part of the scenario.

Since we cannot solve the above optimization problem, we will never know what the global optimum is for any given scenario. Therefore will never know the optimality gap, or in other words, how far are we from the best possible solution. To measure success, we have to start from the other way around. That is, we shall come up with a viable baseline solution and iterate over it.

> A solution iteration should be acceptable if it outperforms the previous baseline on a varied set of scenarios.

This brings us to the next stage in the project, and hopefully the next blog post. We have two major steps to take care of:
1. Define one or more racing scenarios.
2. Implement a baseline solver.

See you in that post and until then, take care!