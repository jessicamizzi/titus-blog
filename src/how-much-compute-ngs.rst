How much compute power do you need for next-gen sequencing?
###########################################################

:author: C\. Titus Brown
:tags: ngs,predicting_the_future
:date: 2011-07-20
:slug: how-much-compute-ngs
:category: misc


During our `next-gen course
<http://ivory.idyll.org/blog/jun-11/ngs-2011>`__, a "student" (really
a professor from Australia ;) asked me if I could provide some
guidance on what computational infrastructure was necessary to handle
next-gen sequencing data.  While we used Amazon Web Services during
the course, she was interested in finding out if they could use their
local HPC, or some other dedicated compute center, to process their
data.

So here are my estimates for what you would need if you were planning
to buy an Illumina HiSeq machine and needed to do all the processing
downstream of "I have sequence", i.e. from the basic quality-passed
FASTQ sequences that most sequencing centers give you.

This is actually a pretty important question to nail down.  Most data
centers have no idea about the biology, and most biologists have no
idea about the data centers.  So biologists may end up asking vague
questions and acting helpless, while the HPC folk buy lots of CPUs for
them (i.e. the wrong thing; see below).  Like may of the little
frustrations in bioinformatics it boils down to a communication
problem!

The below estimates are based on my personal experience over the last
year or two, and should be regarded as the *minimum* for effective
functioning.  Corrections or other opinions welcome in the comments,
or write your own dang blog post and I'll link to it... There is really
a lot of hand-waving on my part so I won't be offended!

**Hard disk capacity**

Just to archive the sequences from a single run, you'll need on the
order of 100 GB (1-2 copies of the basic gzipped FASTQ data).
Temporary disk space for working with the data will average on the
order of 500 GB (uncompressing, copying, moving, temporary data files,
index data files, etc.)  That space can be re-used once the basic
analysis has been done.  Assuming you generate 4-8 data sets each month
with a single Illumina machine, I would guesstimate that about 1 TB
a month of permanent archival space, and 4 TB of working disk space,
would be good.

So, 100 GB disk per data set, permanent, and 500 GB working disk, 
per data set per month, for 8 data sets, + fudge factor: 1 TB of
disk space a month, permanent, and 4 TB of working disk space.
Cost?  Negligible, \\$200/mo plus \\$2000 a year.

**Compute capacity**

Oddly enough, CPU is rarely a huge concern (in my experience).  Unless
you're doing things like really, really large BLASTs against
unassembled short reads (which is inadvisable on pretty much any
planet, not just ours), you probably will have enough CPU on medium
sized computers your data center has.  4 to 8 cores, for 1 month per
data set, are probably enough to do the basic mapping or assembly
analyses, although of course more is better.  Mapping to a reference
genome/transcriptome is particularly parallelizable so you can take
advantage of as many cores as you have.  Bottom line, if you have one
reasonably sized dedicated computer per data set, you should be OK.  I
would suggest 8-16 GB of RAM minimum (but see next section) on a 2 to
4 CPU machine, with each CPU having 2 to 4 cores.  You can easily buy
this kind of thing for way less than \\$5000 -- it's what a lot of kids
have at home for gaming these days, I think.

So, 1 medium sized computer (2-4 multicore CPUs, 8 GB of RAM) for 1
month, per data set, for 8 data sets: 8 computers. Cost: let's say
\\$40,000/year.

**Memory/RAM**

Memory is sort of the big bugaboo for me.  I've been focusing on de
novo assembly, which is a memory hog; I've just put in an order for a
500 GB machine, and I'm writing a 1 TB machine into my next grant.
Mapping is much less memory intensive, requiring at most a few GB
(although performance can always be improved by buying more memory, of
course).

Many de novo assemblers scale with the number of unique k-mers in the
data set, which means that for big, deeply sequenced data sets with
lots of sequencing errors, you are going to need lots of memory.  For
bacterial genomes, you only need a few GB.  For anything more
challenging, you will need 100s of GBs.  I would recommend a 512 GB
machine, and strongly suggest a 1 TB machine (because who really wants
to run only one analysis at a time, anyway?)

The only published machine estimate I've seen for assembly, BTW, is
from the Broad Center, in the ALLPATHS-LG paper, where they estimate
that they can assemble a human genome de novo in about two weeks with
under 512 GB of RAM.

If Amazon Web Services wants to be really, really friendly to me, they
can start providing 512 GB RAM machines for rent... and then give them
to me for free, hint hint.

Note that I haven't said much about CPU power.  That's because by the
time you get a machine that has 512 GB of RAM, it probably has enough
CPU power to run the assembly just fine.  Some assemblers can make use
of multiple CPUs: ABySS does, and Velvet recently released an update
supporting it.  I assume ALLPATHS-LG, SOAPdenovo, and others are
keeping pace.

