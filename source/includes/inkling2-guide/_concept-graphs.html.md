# Concept Graphs

Concepts are the ideas that you want to teach your BRAIN. A BRAIN is organized into a directed graph of concepts with information flowing from the input of the graph to the output. A minimal concept graph may contain only a single concept. More complex concept graphs contain additional concepts whose outputs form inputs to yet other concepts. All concepts graphs must designate one concept whose output becomes the output for the entire BRAIN.

For more information about Concepts, refer below to [Concepts][2] and the [Concept Reference][3].

## Building a Concept Graph

As you think about the problem you're trying to solve, consider how it may make sense to break up into subconcepts that can be taught independently and in an incremental manner. Start by thinking about the problem and what criteria must be met to solve it. For example:

* Learning to play a game
* Learning to save electricity in your home
* Learning to guide an agent to a specified location

After you've determined your success criteria, you want to decide what the AI can do to prove it has successfully learned how to achieve those criteria. In a game, this could be, "I think if the AI can get a high score, it has successfully learned to play the game." This becomes your output `concept`, and it also ties into your reward function, which measures how successful the AI is at learning this concept.

The input to the concept graph typically mirrors the observable inputs for your systems. For example, if you are learning to play a game, the input may be the rasterized image of the game board. If you are learning to save electricity in your home, the inputs may consist of the current outside temperature, current inside temperature, the desired temperature, current electrical rate, etc.

The output of the concept graph typically includes one or more actions that can be effected in your system. For a game, the output may consist of the X and Y values of a joystick controller and the state of one or more buttons. If you are learning to save electricity in your home, the output may consist of control signals that turn on the air conditioner or heater.

## Multi-Concept Graphs

For more complex problems, it may be beneficial to break down the problem into other concepts. The outputs of these concepts can act as inputs for later concepts. By teaching these earlier concepts independently, it is often possible to speed up the overall training time.

For each concept, you will need to determine what inputs are required and what data is expected to be computed as an output. Once a concept is successfully trained, it acts as a learned function that transforms its inputs into outputs. A concept's inputs can come from the graph input or from other concepts in the graph.

Your concept graph can be drawn from left to right, starting with the graph input. Your other concepts come next, followed by your output concept, and finally, to the right, is the output. Concepts are trained independently with earlier concepts being trained before later concepts. To visualize this more clearly, see our expanded example below.

## Examples

### Example: Breakout

```inkling2
graph (input: GameState) {
  concept get_high_score(input): PlayerMove {
    # Curriculum omitted
  }

  output get_high_score
}
```

This program defines a single-concept graph. The name of the concept is get_high_score, and its input is the same as the input to the graph. It outputs a data type called PlayerMove. (More on types and type declarations below.)

The output statement indicates that the get_high_score concept is the output for the entire graph.

![][1]

Single concept graph for learning Breakout

The `input` is a snapshot of the game in its current state.

The concept `get_high_score`, is the output `concept`. It describes what we want the BRAIN to learn (in this case, to get a high score in the simulated Breakout game). This single-concept graph is sufficient to train a BRAIN to play the game, but training may take a long time.

The `output` type PlayerMove describes the move that the AI wants to make. It chooses from left, right, or no move. The definition of the GameState and PlayerMove types are not shown.

To learn the `get_high_score` concept more quickly, additional concepts can be added to the concept graph.

```inkling2
graph (input: GameState) {
  concept BallLocation(input): Coordinates {
    # Curriculum omitted
  }

  concept KeepPaddleUnderBall(input, BallLocation): PlayerMove {
    # Curriculum omitted
  }

  concept GetHighScore(input, KeepPaddleUnderBall): PlayerMove {
    # Curriculum omitted
  }

  output GetHighScore
}
```

In this example, we've added two extra concepts that support our output concept and give our BRAIN more information to work with. The concept of `ball_location` computes the coordinates of the ball, which are provided as inputs to the concept `keep_paddle_under_ball`, and that concept provides input to `get_high_score`.

The graph input remains the same as before. The AI has two additional concepts to train. When it is training the `ball_location` concept, it takes in the input and returns the coordinates of the ball. On the `keep_paddle_under_ball` concept, it takes in the input _and_ the ball coordinates from `ball_location`. Then, that concept returns the best move to attempt to keep the paddle underneath the ball. The final concept, `get_high_score`, takes in the input and the move returned from `keep_paddle_under_ball` and returns the move that will optimize the high score. Each of these concepts are trained from input to output.

[1]: ../images/mental-model.png
[2]: #concepts
[3]: ./../references/inkling2-reference.html#concepts
