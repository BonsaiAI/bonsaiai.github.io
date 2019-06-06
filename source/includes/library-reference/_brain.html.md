# Brain Class

Manages a BRAIN instance, talks with the server backend, and contains
information about the BRAIN state. In future versions will be used to upload
and download Inkling to and from the BRAIN on the server.

Requires a configuration and a BRAIN name. The BRAIN name can be set in
several places, and there is an order of what takes precedence over the other as follows:

Brain() >> --brain >> .brains >> .bonsai[profile] >> .bonsai[DEFAULT] >> env[BONSAI_TRAIN_BRAIN]

such that ">>" indicates a decreasing order of precedence. Note that failure to set
BRAIN name in at least one of these places will result in a friendly error.

## Brain(config, name)

```python
config = bonsai_ai.Config(sys.argv)
brain = bonsai_ai.Brain(config)
print(brain)
```

Creates a local object for interacting with an existing BRAIN on the server.

| Argument | Description |
| ---      | ---         |
| `config` | Object returned by previously created `bonsai_ai.Config`. |
| `name`   | BRAIN name as specified on the server. If name is empty, the BRAIN name in `config` is used instead. |

## update()

```python
brain.update()
```

Refreshes description, status, and other information with the current state of the BRAIN on the server.
Called by default when constructing a new Brain object.

## name

```python
print(brain.name)
```

Returns the name of the BRAIN as specified when it was created.

## description

```python
print(brain.description)
```

Returns the user-provided description for the BRAIN.

## version

```python
print(brain.version)
```

Returns the current version number of the BRAIN.

## latest_version

```python
print(brain.latest_version)
```

Returns latest version number of the BRAIN.

## Config config

```python
print(brain.config)
```

Returns the configuration used to locate this BRAIN.

## Status

```python
print(brain.status)
```

Returns the current status of the target BRAIN

## sample_rate

```python
print(brain.sample_rate)
```

Returns the sample rate in iterations/second for all simulators connected to the brain

## iteration_metrics

```python
print(brain.iteration_metrics)
```

Returns iteration data for a given version of a BRAIN. 

Defaults to configured version if none is given. Iterations contain data for the number of iterations that have occured in a simulation and at what timestamp. This data gets logged about once every 100 iterations. This can be useful for long episodes when other metrics may not be getting data.

| Argument | Description |
| ---      | ---         |
| `version` | Returns data about each training episode for a given version of a BRAIN. Defaults to configured version if none is given. |

## training_episode_metrics

```python
print(brain.training_episode_metrics)
```

Returns test pass data for a given version of a BRAIN. Defaults to configured version if none is given. Test pass episodes occur once every 20 training episodes during training for a given version of a BRAIN. The value is representative of the AI's performance at a regular interval of training.
 
| Argument | Description |
| ---      | ---         |
| `version` | Returns data about each training episode for a given version of a BRAIN. Defaults to configured version if none is given. |