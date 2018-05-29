# Simulator Class

>![Simulator state image](../images/simulator_state.svg)

This class is used to interface with the server while training or running predictions against
a **BRAIN**. It is an abstract base class, and to use it a developer must create a subclass.

The `Simulator` class is closely related to the **Inkling** file that is associated with
the **BRAIN**. The name used to construct `Simulator` must match the name of the simulator
in the *Inkling* file.

There are two main methods that you must override, `episode_start` and `simulate`. The diagram
demonstrates how these are called during training. Optionally, one may also override
`episode_finish`, which is called at the end of an episode.

| Property          | Description |
| ---               | ---         |
| `brain`           |  The simulators Brain object. |
| `name`            |  The simulators name. |
| `objective_name`  |  The name of the current objective for an episode. |
| `episode_reward`  |  Cumulative reward for this episode so far. |
| `episode_count`   |  Number of completed episodes since sim launch. |
| `episode_rate`    |  Episodes per second. |
| `iteration_count` |  Number of iterations for the current episode. |
| `iteration_rate`  |  Iterations per second. |


## Simulator(brain, name)

> Example Inkling:

```inkling
simulator my_simulator(Config)
    action (Action)
    state (State)
end
```

> Example code:

```cpp
class MySimulator : public Simulator {
 public:
    explicit BasicSimulator(std::shared_ptr<Brain> brain, string name)
        : Simulator(move(brain), move(name)) {
            // your simulator init code goes here.
        }

    void episode_start(const bonsai::InklingMessage& params,
        bonsai::InklingMessage& initial_state) override {
            // your simulation episode reset/init code.
        }

    void simulate(const bonsai::InklingMessage& action,
        bonsai::InklingMessage& state,
        float& reward,
        bool& terminal) override {
            // your simulation stepping code.
        }

    void episode_finish() override {
            // your post-episode code.
        }
};

...

auto config = make_shared<bonsai::Config>(argc, argv);
auto brain = make_shared<bonsai::Brain>(config);
MySimulator sim(brain, "my_simulator");

...

```

```python
class MySimulator(bonsai_ai.Simulator):
    def __init__(brain, name):
        super().__init__(brain, name)
        # your sim init code goes here.

    def episode_start(self, parameters=None):
        # your reset/init code goes here.
        return my_state

    def simulate(self, action):
        # your simulation stepping code goes here.
        return (my_state, my_reward, is_terminal)

    def episode_finish(self):
        # your post episode code goes here.
        pass

...

config = bonsai_ai.Config(sys.argv)
brain = bonsai_ai.Brain(config)
sim = MySimulator(brain, "my_simulator")

...
```

Serves as a base class for running simulations. You should create a subclass
of Simulator and implement the `episode_start` and `simulate` callbacks.

| Argument | Description |
| ---      | ---         |
| `brain`  |  A Brain object for the BRAIN you wish to train against. |
| `name`   |  The name of simulator as specified in the Inkling for the BRAIN. |

## Brain brain()
```cpp
std::cout << sim.brain() << endl;
```

```python
print(sim.brain)
```

Returns the BRAIN being used for this simulation.

## string name()
```cpp
std::cout << "Starting " << sim.name() << endl;
```

```python
print("Starting ", sim.name)
```
Returns the simulator name that was passed in when constructed.

## bool predict()
```cpp
void MySimulator::simulate(const bonsai::InklingMessage& action,
                    bonsai::InklingMessage& state, float& reward, bool& terminal) {
    if (predict() == false) {
        // calculate reward...
    }

    ...
}
```

```python
def simulate(self, action):
    if self.predict is False:
        # calculate reward...

    ...
```
Returns a value indicating whether the simulation is set up to run in predict mode or training mode.

## string objective_name()
```cpp
void MySimulator::episode_start(const bonsai::InklingMessage& params,
                                bonsai::InklingMessage& initial_state) {
    cout << objective_name() << endl;
    ...
}
```

```python
def episode_start(self, params):
    print(self.objective_name)
    ...
```
Property accessor that returns the name of the current objective from Inkling.
The objective may be updated before `episode_start` is called. When running
for prediction and during start up, objective will return an empty std::string.

## episode_start(parameters, initial_state)

> Example Inkling:

```inkling
schema Config
    UInt8 start_angle
end

schema State
    Float32 angle,
    Float32 velocity
end
```

> Example code:

```cpp
void MySimulator::episode_start(const bonsai::InklingMessage& params,
                                bonsai::InklingMessage& initial_state) {
    // training params are only passed in during training
    if (predict() == false) {
        cout << objective_name() << endl;
        angle = params.get_float32("start_angle");
    }

    initial_state.set_float32("velocity", velocity);
    initial_state.set_float32("angle",    angle);
}
```

