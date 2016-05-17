---
layout: page
title: Projects
---

## Circuit Design

### 500Mbps All-Digital True Hardware RNG in 65nm CMOS

For my final project in MIC610 for the Spring 2016 semester, I designed a true hardware RNG capable of generating random bits at around 500 Mbps. I came up with a unique technique to extract entropy from differential oscillator jitter. The design is not polished and the randomness of the output is yet to be confirmed. I hope to finalize the design and publish a paper on the idea.

### 250ps 16-bit Adder in GF 65 nm CMOS

During the Spring 2016 semester, my colleague and I designed and implemented a 16-bit adder based on the carry lookahead Kogge-Stone topology for a course midterm project. We were able to bring the propagation delay down to around **250 ps**. We did not however extensively test the functionality of the circuit due to time constraints.

## Completed

<p class="message">
    All of the source code for the projects listed below can be found on my Github page.
</p>

### [Jadawil](http://jadawil.xyz)

A course scheduler for UAE University students. Jadawil makes it extremely easy to schedule course timings during a busy semester. It also takes final exam conflicts into account. I built a basic version over the course of 2 weeks using Python, Flask, mechanize, and BeautifulSoup. The most difficult part of the project was scraping and parsing the course data - this is what I used mechanize and BS for.

### STracker

A task tracker written during my internship at Schlumberger. The idea was for it to improve workshop efficiency by displaying current maintenance tasks for each technician on a large LCD display. I wrote it using Node.js and NW.js.

### dubizzle

A scraping-based API for Dubizzle, a classified ads website based in the UAE. This project actually got me a job offer at Dubizzle!

### cspice

A simple circuit simulator written in C++. I had ambitious plans for the functionality initially:

* Transient analysis and DC operating point
* SPICE-like syntax parser
* Support for R, L, C, as well as diodes and transistors
* Plotting via gnuplot
* REPL for interactive simulation; insert or delete components in real-time without editing the circuit's netlist

I ended up implementing only a small subset of the above as a course project.

## Planning Phase

### BTC Tunisia

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
