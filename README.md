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
> If you are unaware about complexity, you should learn the following before coding more...  
> [Wikipedia: Computational complexity](https://en.wikipedia.org/wiki/Computational_complexity)


### Be explicit.

Avoid defining global things with constructed names. If you do so, document it very clearly.  
When debugging or discovering new code, it is very disturbing not being able to `grep` what you are looking for.

> [!CAUTION]
>
> Here is a bad SKILL example of "smart" (or dynamic) definitions.
> ```scheme
> ;; Define predicates to check if objects are simple geometric shapes.
> (foreach obj_type '( rect polygon ellipse )
>   (funcall 'defun (concat 'is_ obj_type ?) '( obj )
>     (lsprintf  "Check if OBJ is a %s" obj_type)
>     `(eq ,obj_type obj->objType)
>     ))
> ```
> This might confuse most developers that will have a hard time looking where `is_rect?` is defined.

So use explicit definitions. Choose explicit functions and variables names.
Write explicit error messages. Anyone reading and debugging your code will be thankful. 


### Keep things together.

Instead of having external documentation that will become outdated, put comments at the right location.  
Or even better, write docstrings inside the code.

Avoid external mapping files.


### Follow common practices.

If you write specific, tricky or weird code, document it properly!

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

It often avoids bugs and errors even before running the code.


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


### Use `(car (exists ...))` instead of `(car (setof ...))`

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

  results = (foreach mapcar number dividers 1.0/number)
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
>   (when (car (setof number dividers
>                (progn occurences_by_number[number]++ (zerop number))))
>     (error "One of the dividers is zero: %N" dividers))
>
>   results = (foreach mapcar number dividers 1.0/number)
>   (println results)
>
>   ;; Print values with associated occurences
>   (foreach number occurences_by_number
>     (info "%d : %d\n" number occurences_by_number[number])
>     )
>
>   );let
> ```
>
> <details>
> <summary>A clearer solution could be:</summary>
> 
> But it does not use `exists` nor `setof` and
> one could argue that the previous code is more efficient.
> 
> ```scheme
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
>   (assert (zerop occurences_by_number[0]) 
>     "One of the dividers is zero: %N" dividers)
> 
>   results = (foreach mapcar number dividers 1.0/number)
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

### Avoid nested `if` calls.

Nested `if` calls are hard to understand. Ther are also complicated to maintain.  
Here are a few solutions to avoid them:

#### Use `cond`.

`cond` is the default Lisp statement to group conditions.
It is somewhat equivalent to `if ... elif ... else` in other languages.

```scheme
(defun fibonacci (n)
  "Return the Nth Fibonacci number."
  (if (zerop n)
      0
    (if (onep n)
        1
      (fibonnaci n-1)+(fibonnaci n-2)
      ))
  )

;; The nested `if`s can be replaced by a single `cond`
(defun fibonacci (n)
  "Return the Nth Fibonacci number."
  (cond
    ( (zerop n) 0                               )
    ( (onep  n) 1                               )
    ( t         (fibonnaci n-1)+(fibonnaci n-2) )
    ))
```


#### Use `case` or `caseq`.

When comparing the result of an expression to several values (as done just before), then `case` is more suitable.  
It is somewhat equivalent to `switch` statements in other languages.

```scheme
(defun fibonacci (n)
  "Return the Nth Fibonacci number."
  (case n
    ( 0 0                               )
    ( 1 1                               )
    ( t (fibonnaci n-1)+(fibonnaci n-2) )
    ))

;; It is also possible to group values which return the same output
(defun fibonacci (n)
  "Return the Nth Fibonacci number."
  (case n
    ( ( 0 1 ) n                               )
    ( t       (fibonnaci n-1)+(fibonnaci n-2) )
    ))
```

> [!TIP]
>
> If you want to check if the value is exactly `t` or a list.
> You can use the following
> ```scheme
> (case variable
>  ( ( ( 12 27 ) ) (info "variable is equal to '( 12 27 )\n" ) )
>  ( ( t         ) (info "variable is exactly 't\n"          ) )
>  ( t             (info "variable can be anything...\n"     ) )
>  )
> ```

> [!NOTE]
>
> `caseq` is similar to `case` but it uses `eq` instead of `equal` for the comparison.  
> It is faster but only works when comparing value to symbols or integers.  
> You can stick to `case` as it will work in all cases and the performance gain is often negligible.


#### Use `error` or `assert`.

In many cases, raising errors is the proper things to do.  
It avoids nesting statements as it exits the current stack.

One common use-case, is to check arguments:

```scheme
(defun box_width ( box )
  "Return BOX width."
  (if (isBBox box)
      (rightEdge box)-(leftEdge box)
      ))

