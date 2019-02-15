# Examples Overview

This page contains working examples of Inkling code in conjunction with python and C++ simulator files. All of these examples and the libraries that accompany them can be found within [BonsaiAI's SDK GitHub repo][1] and are also linked for each example.

All of the **Python**, **OpenAI Gym**, and **EnergyPlus** examples can be trained in the cloud with managed simulators. A full list of supported Docker containers for remotely managed simulators can be found in the [Project File Reference][3].

If you have any suggestions of examples you'd like to see us implement please [contact the support team][2].

### Custom Simulator Examples
* **Basic Python/C++ Simulation**: A project called *Find the Center* which walks you through how to create a simple Inkling file that connects to a basic Python or C++ simulator.
* **Stellar Cartpole**: A stand-alone version of Cartpole using the machine teaching pattern of STAR.
* **SimPy Elevator Simulation**: A basic example of how to use SimPy to simulate a scenario. This one is an elevator dropping people off on 3 floors.
* **Tic-tac-toe**: A simple implementation of pixel manipulation through the Image.Gray type for a tic-tac-toe board. Also showcases the use of an algorithm hint.

### OpenAI Gym Examples
* **Mountain Car**: A simple control problem where a car must build up enough momentum to climb a hill.
* **Cartpole**: A simple control problem where a cart must make a simple move to keep a pole balanced on top.
* **Predictor Class Cartpole**: Using the same Inkling as original Cartpole, but using the Predictor Class to obtain simulation predictions via Python script.
* **Event Driven Cartpole**: Using the same Inkling as original Cartpole, but using the Event class to drive the simulation.

### Real World Example
* **HVAC with EnergyPlus**: An example of climate control machine teaching using EnergyPlus and BCVTB for simulation. 

### Simulink Example
* **Simulink Househeat**: The Simulink version of an HVAC system taking into account the outdoor environment, the thermal characteristics of the house, and the house heating system.


[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples
[2]: https://bons.ai/contact-us#contact-page-form
[3]: ../references/cli-reference.html#bproj-file