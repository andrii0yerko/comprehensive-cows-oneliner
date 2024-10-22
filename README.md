# COMPREHENSIVE_COWS: Bulls & Cows one-liner
COMPETITION FOR THE BEST SHITCODE IN UKRAINE by Robot_Dreams, 2024

The *single*-player game of Bulls & Cows written as a *single*-line Python program in a form of a *single* [list comprehension](https://docs.python.org/3.12/tutorial/datastructures.html#list-comprehensions).

Code is here: [link](./comprehensive_cows/__main__.py)

- No unneeded `eval()` metaprogramming
- No third-party dependencies
- Complete user experience
- Only standard Python syntax
- 100% list comprehension

Interested? let's dive in

## Rules

Computer writes a 4-digit number. Your task is to guess it using as less attempts as possible.

After each guess you receive a hint:
- number of "bulls" - the number of digits you guessed correctly, and placed in a correct position in your guess. Achieving 4 bulls means you guessed the number.
- number of "cows" - the number of digits you guessed correctly, but placed on the wrong position.

Digits might repeat.

## Features
- CLI control
- multiple game sessions inside the same program run
- validation & informative messages
- on demand quit

## Usage

If you have python (3.8+) on your machine use one of:
```bash
# just like this
python comprehensive_cows/__main__.py

# or run as a package (I'm a fancy python engineer)
python -m comprehensive_cows
```

If you don't have one - Docker is always an option:
```bash
docker run --rm -it -v ./comprehensive_cows:/comprehensive_cows python:3.12 python -m comprehensive_cows
```


## Technical details
### Some Python concepts to know before:
- `x if y else z` inline if, aka "ternary conditional operator", the same as `y ? x : z` in some other languages ([Docs](https://docs.python.org/3.12/reference/expressions.html#conditional-expressions))
- `[something(x) for x in xs]` list comprehensions and generators. Basically the same behavior as "array.map", but from more imperative side. Also nested loops and filters can be added, like `[smth(x) for y in ys for x in y if condition(x)]` ([Docs](https://docs.python.org/3.12/tutorial/datastructures.html#list-comprehensions))
- `;` semicolon bad boi - you probably know, that Python separate statements using new-lines. But it's not always the case: few statements can be places into a single line, if separated with semicolon `like; that` ([Docs](https://docs.python.org/3.12/reference/compound_stmts.html#compound-statements))
- `x or y` return values. The `or` will return the first value evaluated to True, and the last argument, if all evaluated to False. `and` - the last True, or first False. Also, important to mention, that boolean arithmetic is lazy, and will stop after first `True` in `or` and first `False` without further evaluation. ([Docs](https://docs.python.org/3.12/reference/expressions.html#boolean-operations))
- `print()` returns `None`. There is no void in Python, all functions returns something. If no return value is specified - None is returned. The same is True for `print` as well. Soooo, how about `print(x) or y`?
- `lambda:` anonymous functions. Usually used in maps, filters and other functional stuff. But who said we cannot assign it into variable, and use as usual function 🤭 (we won't, I promise) ([Docs](https://docs.python.org/3.12/reference/expressions.html#lambda))

Pretty basic, huh? A few more, that you probably don't use (that often) even if a Python is your primary tool.

- `:=` aka "walrus operator". Inline assignment - allows assigning part of the expression into variable, and use it later inside or outside the same expression. ([Docs](https://docs.python.org/3.12/reference/expressions.html#assignment-expressions))
- `iter(callable, sentinel)` - our lovely `iter()` function that turns a collection into an iterator, have a dark side. Pass a pair of `callable, sentinel` there, and it will call the callable on each iteration, until the sentinel is returned. ([Docs](https://docs.python.org/3.12/library/functions.html#iter), [mCoding explanation](https://youtu.be/YC-12-0sXR8?si=4S7uAbCJlpMcUnUu))
- `locals()` and `globals()` a dict-like object that allows to access/update variables by name in a current local or global scope correspondingly ([`locals` Docs](https://docs.python.org/3.12/library/functions.html#locals), [`globals` Docs](https://docs.python.org/3.12/library/functions.html#locals))
- `exec()` - the nifty function that executes arbitrary python code passed as a string. However it looks like cheating, so let's avoid this as much, as possible. ([Docs](https://docs.python.org/3/library/functions.html#exec))

### Implementation steps

1. **Two loops**. First of all, let's aim to implement multiple game sessions in a single program run. A session can be defined in a single loop, therefore converting the program to one-liner will be quite simple using semicolon `;`
    ```python
    while not_win(); ask_for_input(); etc;
    ```
    It works great, only until there is no nested loops of ifs.

    So the structure we will be
    ```python
    while continue_playing():
        secret_number = random.randint(1000, 9999)
        while True:
            guess = input()
            if not validation(guess):
                continue
            print("bulls: {...}, cows: {...}")
            if guess == secret_number:
                print("You win")
                break 
    ```
    It won't allow us to use nested loops, and force us to come up with something clever

2. **Takewhile**. As we can keep the outer loop control flow using `while`, we need to rework inner one. The most obvious, and probably the only available option, is to write it in a form of list comprehension.

    We will iterate over users guesses
    The output value will be `print()` result of the hints,
    and we will use `if` statement to check if user won, and iteration should be stopped.

    Sounds like `itertools.takewhile(check_and_display_hint(guess, secret_number), user_inputs_iterator)`, right? Wrong.

    Let's make this iterator as bad, as we can. Using list comprehensions.
    ```python
    [x for x in iterator if not (flag := (flag := locals().get("flag", False)) or not condition(x))]
    ```

    Pretty awesome, is it? The main idea here, that we set flag for True as soon as condition met first time. Then we will continue to iterate until the iterator end, but flag value will be taken from the locals, and the iteration is skipped.

3. **User input**.

    Basically, what we want, is iteration over user inputs like this

    ```python
    iterator = iter(input("Enter a number: ", None))
    ```

    But we want few additional functions here. Namely:
    - make sure that user guesses are valid 4-digit numbers
    - allow user to quit the game
    - stop iteration if user won

    So let's split this into two parts:

    First, the generator for the user input control:

    ```python
    func := (x for x in iter(lambda: input("Enter a number (q for quit): "), "q") if validation(x))
    ```
    That will yield only valid numbers (in case of correct validation of course), and will stop immediately if user typed "q".

    Next, use it in the session loop:

    ```python
    iterator = iter(lambda: print("You win!") if flag else next(func), None)
    ```

    We already have `flag` indicator that shows if user win. We can use it to display the message and produce None, to break the iteration. Otherwise, we will take the next number from our user input generator.
    Notice that we cannot use the secret_number as a sentinel, as it will break iteration right away, without displaying messages about 4 bulls and user victory.

4. **Outer loop**

    Now, as we already have all the components of the inner loop, we can assemble it all together to something like this:

    ```python
    while True;import random;number=random.randint(1000, 9999);[print(...) for x in ... if not (flag := (flag := locals().get("flag", False)) or not x!=number)];exit() if input("play again?")!="y" else None
    ```
    > Notice that random is put inside the loop, because the `while` should be the first statement

    But it's too boring. Let's make the whole program even worse, and combine all together into a single list comprehension.

    Remember this guy?
    ```python
    [smth(x) for x in iterator if smth_other(x)]
    ```
    ~~this is him now~~

    the idea here that it will be executed like this
    ```python
    for x in iterator:
        if smth_other(x):
            smth(x)
    ```

    So `if` part will be executed on each iteration, and the main expression only if `if` returned True.

    Nevertheless, with tuples `(of:=named, expre:=ssions)` we can actually put our code wherever we want.

    But, for sake of **simplicity** (ha ha), let's put all the stuff related to a game session into an `if` part, and the control flow into a main expression. As `while True` was originally used, we will use iteration over `iter(lambda: '', None)`, so sentinel never met.

    > I will add some enters, to give you at least the smallest chance to understand what's going on
    ```python
    import random;
    [
        exit() if input("Wanna play again? (y/n): ").lower()!="y" else None

        for _ in iter(lambda: '', None)

        if (
            number:=str(random.randint(1000, 9999)),
            # all the assignments and prints here,
            ...,
            # game loop
            [
                print(...) for i, guess in iterator... 
                if not (flag := (flag := locals().get("flag", False)) or not guess != number)
            ]
        ) or True
    ]
    ```
    The latest `True` probably does nothing, but I like how it looks (anyway this is a bad code challenge, not a codegolf)

5. **Imports**

    At this moment our program almost fit into a single list comprehension. The only problem are imports, they cannot be embedded into other expressions that easily.

    I tried to postpone this as much as I can, but looks it cannot be avoided... we will use `exec`

    Our program depends on `random` (I don't want to implement another challenge here :)) and `collections.Counter` (because I'm lazy)`. So let's import them in our comprehension.

    Key trick here, that we cannot just do `exec("import random;from collections import Counter")` and hope that it will be fine. The exec runs in a separate scope. To actually import, we need to pass `globals` of our session, into the `exec` session. Luckily for us, it is supported:

    ```
    exec("import random;from collections import Counter", globals())
    ```

    ```python
    [
        exit() if input("Wanna play again? (y/n): ").lower()!="y" else None

        for _ in iter(lambda: '', None)

        if (
            exec("import random;from collections import Counter", globals()),  # <-- here
            number:=str(random.randint(1000, 9999)),
            ...,
            [
                ...
            ]
        ) or True
    ]
    ```

6. **Final result**

    _18+, viewer discretion is advised_

    `.py` file: [link](./comprehensive_cows/__main__.py)

    Or just

    ```python
    [exit() if input("Wanna play again? (y/n): ").lower()!="y" else None for _ in iter(lambda: '', None) if (exec("import random;from collections import Counter;",globals()),number:=str(random.randint(1000, 9999)),flag:=False,print("Number is set! Try to guess it"),[print(f"{i}, {guess=}: {(bulls := sum([guess[i] == number[i] for i in range(4)]))} bulls, {sum((Counter(number) & Counter(guess)).values()) - bulls} cows") for i, guess in enumerate(iter(lambda: print("You win!") if flag else next(((x for x in iter(lambda: input("Enter a number (q for quit): "), "q") if((x.isnumeric() and (len(x) == 4))or x=="q" or print("Invalid input; should have 4 digits or 'q'"))))), None)) if not (flag := (flag := locals().get("flag", False)) or not guess != number)]) or True]
    ```
    > please format it yourself. I'm too tired after it all

Oh, you still there? Thanks for reading!

Write a good code and keep learning 🚀
