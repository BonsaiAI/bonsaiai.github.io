# Simulink Examples

**Simulink**, developed by The MathWorks, is a graphical programming environment for modeling, simulating and analyzing multi-domain dynamical systems.

Bonsai is using **Simulink** as a training environment for a Bonsai BRAIN. We’re supporting a wide range of use cases that can be described as control or machine tuning problems. Please review our simulator and sim model requirements guide at INSERT URL.

## Prerequisites

Installation covered within the specific example.

- Python3
- Bonsai CLI and SDK
- Asynchronous HTTP Client/Server
- MATLAB Engine API for Python
- MATLAB & Simulink (R2017b)

[**Download the full source code on GitHub**](https://github.com/BonsaiAI/bonsai-simulink) or clone into your project directory.

```shell
git clone https://github.com/BonsaiAI/bonsai-simulink
```

## Simulink Cartpole

This is a step-by-step guide for using **Bonsai's Universal Coordinator** in Python to connect the Bonsai Platform to a Simulink model.

You have to have Matlab and Simulink installed. Trial versions can be downloaded from [TheMathworks](http://www.themathworks.com)

#### CLI guide

We're using the Bonsai Command Line Interface (CLI) for this example. Refer to the [**Bonsai CLI guide**](http://docs.bons.ai/guides/cli-install-guide.html) for more details.

#### Install Pre-Requisites

```
pip install aiohttp
pip install bonsai-cli bonsai-ai bonsai-gym
```
Install Asynchronous HTTP Client/Server and the Bonsai CLI and Bonsai libraries needed to run our examples.

#### Install Matlab Dependencies:

```
cd <matlabroot>/extern/engines/python
python setup.py install
```
Install the Matlab API for Python from the root directory of your Matlab installation.

**Matlab Simscape Multibody Add-On** is required for this example.

Go to your Add-On Explorer in Matlab and add Simscape and Simscape Multibody.


#### Running and Training

```
bonsai create simulink-cartpole
bonsai push
bonsai train start



../../coordinator/coordinator --brain=simulink-cartpole
```
Run the following in the simulink-househeat directory:


The `bonsai push` command will upload all needed files to the Bonsai backend. This includes an [**Inkling**](http://docs.bons.ai/references/inkling-reference.html) file which describes state, actions and a learning curriculum that the the Bonsai AI is using for training.

###### Inkling    

Inkling is a declarative, strongly typed programming language specifically designed for artificial intelligence (AI). It abstracts away the vast world of dynamic AI algorithms that require expertise in machine learning and enables more developers to program AI.
Please review our [**Inkling Guide**](http://docs.bons.ai/guides/inkling-guide.html#overview)

###### Schema

```inkling
schema CartpoleState
    Float32 theta,
    Float32 dtheta,
    Float32 x,
    Float32 dx
end
```
The `CartpoleState` schema names four records — `theta`, `dtheta`,`x,` and `dx` — and assigns a type to them.

```inkling
schema CartpoleAction
    Float32 {-10.0:10.0} f
end
```

The `CartpoleAction` schema names a single record — `action` — and assigns a constrained type to it.

```inkling
schema CartpoleConfig
    Int8 dummy
end
```

The `CartpoleConfig` is set to `dummy` as we're not using config in this model.

###### Concept

```inkling
concept balance is estimator
   predicts (CartpoleAction)
   follows input(CartpoleState)
   feeds output
end
```
The concept is named `balance`, and it takes input from the simulator about the state of the model (`Cartpolestate` schema). It outputs to the `CartpoleAction` schema. This is the AI's next control signal to the cartpole.

###### Simulator

```inkling
simulator simulink_sim(CartpoleConfig)
    action (CartpoleAction)
    state (CartpoleState)
end
```

The `simulink_sim` gets information from two schemas. The first schema, `action`, specifies the schema for actions within the simulation. The second schema contains the `state` of the simulator that is sent to the lesson.

###### Curriculum

```inkling
curriculum my_curriculum
    train balance
    with simulator simulink_sim
    objective cartpole_balance
        lesson my_first_lesson
            configure
                constrain dummy with Int8{-1}
            until
                maximize cartpole_balance
end
```

The curriculum is named `my_curriculum`, and it trains the `balance` concept using the `simulink_sim`. This curriculum contains one lesson, called `my_first_lesson`. It configures the simulation, by setting a constraint for the state of the simulator.

The lesson trains until the AI has maximized the objective named `cartpole_balance`.

###### Predicting with a BRAIN

```
bonsai train stop
```
When you are seeing rewards reaching 1000 you can stop training. You may need to Ctrl+C to stop in the terminal.


```
../../coordinator/coordinator --predict
```
Now you can predict using the trained BRAIN.


## Simulink Househeat

This is a step-by-step guide for using Bonsai's Universal Coordinator in Python to connect the Bonsai Platform to a Simulink model.

You have to have Matlab and Simulink installed. Trial versions can be downloaded from [TheMathworks](http://www.themathworks.com)

This example shows how to use Simulink to create the thermal model of a house. This system models the outdoor environment, the thermal characteristics of the house, and the house heating system. Objective for the Bonsai AI is to reach the desired temperature.

#### CLI Guide

We're using the Bonsai Command Line Interface (CLI) for this example. Refer to the [**Bonsai CLI guide**](http://docs.bons.ai/guides/cli-install-guide.html) for more details.

#### Pre-requisites to run the Example

```
pip install aiohttp
pip install bonsai-cli bonsai-ai bonsai-gym
```

Install Asynchronous HTTP Client/Server and the Bonsai CLI and Bonsai libraries needed to run our examples.

#### Install Matlab Engine:

```
cd <matlabroot>/extern/engines/python
python setup.py install
```
Install the Matlab API for Python from the root directory of your Matlab installation.

#### Running and Training

```
bonsai create simulink-househeat
bonsai push
bonsai train start

../../coordinator/coordinator --brain=simulink-househeat
```

Run the following in the simulink-househeat directory:

The `bonsai push` command will upload all needed files to the Bonsai backend. This includes an [**Inkling**](http://docs.bons.ai/references/inkling-reference.html) file which describes state, actions and a learning curriculum that the the Bonsai AI is using for training.

###### Inkling

Inkling is a declarative, strongly typed programming language specifically designed for artificial intelligence (AI). It abstracts away the vast world of dynamic AI algorithms that require expertise in machine learning and enables more developers to program AI.
Please review our [**Inkling Guide**](http://docs.bons.ai/guides/inkling-guide.html#overview)

###### Schema

```inkling
schema HouseheatState
    Float32 set_temp,
    Float32 room_temp,
    Float32 room_temp_change,
    Float32 heat_cost,
    Float32 outside_temp,
    Float32 outside_temp_change
end
```

The `HouseheatState` schema names six records — `set_temp`.`room_temp`,`room_temp_change`,`heat_cost`,`outside_temp`, and `outside_temp_change` — and assigns a type to them.

```inkling
schema HouseheatAction
    Float32{ 0.0:1:1.0 } heater_on
end
```

The `HouseheatAction` schema names a single record — `heater_on` — and assigns a constrained type to it.

```inkling
schema HouseheatConfig
    Float32 outside_phase
end
```

The `HouseheatConfig` schema names one record — `outside_phase` - and assigns a type to it.

###### Concept

```inkling
concept thermostat is classifier
   predicts (HouseheatAction)
   follows input(HouseheatState)
   feeds output
end
```
The concept is named `thermostat`, and it takes input from the simulation model about the state of the temperature in the house (`HouseheatState` schema). It outputs to the `HouseheatAction` schema. This is the AI's next move in the game.

###### Simulator

```inkling
simulator simulink_sim(HouseheatConfig)
    action (HouseheatAction)
    state (HouseheatState)
end
```

The `simulink_sim` gets information from two schemas. The first schema, `action`, specifies the schema for actions within the simulation. The second schema, `state` contains the state of the simulation that is sent to the lesson.

###### Curriculum

```inkling
curriculum my_curriculum
    train thermostat
    with simulator simulink_sim
    objective match_set_temp
        lesson my_first_lesson
            configure
	        constrain outside_phase with Float32{0.0:48.0}
            until
                maximize match_set_temp
end
```

The curriculum is named `my_curriculum`, and it trains the `thermostat` concept using the `simulink_sim`. This curriculum contains one lesson, called `my_first_lesson`. It configures the simulation, by setting constraints for the state of the simulator.

The lesson trains until the AI has maximized the objective named `match_set_temp`.

###### Predicting with a BRAIN


```
bonsai train stop
```
When you are seeing rewards reaching >100 you can stop training. You may need to Ctrl+C to stop in the terminal.

```
../../coordinator/coordinator --predict
```
Now you can predict using the trained BRAIN.


## Using Simulink Coder for Parallel Simulations

Simulink Coder provides a mechanism to compile many Simulink models as a fast running C executable file for your operating system. There are three main benefits to training Bonsai BRAINs with these executables.
First is that Simulink Coder executables return simulations results much faster than raw Simulink files. When training a BRAIN using Deep Reinforcement Learning, this difference in speed can add up. For example, if you train a BRAIN for 1M iterations with a Simulink Coder executable that runs in 0.5s per iteration instead of 1.5s per iteration, your total training time will decrease by 277 hours!
Second, Simulink Coder executables are much easier to connect to the Bonsai platform in parallel because they require a lot less memory and CPU than instances of interactive Simulink models. Continuing the example above, if you run 100 copies your new coder executable in parallel training you've now reduced the training time by an additional 250 hours.
Finally, Simulink Coder executable models can be easily shared with people and teams and used to train Bonsai BRAINS without a Matlab or Simulink license.

Visit The Mathworks for more information on [**Matlab and Simulink Coder**](https://www.mathworks.com/products/simulink-coder.html)

## How to connect your own model

Please review the HOWTO file for additional information on how to connect your own Simulink model to the Bonsai AI platform.
