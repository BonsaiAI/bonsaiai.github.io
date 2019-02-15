# API Overview

```
#######################################################
# REST API request and response will be shown here    #
# protobuf messages will be shown here for Websockets #
#######################################################
```

The Bonsai Platform provides programmers and data scientists with a new way of
building sophisticated ML and AI applications. This API is at the core of how
programmers interact with the AI Engine.

The Bonsai BRAIN API has two feature areas:

* Managing BRAINS
* Connecting simulators for training and use.

The root for all Bonsai Platform API requests is *https://api.bons.ai*.



# Authentication

API calls must include an Authorization header. The value of this header is your [Access Key][2].

#### Headers

| Header | Value |
| --- | --- |
| Authorization | Access key |



# User and BRAIN Status

### BRAIN Versions

BRAIN versions numerically count up each time a BRAIN is trained. There is also
a `latest` version that can be called if desired.

### BRAIN Modes

BRAIN versions have the following modes:

* **ready_to_train:** After Inkling is uploaded and successfully compiled, the
BRAIN version number is incremented. That particular version is in the
ready_to_train state. ready_to_train versions give predictions the same as or
worse than random.
* **training:** After the train command is given, the BRAIN version is
incremented and that version is in the training state. In this state the user
cannot load new Inkling into the BRAIN or restart training. The user must
cancel training to upload new Inkling.
* **trained:** If training is canceled or completed the BRAIN version is in
the trained state. Trained BRAIN versions can give predictions or receive more
training. version is NOT incremented upon training completion.

## User Status

Use GET to list all BRAINs owned by the user.

> Request

```text
GET /v1/{userName}
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user |

> Example JSON Response

```json
{
   "brains":  [{
      "url": "/v1/kgmcauliffe/Cartpole",
      "version": 0,
      "last_modified": "2017-04-19T00:19:46.857000Z",
      "state": "Not Started",
      "name": "Cartpole"
    },
    {
      "url": "/v1/kgmcauliffe/MountainCar",
      "version": 6,
      "last_modified": "2017-03-21T18:33:17.676000Z",
      "state": "Complete",
      "name": "MountainCar"
    },
    {
      "url": "/v1/kgmcauliffe/Starter",
      "version": 2,
      "last_modified": "2017-04-24T21:36:44.740000Z",
      "state": "In Progress",
      "name": "Starter"
    }]
}
```

### Response

| Parameter | Description |
| --- | --- |
| url | URL of the BRAIN |
| version | Current version number of the BRAIN |
| last_modified | Text string of last modified date |
| state | Current status of training |
| name | Name of the BRAIN |

## BRAIN Status

Use GET for information about a BRAIN. This will return every URL for
every version of that BRAIN as well.

> Request

```text
GET /v1/{userName}/{brainName}
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |

> Example JSON Response

```json
{
  "versions": [
    {
      "url": "/v1/kgmcauliffe/Starter/2",
      "version": 2
    },
    {
      "url": "/v1/kgmcauliffe/Starter/1",
      "version": 1
    }
  ],
  "description": "Basic example project.",
  "user": "kgmcauliffe",
  "name": "Starter"
}
```

### Response

| Parameter | Description |
| --- | --- |
| versions | Lists URL for every version of the BRAIN |
| description | Text description of the BRAIN |
| user | Name of the user who has the BRAIN|
| name | Name of the BRAIN |

## BRAIN Status Websocket

Use a websocket connection to get live updates about a BRAIN.

> Request

```text
GET /v1/{userName}/{brainName}/ws'
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |

> Example JSON Response

```json
{
    "type": "ADD_DATA_POINT",
    "metric": "episode_value",
    "value": 34.0,
    "episode": 83,
    "iteration": 120546,
    "concept": "balance",
    "lesson": "balancing"
}

{
    "type": "PROPERTY_CHANGED",
    "property": "status",
    "value": "In Progress"
}

{
    "type": "CONCEPTS_SET",
    "concepts": [{
        "name": "balance",
        "state": "Not Started",
        "algorithm": "DQN",
        "is_estimator": true
    }],
}

{
    "type": "CONCEPT_CHANGED",
    "concept": "balance",
    "state": "In Progress"
}

