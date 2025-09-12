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

> [!NOTE].
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


### Always be in control, no place for random.

   Do not use global variables, it makes your code context dependent.  
   (At least prefix them, so they have less chances to be overwritten.)


### Avoid dependencies (when reasonable).

   If you have to develop a server to communicate with another language to use one or two features.  
   Make sure it is not simpler to implement what's missing.  
   Or that you cannot develop what you need directly in the other language.

   At least, make sure your dependencies are well tested, well documented and well maintained.


## Programming Methodology


### Spend time checking what exists already before coding.

> [!NOTE]
> In Python, many modules are already there, and sometimes even open-source.  
> In SKILL, the code is often kept secret by companies but there are many scripts available here:  
> - [Cadence Community - SKILL Forum](https://community.cadence.com/cadence_technology_forums/f/custom-ic-skill)  
> - [Cadence Support](https://support.cadence.com)
>
> The support search is often returning good and interesting results.  
> It contains many examples with code snippets, you can probably re-use directly or adapt!


### Try to use standard functions as much as possible.  

Do not reinvent the wheel.


### Write tests and specs before coding.

If you know what to expect in most cases, you will get the big picture before you start coding.  
This will help you write clearer, simpler code and avoid rushing headfirst into a wall.  
It will also help debugging as the more you test, the less you break things and the easier you spot errors.


### If you write functions, the minimum requirement is to document them with docstrings.

It should explain the arguments and the output. (Sometimes explain the purpose and detail the behavior.)


# Basic SKILL/SKILL++ advice


### Lint your files.

Follow lint advices and try to get 100% score.  
A high score is reassuring, it means the author made effort cleaning his code. 


### Prefix unused variables with underscores.

Prefixing a variable with '_' is the standard syntax to declare it unused.  
It is usual to define functions that must accept unused arguments.

> [!TIP]
> Unused variables will decrease Lint score, unless they have '_' prefix.

For instance:
```scheme
;; Display a simple form with a string field and a button.
;; When the button is clicked, it prints the value of the string field.
(hiDisplayForm
  (hiCreateLayoutForm (gensym 'custom_form) "Custom Form"
    (hiCreateFormLayout 'main_layout ?items (list
        (hiCreateStringField 
          ?name  'input_field
          ?prompt "Input"
          )
        (hiCreateButton 
          ?name       'print_button
          ?buttonText "Print"
          ;; Here the _field value has to be taken in account, but is never used...
          ?callback   (lambda ( _field form ) (println form->input_field->value))
          )
        ))
    ?buttonLayout 'Close
    ))
```
