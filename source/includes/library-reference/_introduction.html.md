# Python Library Overview

The Python library wraps the API to simplify the process of building simulators
in the Python programming language.  Compatible with Python 2.7+

When the AI Engine trains with the simulator it works in a loop. First, the
simulator connects and registers itself with the AI Engine. Then, the simulator
sends the AI Engine a state and the value of any objectives or rewards; next,
the AI Engine replies with an action. The simulator then uses this action to
advance the simulation and compute a new state. This "send state, receive
action" process is repeated until training stops.  At any time the AI engine
may stop, reconfigure, and reset the simulator.  After doing so it will either
restart this training loop or stop training.  A training loop is sometimes
referred to as an "iteration".