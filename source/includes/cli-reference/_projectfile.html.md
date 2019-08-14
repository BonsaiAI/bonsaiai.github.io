# Projects and Config Files

The Bonsai command-line interface works with three files.  A `.bonsai` file in
your user directory, a `.bproj` project file in the directory containing the Inkling 
files and simulator configuration for a BRAIN, and a `.brains` file that links
a project file to a BRAIN in the same directory as the project file.

## .bonsai file

```ini
[DEFAULT]
port = 443
username = bonsaiuser
accesskey = 55555555-5555-5555-5555-555555555555
webport = 443
webhost = beta.bons.ai
host = api.bons.ai
usessl = True
```

*Note:* the username for "bonsaiuser" is a randomized string that is populated by `bonsai -a configure`. Do not change this value, or you will need to request a new code.

## .bproj file

```json
{
    "files": [
        "mybrain.ink",
        "my_simulator.py"
    ],
    "training": {
        "command": "python my_simulator.py",
        "simulator": "bonsai.ai"
    }
}
```

Project files are created in the same directory as your Inkling files when
you download or create a BRAIN with the CLI. The project file has a name like
`bonsai_brain.bproj` and contains a JSON object that ties together the Inkling
files, simulator files, and simulator configuration needed to train a BRAIN.

### Files

`files` is a list of files included in this BRAIN. Directories
may also be in the `files` list. Specifying a directory will include every
file within that directory in the BRAIN. Only one inkling
file per BRAIN is currently supported. If `files` specifies multiple Inkling files,
only the first will be used and the remainder will be ignored.

There must be at least one valid path in the `files` list.

#### Globbing

> Examples of globbing

```json
{
    "files": [
        "*.ink",    # All inkling files in the current directory
        "*/*.py"    # All python source files from every subdirectory one level below cwd
    ],
    "training": {
        "command": "python my_simulator.py",
        "simulator": "bonsai.ai"
    }
}
```

For users familiar with the Unix command line, file path expansion in the CLI will behave as
expected. There is no ~ expansion (in a Unix shell, this expands to the home directory), but `*`,
`?`, and character ranges expressed with `[ ]` will be correctly matched. Wikipedia has detailed
information on [globbing syntax][5].

You can test these on a generic shell under OSX and try the pattern from your project directory
using the ls command, whose globbing support should mirror the Bonsai CLI's in most cases.

### Training

`training` is an object.  The `simulator` field of that object
points to a pre-configured simulation container inside the platform. The
`command` field describes the command to run to start the simulator.

Current list of supported simulators for Docker cloud-hosted training:

* OpenAI Gym environment: [`openai.gym`][1]
* Bonsai Python `bonsai-ai` library: [`bonsai.ai`][4]
* EnergyPlus Simulator: [`bonsai.energyplus`][3]
* Legacy `bonsai-python` library: [`bonsai.python`][2]


## .brains file

```json
{
    "brains": [{
        "default": true,
        "name": "mybrain"
    }]
}

```

The `.brains` file links a project to a BRAIN on the server. You can link one
project to multiple BRAINs.  The file is located in the same directory as a
project file.


* `name` is a name of one of this user's BRAINs.
* `default` marks a named BRAIN as the default BRAIN to use with command
   operations.


[1]: https://quay.io/repository/bonsai/gym
[2]: https://quay.io/repository/bonsai/python
[3]: https://quay.io/repository/bonsai/energyplus
[4]: https://quay.io/repository/bonsai/bonsai-ai
[5]: https://en.wikipedia.org/wiki/Glob_(programming)