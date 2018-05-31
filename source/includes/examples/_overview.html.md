# Examples Overview

This page contains working examples of Inkling code in conjunction with python simulator files. All of these examples and the libraries that accompany them can be found on [BonsaiAI's GitHub page][1] and are also linked within each Example.

All of the **Python**, **OpenAI Gym**, and **EnergyPlus** examples can be trained in the cloud with Bonsai managed simulators. A full list of supported Docker containers for remotely managed simulators can be found in the [Project File Reference][3].

If you have any suggestions of examples you'd like to see us implement please [contact the support team][2].

### Custom Simulator Examples
* **Basic Python/C++ Simulation**: A project called *Find the Center* which walks you through how to create a simple Inkling file that connects to a basic Python or C++ simulator.
* **SimPy Elevator Simulation**: A basic example of how to use SimPy to simulate a scenario. This one is an elevator dropping people off on 3 floors.

### OpenAI Gym Examples
* **Cartpole**: A simple control problem where a cart must make a simple move to keep a pole balanced on top.
* **Mountain Car**: A simple control problem where a car must build up enough momentum to climb a hill.

### Real World Example
* **HVAC with EnergyPlus**: An example of climate control machine teaching using EnergyPlus and BCVTB for simulation. 

### Simulink Examples
* **Simulink Cartpole**: The compiled Simulink version of a pole balancing on a cart.
* **Simulink Househeat**: The compiled Simulink version of an HVAC system taking into account the outdoor environment, the thermal characteristics of the house, and the house heating system.

### Luminance Example
* **Tic-tac-toe**: A simple implementation of pixel manipulation through the Luminance schema type for a tic-tac-toe board.

[1]: https://github.com/BonsaiAI
[2]: https://bons.ai/contact-us#contact-page-form
[3]: ../references/cli-reference.html#bproj-file