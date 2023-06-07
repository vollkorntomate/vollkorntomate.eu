---
title: "justCTF 2023: ECC for Dummies"
date: 2023-06-04
draft: false
description: "CTF Writeup of the ECC for Dummies challenge during justCTF 2023"
type: posts
series: ["justCTF 2023"]
categories: ["justCTF 2023"]
tags: ["misc", "easy"]
---
---

This is one of the challenges which have a simple solution, that is just tricky to find. Let's explore.

# Exploring the task

We are given a ZIP file containing a Python program (consisting of two files) and a netcat listener that executes the program on the CTF's infrastructure. The task description states:
> Sometimes you have to force logic to do what you want it to do

Let's first connect to the netcat listener to see what we're up against:

```text
Hello there!
Would you like to play a little game?
I have written 5 values (yes, no) on cards. I am curious if you are able to guess which values they are.
You can ask me 8 logic questions.
I know you're smart, so I won't always be honest about the answers. No worries, I will fool you only 3 times.
0.Question: 
```

Whatever we "ask" it, its answer is always either `True` or `False`. After all 8 questions are asked, we are challenged with providing the zeros and ones that the other side wrote down on their cards. Looking into the Python code, the answer format is supposed to be something like `0 1 1 0 1`. The exact combination of zeros and ones is generated randomly at the beginning of each new execution, so there is no use in brute-forcing them (and it would be rather boring).

Trying out different kinds of questions and taking guesses doesn't lead us anywhere, so it's time to look a little deeper into the code.

# Understanding the code

The code in `main.py` is pretty straightforward: it handles all the input and output, and in the end compares the user-provided *cards* with its generated and outputs the flag if those two match. Handling the 8 questions gets a bit more complex.

Each question is given to the `evaluate` function in `sandbox.py`, which makes use of Python's internal `compile` function. Reading the documentation tells us that the `compile` function takes Python code as a string and outputs the abstract syntax tree (AST) of that code so that it could be executed by the interpreter using `exec` or `eval`.

Before going too much into details about the code, we wondered how exactly the randomly generated cards influence the answers to our questions. The variable `cards` is passed into `evaulate` alongside the questions, but is never used afterwards – only its length is relevant (which is always 5 in this example). Thus, the answers to our questions are in no way helpful to find out the arrangement of zeros and ones that we need to get the flag.

# One question to rule them all

So, we now know that no matter we ask, we can only guess what the contents of the cards are. However, our input is treated as regular Python code, compiled and evaluated (i.e. executed). This is nothing less than arbitrary code execution! So, why not try out the following Python code as the first question:
```py
print(cards)
```

This is the answer we get for this question:
```text
0.Question: print(cards)
[False, True, True, True, True]
Question resolves to: True
```

That looks a lot like our `cards` variable that we need to get the flag! All other 7 questions can simply be skipped with an empty input. Then we just need to convert each `True` to a `1` and each `False` to a `0`, provide it as our response and voilà: there is our flag!

```text
Your response: 0 1 1 1 1 
Great work!
This is yours flag:  justCTF{S0me7ime$_L0Gic_1s_n0T_B1n4ry_0101}
```

> Sometimes logic is not binary!


---
Shout-out and a thanks to my teammates [Jonas](https://github.com/jonas-hoebenreich/), [MrGameCube](https://github.com/mrgamecube) and all others!

---