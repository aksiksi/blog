---
layout: page
title: About Me
---

## Who am I?

My name is Assil Ksiksi.

I'm an Electrical Engineering senior at UAE University. Besides most things electrical (except power transmission), I'm in love with computer architecture, programming, compilers, and web app development.

## What have I done?

I've built projects in Python, JavaScript, Node.js, Scala, Java, C++, and MATLAB. I'm familiar with quite a few other langauges, and I'm confident I can pickup any new language, not counting the esoteric like APL and friends, in a short amount of time.

You may have noticed that the projects I've worked on during my free time are not at all related to my major. The way I see it is that I spend more than enough time working on EE subjects, so I'd rather spend my free time learning something else. Thankfully my hobby provides an alternate career path in case EE doesn't work out for some reason.

## Projects

### Planning Phase

#### Online Marketplace for Tunisia

A website that makes it easier for buyers and sellers to find each other in my home country Tunisia. Major hurdles include providing shipping for buyers countrywide at a reasonable cost, and handling payments online.

* Backend
    - Scala
    - Play Framework
    - Slick ORM?
* Frontend
    - Bootstrap
    - React
    - Underscore

#### BTC Tunisia

An online Bitcoin wallet/exchange for use in Tunisia. Due to strict currency controls, it is impossible to send Tunisian Dinar out of the country. As a result, Tunisians cannot buy things online, or send money to friends and relatives abroad. I plan on releasing a MVP with some advertising to see if the idea has potential.

* Backend
    - Scala or Python
    - Play Framework or Django
    - Slick (?) or Django ORM
    - A solid Bitcoin blockchain in Java or Python
* Frontend
    - Bootstrap
    - Backbone or React
    - Underscore

### Completed/Cancelled

<p class="message">
    All of the source code for the projects listed below can be found on my <a href="https://github.com/Cyph0n">Github</a>.
</p>

#### [Jadawil](http://jadawil.herokuapp.com)

A course scheduler for UAE University students. Jadawil makes it extremely easy to schedule course timings during a busy semester. It also takes final exam conflicts into account.

#### dubizzle

A scraping-based API for Dubizzle, a classified ads website based in the UAE. This project actually got me a job offer at Dubizzle!

#### cspice

A simple circuit simulator written in C++. I had ambitious plans for the functionality initially:

* Transient analysis and DC operating point
* SPICE-like syntax parser
* Support for R, L, C, as well as diodes and transistors
* Plotting via gnuplot
* REPL for interactive simulation; insert or delete components in real-time without editing the circuit's netlist

I ended up implementing only a small subset of the above. It was done as a course project.

#### stracker

A task tracker written during my internship at Schlumberger. The idea was for it to improve workshop efficiency by displaying current maintenance tasks for each technician on a large LCD display. I wrote it using Node.js and NW.js.