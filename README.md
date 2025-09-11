# General Coding Advice

The following advice are not specific to SKILL in particular, but I believe, it is always good to remind them.


## Programming Mottos

- Less code less errors.  
  (i.e. from a CAD engineer point-of-view: **less code less support!**)
- Keep it simple.
- Be consistent.


## Programming Mindset

### Write code to make it understandable by anyone.

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

### Be explicit.

Avoid defining global things with constructed names. If you do so, document it very clearly.  
When debugging or discovering new code, it is very disturbing not being able to `grep` what you are looking for.

### Keep things together.

Instead of having external documentation that will become outdated, put comments at the right location.  
Or even better, write docstrings inside the code.

Avoid external mapping files.
  
### Try to follow common practices.

If you don't, document it properly!

> [!WARNING]
> If every other developer is having a hard time understanding your code.  
> Your way is probably not the best, or it hasn't been clearly explicited why it is the good solution!
     
### Always be in control, no place for random

   Do not use global variables, it makes your code context dependent.  
   (At least prefix them, so they have less chances to be overwritten.)

### Avoid dependencies (when reasonable).

   If you have to develop a server to communicate with another language to use one or two features.  
   Make sure it is not simpler to implement what's missing.  
   Or that you cannot develop what you need directly in the other language.
  
   At least, make sure your dependencies are well tested, well documented and well maintained.
