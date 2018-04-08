---
title: "Pipeline scripting - from `bash` to `make` to `snakemake` to `nextflow` to ..."
date: 2018-04-08
draft: true
---

I've been looking around

inspired by jazz

My introduction to this was some PBS bash scripts that Niki had given
me for doing tophat alignment, some years ago. I used those for a
while, asking for HPC resources based on running everything at
once in a looped bash script. To be frank, it makes it real easy to
get to working first.

Later, as I started to integrate the results of multiple datasets,
I turned to make. I tried to hack together a way of doing benchmarks
for barseq with dummy files using make, but it got messy.
I returned to make my own custom bash script, I called it a "stoker"
script.

That works okay, except that it's a custom script put-together for
one purpose and distributed across a couple of different scripts.
And, most importantly, it's in bash. Ick. Sorry. Great technology,
but there's been improvements in making scripting higher level and
I'm starting to grow out of my hard-ass phase.

More recently I heard about snakemake. I really enjoy it, made a
nice pipeline that does my RNA seq nicely, but I had some 
frustrations with getting it to play nice with SLURM and handle
configurations again. Also, for configuring a large project (a paper
containing three different RNA seq pipelines, an amplicon sequencing
pipeline, and a variety of Rmarkdown analyses scripts all hacked
together) I didn't seem like it could scale nicely unless I was doing
makefiles that call other makefiles, and it all felt like `make`
all over again (but in the serpent's tongue (which is fine, totally
fine, but I still miss perl) ).

So I turned to nextflow. I was intimidated by the use of Groovy,
even with the ability to use other languages within the processes.
I'd never used Java before (javascript yes, but not the big J). 
