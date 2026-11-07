---
author: ["Setaareh Nikbin"]
title: "Making a Television for Cellular Automata"
date: "2026-11-07"
description: "It's about presentation."
summary: "It's about presentation."
tags: ["shaders", "code",]
categories: ["projects"]
ShowToc: false
TocOpen: false
weight: 3
---

{{< figure src="images/gavazn_full_sample.gif" alt="Gavazn Machine" align="center">}}

### Visualizing Cellular Automata

If you wanted to experiment with [cellular automata](https://en.wikipedia.org/wiki/Cellular_automaton) in a mature and highly functional environment, there's already [Golly](https://golly.sourceforge.io/). If you wanted to see how something like [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life) could be run entirely on the GPU, there are already a bundle of examples [on Shadertoy](https://www.shadertoy.com/results?query=game+of+life).

But my own goal with this project was to create a cellular automaton visualizer that played like a television from the 70s, with two key aesthetic notes: the mechanically chunky, responsive interface, and a CRT shader to give the little cells on the screen warmth and softness. I wanted to create a machine with tactile form and feedback that invites the user to play with it.

### The Simulation

I needed the project to run in-browser, so a Typescript SPA running WebGL seemed the natural choice. But I've never touched shaders before, much less had shaders run a simulation! The above shadertoy posts provide ample models for how to run a cellular automaton on the GPU, but I found [this explainer](https://benpm.github.io/blog/gol_2/) by Benjamin Mastripolito especially pithy on how to tackle the main problem: calculating the combinatoric rank of neighbourhood states and translating it to a position in a rule texture. The [WebGL Fundamentals](https://webgl2fundamentals.org/) website and [Andrew Adamson's Youtube tutorial series](https://youtube.com/playlist?list=PLPbmjY2NVO_X1U1JzLxLDdRn4NmtxyQQo) were also invaluable for helping me tell left from right in WebGL.

The simulation itself is arbitrarily set to a `1024 x 1024` rectangle with toroidal periodicity. Meanwhile, the rule texture is a `8192 x 8192` square; the larger the texture, the larger `N` the simulation can accommodate, but there are diminishing returns for larger powers of two and obvious limits to storing and manipulating textures. This particular choice allows for `N` up to 21, which also means that the knob controlling `N` can divide neatly into four quarters. 

### The Screen

There are some "lightweight" CRT shaders that essentially superimpose scanlines, some noise, and maybe a bit of chromatic aberration on the underlying image. These can be computationally cheap and do not force the user to think about the resolution of the image, but they are also unsatisfying: they don't actually *look* like CRT displays beyond a passing glance, and are certainly not suited to a project where you'll be looking at the display the whole time!

I decided instead to homebrew a shader as part of my crash course in GLSL, and the first choice I had to make was a matter of scale. Here is the problem: if you want to create a CRT shader that takes seriously the mechanics of a pixel being projected onto the display, each pixel in the base image needs to be assigned a larger space in the output to capture the various analog effects. This is a tradeoff—a more aggressive projection can capture finer details of the screen, but it will also mean that a larger digital screen is required to display the same original image. 

My understanding is that [CRT-Royale](https://github.com/libretro/common-shaders/tree/master/crt/shaders/crt-royale) uses a `9x9` projection, so that a 4K screen can display exactly a 240p image. This scaling works well for a lot of classic games, but is a non-starter for a cellular automaton visualizer: some of the most interesting behaviour cellular automata exhibit occurs at scales of hundreds of cells, and I want users who don't have 4K screens, or even 1080p screens, to be able to observe this behaviour! Nevertheless, *some* scaling factor is required to achieve at least passable emulation of CRT behaviour.

After negotiating with myself for a while, I settled on a `4x4` projection. This squishes some discrete behaviour like chromatic aberration into rather too few pixels, and messes with periodicity: the three channels of R, G, and B split into fours. Still, after implementing the whole pipeline, the effect works remarkably well.

{{< figure src="/images/gavazn_crt_sample.gif" caption="if I may say so myself!" alt="Gavazn CRT Screen" align="center" width="500">}}

### The Machine

All that remained was to house the screen in something plausibly mechanical. I spent a while on the oscilloscope subreddit before I decided I'd gone down a garden path and pivoted to retro televisions instead. I collected reference images piecemeal until a friend put me on to [Sony's FX-300 Jackal](https://swling.com/blog/2021/03/the-sony-fx-300-jackal-a-holy-grail-technological-marvel-of-the-late-70s/). Together with the bold orange contrast of the [Marantz SD930](https://audio-database.com/MARANTZ/player/sd930-e.html), I'd found the bulk of my inspiration.

Still, I'd done very little CSS at this point; it turns out that creating satisfying skeumorphic buttons, knobs, and switches can take up to a half-dozen box shadows on a single element. The stylesheets alone contribute significant loadtime to the machine, but after hours of tinkering I'm confident it's a worthwhile overhead. I learnt early on in the process that two design principles help with creating pleasing virtual input controls: first, of course, everything has to be designed with the same virtual light source(s) in mind. The resulting consistency of shadows provides necessary depth and a more immersive illusion.

The second principle is indispensable if we want the user to actually feel this depth: the operation of an input control has to perceptibly alter its orientation, in a way that feels like you're moving something heavy through space. Thus I had to abandon a switch I'd spent an hour designing, because from the fixed head-on perspective of the user, it would merely slide back and forth without ceremony. Even though the switch changed positions, it was impossible (without going several layers deeper in the HTML/CSS) to make it look like it was a physical appendage being manually shifted. It felt flat and unsatisfying! I replaced every instance of this switch with a light switch modeled after the one in my room, and the visible shift in shadows and shape with each toggle made all the difference.

The last step was something I'd felt ambivalent but optimistic about through the whole project: adding sounds to the switches. None of the samples I hunted for in free libraries were to my satisfaction, so I was forced to use a recording app on my phone while standing breathlessly next to various switches at home, waiting until there were no cars going by so I could grab a less-noisy recording. I didn't even have anything that sounded like how I wanted my push-buttons to sound, so I used a pen button instead and slowed it down in Audacity enough to give it some weight. The soft clicks of the knobs turning, meanwhile, came from a control wheel on my Olympus digital camera. I made sure to load multiple samples for each type of switch and knob, because I didn't want rapid operation to feel like you were just echoing the same sample over and over... imagine how unpleasant! As much as the process was hacked together, the result adds to the weighty texture of the app, and I'm glad I did it.

### Gavazn

I didn't think of what to call my contraption until the very end, but the spark of an idea came blessedly quick when I took the time to ruminate on it. I'd modelled the whole thing heavily off a television called the Jackal, so I thought: why not another animal name? It was a short leap from here to the word [gavazn](https://en.wiktionary.org/wiki/%DA%AF%D9%88%D8%B2%D9%86), meaning "deer" in Persian, the sound of which I've long had an aesthetic fondness for. One of my favourite music artists also has [an album by the same name](https://soundcloud.com/alisorena/sets/gavazn), which you should check out. It turned out that the romanized letters were easy to stylize, leading me to a cute logo in short order.

{{< figure src="/images/gavazn_logo.png" caption="there is a stylized 313 above the Z as well..." alt="Gavazn Logo" align="center" >}}

You can play with Gavazn on [itch](https://s-behrangi.itch.io/gavazn), and look at the code on [GitHub](https://github.com/s-behrangi/cellular_automaton_tv).