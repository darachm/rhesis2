---
title: "Bugs (reports) for the (web) spiders"
date: 2018-01-26
draft: true
tags: 
  - bugs
  - reference
---

Here, I document problems I've encountered and some notes to help
others figure out similar problems of their own

<ul>
<li>
<h3>Perl should be one version</h3>
<p> I ran into this issue originally because texworks stopped working
on an Arch system. The error was coming from pdflatex, complaining
that pdflatex.fmt doesn't match pdflatex.pool . In attempting to
regenerate pdflatex.fmt, I tried to run fmtutil, but it complains of
"perl: symbol lookup error: /home/zed/perl5/lib/perl5/x86_64-linux-thread-multi/auto/Encode/Encode.so: undefined symbol: Perl_xs_apiversion_bootcheck"
Strangely cpan also throws the same error, pretty much.
Both fail at dynamically linking in Encode.so, which is kind of a
big deal.
</p>

<p>It would appear that I needed to rebuild this module, or something.
Problem is that I can't load cpan, or make modules with perl!</p>

<p>After comparing lots of threads 
<a href="https://bbs.archlinux.org/viewtopic.php?id=200301">this one</a> 
actually led to the solution: simply getting rid of the perl5 
directory in $HOME ! jasonwryan pointed out that pacman should never
touch $HOME which is cool. 

So that was
<pre>mv ~/perl5 ~/bkperl5</pre>
and that solved it.
</li>

<li></li>
</ul>

I had used the massive memory machines on the NYU HPC to do some
large data.frame `tidy`-ing and `dplyr`-ing into shape, then made
a tiny little dataframe for plotting. I wanted to `sftp` locally
to plot, but when I opened up the `save()`'d file, R complained:

    Error: vars is not a list

Huh. When I went into `str()` it threw what looked like a tibble
but also a lot of indicies. Turns out, I had not `ungroup()`'d it
before saving, and it was saving the grouping in a way that confused
the save/load system, and it was not properly formed in my local
machine. So `ungroup()` when you save! I kinda wish grouping
of tibbles had scope, to prevent this kind of thing.


---

# `minqa` install failing

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


---

# fiddling around with ggplot

- if you want unicode to save nicely to pdf, say like greek deltas 
  in facets/axis-titles for gene deletions, then you need to
  `library(Cairo)` and use `device=cairo_pdf` in the `ggsave` call.
  ( https://stackoverflow.com/questions/28746938/ggsave-losing-unicode-charaters-from-ggplotgridextra )
---
title: ""
date: 
draft: true
---
SCIENCE
wetbench_bugs,_observations

<p>
Here's a few bits about wetbench biology that might be useful to
Those Who Listen To The Robot Spiders (search engines)
</p>

<h3>Blue VRC</h3>
<p>
One time when I was doing FISH, I used VRC as a precautionary RNAse 
inhibitor. I used Buffer B to keep the pH right. I would make it in 
50ml, which
means the dibasic potassium phosphate is 0.8ml but I messed up and put
in 1ml, dumb error. That pushed the pH low. 
</p>
<p>
Turns out you make VRC by adding VOSO<sub>4</sub>, 
which is very blue, and raising the pH until 
it's green. So the acidic buffer turned the VRC very very blue, and
the cells took up the precipitate. So those were blue cells, and
wouldn't wash off. 
Pretty, but I never tested if the cells were good
</p>
<p>
Turned out they were the middle, informative timepoints of an
control experiment that I always present now. Now I just say that
I didn't capture the middle because I have the start and the end,
and the ones in the middle ... turned blue.
</p>

<h3>Quench before spinning</h3>
<p>
I've optimized a protocol of doing a PFA (paraformaldehyde) fixation
before digestion and hybridization (above). I normally quench the
fix with 0.5M final glycine before centrifugation. One time I didn't
because I wanted to test some shortcuts (my advisor had tried the
protocol recently and found some steps tedious, he was 
looking for where he could trim and automate to get a higher
throughput out of the method).
</p>
<p>
So I didn't quench before the spin, and I got really bad hybridization
by that I mean low signal, and the cells were tiny. I tried the whole
thing again with technical replicate raw samples, but this time I
quenched before. Large and good hyb.
</p>
<p>
I think what happened is that centrifuging cells during a fix, before
the quench, compresses the fixed matrix of connections, making
fixable epitopes more accessible to each other by compressing the
searchable space. Haven't tested this well yet, tho.
</p>

