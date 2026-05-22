---
title: "Procedural Space Stations #1"
author: "Barthélemy Paléologue"
type: ""
date: 2024-06-02T22:12:14+02:00
subtitle: "Modelling solar panel surface from energy requirements"
image: ""
tags: ["cosmos-journeyer", "Space station", "Physics"]
bigimg: [{src: "banner.png", desc: "The International Space Station and its 8 solar arrays"}]
---

How can we create realistic looking space stations? How to make them feel grounded and functional? That's what I want to explore it this series of blog posts while developing the new space stations for [Cosmos Journeyer](https://cosmosjourneyer.com). Here is a sneak peak:

{{<figure src="cosmos.png" alt="View from Cosmos Journeyer's space station" caption="View from Cosmos Journeyer's space station" caption-position="bottom">}}

In this first one, we will talk about solar panels. They are an iconic part of the look of space stations: from Mir to the ISS, and Tiangong to the upcoming Lunar Gateway, they all have them! So it's only natural that my procedural space stations will have them too.

I will create a model to find the solar panel area required to power a given space station. I will use the International Space Station as an example as we have a lot of data on its internal power consumption.

## Why solar panels?

Before anything else, why do we need solar panels for? This might sound trivial but answering this questions leads to interesting insight in the design of space stations.

The first reason is that solar energy is freely available in space, with a greater efficiency than on the ground since you don't have the atmosphere acting like a filter.

But they take a lot of space, why not use some kind of small modular nuclear reactor (SMR) instead? Well we still have to consider heat management. For most of history, mankind has always been using heat to boil up some water to then rotate a turbine to generate electricity. Those are complex thermodynamic systems that need cooling, especially in space where you can lose energy only with radiation.

On the other hand, solar panels do not rely on the heating water trick. They use the photoelectric effect to directly generate electricity. This make solar panels ideal because they don't create as much heat and have a much simpler design (when not taking folding into account).

## Energy requirements