;; The previous function does not guarantee that the output would be a valid number.
;; If anything else than a box is passed as argument, it will return nil.
;; It is cleaner to raise a meaningful error message as soon as the erratic input is found.
(defun box_width ( box )
  "Return BOX width."
  (unless (isBBox box) (error "box_width - input is not a valid bounding box: %N" box))
  (rightEdge box)-(leftEdge box)
  )
```

> [!NOTE]
>
> ```scheme
> (unless <predicate> (error <msg> <msg_inputs...>))
> ;; This is completely equivalent to:
> (assert <predicate> <msg> <msg_inputs...>)
> ```


#### Use `prog`.

`prog` disrupts with Lisp mindset but it remains a very useful statement.  
Like `error` it allows to exit the current stack thanks to `return` calls.  
It might be useful to raise warnings instead of errors.


> [!CAUTION]
>
> I do not advise to use `go` statements inside `prog`.
> It is often cleaner to write dedicated functions.


#### Use `or` and `and`.

Boolean operatons might be useful to group several conditions:

```scheme 
(when (and (stringp obj)
           (not (blankstrp obj))
           )
  (info "obj is a non-blank string!: %N\n" obj)
  )
```


#### Use advanced predicates.

There are many available predicates. Knowing them can avoid grouping conditions.
Here are some examples of useful native predicates:

| Predicate   | Description                                   |
|-------------|-----------------------------------------------|
| `dplp`      |  Object is a Disembodied Property List [DPL]. |
| `blankstrp` |  String is empty or contains only whitespace. |
| `isBBox`    |  Object is a bounding box.                    |
| `atom`      |  Object is an atom (i.e. nil or not a list).  |

Write your own predicates, if you are often repeating the same checks:

```scheme
(defun nonblankstring? (obj)
  "Return t if OBJ is a non-blank string.
Meaning a string that contains at least one character which is not whitespace."
  (and (stringp obj)
       (not (blankstrp obj))
       ))
```


#### Use `(printf "%s" str)` instead of `(printf str)`

When using most printing functions (`lsprintf`, `printf`, `fprintf`, `info`, `warn`,
`error`, ...), avoid using a variable as first argument.  (This is even more true
when you are unaware of the variable content. It can be the result of a command, the
text of a file or a user input.)

```scheme
;; The following will break
(foreach str '( "This is a dummy example string\n"
                "Another one but with a percentage sign %\n"
                )
  (info str)
  )

;; This is much safer
(foreach str '( "This is a dummy example string\n"
                "Another one but with a percentage sign %\n"
                )
  (info "%s" str)
  )
