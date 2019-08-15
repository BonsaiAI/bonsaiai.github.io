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
| `episode_reward`  |  Cumulative reward for this episode so far. |
| `episode_count`   |  Number of completed episodes since sim launch. |
| `episode_rate`    |  Episodes per second. |
| `iteration_count` |  Number of iterations for the current episode. |
| `iteration_rate`  |  Iterations per second. |


## Simulator(brain, name)

> Example Inkling:

```inkling2
simulator my_simulator(action: Action, config: Config): State {
}
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

Returns the BRAIN being used for this simulation.

## string name()

```cpp
std::cout << "Starting " << sim.name() << endl;
```

Returns the simulator name that was passed in when constructed.

## bool predict()

```cpp
void MySimulator::simulate(const bonsai::InklingMessage& action,
                    bonsai::InklingMessage& state, float& reward, bool& terminal) {
    if (!predict()) {
        // calculate reward...
    }

    ...
}
```

Returns a value indicating whether the simulation is set up to run in predict mode or training mode.

## episode_start(parameters, initial_state)

> Example Inkling:

```inkling2
type Config {
    start_angle: Number.UInt8
}

type State {
    angle: Number.Float32,
    velocity: Number.Float32
}
```

> Example code:

```cpp
void MySimulator::episode_start(const bonsai::InklingMessage& params,
                                bonsai::InklingMessage& initial_state) {
    // training params are only passed in during training
    if (!predict()) {
        angle = params.get_float32("start_angle");
    }

    initial_state.set_float32("velocity", velocity);
    initial_state.set_float32("angle",    angle);
}
```

| Argument        | Description |
| ---             | ---         |
| `parameters`    | InklingMessage of episode initialization parameters as defined in inkling. `parameters` will be populated if a training session is running. |
| `initial_state` | Output InklingMessage. The subclasser should populate this message with the initial state of the simulation. |

This callback passes in a set of initial parameters and expects an initial state in return
for the simulator.

This call is where a simulation should be reset for the next round.

The default implementation will throw an exception.

## simulate(action, state, reward, terminal)

> Example Inkling:

```inkling2
type Action {
    delta: Number.Int8<0, 1>
}
```

> Example code:

```cpp
void MySimulator::simulate(const bonsai::InklingMessage& action,
                           bonsai::InklingMessage& state, float& reward, bool& terminal) {
    velocity = velocity - action.get_int8("delta");
    terminal = (velocity <= 0.0);

    // reward is only needed during training.
    if (!self.predict()) {
        reward = get_reward();
    }

    state.set_float32("velocity", velocity);
    state.set_float32("angle",    angle);
}
```

| Argument   | Description |
| ---        | ---         |
| `action`   | Input InklingMessage of action to be taken as defined in inkling. |
| `state`    | Output InklingMessage. Should be populated with the current simulator state. |
| `reward`   | Output calculated reward value. |
| `terminal` | Output terminal state. Set to true if the simulator is in a terminal state. |

This callback steps the simulation forward by a single step. It passes in
the `action` to be taken, and expects the resulting `state`, `reward` for the current
`curriculum`, and a `terminal` flag used to signal the end of an episode. Note that an
episode may be reset prematurely by the backend during training.

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

This callback is called at the end of each episode. You can use it to log
out statistical information, or perform post episode cleanup.

## record_file()

```cpp
my_sim.record_file() == "/path/to/foobar.json";
my_sim.set_record_file("/path/to/barfoo.json");

my_sim.record_file() == "/path/to/foobar.csv";
my_sim.set_record_file("/path/to/barfoo.csv");
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

| Argument   | Description |
| ---        | ---         |
| `keys`     | A list/vector of strings to include as keys in log entries for this simulator. |
| `prefix`   | A `std::string` used as a subdomain for the given `keys`. Entries will appear as `<prefix>.<key>` for each `key` in `keys`. Defaults to empty string. |

## record_append(key, value, prefix="")

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

## close()

Close the internal websocket.

## operator<<(ostream, simulator)

Prints out a representation of Simulator that is useful for debugging.

**Note:** Used in C++ only. For python, use `print(simulator)`

| Argument  | Description |
| ---       | ---         |
| `ostream` | A std c++ stream operator. |
| `simulator`  | A bonsai::Simulator to print out. |

