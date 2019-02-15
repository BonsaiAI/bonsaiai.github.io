# Simulator Overview

> ![Simulator Diagram](../images/brain-sim-marketing.png)

This reference gives you the information you’ll need to train a BRAIN on The Bonsai Platform with simulators. It covers best practices such as running simulations in parallel, general guidelines like how to configure web proxies with the CLI, and what simulators are currently supported.

Currently, The Bonsai Platform depends on simulators as a source of training data for applied deep reinforcement learning (DRL). For more background on applied DRL with simulations read our blog post on [simulations as a training environment][6].

The Bonsai Platform includes client libraries for Python & C++, two common languages used in systems modeling. We also have samples showing how to connect to OpenAI Gym Environments, Simulink models, and EnergyPlus models. To learn more about connecting your simulator, refer to our [Library Reference][4].

## What is a simulator?

A simulator is an imperative model of a process, transitioning from state to state as actions are applied. Robotics, industrial automation, supply chain logistics, and structural engineering are all domains which use simulators to model the behavior of complex systems.

Deep Reinforcement Learning can work with simulated models to train a BRAIN to perform tasks within the modeled system. Tasks can be as simple as "stand this pole upright" or as complex as "learn to walk." 

To be effective, training a BRAIN needs to be done using DRL against a simulated model. The Bonsai SDK allows the DRL system to control your simulator during the training process. Any simulator which has an initial state, and increments through time, can connect to the Bonsai training system.

## Currently Integrated Simulators

The Bonsai Platform supports an API and two client libraries written in Python or C++. The platform also includes connectors built on top of these libraries. Currently available connectors:

* OpenAI Gym Environments (bonsai-gym)
* Simulink Toolbox
* EnergyPlus (via BCVTB)

A list of examples using these are on the [Examples][7] page.

# Integrating a Simulator with Inkling

> ![Simulator Diagram](../images/inkling_simulator_comparison2.png)

This table describes the colors and connections between the various parts of the Inkling file and Simulator file that must be the same or connected. The source code for this example, [Find the Center][11], is available if you wish to copy/paste this example.

| Color               | Description  |
| -                   | -  |
| Purple (dark/light) | The Inkling state type field names and types must match the state dictionaries returned from `episode_start` and `simulate` in the simulator. |
| Blue (dark/light)   | The Inkling action type field names will match the keys in the action dictionary passed to `simulate` in the simulator, and the values will have the types specified in Inkling, and will obey the specified constraints (`<-1, 0, 1>` in the example). |
| Orange (dark/light) | The simulator's configuration passes as `parameters` to the `episode_start`, and will take values from the `constraint` clause in Inkling. |
| Green               | The simulator name must match between the `simulator` clause and the `source` clause in the curriculum. The simulator must pass the same name to the constructor of the `Simulator` class so the AI engine knows which simulator is connected. |

<aside class="notice">
Note that config in __main__ is the brain configuration and remains the same throughout, whereas goal_config (highlighted in orange) is used at the beginning of every episode and must be named the same as it is in Inkling. These configs are unrelated.
</aside>

# Running a Simulator from the CLI

> Simulator config help text

```
$ python <simulator_file>.py --help
usage: <simulator_file>.py [-h] [--accesskey ACCESSKEY] [--username USERNAME]
                 [--url URL] [--proxy PROXY] [--brain BRAIN]
                 [--predict [PREDICT]] [--verbose] [--performance]
                 [--log LOG [LOG ...]] [--record RECORD]

optional arguments:
  -h, --help            show this help message and exit
  --accesskey ACCESSKEY
                        The access key to use when connecting to the BRAIN
                        server. If specified, it will be used instead of any
                        access key information stored in a bonsai config file.
  --username USERNAME   Bonsai username.
  --url URL             Bonsai server URL. The URL should be of the form
                        "https://api.bons.ai"
  --proxy PROXY         Proxy server address and port. Example: localhost:3128
  --brain BRAIN         The name of the BRAIN to connect to. Unless a version
                        is specified the BRAIN will connect for training.
  --predict [PREDICT]   If set, the BRAIN will connect for prediction with the
                        specified version. May be a positive integer number or
                        'latest' for the most recent version. For example:
                        --predict=latest or --predict=3
  --verbose             Enables logging. Alias for --log=all
  --performance         Enables time delta logging. Alias for --log=perf.all
  --log LOG [LOG ...]   Enable logging. Parameters are a list of log domains.
                        Using --log=all will enable all domains. Using
                        --log=none will disable logging.
  --record RECORD       Enable record simulation data to a file (current) or
                        external service (not yet implemented). Parameter is
                        the target file for recorded data. Data format will be
                        inferred from the file extension. Currently supports
                        ".json" and ".csv".
```

Currently, all connectors use a Python program to bootstrap the simulation and connect it to the Bonsai AI Engine. To run any simulation, you call a Python program (shown in the code panel).

During development, you will need to iterate efficiently on your simulation, reward function, and lesson plans. You may also need to connect a debugger, or do additional logging to help you get your simulation ready to train.

During the course of normal operation, one should only need to specify the prediction/training mode on the command line. The rest of the options will be read from configuration files.

The following sections explain further how to use the `--proxy`, `--predict`, and `--record` command line options.

To speed up training, you can run multiple simulators in parallel. See [Running Simulations in Parallel][5] below.

## Proxy Support

> Example Python Proxy

```python
my_config.proxy == 'myproxy:5000'
```

