# Concepts

A **concept** in Inkling is an (abstract) idea. In Inkling, the concept defines what you are going to teach the AI and how to teach it. For more information about using the concept keyword, refer to the [Concept Reference][1].

Every Inkling program needs at least one concept, which provides the output to the concept graph. Additional concepts may be defined to assist in teaching the output concept.

## Examples

### Example: Pac-Man

If you want to teach your AI to play a game, the result of the AI learning this concept should be that it knows how to play the game. For example, consider an Inkling program that teaches an AI to play Pac-Man. The goal of the main concept is to get a high score.

Supporting concepts might include `eat_dots`, `avoid_ghosts`, `eat_fruit`, `avoid_blue_ghosts`, `dot_location`, `ghost_location`, and `ghost_color`.

### Example: Tetris

Consider the game [Tetris][2]. As with Pac-Man, we want the AI to get the highest score possible. To assist in teaching the output concept, we can consider various other concepts that may assist the AI. Here's a short list, but you might be able to think of other facts you'd want to use:

* `active_tetrimino`: the tetrimino (piece) that is active.
* `tetrimino_location`: the location of the active tetrimino.
* `tetrimino_speed`: how fast the tetrimino is moving.
* `clear_base`: fit tetriminos into the base to clear pieces of it.

Any of these could be written into supporting concepts that would give information to the output concept.

### Example: Home Automation

Let's imagine that we want to save energy by turning off the lights in empty rooms. The output concept may be called `turn_off_lights_empty_rooms`. To support the output concept, we need to think about other concepts that may assist the AI. Here's a short list, but you might be able to think of other facts that you'd want to use:

* `light_state`: the on/off state of the lights in each room.
* `people_home`: The total number of people in the house.
* `person_location`: The location of each person/the room each person is in.
* `time_empty`: The amount of time a room has been empty.
* `definitely_empty`: don't shut off lights if someone is coming right back.
* `still_in_use`: (a variant of the one above) don't turn off the lights if an empty room might be reoccupied soon.

Any of these could be written into supporting concepts that would give information to the output concept.

## Writing Concept Statements

In Inkling, concepts are declared using the `concept` keyword followed by the concept's name, input list and output type. For readability, a descriptive name is recommended.

Every learned concept must include a curriculum, which tells the AI engine how to train the concept. These will be covered below.

For more information on the concept statement, see the [Concept Reference][1].

[1]: ./../references/inkling2-reference.html#concepts
[2]: https://en.wikipedia.org/wiki/Tetris
