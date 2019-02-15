# Overview

> ![Brain and Simulator][3]

Welcome to Inkling, Bonsai AI's special purpose programming language for training AI.

Inkling is a declarative, strongly typed programming language specifically designed for artificial intelligence (AI). It abstracts away the vast world of dynamic AI algorithms that require expertise in machine learning and enables more developers to program AI. An Inkling file contains the concepts -- what you want to teach the AI -- and the curriculum, -- how you want to teach the concepts -- necessary to train your BRAIN. Before you read through this section about the Inkling language, you should familiarize yourself with the various components of the Bonsai Artificial Intelligence Engine, which you can read about in our [Getting Started Guide][1].

If you're interesting in finding out more about what's going on under the hood of the Bonsai AI Engine, check out [Understand AI Engine Components][2].

## Inkling: A New Language

### What kind of language is Inkling?

Inkling is a special purpose language for writing AI programs. The language has a blend of declarative and functional styles; it's modeled after ideas in SQL and python. We like to describe Inkling as the world's first machine teaching language.  

This verison of Inkling has been replaced by a newer version. The syntax and terminology of the newer version of Inkling is documented in the [Inkling v2 Guide][5]

### What is Machine Teaching?

Machine teaching is a new programming paradigm. It expresses the solution to a problem in terms of how you teach the computer to find the solution as opposed to how to calculate the solution. Machine teaching focuses on:

* **What do you want the computer to know?**
Machine teaching captures the concepts and their relationship to each other and forms a directed graph of the concepts.

Imagine learning to play baseball; there are a lot of concepts involved. It goes beyond just the rules of the game, and includes things like batting stance, bat grip, eye hand coordination, etc.

* **How do you go about teaching those things?**
Machine teaching focuses on how the problem can be taught, not how it can be calculated.

Imagine teaching a child to play baseball. You can't just explain the whole game to them and expect them to understand how to play. You'd break down learning into pieces, and create a series of lessons that would help them learn how to play baseball.

Let's say, to start, you wanted to teach them the concept of hand-eye coordination. There would be a sequence of lessons to do this. You might start with a whiffle ball on a tee, progress to a tee that pops the ball up, move to underhand pitches, and then overhand pitches. You could continue to build upon your hand eye coordination concept with things like curved pitches, different styles of catching, and other exercises. It's all teaching one concept - hand-eye coordination - using **progressive lessons** to do so.

### Inkling is a Special Purpose Language

Inkling is a special purpose language, meaning that it is a language for writing programs for a particular domain. This is as opposed to a general purpose language, which would allow you to write anything you could imagine. C++, Python, Ruby, and Java are all examples of general purpose programming languages.

### Inkling is a Functional language

In a functional programming language, a function defines a scope and can only modify what is in that scope. In Inkling, every concept takes data as input and sends data as output. Each concept is self contained, and there is no shared information between concepts unless you explicitly declare it. This means that no information is kept as state unless you specifically declare it. This makes Inkling a [functional programming][4] language.

**For example:**

* Python and Javascript can be programmed in a functional way, but they are not pure functional languages.
* F# is a pure functional programming language.
* R, matlab, Java and prolog are not functional programming languages.

### White Space Insensitive

Inkling is white space insensitive. Internally, we prefer to use four spaces to indent lines. This is a stylistic choice. In this document, any mention of an "indentation" or a line "being indented" refers to using four spaces at the start of a line.


[1]: ./getting-started.html#the-bonsai-platform
[2]: ./ai-engine-guide.html
[3]: ../../images/data-flow-simple.png
[4]: https://en.wikipedia.org/wiki/Functional_programming
[5]: ../../guides/inkling2-guide.html#overview
