# Connecting Your BRAIN

> Adaptation of previous RL image to show the translation onto the Bonsai Platform
> ![Overview Image](../images/tutorial1-overview.png)

To connect our simulation to a BRAIN in the Bonsai platform, we will need an Inkling program that describes the problem and how to teach the AI to solve it. Inkling is a programming language specifically designed for artificial intelligence (AI). It abstracts away the vast world of dynamic AI algorithms that require expertise in machine learning and enables more developers and subject matter experts to create AI.

The problem description includes type definitions for the states the BRAIN will receive and the actions it will need to send. The description of how to teach the BRAIN includes the reward, as well as any decomposition of the problem into *concepts* and the sequencing of learning using *lessons*. This first tutorial will use a single concept and a single lesson.

## Types

A [type][1] in Inkling describes the format and allowed ranges for a data value.

#### Exercises

* Fill in the state type
* Fill in the action type

###### Fill in the state type

```inkling2--exercise
type GameState {
    # EXERCISE: Add a field name and type for each variable of x and y position.
    # These have to match the dictionary returned by _get_state() in our simulator.
    # <Your code goes here>
}
```

```inkling2--solution
type GameState {
    # X and Y direction of the point. These names (and types) have to match the
    # dictionary returned by _get_state() our simulator.
    dx: number,
    dy: number
}
```

The state type describes the state of the environment, and must match the state values returned from the simulator. In this case, the state consists of two numbers named `dx` and `dy`. Now, fill in the state type definition. (Refer to the [Inkling reference][2] if you're not sure how.)

<aside class="notice">
Click the "Solution" tab of the code panel to see the answer to this exercise.
</aside>

###### Fill in the action type

```inkling2--exercise
type PlayerMove {
    # EXERCISE: The field names and types must match the parameter to step() in
    # our simulator. You need to specify the range and step size for the action.
    # <Your code goes here>
}
```

```inkling2--solution
type PlayerMove {
    # This field names and types must match the parameter to step() in our
    # simulator. We specify the range and step size for the action.
    # A constraint {0,1.575,3.142, 4.712} would also work
    direction_radius: number<0 .. 6.283 step 1.575>  
}
```

Next, let's define the action type. The action in our problem corresponds to picking a direction, specified as a number of radians and named `direction_radians`. In this case, we want the system to pick from the four cardinal directions: 0, pi/2, pi, 3\*pi/2. Use the Inkling reference to look up the syntax for number type constraints and fill in the action type. (Bonus: there are at least two ways to do it.)

<aside class="notice">
Click the "Solution" tab of the code panel to see the answer to this exercise.
</aside>

## Concepts

```inkling2
graph (input: GameState) {
    concept FindTheTarget(input): PlayerMove {
        # Curriculum omitted
    }

    output FindTheTarget
}
```

A `concept` in Inkling defines what you are going to teach the AI. By declaring a concept, you are instructing the AI Engine to learn a new function that maps a set of inputs to outputs. Each concept must include a curriculum.

In this simulation, we are asking the agent to learn the concept `FindTheTarget` by choosing a direction (of type `PlayerMove`) after seeing the current state (of type `GameState`).

For more information about using the concept keyword, refer to the [Concept Reference][3].

## Curriculum and Lessons

```inkling2
curriculum {
    # Specify a simulator as the data source
    source MoveAPointSim
}
```

A `curriculum` in Inkling is used to define what and how to teach a concept. Each concept must contain a curriculum. It refers to a simulator that acts as a data source for training the concept.

[1]: ../references/inkling2-reference.html#types
[2]: ../references/inkling2-reference.html#type-references
[3]: ../references/inkling2-reference.html#concepts