{
    "type": "FILE_UPDATED",
    "filename": "cartpole.ink",
    "hash": "c06edf7bc96dd461af1357a274734633b0ff2932"
}
```

### Response

The websocket will send JSON messages for events in the BRAIN. Every message
will have a `type` field which can be used to determine what the rest of the payload is.

There are 6 types of messages that are sent on this socket: `ADD_DATA_POINT`,
`PROPERTY_CHANGED`, `CONCEPTS_SET`, `CONCEPT_CHANGED`, `FILE_UPDATED`,
`FILES_UPDATED`, and `TRAINING_INITIALIZED`.

#### ADD_DATA_POINT
| Parameter | Description |
| --- | --- |
| metric | The data series for this data. This will be `episode_value` for training data points and `test_pass_value` for test data points |
| value | The reward value for an episode |
| episode | Which episode the value corresponds to |
| concept | The concept the value corresponds to |
| lesson | The lesson the value corresponds to |

#### PROPERTY_CHANGED
| Parameter | Description |
| --- | --- |
| property | The property that has changed |
| value | The new value for this parameter |

#### CONCEPTS_SET
| Parameter | Description |
| --- | --- |
| concepts | An array of JSON per concept in the BRAIN. The JSON will include the concept's state, the algorithm that the concept will be trained with and whether the concept is an estimator or classifier |

#### CONCEPT_CHANGED
| Parameter | Description |
| --- | --- |
| concept | The concept that has changed |
| state | What the concept's state changed to |

#### FILE_UPDATED
| Parameter | Description |
| --- | --- |
| filename | The file that has changed |
| hash | A hash of the file's content |

#### FILES_UPDATED
This message has no extra data.

#### TRAINING_INITIALIZED
This message has no extra data.


# BRAIN Metrics

The metrics endpoints will return data from the specified version of a BRAIN's
training. They can be useful for analyzing what happened during training.

## Episode Value

Episode value contains data about each training episode for a given version of a BRAIN.
This represents what the AI has been rewarded and what it's currently learning
through simulation.

> Request

```text
GET /v1/{userName}/{brainName}/{version}/metrics/episode_value
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| version | Version of the BRAIN |

> Example JSON Response

```json
[
  {
    "episode": 1,
    "lesson": "balancing",
    "value": 25,
    "iteration": 1354,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.712068096Z"
  },
  {
    "episode": 2,
    "lesson": "balancing",
    "value": 43,
    "iteration": 2784,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.739321856Z"
  },
  {
    "episode": 3,
    "lesson": "balancing",
    "value": 16,
    "iteration": 4184,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.769031936Z"
  }
]
```

### Response

| Parameter | Description |
| --- | --- |
| value | The episode's cumulative reward value |
| episode | The episode for this value |
| iteration | Total number of simulator iterations that have occurred |
| time | Timestamp for when this value occurred |
| concept| The concept this value corresponds to |
| lesson | The lesson this value corresponds to |

## Test Pass Value

Test pass value contains data for test pass episodes that occur once every
20 training episodes during training for a given version of a BRAIN.
This value is representative of the AI’s performance at a regular interval of training.

> Request

```text
GET /v1/{userName}/{brainName}/{version}/metrics/test_pass_value
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| version | Version of the BRAIN |

> Example JSON Response

```json
[
  {
    "episode": 1,
    "lesson": "balancing",
    "value": 25,
    "iteration": 5354,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.712068096Z"
  },
  {
    "episode": 2,
    "lesson": "balancing",
    "value": 43,
    "iteration": 10784,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.739321856Z"
  },
  {
    "episode": 3,
    "lesson": "balancing",
    "value": 16,
    "iteration": 16184,
    "concept": "balance",
    "time": "2017-11-10T06:37:11.769031936Z"
  }
]
```

### Response

| Parameter | Description |
| --- | --- |
| value | The episode's cumulative reward value |
| episode | The test pass episode for this value |
| iteration | Total number of simulator iterations that have occurred |
| time | Timestamp for when this value occurred |
| concept| The concept this value corresponds to |
| lesson | The lesson this value corresponds to |

## Iterations

Iterations contains data for the number of iterations that have occurred in a
simulation and at what timestamp. This data gets logged about once every 100 iterations.
This can be useful for long episodes when other metrics may not be getting data.

> Request

```text
GET /v1/{userName}/{brainName}/{version}/metrics/iterations
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| version | Version of the BRAIN |

> Example JSON Response

```json
[
  {
    "value": 134,
    "time": "2017-08-22T19:40:47.631161088Z"
  },
  {
    "value": 267,
    "time": "2017-08-22T19:40:48.393937152Z"
  },
  {
    "value": 374,
    "time": "2017-08-22T19:40:49.018401024Z"
  },
  {
    "value": 503,
    "time": "2017-08-22T19:40:49.508259072Z"
  }
]
```

### Response

| Parameter | Description |
| --- | --- |
| value | Total number of simulator iterations that have occurred |
| time | Timestamp for when this value occurred |