First, we must estimate how much energy we need to power our space station. In the case of the ISS, we find that the energy consumption is about 90kw (https://www.edn.com/international-space-station-iss-power-system/) to sustain a crew of 7 astronauts.

When considering larger stations that would work more like cities, we might consider the average energy consumption per capita in the US, which is about 80,000 kWh per year, so about 9kW per person. This is a rough estimate, but it gives us a good starting point. For now we will focus on the ISS to sanity check our model.

## Energy efficiency

The next important parameter is the energy efficiency. It is defined as the ratio of the energy produced by the panels vs the total energy received from the sun. Solar panels will reach about 40% efficiency in the near future, so we will use that for Cosmos Journeyer. However, the ISS is much older, so the efficiency we need for our computation will be lower. Let's calculate it to understand how it works!

### Efficiency of the ISS solar panels

According to https://www.edn.com/international-space-station-iss-power-system/, the ISS solar panel surface is about 2,500 m² and produces 120kW of power. To get the efficiency, we must first compute how much energy is received from the sun. You might find weird that we use the surface of solar panels of the ISS to compute the solar panel efficiency in order to compute the solar panel surface. This is only because I want to check our model gives plausible results, and to prove I'm not cheating.

### Solar energy received

This is the part with some math, but I will try to make it as painless as possible.

{{<figure src="thermo.gif" alt="Thermodynamics">}}

The energy we receive from the sun depends on multiple factors:

- Its temperature: if the sun is twice as hot then we will get toasted very quickly.
- Its radius: if the sun was twice as big, we would also have a bad time.
- Its distance: if we were twice as far from the sun, we would be frozen in thick ice.

Moreover, the sun is in space (yeah, I know, big surprise), so we can rule out convection and conduction for heat transfers. This leaves us with radiation as the only way to transfer energy from the sun to the ISS.

We can compute the radiation flux coming from every unit of surface of our sun with the Stefan-Boltzmann law:

$$
\Phi = \sigma T^4 \quad \text{[W m}^{-2}\text{]}
$$
$$
\text{where: } \sigma = 5.67 \times 10^{-8} \text{ W m}^{-2}\text{ K}^{-4} \ \text{and} \ T = 5,778 \text{ K}
$$

Sigma is the Stefan-Boltzmann constant, and T is the temperature of the sun.

If we want to compute the total energy radiated from the sun, we simply multiply the flux by the surface of the sun.

The sun is a sphere so its surface is given by:

$$
S = 4 \pi R^2
$$

where R is the radius of the sun, which is about 696,340 km = 696,340,000 m. Just for comparison, Earth is only 6,371 km = 6,371,000 m.

{{<figure src="image-1.png" alt="It's big. src: https://www.universetoday.com/wp-content/uploads/2010/05/sunearthcompared.jpg" caption="It's big. src: https://www.universetoday.com/wp-content/uploads/2010/05/sunearthcompared.jpg" caption-position="bottom">}}

We get the following formula for the total energy radiated by the sun:

$$
E = \Phi S = 4 \pi \sigma T^4 R^2
$$

But we want to know the energy received by the ISS, which accounts for the distance to the sun. As we get further away, the energy of the sun gets spread out over a larger area (think about a pebble thrown in a pond: the wave will start strong but fade away the further it goes because of the energy spread). 

{{<figure src="./flux.png" alt="What an awesome illustration!" caption="What an awesome illustration!" caption-position="bottom">}}

If we stand at distance D from the sun, the total energy emitted by the sun will be spread over a sphere of radius D. Therefore we can compute the energy flux received by the ISS as:

$$
\Phi_{\text{ISS}} = \frac{E}{4 \pi D^2}
$$

Notice how the 4 pi cancels out. We get the final formula for the energy flux received by the ISS:

$$
\Phi_{\text{ISS}} = \sigma T^4 \frac{R^2}{D^2}
$$

We get a nice inverse square law here, which is a common occurrence in physics, as a lot of stuff spreads out in a spherical way (think about sound, gravity, etc.).

When taking D as the distance between the Earth and the sun, which is about 149,600,000 km = 149,600,000,000 m, we get a flux of about 1,360 W m\\(^{-2}\\).

### Back to the ISS

Did you survive the math? Congratulations! Now that we know the flux of energy received by the ISS, and that we know the solar panel surface of the ISS, we can get the total energy received by just multiplying the two:

$$
E_{\text{ISS}} = \Phi_{\text{ISS}} S_{\text{ISS}}
$$

The resulting energy is about 3,400kw for 2,500 m² of solar panels. This gives us an efficiency of about 7%. But that's considering the solar panels are always lit by the sun. The situation is more like this:

{{<figure src="./shadow.png" alt="ISS orbiting the Earth">}}

As the ISS spends half of the time in the shadow of the Earth, we will divide the energy produced by 2:

$$
E_{\text{ISS}} = \frac{\Phi_{\text{ISS}} S_{\text{ISS}} \eta_{\text{ISS}}}{2}
$$

where eta is the efficiency of the ISS solar panels.

## Find the solar panel surface of the ISS

Now we can check if this equation gives us back a coherent value for the surface are of the ISS' solar panels. We can rewrite the equation to isolate the surface as follows:

$$
S_{\text{ISS}} = \frac{2 E_{\text{ISS}}}{\Phi_{\text{ISS}} \eta_{\text{ISS}}}
$$

As a refresher, we have the following values:

- required energy: 90,000 W
- energy flux from the sun: 1,360 W/m²
- efficiency of the ISS solar panels: 7%

Which gives us 1,890m² of solar panels. We are in the same range! But the result is not exact, why is that? Well, the ISS does not produces exactly the amount of energy necessary to sustain itself. It generates a bit more to recharge batteries and to have some margin in case of emergency. The energy produced is more like 120kW. Plugging this in our equation, we get back 2,521m², which is very close to the actual value of 2,500m².

{{<figure src="image-2.png" alt="Great success!" attr-link="https://1.bp.blogspot.com/_5PFJfNCAwkM/TGXdmJm5toI/AAAAAAAABHY/zDGeu9CnjH0/s1600/borat_great_success-450x337.jpg">}}

This little exercise demonstrates that our model is coherent with the data we have on the ISS. We will now use it to generalize to Cosmos Journeyer's space stations.

## Putting Nantes into space

For Cosmos Journeyer, I see space stations being analog to our cities on Earth. This gives us a rough number of people that should be sustained on board. I will take Nantes as an example, as it is the city I spent most of my childhood in. Moreover, there is a huge debate to know if Nantes is part of Bretagne or not, so we will just put it into space to piss off everybody.

{{<figure src="nantes.png" alt="Nantes in space" caption="Nantes in space" caption-position="bottom">}}

Our population increases dramatically: from 7 crew members, we go to about 323,000 people. The energy consumption is also not the same, as this is no longer a scientific space station but a space city. In France, the energy consumption per capita is about 40,000 kwh per year, so about 4.5kW per person. This gives us a total energy consumption of about 1.45GW. The ISS uses one more third of the energy used to recharge batteries, so we can also account for that and get a total energy production of about 1.9GW.

For the solar panel efficiency, we will use 40% as it is a realistic value for the near future.

Finally, for the star parameters, we will use the values of our sun for this exercise, but our formula works for any kind of star at any distance, so tweaking it is easy.

Running the numbers, we get about 7km² of solar panels. That's a huge surface, but there is plenty of space out there in space!

## Conclusion

We now have a robust model to compute the solar panel surface required to power any space station. We have validated it with the ISS and used it to compute the solar panel surface for a space city the size of Nantes.

If you want to play with the model, I have put it in a Jupyter notebook that you can find [here](https://github.com/BarthPaleologue/CosmosJourneyer/blob/927fc2aad24245d3c56e9eed8311884f2a6c4451/research/solarPanels/solarPanels.ipynb).

The next step will be to design believable habitats for our space city and explore artificial gravity. Stay tuned!  