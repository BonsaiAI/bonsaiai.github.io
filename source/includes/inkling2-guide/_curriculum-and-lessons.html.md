# Curriculum and Lessons

A **curriculum** in Inkling defines how to teach a concept. Each concept declaration must contain a curriculum. A curriculum also specifies the data source (the simulator) used to train the concept.

A **lesson** is part of a curriculum. It teaches a specific piece of the concept by specifying training parameters. Lessons enable the AI to learn the concept in stages instead of all at once. Lessons are contained within curriculum statements. If no lessons are specified within a curriculum, a default lesson is assumed.

Each lesson provides an optional simulator configuration. If multiple lessons are specified, they should progress in difficulty with simple configurations followed by increasingly-difficult configurations.

```inkling2
concept Balance
  curriculum {
    source CartpoleSimulator

    lesson Slow {
      constraint { MaxSpeed: 10 }
    }

    lesson Medium {
      constraint { MaxSpeed: 20 }
    }

    lesson Fast {
      constraint { MaxSpeed: 30 }
    }
  }
}
```

For more information and examples, refer to the [Curriculum Reference][1].

## Defining Lessons

### Simulator Configuration Constraints

Each lesson may specify a **constraint** for the simulator's configuration. Earlier lessons must provide more restrictive constraints, while later lessons loosen the constraints. This allows the AI training to learn a simpler form of the concept before learning a more complex form.

The **constraint** keyword must be followed by a type that constrains the simulator's declared configuration type. For more information and examples, refer to the [Lesson Reference][2].

## Reward

The **reward** is a function that returns a numeric value indicating how well the AI is doing.

This idea may be easier to understand with a few examples:

In our Breakout example, our concept was GetHighScore. The **score** is how we are measuring how well the AI is doing at learning that concept.

If we are teaching the AI Pac-Man and we are training the concept of `avoid-non-blue-ghosts`, we may measure how well the AI is learning this concept by measuring the amount of time the AI manages to avoid non-blue ghosts. Our reward function uses **time** as the reward.

###### Defining Rewards

There are several factors you can use to determine what your reward should be. A good reward function should reward the AI more often and be proportional to the quality of the behavior.

### Sparsity of Rewards

The frequency of feedback affects the speed of training. Ideally, the time between the AI's decision making and the reward for the behavior should be short. For example, if you're teaching an AI to play Pac-Man, you want to choose a reward like `score` instead of a reward like `beat level`. In Pac-Man, your score changes often, typically many times during a level, but your level can change only 256 times, and only after many actions. Your AI would need to beat an entire level to be rewarded, which is difficult to do without getting intermediate feedback about which actions along the way were beneficial.

### Proportional Feedback

The reward should be in proportion to the quality or correctness of the behavior. When you are implementing your reward function, you need to consider that the AI will learn exactly what you ask it to learn. This can result in learning what you specified but not necessarily what you wanted. For example, if you ask an AI to play Pong (controlling both agents), and your reward function is based on `game time`, the AI may learn that to get the longest time played, the paddle should remain in the same place. The ball will bounce back and forth between the two unmoving paddles. The AI learned exactly what you told it to learn -- how to maximize the amount of time, but it didn't learn how to play the game.


[1]: ./../references/inkling2-reference.html#curriculums
[2]: ./../references/inkling2-reference.html#lessons
