# EnergyPlus HVAC Optimization

> ![EnergyPlus Graph](../images/energyplus-graph.png)

[**Download the full source code on GitHub**][1] if you want to run this simulator locally. If you want to run EnergyPlus remotely on the Bonsai Platform as a managed simulator, create a new BRAIN selecting the EnergyPlus demo on [beta.bons.ai][4].

In this example, we'll walk you through the statements that are part of a sample implementation of [EnergyPlus][2] on the Bonsai Platform. This is a real-world example of how to use the Bonsai Platform for HVAC control using BCVTB and EnergyPlus.

While this BRAIN is training, the Bonsai AI Engine invokes the EnergyPlus simulator for every episode. The *energyplus_simulator.py* then drives the simulator forward a step at a time until it finishes the episode and then resets it for the next episode, driving the actions into it and sending states back to the Bonsai AI Engine.

## Inkling File

###### Types

```inkling2
const minIrradiation: Number.Int32 = 0
const maxIrradiation: Number.Int32 = 10
type SimState {
    SolarIrradiation: Number.Int32<minIrradiation .. maxIrradiation>
}
```

The `SimState` type describes the value returned from the Python simulation's `simulate` method to the BRAIN. We have added constants to demonstrate how they can be used in defining types.

```inkling2
type SimAction {
    shade: number<Up = 0, Down = 1>
}
```

The `SimAction` type defines the control signals this AI can send to the climate control. In this case, the shade can be either up or down.

###### Concept Graph

```inkling2
graph (input: SimState): SimAction {

    concept my_concept(input): SimAction {
        curriculum {
            source energyplus_simulator
        }
    }

    output my_concept
}
```

This concept is named `my_concept` which outputs a `SimAction` given a `SimState`. In this simple demo, we ask the Bonsai Platform to generate any model that can learn to control the system using these inputs and outputs.

The curriculum trains `my_concept` using `energyplus_simulator`. This curriculum has no defined lesson, so a default lesson is assumed.

###### Simulator

```inkling2
simulator energyplus_simulator(action: SimAction): SimState {
}
```

This statement declares that a simulator named `energyplus_simulator` will be connecting to the server for training. This code snippet binds the previous types to this simulator. To define the training relationship between the simulator and the concept, we must begin by defining the simulator. `energyplus_simulator` expects an action of type `SimAction` as an input, and it replies with a state of type `SimState` as output.

## Simulator Excerpt

```python
# Excerpt of simulator class from the energyplus_simulator.py file

class EnergyPlusSimulator(Simulator):
    """
    Runs the Actuator model for training or prediction by launching it
    against the Ptolemy server above. This uses the Bonsai Simulator
    base class to interface with the BRAIN server.
    """
    model = ePlus85Actuator()
    server = None

    clientState = {'SolarIrradiation': 0}
    shade = 0.
    is_terminal = True

    def episode_start(self, parameters):
        """
        Callback called when an training episode starts. We use this
        to reset the Ptolemy server and start a new simulation session.
        Returns the initial state.
        """
        self.restartPtolemyServer()
        return self.clientState

    def simulate(self, action):
        """
        Callback called  when stepping the simulation. It sends actions to the
        model and returns the state of the simulation to the BRAIN.
        """

        # take the action
        self.shade = action['shade'] * 6.

        if self.is_terminal:
            self.restartPtolemyServer()
        else:
            self.server.writeToClient([self.shade])
            self.readFromPtolemyClient()

        # you like graphs? WE HAVE GRAPHS. SO MANY GRAPHS.
        if self.is_terminal:
            graph = self.model.grapher()
            write_graph(graph)

            # clear old data
            self.model.data = ([], [], [], [], [])

        reward = self.reward_function()
        return self.clientState, float(reward), self.is_terminal

    def finished(self):
        """
        Called at the end of the simulation to output the graph.
        """
        graph = self.model.grapher()
        py.plot(graph, filename="graph.html")

    def readFromPtolemyClient(self):
        """
        Utility method used to do the read portion of the exchange
        with the Ptolemy server and client.
        """
        self.server.readFromClient()
        if self.model.fromClient and len(self.model.fromClient) == 4:
            self.clientState = {
                'SolarIrradiation': int(self.model.fromClient[2]/100)
                }

            # save the client input in our graph
            for n in range(len(self.model.fromClient)):
                value = self.model.fromClient[n]
                # scale some of the values for readability
                if n == 2:
                    value /= 100.
                self.model.data[n].append(value)

        self.is_terminal = self.model.exitFlag != 0

    def restartPtolemyServer(self):
        """
        Used to restart the server and setup the initial state.
        """

        # set some default values for get_state
        self.is_terminal = True
        self.clientState = {'SolarIrradiation': 0}

        # close the old connections if they're still open
        if self.server:
            self.server.close()

        # star a new episode
        print("EnergyPlusSimulator: starting PtolemyServer")
        self.server = PtolemyServer(self.model)

        try:
            self.server.start()
            self.server.waitForClient()
            # get initial state
            self.readFromPtolemyClient()

        except OSError as msg:
            print("EnergyPlusSimulator: error on restart:", msg)
            self.server = None

    def reward_function(self):
        """
        Calculates the reward for the current state of the simulation
        """
        print("EnergyPlusSimulator: reward_function")

        # largest reward is best reward (maximize)
        reward = 0
        if self.model.fromClient and len(self.model.fromClient) == 4:
            # SolarIrradiation === Shades down === good
            SolarIrradiation = self.model.fromClient[2] / 100.

            # sun is down
            if SolarIrradiation <= 1:
                if self.shade > 0:
                    reward = -1  # shades on
                else:
                    reward = 1  # shade off

            # sun is out
            else:
                if self.shade > 0:
                    reward = 1  # shades on
                else:
                    reward = -1  # shades off

            self.model.data[4].append(reward)

        print("EnergyPlusSimulator reward:", reward)
        return reward
```

The full simulator file *energyplus_simulator.py* for this example is with the rest of the [energyplus-sample code][1] on GitHub.

This is a Python simulator for integrating the EnergyPlus simulator into the Bonsai AI Engine. This *energyplus_simulator.py* file repeatedly runs the EnergyPlus simulator in the background with new actions sent from the Bonsai AI Engine by passing the state from EnergyPlus to the backend, and the action from the backend back to EnergyPlus.

For more information on the functions inside of this simulator class and how to implement them see the [Library Reference][3].

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/energyplus-sample
[2]: https://energyplus.net/
[3]: http://docs.bons.ai/references/library-reference.html
[4]: https://beta.bons.ai/new
