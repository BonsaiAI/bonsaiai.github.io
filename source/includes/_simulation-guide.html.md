# Simulation Overview

[**The Bonsai platform**](http://docs.bons.ai/guides/ai-engine-guide.html#major-components-overview) uses simulations to train a BRAIN. This guide describes requirements and recommendations for simulators that are well suited to use with Reinforcement Learning. Some of these are related to the simulation model itself while others are directed at the simulator platform that runs the models. Typically, model related requirements are focused on whether Deep Reinforcement Learning is applicable or not, while simulator platform requirements determine if and how the Bonsai backend can connect and control the simulation environment through the Bonsai SDK.

# Simulating an Environment

Most fundamentally, the simulation model should simulate the environment of interest, starting at an initial state, and modeling the effects of chosen actions on the environment.
The environment state should be well-defined and accessible at each step of the simulation.

## States

Examples of states for different settings:

* **HVAC:** current temperature, humidity inside and outside the building, desired temperature.
* **Driving:** position and velocity of the car, current steering angle, positions of relevant outside objects, location of next waypoint, etc.

## Actions

There should be **identifiable actions** that an agent can control.
E.g. in a simulation of a manufacturing machine, control actions affect whether the part is made correctly. By contrast, in a weather model, the simulation computes what’s likely to happen tomorrow, but there isn’t much that an agent can do to affect the outcome.

* The simulation should be able to evaluate when the system gets into “terminal” states when it does not make sense to proceed further. <br> Typical scenarios:
    * The agent succeeded at a given task (e.g. “machine successfully manufactured a part within tolerance”, or “vehicle successfully navigated around obstacle”)
    * The simulated system got into a bad or invalid state (e.g. “steering tried to drive off a cliff”)
    * Time limit hit (e.g. “the model simulates one year of weather and demand for a building HVAC system, and we’ve hit day 365”)
  
* The model should capture the key features of the environment well enough that an action policy that works well in the simulation is likely to work well in the real environment.
* The needed level of fidelity depends on the precision required in individual actions, which in turn can depend on how feasible it is to recover from an imperfect action at one step. e.g. in driving, if the real world car turns left/right just a bit faster than the simulation in response to the same command, this may be ok - the agent will slightly oversteer, then correct. If the difference is too large, it may lead to unstable control.
* The best practice is to run experiments in the real system as early as practical to understand the effects of simulation limitations.

# Integrating with Bonsai

To use your simulation with the Bonsai platform, you’ll need to connect it. This requires the following:

A way to integrate with the [**Bonsai SDK**](http://docs.bons.ai/references/library-reference.html), currently supported in Python and C++.

Ideally, the simulator provides an API or SDK that can be used for integration. For custom simulations, it may be possible to directly edit the simulation code to integrate with Bonsai.

## Integration Components:

Reset the simulation into an initial state. Ideally, the simulation can be initialized into a variety of different states, to allow the system to learn from many starting point. Execute a specified action and return the resulting state, and whether that state is a terminal.

A way to programmatically configure internal parameters of the simulation, as needed to cover the relevant range of simulated environments. e.g. in an HVAC simulator, parameters such as weather variability and building size can be used to learn an agent that’s robust across varied environments.

Once your simulation is integrated, you will need to define a reward function that helps the system learn what actions to take. The details of reward function definition are beyond the scope of this document, but it is worth considering whether there are any rules for what good and bad actions or good and bad states are. See our [**blog**](https://bons.ai/blog/reward-functions-reinforcement-learning-video) post on writing good reward functions for more.

An additional consideration is facilitating visualization and analysis of performance: ideally, the simulation should output useful visualizations or other data while controlled by the AI agent.

## Simulation Performance

Training reinforcement learning algorithms can require simulating millions of actions and resulting state transitions. To make this practical, your simulation should:

* Simulate the effects of each action in no more than a few seconds. Milliseconds is preferred.
* Some simulators (e.g. Simulink) allow compiling models to a C program that runs much faster than the original. When possible, we encourage this.
* It is ideal if the simulation can be easily run many times on the same machine, and on many machines in parallel.
Simulators that allow many models to be run without requiring separate licenses for each make it easier to parallelize.

# Next Steps

Now that you’ve reviewed this guide, you can:

* Download the [Bonsai SDK][1] to build or integrate your simulator
* Read the [Simulator Reference][2] for details on how to use the SDK
* [Learn the Inkling Language][3]
* Download [Bonsai Platform Examples][4] to test out an example on your computer


[1]: ./sdk-install-guide.html
[2]: ../references/simulator-reference.html
[3]: ./inkling2-guide.html
[4]: ../examples.html

