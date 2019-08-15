# Config Class

> Example `~/.bonsai` file

```ini
[DEFAULT]
username = admin
accesskey = None
profile = dev

[dev]
url = http://localhost:5000
username = admin
accesskey = 00000000-1111-2222-3333-000000000001
```

Manages Bonsai configuration environments.
Config files can be specified in the user home directory, `~/.bonsai`,
or in a local directory. Configuration parameters can also be parsed from
the command line.

The `profile` key can be used to switch between different profiles in
one configuration file.

## Config(profile)

```cpp
#include <bonsai.hpp>
int main(int argc, char** argv) {
    auto config = bonsai::Config();
    std::cout << config << std::endl;
}
```

Constructs a default configuration.

| Argument | Description |
| ---      | ---         |
|`profile` | Optional `.bonsai` profile name. Will use the DEFAULT profile if not specified. |

Configurations are stored in `~/.bonsai` and `./.bonsai` configuration files.
The local configuration file overrides settings in the configuration file in the user home directory.

## Config(argc, argv, profile)

```cpp
#include <bonsai.hpp>
int main(int argc, char** argv) {
    auto config = bonsai::Config(argc, argv);
    std::cout << config << std::endl;
}
```

Constructs a config by looking in the configuration files and parsing the command line arguments.
To see the list of command line arguments, pass in the `--help` flag.

| Argument  | Description |
| ---       | ---         |
| `argc`    | As passed to main(). (C++)|
| `argv`    | As passed to main(). (C++/Python)|
| `profile` | Optional `.bonsai` profile name. Will use the DEFAULT profile if not specified. |

Unrecognized arguments will be ignored.

## accesskey()

```cpp
my_config.accesskey() == "00000000-1111-2222-3333-000000000001";
my_config.set_accesskey("00000000-1111-2222-3333-000000000001");
```

Server authentication token.
Obtained from the bonsai server. You need to set it in your config.

## username()

```cpp
my_config.username() == "alice";
my_config.set_username("alice");
```

Account user name.
The account you signed up with.

## url()

```cpp
my_config.url() == "https://api.bons.ai";
my_config.set_url("https://api.bons.ai");
```

Server URL.
Address and port number of the bonsai server. Normally you should not need to change this.

## proxy()

```cpp
my_config.proxy() == "myproxy:5000";
my_config.set_proxy("myproxy:5000");
```

Proxy Server.
Address and port number of the proxy server to connect through.

## brain()

```cpp
my_config.brain() == "scarecrow";
my_config.set_brain("scarecrow");
```

BRAIN name.
Name of the BRAIN on the server.

## predict()

```cpp
my_config.predict() == true;
my_config.set_predict(true);
```

Simulator mode.
The mode in which simulators will run. True if running for prediction, false for training.

## brain_version()

```cpp
my_config.brain_version() == 0;
my_config.set_brain_version(0);
```

BRAIN version.
The version of the brain to use when running for prediction. Set to 0 to use latest version.

## record_file()

```cpp
my_config.record_file() == "foobar.json";
my_config.set_record_file("foobar.json");
```

This property defines the destination for log recording. Additionally, the format for log recording is inferred from the file extension. Currently supported options are `json` and `csv`. Missing file extension or use of an unsupported extension will result in runtime errors.

## record_enabled()

```cpp
my_config.record_enabled() == true;
my_config.set_record_enabled(true);
```

## record_format()

```cpp
my_config.record_file() == "foobar.json";
my_config.record_format() == "json";
```

**Note:** This property cannot be set directly. It reflects the file extension of the currently configured `record_file`. `json` or `csv` are valid.

## operator<<(ostream, config)

Will print out a representation of Config that is useful for debugging.

| Argument  | Description |
| ---       | ---         |
| `ostream` | A std c++ stream operator. |
| `config`  | A `bonsai::Config` object to print. |

