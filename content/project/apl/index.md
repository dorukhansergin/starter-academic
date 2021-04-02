---
title: Active Preference Learning with Discrete Choice Data
summary: A Python repo for modeling user preferences with pairwise queries from a set of items.
tags:
- Bayesian Learning
- Preference Learning
- Gaussian Processes
date: "2021-04-01T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Photo by rawpixel on Unsplash
  focal_point: Smart

url_code: "https://github.com/dorukhansergin/APL-Brochu"
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---

# Active Preference Learning with Discrete Choice Data
I just released an unofficial implementation of the [Active Preference Learning with Discrete Choice Data](https://papers.nips.cc/paper/2007/hash/b6a1085a27ab7bff7550f8a3bd017df8-Abstract.html) by Brochu et al. as published in NIPS 2007.

### Why would this package be useful for you?

> Imagine a scenario where you are trying to find a place to have lunch today. 
> There are tons of places to eat around. 
> An app suggests you two restaurants to compare at a time and can help you reach a good enough restaurant in as few queries as possible.
> Each time you pick a restaurant, the model gradually learns what you want and hopefully, its suggestions get gradually better.
> This will save you a lot of time and when designed well, can be a much more fun way to search as opposed to going through a boring list view.

In general, if the following conditions are present, active preference learning can be useful for you:
- User is searching for an item in a very large set of items that's impossible go through one by one.
- User is okay with a good enough solution if it is going to be found shortly.
- Items can be embedded in a vectors space where proximity in that space implies similarity in preference between items.

{{% callout note %}}
Interested? Go to the [repo](https://github.com/dorukhansergin/APL-Brochu), install it on your system and play with the demo!
{{% /callout %}}

### Why not other rating-based methods as a way to model preferences?
Amazon uses 5-star ratings as feedback for items by users, also called [Likert scale](https://www.simplypsychology.org/likert-scale.html). 
Stitch Fix uses [Style Shuffle](https://www.stitchfix.com/women/blog/inside-stitchfix/how-style-shuffle-works/), which is basically a liked or didn't like decision.
Then, why would you use binary comparisons instead of these two options?
According to [Kingsley and Brown (2010)](https://www.fs.fed.us/rm/value/docs/preference_uncertainty_learning_paired_comparison.pdf), people get more consistent as they progress towards binary comparisons of alternatives. This might be a cleaner feedback from the user as opposed to absolute ratings such as 1-to-5 scale or like-or-dislike input.
This is the motivation behind this paper.

## Why I built this
I just think amazing things can be made with active preference learning.
Especially given that we are in the age of GANs and other powerful generative models.
Imagine a GAN creating music pairs of music and the user rates them to quickly generate enjoyable tunes.
Or quick art forms that reflect the current mood of the user.

I couldn't find well-maintained repos that provide active preference learning alternatives. I wanted to spark a fire.
I hope me or someone else can extend this library further and built great things with it.

## Roadmap
- [ ] Extend the package such that it models two more options: 'liked both' and 'disliked both'.
- [ ] Allow the option to search in the (possible bounded) continous space as opposed to the finite set of elements.
- [ ] Incorporate a more accurate posterior inference engine than Laplace's approximation.
- [ ] Create more examples.
- [ ] Package for PyPI, add CI/CD procedures and more tests.