```python
def episode_start(self, params):
    # training params are only passed in during training
    if self.predict == False:
        print(self.objective_name)
        self.angle = params.start_angle

    initial = {
        "angle": self.angle,
        "velocity": self.velocity,
    }
    return initial
```

| Argument        | Description |
| ---             | ---         |
| `parameters`    | InklingMessage of episode initialization parameters as defined in inkling. `parameters` will be populated if a training session is running. |
| `initial_state` | Output InklingMessage. The subclasser should populate this message with the initial state of the simulation. |

This callback passes in a set of initial parameters and expects an initial state in return
for the simulator. Before this callback is called, the property `objective_name` will be
updated to reflect the current objective for this episode.

This call is where a simulation should be reset for the next round.

The default implementation will throw an exception.

## simulate(action, state, reward, terminal)

> Example Inkling:

```inkling
schema Action
    Int8{0, 1} delta
end
```

> Example code:

```cpp
void MySimulator::simulate(const bonsai::InklingMessage& action,
                           bonsai::InklingMessage& state, float& reward, bool& terminal) {
    velocity = velocity - action.get_int8("delta");
    terminal = (velocity <= 0.0);

    // reward is only needed during training.
    if (self.predict() == false) {
        reward = reward_for_objective(objective_name());
    }

    state.set_float32("velocity", velocity);
    state.set_float32("angle",    angle);
}
```

```python
def simulate(self, action):
    velocity = velocity - action.delta;
    terminal = (velocity <= 0.0)

    # reward is only needed during training
    if self.predict == False:
        reward = reward_for_objective(self.objective_name)

    state = {
        "velocity": self.velocity,
        "angle": self.angle,
    }
    return (state, reward, terminal)
```

| Argument   | Description |
| ---        | ---         |
| `action`   | Input InklingMessage of action to be taken as defined in inkling. |
| `state`    | Output InklingMessage. Should be populated with the current simulator state. |
| `reward`   | Output reward value as calculated based upon the objective. |
| `terminal` | Output terminal state. Set to true if the simulator is in a terminal state. |

This callback steps the simulation forward by a single step. It passes in
the `action` to be taken, and expects the resulting `state`, `reward` for the current
`objective`, and a `terminal` flag used to signal the end of an episode. Note that an
episode may be reset prematurely by the backend during training.

For a multi-lesson curriculum, the `objective_name` will change from episode to episode.
In this case ensure that the simulator is returning the correct reward for the
different lessons.

Returning `true` for the `terminal` flag signals the start of a new episode.

The default implementation will throw an exception.

## bool run()

```cpp
MySimulator sim(brain);

if (sim.predict())
    std::cout << "Predicting against " << brain.name() << " version " << brain.version() << endl;
else
    std::cout << "Training " << brain.name() << endl;

while( sim.run() ) {
}
```

```python
sim = MySimulator(brain)

if sim.predict:
    print("Predicting against ", brain.name, " version ", brain.version)
else:
    print("Training ", brain.name)

while sim.run():
    continue
```

Main loop call for driving the simulation. Returns `false` when the
simulation has finished or halted.

The client should call this method in a `while` loop until it returns `false`.
To run for prediction, `brain()->config()->predict()` must return `true`.

## episode_finish()

```cpp
void MySimulator::episode_finish() {
    cout << 'Episode: ' << episode_count() << ' reward:' << episode_reward() << endl;
}
```

```python
def episode_finish(self):
    print('Episode:', self.episode_count,
          'reward:', self.episode_reward)
```

This callback is called at the end of each episode. You can use it to log
out statistical information, or perform post episode cleanup.

## record_file()

```cpp
my_sim.record_file() == "/path/to/foobar.json";
my_sim.set_record_file("/path/to/barfoo.json");

my_sim.record_file() == "/path/to/foobar.csv";
my_sim.set_record_file("/path/to/barfoo.csv");
```

```python
my_sim.record_file == "/path/to/foobar.json"
my_sim.record_file = "/path/to/barfoo.json"

my_sim.record_file == "/path/to/foobar.csv"
my_sim.record_file = "/path/to/foobar.csv"
```

Getter and setter for analytics recording file.

When a new record file is set, the previous file will be closed immediately. Subsequent log lines will be written to the new file.

## enable_keys(keys, prefix=None)

This function adds the given keys to the log schema for this writer.
If one is provided, the prefix will be prepended to those keys and
they will appear as such in the resulting logs.
If recording is not enabled, this method has no effect.
    
You should enable any keys you expect to see in the logs. If you
attempt to insert objects containing keys which have not been
enabled, those keys will be silently ignored.