> Example CLI Proxy Usage

```shell
$ python <simulator_file>.py --proxy=localhost:3128
```

To connect a simulator from behind a corporate proxy, tools like the Bonsai SDK need to know where to send traffic. This is done by providing the proxy server address and port to the simulator config when running a simulator. There are multiple ways that you can do this such as through the CLI or in the simulator code itself.

### Environment Variables

> Set Windows Environment Variable

```text
# HTTPS_PROXY and HTTP_PROXY are also supported

| Variable       | Value                                      |
| -------------- | ------------------------------------------ |
| ALL_PROXY      | http://username:password@URL:3456          |
```

> Set macOS/Linux Environment Variable

```sh
# HTTPS_PROXY and HTTP_PROXY are also supported

$ export ALL_PROXY=http://username:password@URL:3456 
```

If the previous usage for the SDK does not work for the CLI as well you may need to specify the proxy details as environment variables.  Proxy details differ between companies, so ask your IT staff for the proxy URL, port number, and if any login information such as username/password is also needed.

The examples shown use `ALL_PROXY` but `HTTPS_PROXY` and `HTTP_PROXY` are also supported.

**Windows**

Go to System -> Advanced system settings -> Environment Variables to set these in Windows.

**macOS/Linux**

Use `export` in macOS or Linux in a terminal window to set these variables into your PATH. You may wish to copy and paste this into your `.bash_profile` for future use.

## Prediction

```python
# Default version is latest
python <simulator_file>.py --predict

# OR
Python <simulator_file>.py --predict=1
```

After your BRAIN is finished training it can be used to "predict" or perform in the simulation. How well it does depends on how long you let it train! Using your BRAIN involves running your simulator file as you did when training, but now in prediction mode with `--predict`. The default is `--predict=latest` which will use the version of the latest training session that you just ran. You can use a different version of your BRAIN if you have trained it multiple times by replacing `latest` with the version number.

## Recording Data to File

> Record file example

```python
# Excerpt of find_the_center_sim.py

if __name__ == "__main__":
    config = bonsai_ai.Config()
    
    config->set_record_enabled(true);
    config->set_record_file("find_the_center.json");

    brain = bonsai_ai.Brain(config)
    sim = BasicSimulator(brain, "find_the_center_sim")
    sim.enable_keys(["delta_t", "goal_count"], "ftc")

    print('starting...')
    last = clock() * 1000000
    while sim.run():
        now = clock() * 1000000
        sim.record_append(
            {"delta_t": now - last}, "ftc")
        last = clock() * 1000000
        continue
```

The full code for how to use this feature within an example can be found in the [find-the-center example][14].

Analytics recording can be enabled in code (shown in the code panel) or at the command line. The code example shown has the same effect as invoking this script with `--record=find_the_center.json`. Alternatively, invoking with `--record=find_the_center.csv` enables recording to CSV.

The syntax for how to use this feature within the SDK libraries see [`record_file`][12] for Python and [`record_file()`][13] for C++.

# Cloud Hosted Simulators

The Bonsai Platform can run several simulators for you in our cloud environment. This means you can start and stop training all from the web interface, without needing to download and install the Bonsai CLI or SDK. You will still need to run the simulator locally using the CLI to obtain predictions, however.

Cloud-hosted simulators are useful for prolonged training to avoid your computer becoming idle and stopping training. At this time, you can only run one simulator cloud-hosted, and cannot take advantage of training speed increases from [running simulators in parallel][5].

Current list of supported libraries for cloud-hosted training:

* OpenAI Gym environment: [`openai.gym`][1]
* Bonsai Python `bonsai-ai` library: [`bonsai.ai`][10]
* EnergyPlus Simulator: [`bonsai.energyplus`][3]
* Legacy `bonsai-python` library: [`bonsai.python`][2]

In order to set your BRAIN to use one of these cloud-hosted simulators, you’ll either need to start with a starter project and modify your BRAIN’s [project (`.bproj`) file][8] or use one of the Bonsai demo projects.

# Running Simulations in Parallel

> ![Parallel Simulators](../images/multiple-sims.png)

Training can take a long time. Depending on the complexity of the problem and the way your lesson plans are written, training can take several hours to several days. Fortunately, training can happen in parallel. Each new simulator you connect to a running training session will increase the episode per second training rate which will help the BRAIN find better solutions faster. How much you can increase training depends on the speed of your simulation: if your simulation takes 1s per iteration, it will be beneficial to run dozens in parallel.

Running simulations in parallel is currently only supported either when running your simulator locally via the Bonsai CLI, or in your own private cloud on AWS or Azure. Simply connect multiple instances of your simulation to the same BRAIN to take advantage of this feature. Eventually, the system will no longer handle any more messages per second and you will see a plateau of improvement in training.


[1]: https://quay.io/repository/bonsai/gym
[2]: https://quay.io/repository/bonsai/python
[3]: https://quay.io/repository/bonsai/energyplus
[4]: ./library-reference.html
[5]: #running-simulators-in-parallel
[6]: https://bons.ai/blog/simulators-deep-reinforcement-learning
[7]: ../examples.html
[8]: ./cli-reference.html#bproj-file
[9]: ./library-reference.html#proxy
[10]: https://quay.io/repository/bonsai/bonsai-ai
[11]: ../../examples.html#inkling-file
[12]: ./library-reference.html#record_file
[13]: ./cpp-library-reference.html#record_file
[14]: ../examples.html#simulator-file