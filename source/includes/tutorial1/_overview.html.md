# Problem Overview 

> ![RL Overview Image](../images/tutorial1-rl-overview.png)

In this tutorial you will learn about reinforcement learning (RL), and using RL to solve a simple simulated problem using Inkling and the Bonsai Platform. This will include learning the essential components of the Inkling language needed to connect the simulation to the platform.  You will familiarize yourself with the simulation, test it, then fill in the necessary Inkling types (states and actions) needed to connect the simulation to the platform. By the end of this tutorial, you can expect to be comfortable with the relationship between a BRAIN and a simulation, as well as how to train a BRAIN inside the Bonsai Platform using custom Inkling code, and basic knowledge of Bonsai’s Command Line Interface (CLI).

We expect you to already have read the [Quick Start tutorial overview][1] and [install the CLI][2] before following this tutorial so you have a general understanding of how the platform works and have installed and configured the Bonsai CLI. Please do so before continuing with this tutorial.

The problem we are trying to solve using this simulation is to teach an agent (the Bonsai BRAIN) to move to a target point in a simple planar world. The agent is told where it is relative to the goal, and has to decide which way to move. The agent sends its chosen action to the simulation, which simulates the step, moving the agent to a new location, and this repeats until the agent makes it to the goal, or time runs out.

First, we’ll cover a bit about reinforcement learning for those who are unfamiliar with this style of machine learning.

## How Reinforcement Learning Works

> Visualization of one iteration on the Bonsai Platform
> ![Iteration visualization](../images/tutorial1-iterations.png)

Reinforcement learning (RL), illustrated here, is a machine learning technique for controlling or optimizing a system or process. In RL, an *agent* takes *actions* in an *environment*, getting feedback in the form of *reward*. The agent explores different action strategies or policies and learns to maximize cumulative reward. One cycle from the environment to the agent and back is called an iteration.

### Terminology

This table describes the key terms used in RL:

|     |     |
| --- | --- |
| state | The state of the environment at each iteration (ex: current agent position relative to the target). The agent uses this to decide what action to select. |
| action | Actions define what the agent can do at each iteration (ex: which direction to move in). The goal of RL is to learn to select the right series of actions to achieve an objective. |
| reward | The reward at each iteration gives the agent feedback that helps it learn (ex: higher reward when moving toward the target). An important note is that the agent's goal is to maximize *cumulative* future reward, not the instantaneous reward at each iteration. |
| iteration | An iteration is one state → action → reward → new-state transition in the environment. The agent uses the state to select an action, which causes the environment to transition to a new state, and results in a reward. |
| episode | An episode is a series of iterations, starting in some initial state and ending when the environment hits a termination condition and the environment resets for the next episode. (ex: episode is terminated either when agent reaches the target (success) or time runs out (failure)) Episodes can vary in length, with termination conditions typically defined based on succeeding at the task, failing at the task, getting too far from success, or running out of time. |
| cumulative reward | Cumulative reward is the sum of the per-iteration rewards over an episode. The agent's goal is to select actions that maximize cumulative future reward through the end of the episode. |
| terminal condition | Terminal conditions specify when to end an episode. |
| policy | A policy determines what action the agent selects for every possible state. The goal of RL is to learn a policy that maximizes cumulative future reward. | 

To learn more about reinforcement learning, watch our [training video][3] about types of machine learning.

# Create a BRAIN

Now that you’ve gotten a taste of Reinforcement Learning, let us begin by downloading the code for this tutorial and creating a new BRAIN on the Bonsai Platform for training.

As mentioned in the introduction, you’ll need to have the Bonsai CLI installed before you can run these commands. If you haven’t already done so, please [Install the CLI][4].

```shell
git clone https://github.com/BonsaiAI/bonsai-tutorials.git
cd bonsai-tutorials/tutorial1
```

If you don’t have git installed on your computer you can [download a .zip file][5] of the materials instead.

```shell
# The name of your BRAIN will be move-a-point
bonsai create move-a-point
```

Once you have either git cloned or downloaded the files for this tutorial, navigate on your command prompt inside of the tutorial1 folder and run `bonsai create move-a-point` which will create a new BRAIN for your account called move-a-point. This command also silently uploads your project files to the server so it’s important to use this command within the tutorial1 folder where your Inkling and simulation files are.

<aside class="notice">
The CLI will ask you to run a push command to upload your files after you create your BRAIN. If you do this you will get an Inkling compile error since your Inkling code is not yet complete. Don't worry! We will do this a few sections later in this tutorial.
</aside>

[1]: ../guides/getting-started.html
[2]: ../guides/cli-install-guide.html
[3]: https://www.youtube.com/watch?v=VcwzDqReLPk&index=1&list=PLAktfMEMCsOY9HUZKIuGI6yqefGBuszAV
[4]: ../guides/cli-install-guide.html
[5]: https://github.com/BonsaiAI/bonsai-tutorials/archive/master.zip