```cpp
int main(int argc, char** argv) {
    auto config = std::make_shared<bonsai::Config>(argc, argv);
    auto brain = std::make_shared<Brain>(config);
    MySim sim(brain);
    sim.enable_keys({"foo", "bar"});
    sim.enable_keys({"baz"}, "qux");
}
```

```python
if __name__ == "__main__":
    config = Config(sys.argv)
    brain = Brain(config)
    sim = MySimulator(brain)
    sim.enable_keys(["foo", "bar"])
    sim.enable_keys(["baz"], "qux")
```

| Argument   | Description |
| ---        | ---         |
| `keys`     | A list/vector of strings to include as keys in log entries for this simulator. |
| `prefix`   | A `std::string` used as a subdomain for the given `keys`. Entries will appear as `<prefix>.<key>` for each `key` in `keys`. Defaults to empty string. |

## record_append(key, value, prefix="")

**Note:** Used in C++ only. For python, use `record_append(obj, prefix)`.

Adds the given key (prepended by `prefix`, if provided) and value to the current log entry. If the specified
key is not enabled or recording is not enabled, this method has no effect.

The following `value` types are supported:
- `int64_t`
- `size_t`
- `double`
- `std::string`
- `bool`

**Note:** The template specializations in the following snippet are included for completeness. If the `value` parameter is of one of the supported types, the correct specialization will be deduced by the compiler.

```cpp
int main(int argc, char** argv) {
    auto config = std::make_shared<bonsai::Config>(argc, argv);
    config.set_recording_enabled(true);
    config.set_record_filie("foobar.json");
    auto brain = std::make_shared<Brain>(config);
    MySim sim(brain);
    sim.enable_keys({"foo", "bar", "oof", "rab"});
    sim.enable_keys({"baz"}, "qux");

    while (sim.run()) {
        sim.record_append<int64_t>("foo", -23);
        sim.record_append<size_t>("bar", 123);
        sim.record_append<double>("oof", .023);
        sim.record_append<std::string>("rab", "foobar");
        sim.record_append<bool>("baz", true, "qux");
        sim.record_append<int64_t>("nope", 23);
    }
}
```

| Argument   | Description |
| ---        | ---         |
| `key`      | A `std::string` used as an index into the current log entry. |
| `value`    | The value to add under `<prefix>.<key>`. May be any of `int64_t`, `size_t`, `double`, `std::string`, or `bool` |
| `prefix`   | String prefix for `key`. Keys should be enabled and added with the same prefix. |

## record_append(obj, prefix=None)

**Note:** Used in Python only. For C++, use `record_append(key, value, prefix)`.

Adds the keys (prepended by `prefix`, if provied) from the given dictionary to the current log entry. If recording is not enabled, this method has no effect. If a particular subset of the keys in `obj` are not enabled, they will be ignored silently.

```python
if __name__ == "__main__":
    config = Config(sys.argv)
    brain = Brain(config)
    sim = MySimulator(brain)
    sim.enable_keys(["foo", "bar"])
    sim.enable_keys(["baz"], "qux")
    while sim.run():
        sim.record_append({
            "foo": 23,
            "bar": 5
        })
        sim.record_append({
            "baz": "zabow"
        }, "qux")
```

| Argument   | Description |
| ---        | ---         |
| `obj`      | A dictionary containing data to be added to the current log entry. |
| `prefix`   | String prefix for the keys in `obj`. |

## get_next_event()
Advance the SDK's internal state machine and return an event for processing.

This is the primary entrypoint for the "Event Pump" interface. With this,
custom run loop implementations are possible in user code. Rather than calling
`Simulator.run` in a loop, communication between simulation code and Bonsai backend
can be accomplished step by step.

```python
if __name__ == "__main__":
    config = Config(sys.argv)
    brain = Brain(config)
    sim = MySimulator(brain)
    
    while True:
        event = sim.get_next_event()
        if isinstance(event, EpisodeStartEvent):
            state = sim.episode_start(event.initial_properties)
            event.initial_state = state
        elif isinstance(event, SimulateEvent):
            state, reward, terminal = sim.simulate(event.action)
            event.state = state
            event.reward = float(reward)
            event.terminal = bool(terminal)
        elif isinstance(event, EpisodeFinishEvent):
            sim.episode_finish()
        elif isinstance(event, FinishedEvent):
            sim.close()
            break
        elif isinstance(event, UnknownEvent):
            pass
```

## close()

Close the internal websocket.

## operator<<(ostream, simulator)

Prints out a representation of Simulator that is useful for debugging.

**Note:** Used in C++ only. For python, use `print(simulator)`

| Argument  | Description |
| ---       | ---         |
| `ostream` | A std c++ stream operator. |
| `simulator`  | A bonsai::Simulator to print out. |

