# Let's Get Started!

Before you begin, you will need to have access to our Beta. If you don't have access yet, request access at [bons.ai][1] and then come visit us in the Forums and let us know why you're excited to try out Bonsai!

The next section walks you through how to install Python, Git, and Pip, but if you're already ahead of the curve and have them all installed, skip down to [Setup the Bonsai CLI][2]. 

**Note:** We highly recommend Mac users use HomeBrew or download Python directly in the General Install instructions so they can perform the following steps on user-land Python rather than [system Python][13].

# Install Prerequisites

If you have Anaconda installed on your system, skip down to [Install with Anaconda][9].

If you *do not* have Anaconda installed on your system continue to follow our **General Install** instructions.

## General Install

### Windows

> [Chocolatey][14] commands

```shell
choco install git
choco install python # or you can install python2 if you need version 2.7
```

1. **Git**: Download and install Git from [Git for Windows][11] or with [Chocolatey][14].
2. **Python**: Download and install Python from [python.org][3] or with [Chocolatey][14].
3. **Pip**: Python version 2.7.9 and greater come with pip, but if for some reason you need to use a different version of python please follow [these instructions][12].

**Note:** You may need to reboot your computer after installing Python and Git to make sure all install settings take effect.

### Mac OS X

> [Homebrew][6] commands

```shell
brew install git
brew install python3 # or you can install python if you need version 2.7
```

1. **Git**: Git is more than likely already installed on your computer if you have Xcode, but if not, run the git command or install with [Homebrew][6].
2. **Python**: Download and install Python from [python.org][3] or with [Homebrew][6].
3. **Pip**: Python version 2.7.9 and greater come with pip, but if for some reason you need to use a different version of python please follow [these instructions][12].

## Install with Anaconda

If you *do not* have Anaconda installed on your system, please skip this section.

conda is a tool for installing and managing Python and R dependencies. For more information, refer to the [Anaconda website][10].

> Windows

```shell
conda create -n gym-env
activate gym-env
conda install pip
```

> Mac OS

```shell
conda create -n gym-env
source activate gym-env
conda install pip
```

# Setup the Bonsai CLI

Install the Bonsai Command Line Interface tool and configure it. You'll need to have your access code (accessKey). You can find your access code in your [Bonsai Account Settings][8].

```shell
pip install bonsai-cli
bonsai configure # Follow the instructions to retrieve your key, and enter it when prompted
```

## Clone a sample project

Clone a sample simulation project with git. For this guide we'll be walking you through [OpenAI Gym's][4], Mountain Car simulation.

```shell
git clone https://github.com/BonsaiAI/gym-mountaincar-sample
```

## Install requirements

Enter into the Mountain Car folder and then install the mountain car requirements from OpenAI Gym.

```shell
cd gym-mountaincar-sample
pip install -r requirements.txt
```

# Create a BRAIN

Create your BRAIN and give it a name. You can also create a BRAIN from your beta.bons.ai BRAIN Dashboard instead of using the command line. Either way, you will view your BRAIN's progress on the BRAIN Details page.

### Create from Web

![BRAINS Dashboard Page][15]

The dashboard has a NEW BRAIN button, pictured above. Click on it to start the BRAIN creation process, where you're provided a form to give your brain a name and description. Name can contain alpha, numerals, or hyphens; the description is arbitrary and should contain human-readable text that explains your intent for the BRAIN.

### Create from Command Line

```shell
bonsai create myMountainCarBrain
```
You may give you brain a name with `bonsai create BRAIN_NAME` but there is no option for a description with this method.

# Train a BRAIN

## Load sample Inkling into your BRAIN

```shell
bonsai load
```

Load our sample Inkling file for the Mountain Car simulation.

## Start Training Mode

```shell
bonsai train start
```

Almost there! Time to tell the AI engine to prepare a new BRAIN version for training. Start the training mode for your BRAIN.

## Connect the simulator for training

```shell
python mountaincar_simulator.py --train-brain=myMountainCarBrain --headless
```

Everything is in place, it's time to start the simulator and test things out. To do this, call Python and then the simulator file. Training will begin automatically and if the simulator gets disconnected, training resumes from the same point when the simulator is reconnected if it's within an hour of the disconnect. Remember, as mentioned in [2.3 What is a simulator?][7] if you want your training to take place overnight, make sure that your computer won't go to sleep and disconnect for more than an hour or that training time will be lost!

**Note:** we use headless here to indicate we don't need to see a graphical display from the simulator; if you'd like to see it and watch the simulator learn, omit this option. You'll need to remember to close this window when you're done with training before you can stop the training mode.

## View your BRAIN training status

View your BRAIN's training status as it trains on the simulator by going to the BRAIN's Dashboard page on [beta.bons.ai][5]. Training Mountain Car takes about an hour to get sufficient training to beat the game most of the time. If you want flawless victories each time the simulator can take up to 2 hours before you'll see the graph level out.

![Fully Trained BRAIN][16]

There is no automatic ending to training, you can train forever, but there will be diminishing returns after a couple of hours. You can play around with training for 15 mins, 30 mins, 1 hour, etc and use your BRAIN to see how well it plays each time! It takes about 700 episodes to train the BRAIN correctly. Our ideal target is an average reward of less than -195 or higher over 100 consecutive episodes.

## Stop training

```shell
bonsai train stop
```

Once we've gotten to this level of performance (or sooner if you prefer), CTRL-C to stop the simulator and proceed to Prediction.

# Use your BRAIN

```shell
python mountaincar_simulator.py --predict-brain=myMountainCarBrain --predict-version=latest
```

After your BRAIN is finished training it can play the mountain car game. How well it does depends on how long you let it train! Using your BRAIN involves calling Python on the same simulator file, but now in predict mode, and `predict-version=latest` will use the latest training session that you just ran.


[1]: http://pages.bons.ai/apply.html
[2]: ./getting_started.html#setup-the-bonsai-cli
[3]: https://www.python.org
[4]: https://gym.openai.com/envs/MountainCar-v0
[5]: https://beta.bons.ai
[6]: http://brew.sh/
[7]: ./getting_started.html#what-is-a-simulator
[8]: https://beta.bons.ai/accounts/settings
[9]: ./getting_started.html#install-with-anaconda
[10]: https://www.continuum.io/anaconda-overview
[11]: https://git-for-windows.github.io/
[12]: https://pip.pypa.io/en/stable/installing/
[13]: https://github.com/MacPython/wiki/wiki/Which-Python
[14]: https://chocolatey.org/
[15]: ../images/no_brains_image.png
[16]: ../images/fully_trained_brain.png