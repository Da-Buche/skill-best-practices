# General Coding Advice

The following pieces of advice are not specific to SKILL in particular, but I believe, it is always good to remind them.


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
>
> SKILL is a high level language, most scripts are called by human interaction (GUI, click, ...).
>
> A few seconds, sometimes even minutes, to run are often acceptable.  
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
>
> If every other developer is having a hard time understanding your code.  
> Your way is probably not the best, or it hasn't been clearly explicited why it is the good solution!


### Always be in control, no place for random.

   Avoid global variables, it makes your code context dependent.  
   (At least prefix them, so they have less chances to be overwritten.)


### Avoid dependencies (when reasonable).

   If you have to develop a server to communicate with another language to use one or two features.  
   Make sure it is not simpler to implement what's missing.  
   Or that you cannot develop what you need directly in the other language.

   At least, make sure your dependencies are well tested, well documented and well maintained.


## Programming Methodology


### Check what exists already.

Before coding in a rush, make sure there are no solutions available.

> [!NOTE]
> 
> In Python, the community is active and many open-soucre modules are already there.  
> In SKILL, although most code is proprietary, many scripts are available here:  
> - [Cadence Community - SKILL Forum](https://community.cadence.com/cadence_technology_forums/f/custom-ic-skill)  
> - [Cadence Support](https://support.cadence.com)
>
> The support search is often returning good and interesting results.  
> It contains many examples with code snippets, you can probably re-use directly or adapt!


### Use standard functions.

As much as possible, try to not reinvent the wheel.


### Write tests and specs before coding.

If you know what to expect in most cases, you will get the big picture before you start coding.  
This will help you write clearer, simpler code and avoid rushing headfirst into a wall.  
It will also help debugging as the more you test, the less you break things and the easier you spot errors.


### Add docstrings to functions.

The minimum requirement is to list the arguments and the output.  
It is also nice to explain the purpose and detail the behavior.

Docstrings are better than regular comments as they are easier to manipulate,   
when generating documentation, for instance.


# Basic SKILL/SKILL++ Advice


## Simple Improvements

The following advice are low hanging fruits but can drastically improve code quality and robustness.


### Lint your files.

Follow lint recommendations and try to reach 100% score.  
A high score is reassuring, it means the author made effort cleaning his code. 


### Prefix unused variables with underscores.

Prefixing a variable with '_' is the standard syntax to declare it unused.  
It is common to define functions that must accept unused arguments.

> [!TIP]
> 
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


### Use (car (exists ...)) instead of (car (setof ...))

I have seen this one many times, SKILL developers love `setof`... Of course, it is super useful.  
But if you only need the first element of a filtered list, then you probably don't need to parse it entirely.

In 99% cases, you can blindlessly replace `(car (setof ...))` by `(car (exists ...))`.  
The output will stay the same but it will run faster.

```scheme
;; Generate a list of random values, just for the sake of this example
(let ( dividers
       results
       )
  (for _ 0 99 (push (random 100) dividers))

  ;; Here we browse all dividers and keep only the ones that are zero
  ;; Then we take the first one of those zeros
  (when (car (setof number dividers (zerop number)))
    (error "One of the dividers is zero: %N" dividers))

  ;; The following will give the same result
  ;; But it stops as soon as one divider equal to zero is found
  (when (car (exists number dividers (zerop number)))
    (error "One of the dividers is zero: %N" dividers))

  results = (foreach mapcar number dividers 1/number)
  (println results)

  ;; The worst-case complexity is still identical, if there are no zero in dividers
  ;; the list will be browsed entirely in both cases.
  ;; But the average complexity is way improved for `exists`.

  );let
```

> [!CAUTION]
>
> The 1% cases where this is not valid are the cases where the loop contains destructive (or constructive) code.  
> i. e. when operations inside `setof` modify the environment.
>
> This is not a good practice though.
>
> ```scheme
> ;; Let's re-use the previous example
> ;; But this time we also want to count the occurences of each generated number
> (let ( ( occurences_by_number (makeTable 'occurences 0) )
>        dividers
>        results
>        )
>   (for _ 0 99 (push (random 100) dividers))
>
>   ;; Here we cannot replace `setof` by `exists`
>   ;; Otherwise the occurences count will be wrong as soon as zero is encountered
>   (when (car (setof number dividers (progn occurences_by_number[number]++ (zerop number))))
>     (error "One of the dividers is zero: %N" dividers))
>
>   results = (foreach mapcar number dividers 1/number)
>   (println results)
>
>   ;; Print values with associated occurences
>   (foreach number occurences_by_number
>     (info "%d : %d\n" number occurences_by_number[number])
>     )
>
>   );let
> ```
> <detaisl><summary>A better solution could be:</summary>

> ```scheme
> 
> (let ( ( occurences_by_number (makeTable 'occurences 0) )
>        dividers
>        results
>        )
>   (for _ 0 99 (push (random 100) dividers))
> 
>   ;; Count occurences of each number
>   (foreach number dividers
>     occurences_by_number[number]++
>     )
> 
>   ;; Make sure zero is not present
>   (assert (zerop occurences_by_number[0]) "One of the dividers is zero: %N" dividers))
> 
>   results = (foreach mapcar number dividers 1/number)
>   (println results)
> 
>   ;; Print values with associated occurences
>   (foreach number occurences_by_number
>     (info "%d : %d\n" number occurences_by_number[number])
>     )
> 
>   );let
> ```
> </details>

<details>
<summary>A better solution could be:</summary>

But it does not use `exists` nor `setof` and
one could argue that the previous code is more efficient.

```scheme

(let ( ( occurences_by_number (makeTable 'occurences 0) )
       dividers
       results
       )
  (for _ 0 99 (push (random 100) dividers))

  ;; Count occurences of each number
  (foreach number dividers
    occurences_by_number[number]++
    )

  ;; Make sure zero is not present
  (assert (zerop occurences_by_number[0]) "One of the dividers is zero: %N" dividers))

  results = (foreach mapcar number dividers 1/number)
  (println results)

  ;; Print values with associated occurences
  (foreach number occurences_by_number
    (info "%d : %d\n" number occurences_by_number[number])
    )

  );let
```

</details>

