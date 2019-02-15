# SimPy Elevator Simulation

> ![ASCII Elevator Output](../images/elevator_output.png)

[**Download the full source code on GitHub**][1] if you want to run this simulator locally.

In this example, [SimPy][2], a process-based discrete-event simulation framework based on standard Python, is used to simulate an elevator effectively transporting people to their desired floor. The simulated elevator gets rewarded by having people wait for the least time. This example includes a simple elevator SimPy simulator, a Python simulation, and an Inkling file.

"Processes in SimPy are defined by Python generator functions and may, for example, be used to model active components like customers, vehicles or agents. SimPy also provides various types of shared resources to model limited capacity congestion points (like servers, checkout counters and tunnels)." - SimPy docs

This simulation consumes actions (up, down, open doors) for an elevator given floor requests from randomly-arriving passengers. SimPy has a great framework for simulating time only when state changes occur. This speeds up training for systems that would otherwise be mostly waiting.

In the image to the right, the elevator logs output every 100 seconds, then shows the state of the world, and then a list of recent passengers. The world state is the floor, the number of people waiting, plus the elevator, and the number of people inside.

`1: 0| 2: 1| 3: 0| [_1_]` shows zero people on the first floor, one person waiting on the second floor, and one person in the elevator on the third floor.

For more ideas of how SimPy can simulate real world problems see the [SimPy Examples page][4].

## Inkling File

###### Types

```inkling2
type FloorState {
    Position: number<0, 1, 2>,
    Floor1: number<0, 1>,
    Floor2: number<0, 1>,
    Floor3: number<0, 1>
}
```

The `FloorState` type defines the fields returned by the simulation's `advance` method.

```inkling2
type Action {
    command: number<up = 0, open = 1, down = 2>
}
```

The `Action` type defines the possible commands accepted by the elevator. In this case, the command given to the elevator is '0' to open the door, '1' to go up a floor, and '2' to go down a floor.

```inkling2
type ElevatorConfig {
    episode_length: Number.Int8
}
```

###### Concept Graph

```inkling2
graph (input: FloorState): Action {
    concept elevator_plan(input): Action {
        curriculum {
            source elevator_simulator
        }
    }

    output elevator_plan
}
```

This concept is named `elevator_plan`, and it computes an `Action` given the current `FloorState`. In this simple example, we are training the concept to make an action (go up a floor, go down a floor, or open the doors) based on the current state of the floor.

The curriculum uses simulator `elevator_simulator` as its data source. No lesson is specified, so a single simple lesson is assumed.

###### Simulator

```inkling2
simulator elevator_simulator(action: Action): FloorState {
}
```

The simulator clause declares that a simulator named `elevator_simulator` will be connecting to the server for training. This `elevator_simulator` expects an input action of type `Action` and replies with a value of type `FloorState`.


## Simulator Excerpt

```python
# Excerpt of simulator class from the elevator_simulator.py file

class ElevatorSimulator(Simulator):

    def episode_start(self, parameters=None):
        print('called episode_start')
        self.env = env = simpy.Environment()
        floors = [simpy.Resource(env, 1) for _ in range(0, BUILDING_SIZE)]
        store = simpy.Store(env, 1)
        state = elevator.Lstate()
        person_score = []
        reqs = []
        env.process(elevator.claim(floors, reqs))
        env.process(elevator.person_generator(env, floors, person_score))
        env.process(elevator.display_process(env, person_score, state))

        # We use the single step version of elevator (elevator_one)
        # this allows the simulator to run until the elevator uses a command.
        ep = env.process(
            elevator.elevator_one(env, floors, state, store, reqs))

        self.floors = floors
        self.store = store
        self.state = state
        self.person_score = person_score
        self.reqs = reqs
        self.ep = ep

        return self._get_state()

    def simulate(self, action):
        command = action['command']
        env = self.env
        # print('[advance]', end='')
        # print('command: {}'.format(command))
        self.state.command = command

        # pass our command to a resource by way of this doit() method
        env.process(doit(self.store, command))

        env.run(until=self.ep)
        self.ep = env.process(elevator.elevator_one(
            self.env, self.floors, self.state, self.store, self.reqs))
        # print('stepped to {}'.format(env.now))

        state = self._get_state()
        done = self._get_done()
        reward = None

        # only calculate reward for training mode
        if not self.predict:
            reward = self._elevator_objective()

        return self._get_state(), reward, done

    def _get_state(self):
        """ return the current state of the simulation """

        # print('[get_state]', end='')

        # if a floor is requested, state=1
        values = [min(len(q.queue), 1) for q in self.floors]
        state = {'Floor{}'.format(ix+1): v for ix, v in enumerate(values)}
        state['Position'] = self.state.floor
        # print(state)

        return state

    def _get_done(self):
        self.done = done = self.env.now > SIM_TIME
        return done

    def _elevator_objective(self):
        # print('[objective]', end='')
        waiting = elevator.count_waiting(self.person_score)
        # print("returning score %d for %d people" % (active, len(scores)))

        # return as negative because the simulator maximizes this value.
        return -waiting
```

The full simulator file *elevator_simulator.py* and elevator simulation file *elevator.py* for this example is with the rest of the [simpy-elevator code][1] on GitHub.

This is a Python simulator which uses the elevator.py custom Python simulation using SimPy. This *elevator_simulator.py* file repeatedly runs the elevator simulation for each episode of training to get details of where people start out, how many are on the elevator, and what floor they are going to, etc. Each episode the curriculum in Inkling trains the concept by sending a new `Action` to the BRAIN based on the `FloorState` of the simulator.

The `_elevator_objective` function returns a negative waiting value because this value is going to be maximized, and we want to actually minimize the collective group of people's wait time.

For more information on the functions inside of this simulator class and how to implement them see the [Library Reference][3].

Also note that if you would like to see how this simulator can be run without Bonsai - to demonstrate how the emulator behaves with a hard-coded algorithm - you can do so by running elevator.py simply with Python itself.

[1]: https://github.com/BonsaiAI/bonsai-sdk/tree/master/samples/elevator-sim
[2]: https://simpy.readthedocs.io/en/latest/index.html
[3]: http://docs.bons.ai/references/library-reference.html
[4]: https://simpy.readthedocs.io/en/latest/examples/index.html#examples