But the overall problem is it only takes ~1 week to generate a data
set that can require 2-4 weeks to assemble in 512 GB of RAM.  And
these machines are expensive: figure \\$20-40k for something robust,
with decent CPU and memory performance.  And you need one of these
babies per de novo assembly project, dedicated for 1-3 months (because
de novo assembly is **slow** and **data intensive**).

If you're an HPC admin sitting there, sweating, you might think you
don't need to worry, because biologists will tell you that they're
going to be resequencing lots of genomes, and doing lots of
transcriptomes, etc., so de novo assembly isn't going to be required
much.  They'll tell you it'll mostly be mapping.

Unfortunately I think they're wrong.  De novo assembly is going to be
a big challenge going forward, as we sequence more and more odd
genomes.  I think humanity is going to sequence between 10**3 and
10**6 more novel genomes in the next 5 years than we have to date, and
many of these genomes will have no reasonably close reference.  (Don't
believe me?  Check out the `Tree of Life
<http://pacelab.colorado.edu/images/Big_Tree_Bold_Letters_white.png>`_
from `Norm Pace's Web site <http://pacelab.colorado.edu/>`__.  Humans
and corn are the two little teeny branches over on the upper left of
the Eucarya branch; I believe we have fewer than 20 draft genomes from
the non-plant/animal/fungi segments of the Eucarya branch, i.e. it's
completely unsampled!)

In sum, at least 1 bigmem computer (512 GB RAM) available for dedicated
use by biologists doing assembly, preferably more.  Cost?  \\$50k/year
for one.

---

In summary:

Hard disk: \\$5000/yr (not counting RAID, NAS, permanent backups, etc.)

Compute: approx. 8 medium computers, \\$40,000/yr (not counting air conditioning)

Memory: 1x bigmem computer minimum, \\$50,000/yr (not counting air conditioning)

---

The numbers above feel OK to me, but may be a little bit light.  If I
was doing nothing other than running such a center, I'd want about
double that (so figure \\$100-200k/year, hardware costs), which would result
in a fair amount of overcapacity that others could use.  But if I had to
give the minimum budget, \\$100k/year for preliminary sequence analysis sounds
about right.

So why am I wrong? :)  Inquiring minds want to know!

--titus


----

**Legacy Comments**


Posted by Rich Enbody on 2011-07-20 at 16:01. 

::

   Sounds reasonable to me.    The only thing missing is bus speed.  That
   is frequently the bottleneck already and is almost certainly going to
   be a factor in the scenario you describe.  There if often little
   choice on bus speed, but it isn't unusual to get a choice between two
   generations, especially at generation transition time.


Posted by Nick Barnes on 2011-07-20 at 18:41. 

::

   So is there a lot of money to be made by coming up with a de novo
   assembly algorithm which doesn't need as much RAM?


Posted by Aaron J. Grier on 2011-07-20 at 22:28. 

::

   how about specialized hardware?    <a href="http://www.conveycomputer.
   com/">http://www.conveycomputer.com/</a> claims a 400x speedup for
   smith-waterman over a single core using FPGAs, and announced 2.2 to
   8.4x speedup for de novo assembly back in May
   (http://www.genomeweb.com/blog/fpga-coprocessor-solution-de-novo-
   genome-assembly-0) although it's unclear what their comparison basis
   was.    <a
   href="http://www.nallatech.com/">http://www.nallatech.com/</a> has
   created socket-connected FPGA modules for FSB.    the socket-connected
   FPGA hardware has gone through an additional generation (to QPI), and
   continues to be under development for newer bus technologies.  <a
   href="http://www.pactroninc.com/">http://www.pactroninc.com/</a> has
   announced QPI hardware.    my employer would be happier if I didn't
   mention it, but AMD likely has similar options for hypertransport-
   connected FPGAs.    admittedly the number of computational biology
   researchers who can write verilog or VHDL is limited, but there's got
   to be a potential here.    for highly parallel workloads (which de
   novo sounds like it isn't) Intel has many integrated core (MIC) which
   is projected to be commercially available next year...    (I am
   employed by Intel and work on these technologies.)


Posted by David Smith on 2011-07-21 at 05:29. 

::

   What type of permanent archival space are we talking about? Should a
   research group invest in this or should they rely on an external data
   center?


Posted by Titus Brown on 2011-07-21 at 09:37. 

::

   Nick, good question.  A lot of smart people (and whatever my group is,
   too) are working on such things.  I harbor a general distrust of
   closed software, too -- it's not science if you can't look at the
   source code for your analysis! I particularly don't like things like
   CLC Genomics, which is fast and memory efficient but returns
   assemblies that look very different from other programs.    Aaron,
   I've talked with some of those companies.  Assembly is a horrible
   situation because it's very memory intensive, and not very
   distributable, so it requires different hardware optimization
   approaches.  And what makes you think I'm not solving the problem in
   software, anyway? ;)    David, I am a fan of AWS EBS for storing data,
   but I guess that gets expensive!  I like the idea of using an external
   data center for archival purposes.  It doesn't need to be online
   accessible. I think there are commercial services that offer some or
   all of this, of course; I just don't know much about them.

