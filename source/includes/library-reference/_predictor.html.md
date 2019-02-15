# Predictor Class

> Example Inkling:

```inkling2
simulator MySimulator(action: Action, config: Config): State {
}
```

> Example code:

```python
# As a context manager:
config = bonsai_ai.Config(sys.argv)
brain = bonsai_ai.Brain(config)
predictor = Predictor(brain, "my_simulator")

with predictor:
    action = predictor.get_action(state)

# Without context manager:
config = bonsai_ai.Config(sys.argv)
brain = bonsai_ai.Brain(config)
predictor = Predictor(brain, "my_simulator")

action = predictor.get_action(state)
predictor.close()
```

This class is used to interface with the server to obtain predictions for a specific BRAIN and
is a subclass of Simulator.

The `Predictor` class is closely related to the Inkling file that is associated with the BRAIN.
The name used to construct `Predictor` must match the name of the simulator in the Inkling file.

| Argument | Description |
| ---      | ---         |
|`brain`   | The name of the BRAIN to connect to. |
|`name`    | The name of this simulator. Must match simulator in Inkling. |

<aside class="notice">
predict(), simulate(), and episode_start() are available methods in this class but should not be overwritten.
</aside>

## get_action(self, state)

Receives the Inkling action when sent a state.

## close(self)

Closes a websocket connection. This is recommended when `predictor()` is used outside of the context manager.
