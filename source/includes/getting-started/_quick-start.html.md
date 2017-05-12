# Create Your BRAIN

> Open AI's Cartpole Gym
> ![Cartpole Trained][12]

Before you begin, you will need to have access to our private beta. If you don't have access yet,
request access at [bons.ai][1].

In this guide, we’ll walk you through creating a BRAIN to train the [OpenAI Gym][4] environment for
Cartpole, a simple balance control problem. The walkthrough of the Inkling code and simulator
interface can be found in our [Examples][2] and the full source code for you to take a look at
is on [Bonsai's GitHub][3].

If you'd prefer not to use our web interface, head to our [CLI Guide][11] to follow a similar guide
to this one on your local computer.

## BRAIN Dashboard

> ![No BRAIN Dashboard][5]

The dashboard has a New BRAIN button as pictured. Click on it to create a BRAIN.

> ![Setup Cartpole BRAIN][6]

Select ‘Cartpole’ from the set of BRAIN templates. The Cartpole simulation can be managed on Bonsai’s
servers, so you don't need to run anything locally on your computer to train this simulation. At
this time, you will also be asked to give your BRAIN a name and a description.

Click on the "Create" button which will take you to the BRAIN Details page.

## BRAIN Details

> BRAIN Graph
> ![Untrained BRAIN graph][7]

### BRAIN Training Graph

Do not be afraid! You don't have data yet because you haven't trained your new BRAIN! That's the
next step. This graph will display each of your concepts to be trained (if you look in the Inkling
code you'll see that Cartpole only has one).

> Code Editor Window
> ![Code Editor Window][9]

### Code Editor Window

The code editor shows you all of the files that are contained within your project and the contents
of each file if you click on them on the left.

Your code will automatically compile (if it's Inkling) and save every few seconds while you are
editing. You never need to edit code in the browser if you don't want to. Check out our
[CLI Guide][11] to install the Bonsai CLI locally, download these files, and run the rest of this
guide locally if you want.



# Train Your BRAIN

> ![Start Training BRAIN][8]

> ![Trained BRAIN][10]

When you're done checking out the code editor and ready to start training this BRAIN on the
Cartpole simulator, simply click the **Start Training** button below the graph.

What this is going to do is spin up a container for the [AI Engine][14], connect up to the
`cartpole_simulator.py` file, and start the BRAIN training. All with one button!

Training Cartpole for ~2,000 episodes is ideal, and might take between 15-30 minutes depending on
how well your BRAIN is performing. You can see in the images that it only took about a minute for
the first ~1,000 episodes but the second half took 14 minutes. Your graphs will vary greatly so
don't worry if they don't look like these.

The server will automatically end training once the BRAIN reaches past a certain accuracy, but you
will most likely want to stop training before it gets there because there will be diminishing returns
for training past about 2,000 episodes. You can play around with training for
15 mins, 30 mins, or even 1 hour and use your BRAIN to see how well it plays each time!



# Use Your BRAIN

> ![Cartpole Trained][12]

Congratulations on training your first BRAIN! "What do I do with it now?" you may ask.

To use the BRAIN you've just trained you'll need to locally run the python simulator for OpenAI Gym.
If this was your own simulation you had written, this would be the part where you hook it up to
your application and get a prediction of your own!

If you **decide to skip this setup** for any reason or have trouble installing the Bonsai CLI you
can get the satisfaction of what the simulation would look like by watching this pole balance on
the cart in gif form.

## Install Bonsai CLI

> Install Bonsai CLI

```shell
pip install bonsai-cli

# If you're running python3 on macOS:
pip3 install bonsai-cli
```

The Bonsai Command Line Interface (CLI) is a command line tool that enables you to configure the
Bonsai AI Engine. The CLI is especially useful for automation and connection to other tools.
Currently, there are some actions that can only be performed using the CLI, such as loading your
Inkling file and connecting your simulator.

If you already **have python installed** you should be able to run the python package manager `pip`
to install the Bonsai CLI. Otherwise, check out the [CLI Guide Install Prerequisites][13] section
to get everything installed and then come back here.

## Run Prediction

> Download Project Files

```
bonsai download myCartpole
cd myCartpole
```

> Install Simulator's Requirements

```shell
pip install -r requirements.txt

# If you're running python3 on macOS:
pip3 install -r requirements.txt
```

The `cartpole_simulator.py` simulator file that you need to run is one of the project files the
server created when you made your BRAIN. You can download these project files locally with the
`download` command, and then `cd` into that folder to look around.

You'll also want to run the `requirements.txt` file which contains all of the packages your simulator needs to run. This `pip` command will make sure all of them are installed, and if not, install them for you.

> Use Your BRAIN

```shell
python cartpole_simulator.py --predict-brain=myCartpole --predict-version=latest

# If you're running python3 on macOS:
python3 cartpole_simulator.py --predict-brain=myCartpole --predict-version=latest
```

It's finally time to run the simulator using predictions from your BRAIN! How well it does depends
on how long you let it train. Using your BRAIN involves calling Python on the project's simulator
file, in predict mode, and `--predict-version=latest` will use the latest training session that you just ran.



[1]: http://pages.bons.ai/apply.html
[2]: ../examples.html#openai-gym-cart-pole
[3]: https://github.com/BonsaiAI/gym-cartpole-sample
[4]: https://gym.openai.com/envs/CartPole-v1
[5]: ../../images/no_brains_image.png
[6]: ../../images/quick-start-create-brain.png
[7]: ../../images/quick-start-no-training.png
[8]: ../../images/quick-start-start-training.png
[9]: ../../images/quick-start-code-editor.png
[10]: ../../images/quick-start-trained-brain.png
[11]: ./cli-guide.html
[12]: ../../images/cart-pole-balance.gif
[13]: ./cli-guide.html#install-prerequisites
[14]: #what-are-brains