# Project Files

## Retrieve File

GET the contents of a specified project file, for example, the Inkling
code in the *.ink* file for a BRAIN.

> Request

```text
GET /v1/{userName}/{brainName}?file={fileName}
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| fileName | Full name of the file to get |

### Response

The full text of the file will be returned in the body.

## Change File

Uses the PUT request method to change a project file for the given
BRAIN. You cannot PUT new file contents while a BRAIN is training.

> Request

```text
PUT /v1/{userName}/{brainName}?file={fileName}
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| fileName | Name of the file to be changed |

#### Headers

| Header | Value |
| --- | --- |
| Content‐Type | Set to `text/x-inkling` or `text/plain` if not Inkling code|
| Content‐Length | Size of file in bytes |

> Example JSON Response

```json
{
  "success": true,
  "errors": [],
  "compiler_version": "1.8.25",
  "warnings": []
}
```

### Response

| Parameter | Description |
| --- | --- |
| success | Whether the Inkling code was compiled successfully |
| errors | Compiler errors will be listed if there are any |
| compiler_version| Version of the compiler used |
| warnings |  Compiler warnings will be listed if there are any |

This is an example response for an Inkling file with no errors or warnings to report.



# Simulators

## Simulator Information

GET information for a simulator connected to a BRAIN.

> Request

```text
GET /v1/{userName}/{brainName}/sims
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |

> Example JSON Response

```json
{
   "name": "breakout",
   "connected": 2,
   "instances": [{
       "state": "ready_to_play".
       "episode": 1
   }]
}
```

### Response

| Parameter | Description |
| --- | --- |
| name | Name of the simulator |
| connected | Count of how many simulators are connected |
| instances | Array of connected simulators with their status and episode count |

## Simulator Logs Websocket

Use the websocket connection to get real-time simulator messages during training.

> Request

```text
GET /v1/{userName}/{brainName}/{brainVersion}/sims/1/logs/ws
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| brainVersion | Version of the BRAIN |

### Response

The websocket will send one message for each log message (starting with the first
message logged in the simulator). This websocket will automatically close when
all log messages have been sent or training is complete.

## Simulator State

Use a websocket to GET the state of values in the simulator during training.

> Request

