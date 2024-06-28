---
title: "Procedural Space Stations 3"
author: ""
type: ""
date: 2024-06-14T15:18:33+02:00
subtitle: ""
image: ""
tags: []
draft: true
bigimg: [{src: "verticrop.jpg", desc: "Lettuce in a vertical farm"}]
---


In the [previous blog post](https://barthpaleologue.github.io/Blog/posts/procedural-space-stations-2/), we found the ideal size for our habitats in order to produce comfortable artificial gravity.

To finalize our size requirements, we now need to estimate the surface needed to sustain a given population. The energy requirements have already been settled in the [first post of this series](https://barthpaleologue.github.io/Blog/posts/procedural-space-stations-1/), and now is the time for food production!

## Space production vs planetary imports

The first question we need to ask is whether the food supply comes from local production or imports. Importing would save a lot of space, but it would undermine each station's autonomy dramatically. What if the supply chain is broken? The inhabitants of the station would starve without being able to do anything about it.

But then on Earth, food is not produced in the middle of the city and a supply chain disruption could also be dramatic. While I agree, I would argue it is always harder to figure out solutions when you are stranded in space.

Moreover, imports would mean lifting tons of cargo from the surface of planets to bring them to the station, which is costly.

The true answer to this kind of binary questions is almost always: both! You can produce locally to cover your basic needs, and then import to get a greater variety of products, and also trade your products as well with other stations for example.

This means every station will have an importation/local production balance for food supply. There will also be a small surplus in order to export to other stations. We have the following relation:

$$
F_{local} = F_{needed} - F_{imports} + F_{exports}
$$

## Estimating the required calories

According to [NHS](https://www.nhs.uk/live-well/healthy-weight/managing-your-weight/understanding-calories/), a man needs 2,500 kcal each day on average, while women need 2,000 kcal

We will consider a population composed of 50% male and female individuals for the computations. We will use 2,250 kcal per day per person:

$$
F_{needed} = 2.250 \times N
$$

With `N` being the population of the station. We can rewrite the 1st formula as follows:

$$
F_{local} = 2.250 \times (N - N_{import} + N_{exports})
$$

With `N_imports` and `N_exports` representing the number of persons sustained by the imports and exports.

## Estimating the surface needed

The surface needed to produce a given quantity of food depends on many factors: climate, crop species, farming methods and more!

Luckily for us, a space station is a closed environment with a constant controlled climate. This will simplify our calculation.

For the crop species, we have to consider them separately to compute the resulting area needed. For example, 100g of tomato only provide 18 calories while 100g of rice provide 130 calories. Depending on the mix of each space station, we will get different results!

The more interesting part I think is the farming method. Do we use biological farming? With intrans? What about hydroponic farming and even aeroponic?

This [stack exchange discussion](https://worldbuilding.stackexchange.com/a/9601) gives interesting insights on the matter. Compared to regular farming, hydro and aeroponics provide superior yields by a factor of about 250%. Moreover, they are stackable. This means we can divide the required surface by the number of farming layers we want. This can save a ton of space, so our space stations will go the hydroponic way.

Fun fact: Nasa is already experimenting with hydroponics in space: https://science.nasa.gov/science-research/science-enabling-technology/technology-highlights/how-do-you-water-plants-in-space/

I will use the data from https://www.fao.org/4/t0207e/T0207E04.htm#4.%20Nutritive%20value to get the `kcal/ha/day` that we need to compute the surface. The formula for a single type of crops is as follow:

$$
S = (N - N_{imports} + N_{exports}) \frac{E_{individual}}{E_{crop} N_{layers}} \newline
\text{where: } E_{individual} = 2,250 \text{ kcal/day} \newline 
E_{crop} \text{ is the edible energy of the crop type in kcal/ha/day} \text{ and}  \newline
N_{layers} \text{ is the number of layers used in our hydroponic infrastructure}
$$

Consequently, our result is in `ha`, which we can convert back to `m²` by multiplying by `1000`.

And when we consider the 250% factor, we get:

$$
S = (N - N_{imports} + N_{exports}) \frac{E_{individual}}{3.5 E_{crop} N_{layers}}
$$

For each crop that grows in our space station, we assign a weight `a_crop`, representing its share of the local production.

This gives us the complete formula:

$$
S = (N - N_{imports} + N_{exports}) E_{individual} \sum_{crops} \frac{a_{crop}}{3.5 E_{crop} N_{layers}}
$$

## Results

Let's put this formula to use with a simple case: no imports and no exports, and we only produce rice on a single hydroponic layer:

$$
S = N E_{individual} \frac{1}{3.5 E_{rice}}
$$

We will consider a population of 300,000 people. Still according to https://www.fao.org/4/t0207e/T0207E04.htm#4.%20Nutritive%20value, the yield of rice is `49,000 kcal/ha/day`.

Putting all of this together, we get about `4,000 ha` necessary: `40 km²` or `400,000 m²` depending on your unit of choice. If this is too much for you, we can simply stack hydroponic layers on top of each other (like 50 for example), putting us under a single square meter!
