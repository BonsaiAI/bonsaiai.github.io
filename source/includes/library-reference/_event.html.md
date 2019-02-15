# Event Class

Internally, the Bonsai library uses a state machine to drive activity in user
code (i.e. advancing and recording simulator state, resetting a simulator, etc).
State transfer is driven primarily by a websocket-based messaging protocol
shared between the library and the Bonsai AI platform. For your convenience,
the details of this protocol have been hidden behind a pair of API's, one based
on callbacks in `bonsai_ai.Simulator` and the other event driven.

Filling out the callbacks in `bonsai_ai.Simulator` and relying on
`Simulator.run` to invoke them at the appropriate time will be sufficient for
many use cases. For example, if you have a simulator which can be advanced,
reset, and observed in a synchronous manner from Python or C++ code, your
application is likely amenable to our callback API. However, if, for example,
your simulator is free running and communicates with your application code
asynchronously, your application will likely need to employ the event driven
API described below.

In the **event-driven mode of operation**, you application code should implement
its own run loop by requesting successive events from the Bonsai library and
handling them in a way that is appropriate to your particular simulation or
deployment architecture. For example, if your simulator invokes callbacks into
your code and is reset in response to some outgoing signal (i.e. not via a
method call), you might respond to an `EpisodeStartEvent` by setting the
appropriate flag, returning control to the simulator, and returning the
resulting state to the Bonsai platform the next time your callback gets invoked.

## EpisodeStartEvent

```python
event = sim.get_next_event()
if isinstance(event, EpisodeStartEvent):
    state = sim.episode_start(event.initial_properties)
    event.initial_state = state
```

This event is generated at the start of a training episode.
It is triggered either by a terminal condition in the simulator
or by the platform itself.

| Attribute | Description |
| ---      | ---         |
| `initial_properties`  |  Configuration properties for the simulator. |
| `initial_state`   |  Assign the state resulting from a model reset. |

`initial_properties` is the configuration sent down from the server
in the form of a dictionary matching the config specified in Inkling.

## SimulateEvent

```python
event = sim.get_next_event()
if isinstance(event, SimulateEvent):
    state, reward, terminal = sim.simulate(event.action)
    event.state = state
    event.reward = float(reward)
    event.terminal = bool(terminal)
```

This event is generated when an action (prediction) is ready
to be fed into the simulator.

Assign values to a `SimulateEvent` by assigning a dictionary whose fields match
the state type. You will need to set the values before the next
`Simulator.get_next_event` call. The values get sent back to the brain during
the call to Simulator.get_next_event.

| Attribute | Description |
| ---      | ---         |
| `action`  |  Next action (prediction) in the queue. |
| `state`   |  Assign the resulting state after updating the model. |
| `reward`   |  The reward calculated from the updated. |
| `terminal`   |  Whether the updated state is terminal. |

## FinishedEvent

```python
event = get_next_event()
if isinstance(event, FinishedEvent):
    self.close()
```

Indicates that the Bonsai Platform has terminated training.

## UnknownEvent

Catch-all event for other internal states. This event can be safely ignored,
but it is provided for completeness and is handy for explicitly tracking state
transitions from client code.