```text
GET, /v1/{userName}/{brainName}/{brainVersion}/sims/1/state/ws'
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |
| brainVersion | Version of the BRAIN |


> Example JSON Response

```json
{
   "action": { 
      "command": { 
         "value": 0 
      }
   },
   "reward": 1,
   "state": {
       "angle": {
         "value": 0.1234
       },
       "position": {
         "value": 0.02521
       }
   }
}
```

### Response

The websocket will send JSON messages for each state transition in the simulator.
The payload will have `action`, `reward`, and `state` keys. The `reward` value
will be the reward the simulator gives for this state transition. The `action`
will be JSON with keys which correspond to the Inkling's action type and the
`state` will be JSON with keys corresponding to the Inkling's state type.

# Training

## Start/Stop Training

Use PUT to start or stop training.

> Request

```text
PUT /v1/{userName}/{brainName}/train
```

| Parameter | Description |
| --- | --- |
| userName | Name of the user who has the BRAIN |
| brainName | Name of the BRAIN |

> Example JSON Response

```json
{
  "user": "kgmcauliffe",
  "simulator_connect_url": "/v1/kgmcauliffe/Starter/sims/ws",
  "name": "Starter",
  "simulator_predictions_url": "/v1/kgmcauliffe/Starter/1/predictions/ws",
  "version": 1,
  "manage_simulator": false,
  "compiler_version": "1.8.25",
  "brain_url": "/v1/kgmcauliffe/Starter/1"
}
```

### Response

| Parameter | Description |
| --- | --- |
| user | Name of the user who has the BRAIN |
| simulator_connect_url | URL the simulator connects to for training |
| name | Name of the BRAIN |
| simulator_predictions_url | URL the simulator connects to for prediction |
| version | Current version number of the BRAIN |
| manage_simulator | Whether or not the simulator is managed |
| compiler_version| Version of the compiler used |
| brain_url | URL of the BRAIN |

## Training Protocol

Connect to the server and upgrade to a websocket to initialize the training
protocol.

> Request

```text
GET /v1/{userName}/{brainName}/sims/ws
```

| Parameter | Description |
| --- | --- |
| userName | name of the user who has the BRAIN |
| brainName | name of the BRAIN |

#### Headers

| Header | Value |
| --- | --- |
| Upgrade | websocket |
| Connection | upgrade |

### Protocol

> Simulator to Server

```proto
message SimulatorToServer {
    enum MessageType {
        UNKNOWN = 0;
        REGISTER = 1;
        READY = 2;
        STATE = 3;
    }
    MessageType message_type = 1;

    // Nested message data. Which of these fields is populated depends
    // upon the message type. Some message types do not have any
    // additional data.
    RegisterData register_data = 2;
    repeated SimulationSourceData state_data = 3;
    fixed32 sim_id = 4;
}
```

> Server to Simulator

```proto
message ServerToSimulator {
    enum MessageType {
        UNKNOWN = 0;
        ACKNOWLEDGE_REGISTER = 1;
        SET_PROPERTIES = 2;
        START = 3;
        STOP = 4;
        PREDICTION = 5;
        RESET = 6;
        FINISHED = 7;
    }
    MessageType message_type = 1;

    // Nested message data. Which of these fields is populated depends
    // upon the message type. Some message types do not have any
    // additional data.
    AcknowledgeRegisterData acknowledge_register_data = 2;
    SetPropertiesData set_properties_data = 3;
    repeated PredictionData prediction_data = 4;
}
```

> Register

```proto
message RegisterData {
    string simulator_name = 1;
}
```

> Acknowledge Register

```proto
message AcknowledgeRegisterData {
    google.protobuf.DescriptorProto properties_schema = 1;
    google.protobuf.DescriptorProto output_schema = 2;
    google.protobuf.DescriptorProto prediction_schema = 3;
    fixed32 sim_id = 4;
}
```

> Set Properties

```proto
message SetPropertiesData {
    bytes dynamic_properties = 1;
    string reward_name = 2;
    google.protobuf.DescriptorProto prediction_schema = 3;
}
```

> State

```proto
message SimulationSourceData {
    bytes state = 1;
    float reward = 2;
    bool terminal = 3;
    bytes action_taken = 4;
}
```

> Prediction

```proto
message PredictionData {
    bytes dynamic_prediction = 1;
}
```


![Training Message Protocol][3]



# Prediction

## Prediction Protocol

Connect to the server and upgrade to a websocket to initialize the prediction
protocol.

> Request

```text
GET /v1/{userName}/{brainName}/{version}/predictions/ws
```

| Parameter | Description |
| --- | --- |
| userName | name of the user who has the BRAIN |
| brainName | name of the BRAIN |
| version | version of the BRAIN to get predictions from. Use "latest" for latest version |

#### Headers

| Header | Value |
| --- | --- |
| Upgrade | websocket |
| Connection | upgrade |

### Protocol

> Simulator to Server

```proto
message SimulatorToServer {
    enum MessageType {
        UNKNOWN = 0;
        REGISTER = 1;
        READY = 2;
        STATE = 3;
    }
    MessageType message_type = 1;

    // Nested message data. Which of these fields is populated depends
    // upon the message type. Some message types do not have any
    // additional data.
    RegisterData register_data = 2;
    repeated SimulationSourceData state_data = 3;
    fixed32 sim_id = 4;
}
```

> Server to Simulator

```proto
message ServerToSimulator {
    enum MessageType {
        UNKNOWN = 0;
        ACKNOWLEDGE_REGISTER = 1;
        SET_PROPERTIES = 2;
        START = 3;
        STOP = 4;
        PREDICTION = 5;
        RESET = 6;
        FINISHED = 7;
    }
    MessageType message_type = 1;

    // Nested message data. Which of these fields is populated depends
    // upon the message type. Some message types do not have any
    // additional data.
    AcknowledgeRegisterData acknowledge_register_data = 2;
    SetPropertiesData set_properties_data = 3;
    repeated PredictionData prediction_data = 4;
}
```

> Register

```proto
message RegisterData {
    string simulator_name = 1;
}
```

> Acknowledge Register

```proto
message AcknowledgeRegisterData {
    google.protobuf.DescriptorProto properties_schema = 1;
    google.protobuf.DescriptorProto output_schema = 2;
    google.protobuf.DescriptorProto prediction_schema = 3;
    fixed32 sim_id = 4;
}
```

> State

```proto
message SimulationSourceData {
    bytes state = 1;
    float reward = 2;
    bool terminal = 3;
    bytes action_taken = 4;
}
```

> Prediction

```proto
message PredictionData {
    bytes dynamic_prediction = 1;
}
```

![Prediction Message Protocol][4]


[1]: https://api.bons.ai
[2]: https://beta.bons.ai/accounts/key
[3]: ../images/training.svg
[4]: ../images/prediction.svg
