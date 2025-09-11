# SKILL & SKILL++ Best Practices
Cadence SKILL/SKILL++ coding advice.


## General Coding Advice

The following advice are not specific to SKILL in particular, but I believe, it is always good to remind them.


### Programming mottos

- Less code less errors.  
  (i.e. from a CAD engineer point-of-view **less code less support!**)
- Keep it simple.
- Be consistent.


### Programming mindset

#### Write code to make it understandable by anyone.

Your future self (and your coworkers) will be thankful!
  
> Programs must be written for people to read, and only incidentally for machines to execute.  
> -- <cite>[Harold Abelson, Structure and Interpretation of Computer Programs, MIT][1]</cite>

[1]: https://www.goodreads.com/quotes/9168-programs-must-be-written-for-people-to-read-and-only    

> [!NOTE]
> Useful information that users should know, even when skimming content.
> SKILL is a high level language, most scripts are called by human interaction (GUI, click, ...).  
> 
> A few seconds, or even minutes sometimes, to run are often acceptable.  
> So [always] prefer clarity over efficiency. (Effiency is often achieved while clarifying your code.)

> [!WARNING]
> Still, avoid square or exponential complexities...  
> If you don't know what complexity is, you should probably learn about it before coding more...  
> [Wikipedia: Computational complexity](https://en.wikipedia.org/wiki/Computational_complexity)