```


### Construct Graphical User Interfaces [GUIs] with layout forms

> [!WARNING]
>
> Forms with hardcoded coordinates are hard to modify and maintain.
> ```scheme
> (defun create_custom_form ()
>   "Create and return an example form."
>   (hiCreateAppForm
>     ?name      (gensym 'custom_form)
>     ?formTitle "Example Form"
>     ?fields
>     (list
>       ;; String
>       (list
>         (hiCreateStringField
>           ?name   'string_field
>           ?prompt "string"
>           )
>         0:0 300:30 100
>         )
>       ;; Combo
>       (list
>         (hiCreateComboField
>           ?name   'combo_field
>           ?prompt "combo"
>           ?items '( "Choice A" "Choice B" "Other" )
>           )
>         0:30 300:30 100
>         )
>       ));list ;hiCreateAppForm
>   );defun
> 
> (hiDisplayForm (create_custom_form))
> ```

> [!TIP]
>
> Creating a similar form using layouts is simpler.  
> The coordinates are calculated automatically.  
> The form will be more responsive (it will adapt when resized, etc.).
> ```scheme
> (defun create_custom_form ()
>   "Create and return an example form."
>   (hiCreateLayoutForm (gensym 'custom_form) "Example Form"
>     (hiCreateFormLayout 'main_layout
>       ?items
>       (list
>         ;; String
>         (hiCreateStringField
>           ?name   'string_field
>           ?prompt "string"
>           )
>         ;; Combo
>         (hiCreateComboField
>           ?name   'combo_field
>           ?prompt "combo"
>           ?items '( "Choice A" "Choice B" "Other" )
>           )
>         ));list ;hiCreateFormLayout
>     ));hiCreateLayoutForm ;defun
> 
> (hiDisplayForm (create_custom_form))
> ```

Layout forms are very flexible, you just need to know the right [functions](#layout-form-functions)


## Useful functions, macros & syntax forms

Here are some useful statements that are worth knowing while coding in SKILL or SKILL++.


### Type conversion

| Function    | Description                                                            |
|-------------|------------------------------------------------------------------------|
| `concat`    | Concatenate any number of strings, symbols and integers into a symbol. |
| `strcat`    | Concatenate any number of symbols and strings into a string.           |
| `atoi`      | Convert a string into an integer.                                      |
| `atof`      | Convert a string into a floating-point number.                         |

> [!TIP] 
> 
> `concat` & `strcat` can also be used to simply convert a string into a symbol or a symbol into a string.  
> They are faster to type and to execute than `stringToSymbol` & `symbolToString`.
> So you can forget about those two last ones.


### List splitting

I know combinations of `car` and `cdr` are very Lispy, but they are often hard to read.
You can often use `nth` and `nthcdr` which are easier to read.
Or even better, use `destructuringBind`.

> [!WARNING]
> 
> ```scheme
> ;; Bad example of how to split a bounding box into coordinates
> (letseq ( ( cellview (geGetEditCellView) )
>           ( box      cellview->bBox      )
>           ( x0       (caar   box)        )
>           ( y0       (cadar  box)        )
>           ( x1       (caadr  box)        )
>           ( y1       (cadadr box)        )
>           )
>   ...
>   )
> 
> ;; Better solution but it can still be improved
> (letseq ( ( cellview (geGetEditCellView) )
>           ( box      cellview->bBox      )
>           ( x0       (leftEdge   box)    )
>           ( y0       (bottomEdge box)    )
>           ( x1       (rightEdge  box)    )
>           ( y1       (topEdge    box)    )
>           )
>   ...
>   )
> ```

> [!TIP]
>
> It is much cleaner using `destructuringBind`:
> ```scheme
> (destructuringBind ( ( x0 y0 ) ( x1 y1 ) ) (geGetEditCellView)->bBox
>   ...
>   )
> ```

> [!NOTE]
>
> Lisp is meant to manipulate lists using functions.
> `destructuringBind` is simply a macro that defines local functions and apply them to a list.
> ```scheme
> (destructuringBind ( a b @key c @rest _ ) mylist
>   ...
>   )
> ;; is completely equivalent to
> (apply (lambda ( a b @key c @rest _ ) ...) mylist)
> ```


### Using predicates with lists

The following functions are very useful to apply a predicate to list elements:

| Function | Description                                                    |
|----------|----------------------------------------------------------------|
| `setof`  | Return elements that pass the predicate.                       |
| `exists` | Return first sublist whose first element passes the predicate. |
| `forall` | Return t if all elements pass the predicate, nil otherwise.    | 

```scheme
;; Get all rectangle shapes from cellview
(setof shape (geGetEditCellView)->shapes (equal "rect" shape->objType))

;; Check if cellview contain an ellipse
(when (exists shape (geGetEditCellView)->shapes (equal "ellipse" shape->objType))
  (warn "Cellview contains an ellipse"))

;; Check if all shapes are polygons
(when (forall shape (geGetEditCellView)->shapes (equal "polygon" shape->objType))
  (warn "All shapes are polygons"))
```


### Setting anything

"setters" and "getters" are very common.  
They are functions to access and manage properties.  
However their syntax can be confusing, for instance:
```scheme
;; Set property 'a to 12 in a DPL
(let ( ( dpl (list nil) )
     )
  ;; `putprop` arguments order is : object, value, key
  (putprop dpl 12 'a)
  (get dpl 'a)
  )
  
;; Set property 'a to 12 in an instance
(defclass dummy () ( ( a ) ))
(let ( ( obj (makeInstance 'dummy) )
       )
  ;; `setSlotValue` arguments order is : object, key, value...
  (setSlotValue obj 'a 12)
  (slotValue obj 'a)
  )
```

This is where `setf` comes in handy.  
If we know how to access a property, it should be equally simple to set it.  
`setf` is a macro meant for that, it abstracts setter syntax.

Let's update the previous example:
```scheme
;; Set property 'a to 12 in a DPL
(let ( ( dpl (list nil) )
     )
  (setf (get dpl 'a) 12)
  (get dpl 'a)
  )
  
;; Set property 'a to 12 in an instance
(defclass dummy () ( ( a ) ))
(let ( ( obj (makeInstance 'dummy) )
       )
  ;; `setSlotValue` arguments order is : object, key, value...
  (setf (slotValue obj 'a) 12)
  (slotValue obj 'a)
  )
```

It is simpler, easier to remember and more consistent!

> [!NOTE]
>
> `setf` works with almost every getter:
> `get`, `getShellEnvVar`, `status`, `car`, `nth`, ...
>
> And in the few cases where it does not work, it is always possible to define it.
> ```scheme
> ;; `(setf (envGetVal ...) ...)` seems to be missing, let's define it
> (defun setf_envGetVal ( value tool name type )
>   "`setf` helper for `envGetVal`."
>   (envSetVal tool name type value)
>   )
>
> ;; Now we can use the following syntax to set the number of input lines in the CIW
> (setf (envGetVal "ui" "ciwCmdInputLines" 'int) 10)
> ```


### Layout form functions

To fully exploit User Interfaces creation in SKILL, you should have a look at the following functions:
- `hiCreateFormLayout`
- `hiCreateLayoutForm`
- `hiCreateVerticalBoxLayout`
- `hiCreateHorizontalBoxLayout`
- `hiCreate...Field` (All field creation functions)

You can also create interactive Library, Cell, View fields using:
- `ddHiCreateLibraryComboField`
- `ddHiCreateCellComboField`
- `ddHiCreateViewComboField`
- `ddHiLinkFields`

You can create a layer selection field using:
- `leCreateLayerField`

`hiSetFieldMinSize` can be useful to align several fields or headers.  
It can also be used to reset the height of a layer field created with `leCreateLayerField` which appears bigger than other fields by default.

> [!TIP]
>
> You can use `(hiDisplayForm custom_form -1:-1)` to display a form under the mouse.


### Regular expressions

If you are unaware about regular expressions.
You should definitely have a look at the following resources:

| Site                                             | Description                                                 |
|--------------------------------------------------|-------------------------------------------------------------|
| [Regex One](https://regexone.com/)               | Interactive courses to learn regular expressions from zero. |
| [Regular Expressions 101](https://regex101.com)  | Test and explain regular expressions interactively.         |

In SKILL, instead of `rex...` functions, I advise using `pcre...` ones.  
They are following [Perl Compatible Regular Expressions \[PCRE\]](https://www.pcre.org) standards.

- `pcreCompile`
- `pcreMatchp`
- `pcreReplace`
- `pcreSubstitute`
- `pcreGenCompileOptBits`


### Comparison of strings containing numbers

`alphalessp` is limited when comparing strings with numbers.
This is not the case of its lesser known cousin `alphaNumCmp`
which compares numbers inside strings in a "natural" way:

```scheme
(let ( ( file_names '( "video_0.mp4"
                       "video_12.mp4"
                       "video_27.mp4"
                       "video_10.mp4"
                       "video_1.mp4"
                       "video_111.mp4"
                       ) )
       )
  (println (sort (copy file_names) 'alphalessp))
  ;; > ("video_0.mp4" "video_1.mp4" "video_10.mp4" "video_111.mp4" "video_12.mp4" "video_27.mp4")
  
  (println (sort (copy file_names) (lambda ( str0 str1 ) (negativep (alphaNumCmp str0 str1)))))
  ;; > ("video_0.mp4" "video_1.mp4" "video_10.mp4" "video_12.mp4" "video_27.mp4" "video_111.mp4")
  )
```


### Resolving shell variables and symlinks in paths

Instead of playing with `getShellEnvVar`, `strcat` and `sprintf`.  
`simplifyFilename` is the right function to expand shell variables in paths.  
It works like Unix `realpath` (or `readlink -f`).

```scheme
;; Return the path of the SKILL interpreter utility
(simplifyFilename "$CDS_INST_DIR/tools.lnx86/dfII/bin/skill")
```


### Geometry

When working on geometrical functions, use `complex` numbers.
They easily solve most problems regarding angles.

| Function  | Description                                                       |
|-----------|-------------------------------------------------------------------|
| `complex` | Build a complex number, usually from x:y coordinates.             |
| `real`    | Return the real part of a complex number (its x coordinate).      |
| `imag`    | Return the imaginary part of a complex number (its y coordinate). |
| `abs`     | Return the modulus of a complex number.                           |
| `phase`   | Retunr the phase (or angle) of a complex number.                  |

Here is a simple example of a function to round the corners of a rectangle:

```scheme
(defun round_rectangle ( rect radius points "dnx" )
  "Round each corners of RECT using RADIUS and a number of POINTS."
  (assert (eq "rect" rect->objType) "round_rectangle - first argument should be a rectangle")
  (letseq ( ( pi      (acos -1)              )
            ( step    (pi/2.0)/(sub1 points) )
            ( new_pts ()                     )
            )
    (destructuringBind ( ( x0 y0 ) ( x1 y1 ) ) rect->bBox
      ;; Make sure that radius fits in rectangle
      (assert radius <= (x1-x0)/2.0 "round-rectangle - radius %N > half width %N"  radius (x1-x0)/2.)
      (assert radius <= (y1-y0)/2.0 "round-rectangle - radius %N > half height %N" radius (y1-y0)/2.)
      ;; Browse each corner
      ;; Each corner is represented by center of the circle used for rounding and the start angle
      (foreach tuple (list
                       (list (complex x1-radius y1-radius) 0      ) ; top-right
                       (list (complex x0+radius y1-radius) pi/2   ) ; top-left
                       (list (complex x0+radius y0+radius) pi     ) ; bottom-left
                       (list (complex x1-radius y0+radius) pi+pi/2) ; bottom-right
                       )
        (destructuringBind ( z angle ) tuple
          (for _ 1 points
            ;; Use complex exponential representation
            (push z+radius*(exp (complex 0 angle)) new_pts)
            (setq angle angle+step)
            ))
        ))
    ;; Transform rectangle using calculated points
    (dbConvertRectToPolygon rect)
    (setf rect->points (foreach mapcar z new_pts (real z):(imag z)))
    ))
    
;; The previous function is just an example, it is [almost] always better to use native functions
(defun round_rectangle ( rect radius points "dnx" )
  "Round each corners of RECT using RADIUS and a number of POINTS."
  (leModifyCorner rect '(t t t t) nil radius (sub1 points))
  )
```


## Tricky errors that confuse beginners


### `sort` is destructive

Be careful when using `sort`, it one of the very few destructive functiosn available in SKILL.  
It is a tricky one because its desctructive behavior is implicit.  
The output might be what you expect, but the input list is often messed up in the process...

> [!CAUTION]
>
> ```scheme
> ;; The following code is not sorting the list but destructs it instead
> (let ( ( dummy_list '( 12 27 42 3 2 1 101 ))
>        )
>   (sort dummy_list 'lessp)
>   (println dummy_list)
>   )
> ```

There are several solutions depending on the use-case:

1. You actually want to modify the sorted list.
   ```scheme
   (setq dummy_list (sort dummy_list 'lessp))
   ```
   
1. You use the sorted list but need to keep the original intact.
   ```scheme
   (println (sort (copy dummy_list) 'lessp))
   ```
   
1. You are sorting a list which is generated on-the-fly, then you don't care about destroying it.
   ```scheme
   (println (sort (setof num dummy_list (evenp num)) 'lessp))
   ```


### `quote` or `'` returns a fixed pointer

If you return a symbol or a list constructed using `'`, the value is actually an object at a fixed address in memory.  
Here is an example to show how this can mess up your code:


> [!CAUTION]
> ```scheme
> ;; The following function seems perfectly valid and harmless
> (defun new_dpl ()
>   "Build an empty disembodied property list [DPL] and return it."
>   '( nil )
>   )
>
> ;; Let's use it to create a DPL and set its properties
> (let ( ( dpl (new_dpl) )
>        )
>   (setf dpl->a 12)
>   (setf dpl->b 27)
>
>   ;; This will print (nil b 27 a 12) as expected
>   (println dpl)
>   )
>
> ;; However the behavior of our function seems broken
> (println (new_dpl))
> ; > (nil b 27 a 12)
> ;; The return value is not an empty DPL anymore!
>
> ;; What happened is that when evaluating '( nil ) the SKIL interpreter built a list
> ;; and stored its pointer directly inside the function
> 
> ;; You can confirm this behavior with the following statement
> ;; (You will see the updated DPL directly inside the function code)
> (pp new_dpl)
> ```

> [!TIP]
>
> Instead of constructing list using `'`, prefer using `list`.  
> This will reconstruct the list everytime the function is called and avoid unwillingly caching values.
>
> ```scheme
> ;; This is a much safer and valid approach
> (defun new_dpl ()
>   "Build an empty disembodied property list [DPL] and return it."
>   (list nil)
>   )
>
> ;; Everytime `new_dpl` is called, it will construct a new list containing nil and return nil
> ;; This will avoid unwanted behaviors
> 
> ;; You can check it with the following statement
> (equal (new_dpl) (new_dpl))
> ; > t
> (eq (new_dpl) (new_dpl))
> ; > nil
> ```
> `eq` returns nil, this means that even if objects have the same value (as stated by `equal`), they are different.  
> The previous implementation using `'( nil )` instead of `(list nil)` will return t for both `equal` and `eq`.


### `nil` as a symbol

The SKILL Language user guide states that:
> Both nil and t always evaluate to themselves and must never be used as the name of a variable.  
> -- [Cadence SKILL Language User Guide IC25.1 - Atoms](https://support.cadence.com/apex/techpubDocViewerPage?xmlName=sklanguser.xml&title=Cadence%20SKILL%20Language%20User%20Guide%20--%20Language%20Characteristics%20-%20Atoms&hash=pgfId-1008778&c_version=IC25.1&path=sklanguser/sklanguserIC25.1/chap2.html#pgfId-1008778)

I fully agree that they must never be used as the name of a variable.  
But the first part stating that nil always evaluate to itself is not completely true...

This is very unlikely, but you can obtain `nil` as a symbol.  
This can lead to unexpected behavior:
```scheme
(let ( ( nil_var (concat "nil") )
       )
  (println nil_var)
  ; > nil
  (eq nil_var nil)
  ; > nil
  (equal nil_var nil)
  ; > nil
  )
```

As stated before, this should not happen in regular code.  
But you might encounter issues due to this if you read a string from a user input and use `concat` on it.


### `rexMagic` is a vey nasty switch

If you want to mess with a Virtuoso session, you can type:
```scheme
(rexMagic nil)
```

This will break things like `rexMatchp`, `rexReplace` which is expected.  
But it will also subtly break unexpected functions like `pcreSubstitute` or `simplifyFilename`.

> [!WARNING]
>
> ```scheme
> (defun split_email ( email )
>   "Split EMAIL string into first name, last name and website.
> Return nil when EMAIL cannot be parsed properly."
>   (when (pcreMatchp "([^.@]+)\\.?(.*?)@(.+)" email)
>     (mapcar 'pcreSubstitute '( "\\1" "\\2" "\\3" ))
>     ))
>     
> (rexMagic t)
> (println (split_email "first_name.last_name@example.com"))
> ; > ("first_name" "last_name" "example.com")
> 
> (rexMagic nil)
> (println (split_email "first_name.last_name@example.com"))
> ; > ("\\1" "\\2" "\\3")
> ```

> [!TIP]
> 
> You should always set `rexMagic` when using one of the functions mentioned above.  
> Here is a clean way to do so:
> ```scheme
> (defun split_email ( email )
>   "Split EMAIL string into first name, last name and website.
> Return nil when EMAIL cannot be parsed properly."
>   ;; Properly set and re-set `rexMagic`
>   (let ( ( magic (rexMagic) )
>          )
>     (unwindProtect
>       (progn
>         (rexMagic t)
>         (when (pcreMatchp "([^.@]+)\\.?(.*?)@(.+)" email)
>           (mapcar 'pcreSubstitute '( "\\1" "\\2" "\\3" ))
>           ))
>       (rexMagic magic)
>       ))
>   )
> ```

