# CLI Training Overview

```powershell
###########################################################
# Windows specific command prompt instructions shown here #
###########################################################
```

```shell
#########################################################
# macOS specific command prompt instructions shown here #
#########################################################
```

This guide uses the Bonsai Command Line Interface (CLI) to walk you through how to setup your first project and get a simulator running, all from your local computer or virtual machine. If you haven’t yet, it will be helpful to read the [Quick Start Overview][1] to give context to the commands you’ll be running in this guide. The CLI is especially useful for automation and connection to other tools. Currently, there are some actions that can only be performed using the CLI, such as loading your Inkling file and connecting your simulator.

The reference document for all of the commands you will see in this guide can be found in the [CLI Reference][3].


# Setup Your Project

## Install the Bonsai CLI

If you haven't already, please follow the [Install the CLI][2] guide. This guide will walk you through any prerequisites you may need and then will link you back to this guide when you're done.

## Clone a Sample Project

```
git clone https://github.com/BonsaiAI/bonsai-sdk
```

Clone the Bonsai Python SDK with `git clone`. This will provide you with all of the samples, including [OpenAI Gym's Mountain Car][4] environment, which we will be using for this guide.

If you wish to experiment with your own project at this stage you can. Your project will need an Inkling (.ink) file and a simulator file. Many other projects can be found in the samples folder of the SDK you have just cloned.

## Install Project Requirements

> Python 2 and Anaconda

```
cd bonsai-sdk/samples/openai-gym/gym-mountaincar-sample
pip install -r requirements.txt
```
> Python 3

```powershell
# same as above
```
```shell
cd bonsai-sdk/samples/openai-gym/gym-mountaincar-sample
pip3 install -r requirements.txt
```

Enter into the Mountain Car folder in the samples folder of the SDK and then install the Mountain Car requirements from OpenAI Gym with `pip`.




# Create Your BRAIN

```
bonsai create myMountainCarBrain
```

Create your BRAIN (and its related project file) with [`bonsai create`][18] and give it a name. We used *myMountainCarBrain* in the example code but you can name it whatever you'd like. You will view your BRAIN's training progress on the BRAIN's Details page which can be found on the [BRAIN dashboard][5].

```
bonsai push
```

Use [`bonsai push`][19] to upload the cloned Mountain Car project and its associated files to the Bonsai AI Engine for training.




# Train Your BRAIN

Almost there! Time to tell the Bonsai AI Engine to prepare a new BRAIN version for training. There are currently two options for training a BRAIN, one is to run a simulator remotely using Bonsai's servers, and the other is to run a simulator locally with your own computer.

Simulators which can be trained remotely include EnergyPlus, OpenAI Gym's, and Python simulators all found on our [Examples documentation][23]. If the simulator is not one that we support remotely, you'll need to run it locally.

## Train Remotely or Locally

### Remotely

```
bonsai train start --remote
```

Start training your BRAIN with [`bonsai train start --remote`][20].

Remote training is the best way to train BRAINs that need to run overnight or for multiple days at a time. There is no disconnection of the simulator to the server like there can be when training locally.

### Locally

```
bonsai train start
```

> Python 2

```
python mountaincar_simulator.py --brain=myMountainCarBrain --headless
```
> Python 3

```powershell
# same as above
```
```shell
python3 mountaincar_simulator.py --brain=myMountainCarBrain --headless
```

Start training your BRAIN with [`bonsai train start`][20].

Once you have started training mode it's time to start running your simulator locally by calling Python and then the simulator file. Training will begin automatically and if the simulator gets disconnected, training resumes from the same point when the simulator is reconnected if it's within an hour of the disconnect. If you want your training to take place overnight, make sure that your computer won't go to sleep and disconnect for more than an hour or that training time will be lost!

<aside class="notice">
We use headless here to indicate we don't need to see a graphical display from the simulator; if you'd like to see it and watch the simulator learn, omit this option.
</aside>

## View your BRAIN training status

> ![Mountaincar Training][16]

View your BRAIN's training status as it trains on the simulator by going to the BRAIN's Dashboard page on [beta.bons.ai][5]. Training Mountain Car takes about an hour to get sufficient training to beat the game most of the time. If you want flawless victories each time the simulator can take up to 2 hours before you'll see the graph level out.

There is no automatic ending to training, you can train forever, but there will be diminishing returns after a couple of hours. You can play around with training for 15 mins, 30 mins, 1 hour, etc and use your BRAIN to see how well it plays each time! It takes about 700 episodes to train the BRAIN correctly. Our ideal target is an average reward of -195 or higher over 100 consecutive episodes.

[//]: # (Update this when we have multiple concepts and smart ending)

## Stop Training

```
bonsai train stop
```

Once the BRAIN has gotten to this level of performance (or sooner if you prefer), CTRL-C to disconnect the simulator (if you are running it locally), then [`bonsai train stop`][21] will end the training, and proceed to prediction.

# Predict with Your BRAIN

> Python 2

```
python mountaincar_simulator.py --predict=latest
```
> Python 3

```powershell
# same as above
```
```shell
python3 mountaincar_simulator.py --predict=latest
```

After your BRAIN is finished training it can play the Mountain Car game. How well it does depends on how long you let it train! Using your BRAIN involves calling Python on your simulator file, but now in prediction mode with `--predict=latest` which will use the version of the latest training session that you just ran. You can also specify a number like `--predict=1` if you want to predict from a different version of your brain if you have trained it multiple times.

# Next Steps

Now that you've completed this guide, you can:

**Check out Tutorial 1 - [Running Simulations and Writing Inkling][6]**

It contains all of the information you need to get up and running with your own projects on the Bonsai Platform. You'll learn:

* The basics of reinforcement learning 
* How to hook up a simulation to the Bonsai Platform
* The core components of the Inkling language
* How to train a BRAIN with custom Inkling code

And we have these other resources that will enable you to maximize your AI development experience:

* [Understand BRAIN Graphs][7] Guide
* VIDEO: [Writing Great Reward Functions][8]
* [Learn the Inkling Language][9] Guide


[1]: getting-started.html#overview
[2]: cli-install-guide.html#install-prerequisites
[3]: ../references/cli-reference.html
[4]: https://gym.openai.com/envs/MountainCar-v0
[5]: https://beta.bons.ai
[6]: ../tutorials/tutorial1.html
[7]: ./web-graphs-guide.html
[8]: https://www.youtube.com/watch?v=0R3PnJEisqk&list=PLAktfMEMCsOY9HUZKIuGI6yqefGBuszAV&index=4
[9]: ./inkling2-guide.html
[16]: ../images/graph-mountaincar-training.png
[17]: ../references/cli-reference.html#bonsai-configure
[18]: ../references/cli-reference.html#bonsai-create
[19]: ../references/cli-reference.html#bonsai-push
[20]: ../references/cli-reference.html#bonsai-train-start
[21]: ../references/cli-reference.html#bonsai-train-stop
[22]: https://github.com/BonsaiAI
[23]: ../examples.html
