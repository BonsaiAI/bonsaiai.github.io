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

### enum class Type

```cpp
auto event = get_next_event();
if (event->type() == Type::Episode_Start) {
    auto es_E = dynamic_pointer_cast<EpisodeStartEvent>(event);
    auto initial_properties = es_E->initial_properties;
    auto initial_state = es_E->initial_state;
    // process initial properties/state
} else if (event->type() == Type::Simulate) {
    auto sim_E = dynamic_pointer_cast<SimulatorEvent>(event);
    auto prediction = sim_E->prediction;
    auto state = sim_E->state;
    auto reward = sim_E->reward;
    auto terminal = sim_E->terminal;
    // process simulator step 
} else if (event->type() == Type::Finished) {
    close();
}
```

Event is an abstract base class for encapsulating Simulator state. Event types correspond to different
procedures in client code as shown in the table below.

| Value          | Description |
| ----           | ----        |
| `Episode_Start`|  Reset the simulator and set initial state. |
| `Simulate`     |  Advance the simulation with the next prediction and record resulting state. |
| `Finished`     |  Simulation complete. BRAIN does not expect further state data. |
| `Unknown`      |  No event corresponding to last message exchange. |

### Type type()
Returns the Type of the Event. Implemented for each specialization of Event.

## EpisodeStartEvent Class
Signals a boundary between training episodes. Requires resetting the simulation environment
and returning its initial state to the **BRAIN**.

###### std::shared_ptr<const InklingMessage> initial_properties
Settings used when resetting the simulation environment.

###### std::shared_ptr<InklingMessage> initial_state
Directly manipulate to reflect the state resulting from sim environment reset.

See `InklingMessage` header for details.

## SimulateEvent Class
Signals that the **BRAIN** is ready to receive the simulator state resulting from
the next prediction in the queue.

###### std::shared_ptr<const InklingMessage> prediction
The prediction (action) intended for the next simulation step.

###### std::shared_ptr<InklingMessage> state
Directly manipulate to reflect the state resulting from applying `prediction`
to the simulation environment.

See `InklingMessage` header for details.

###### std::shared_ptr<float> reward
Directly manipulate to reflect the reward corresponding to `state`.

###### std::shared_ptr<bool> terminal
Directly manipulate to reflect whether the current `state` is terminal.

## FinishedEvent Class
Signals that the **BRAIN** is done training. No more simulation steps are expected.

## UnknownEvent Class
Signals that no action is required.

