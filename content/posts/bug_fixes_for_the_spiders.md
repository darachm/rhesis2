---
title: "Bugs (reports) for the (web) spiders"
date: 2018-01-26
draft: false
tags: 
  - bugs
  - reference
  - computer
  - benchwork
  - science
  - methods
---

Here, I document problems I've encountered and some notes to help
others figure out similar problems of their own

# Code 

## `Makefiles` and `rmarkdown::render` fighting over reproducible research

180417

In effort to both provide a reproducible setup for the analysis for
a paper I was writing, and to placate my perfectionism, I tried 
using `Makefiles` to run a set of `rmarkdown::render`-typeset documents,
for all the R code (and more) I used. It was working fine, then we
sent it out for review. Dumbly, I then updated my laptop, which
then updated libraries, which required `tidy` to update, which
was a mess...

First off, the 1.9 version of `rmarkdown::render` drops support
for the usual latex engine, and requires that you use the author's
own custom reboot of the code. So that was a warning sign, but I
just reverted to 1.8 and it seemed to work.

Later, I realized that the `Makefile` was running `rmarkdown::render`
commands, but they weren't doing the same thing they did a few months
ago! I could swear the options were different, but I don't know how
to pull the old documentation short of installing it so I just tried
to debug it.  
It became a multi-hour nightmare of debug testing lots of different
options, with the order of operations and cleaning mattering for the
output in strange ways.

The main error was pandoc 67, it couldn't find a png it was looking
for during the typsetting, but `rmarkdown::render` thought it had
made it. Specifically, `*2-1.png` was needed but only `*3-1.png` was
in the `*_files` folder.

It ended up being that if you interrupt a `Makefile` run to debug
something, it'll delete some but not all files that aren't protected.
So it'll leave a `*_cache` folder around for `rmarkdown::render` 
to find, and `rmarkdown::render` assumes that means the files are
made, and skip it.

Solution was to deep clean and delete everything. Also started to
use the `.SECONDARY` directive in the `Makefile`, but didn't spend
much time since I plan on not investing too much time in learning
any more of GNU make (except what's needed to launch `nextflow`).

## Perl should be one version (duh)

I ran into this issue originally because `texworks` stopped working
on an Arch system. The error was coming from `pdflatex`, complaining
that `pdflatex.fmt` doesn't match `pdflatex.pool` . In attempting to
regenerate `pdflatex.fmt`, I tried to run `fmtutil`, but it complains of

    perl: symbol lookup error: /home/zed/perl5/lib/perl5/x86_64-linux-thread-multi/auto/Encode/Encode.so: undefined symbol: Perl_xs_apiversion_bootcheck

Strangely `cpan` also throws the same error, pretty much.
Both fail at dynamically linking in `Encode.so`, which is kind of a
big deal.  
It would appear that I needed to rebuild this module, or something.
Problem is that I can't load `cpan`, or make modules with `perl`!

After comparing lots of threads 
<a href="https://bbs.archlinux.org/viewtopic.php?id=200301">this one</a> 
actually led to the solution: simply getting rid of the `perl5`
directory in `$HOME`! _jasonwryan_ pointed out that `pacman` should never
touch `$HOME` which is cool. 

So that was
`mv ~/perl5 ~/bkperl5
and that solved it.

## R

### `ungroup()` in the `tidyverse`

I had used the massive memory machines on the NYU HPC to do some
large `data.frame` `tidy`-ing and `dplyr`-ing into shape, then made
a tiny little dataframe for plotting. I wanted to `sftp` locally
to plot, but when I opened up the `save()`'d file, R complained:

    Error: vars is not a list

Huh. When I went into `str()` it threw what looked like a tibble
but also a lot of indicies. Turns out, I had not `ungroup()`'d it
before saving, and it was saving the grouping in a way that confused
the save/load system, and it was not properly formed in my local
machine. So `ungroup()` when you save! I kinda wish grouping
of tibbles had scope, to prevent this kind of thing.


### `minqa` install failing

`lme4` and `minqa` and all that rock. But there's a little bit of
a snag I've observed in trying to install them (for the purposes
of the `variancePartition` package in Bioconductor). Threw some 
errors, but here's the issue with an install of `minqa`:

    g++ -shared -L/usr/lib64/R/lib -Wl,-O1,--sort-common,--as-needed,-z,relro -o minqa.so altmov.o bigden.o biglag.o bobyqa.o bobyqb.o lagmax.o minqa.o newuoa.o newuob.o prelim.o rescue.o trsapp.o trsbox.o trstep.o uobyqa.o uobyqb.o update.o updatebobyqa.o NULL -lgfortran -lm -lquadmath -L/usr/lib64/R/lib -lR
    g++: error: NULL: No such file or directory

So g++ is getting fed a NULL, where's it coming from? To fix this,
I extracted the archive in the `/tmp` folder, edited the
`src/MakeVars` file to comment out the one line there, so

    PKG_LIBS = `$(R_HOME)/bin/Rscript -e "Rcpp:::LdFlags()"`              

That Rcpp call was returning `NULL` and feeding that to g++. So after
that edit, tar-gzip'd it back up, and installed that tgz. Then
it worked fine.

So why `NULL`? I hadn't `Rcpp` installed! why is this not a 
dependency marked for auto-install? dunno

### Plotting greek and italics

If you do yeast genetics, you've got to get the nomenclature
right. I suppose that means in plots too. So for doing things
in `R` and `ggplot`, here's some ways. This is my understanding
of nomenclature usage, but I've only ever looked at mRNA and
knockouts so I'm probably not using it right with respect to
counting alleles or thinking about dominance.

Genes are capitalized and italic. So to do that in a plot, let's
say the x-axis, you'd add on 
`xlab( expression(paste( italic(GAP1)," mRNA signal" )))`.
Do note that that `paste` seems to not put spaces in, so it looks
like a different `paste` than the one I'm used to --- or different
arguments interally or something.

Mutants are italicized, lower case, and with a 
[Δ](https://unicode-table.com/en/0394/). So put that in the above
string. I don't know if `expression` has a short hand for that,
I wouldn't be suprised.

To save plots with deltas, either save it as PNG (preferred), but
if you have someone who really likes PDFs for their vector graphics
(I assume that's the advantage?), then you need to use cairo.
So for `ggplot`, that's `library(Cairo)` and put `device=cairo_pdf`
in the `ggsave` call. 
[That's from here]( https://stackoverflow.com/questions/28746938/ggsave-losing-unicode-charaters-from-ggplotgridextra).

# Computers 

## t420 fails to boot, CMOS battery

180126

I've had a 2011 t420 thinkpad for a while. I got it used, and it's
been robustly performing. 

Then I started to get some issues with booting. 
Usually it's on or crunching over night plugged in, 
with brief reboots every few weeks.
On the infrequent times that I tell it to suspend, I started seeing
a behaviour where it would cycle with power light coming on, 
then it would try to boot, fail, and keep cycling ad infinitum.
Lights on the back of the case were on. The way to get out was to
remove the main battery and unplug, wait a bit, then re-power and
try turning it on.

The day of submitting a paper, in the evening after it was submitted
(great timing), I told it to suspend. In the morning, it refused to
turn on. No lights, no nothing, no how. Nothing on the back, tried
different power supplies. Dead as I could tell. 
Taking out main battery, unplugging, and holding power button for
about 2 minutes didn't do it.

The fix was to take the RAM cover off the underside, flip it over,
pry the keyboard towards the back and off, then reach in there with
a screwdriver and get the CMOS battery plug up and off.
After a minute or two, I put it back in, and it started to
boot off of battery power! Then it died. I re-opened, got it wiggled
off, then re-did it, and got it to boot off of battery again
and back to life. 

I am ordering a new CMOS battery now to test if it's just an old
battery.

In conclusion, I think the old dead CMOS battery screws up the boot.
Surprisingly contrary to other reports, it didn't have lights on
the back panel that reacted to power supply events, but nevertheless
it did get fixed.



# Benchwork 

## Blue VRC

One time when I was doing FISH, I used VRC as a precautionary RNAse 
inhibitor. I used Buffer B to keep the pH right. I would make it in 
50ml, which
means the dibasic potassium phosphate is 0.8ml but I messed up and put
in 1ml, dumb error. That pushed the pH low. 

Turns out you make VRC by adding VOSO<sub>4</sub>, 
which is very blue, and raising the pH until 
it's green. So the acidic buffer turned the VRC very very blue, and
the cells took up the precipitate. So those were blue cells, and
wouldn't wash off. 
Pretty, but I never tested if the cells were good

Turned out they were the middle, informative timepoints of an
control experiment that I always present now. Now I just say that
I didn't capture the middle because I have the start and the end,
and the ones in the middle ... turned blue.

## Quench before spinning

I've optimized a protocol of doing a PFA (paraformaldehyde) fixation
before digestion and hybridization (above). I normally quench the
fix with 0.5M final glycine before centrifugation. One time I didn't
because I wanted to test some shortcuts (my advisor had tried the
protocol recently and found some steps tedious, he was 
looking for where he could trim and automate to get a higher
throughput out of the method).


So I didn't quench before the spin, and I got really bad hybridization
by that I mean low signal, and the cells were tiny. I tried the whole
thing again with technical replicate raw samples, but this time I
quenched before. Large and good hyb.


I think what happened is that centrifuging cells during a fix, before
the quench, compresses the fixed matrix of connections, making
fixable epitopes more accessible to each other by compressing the
searchable space. Haven't tested this well yet, tho.


