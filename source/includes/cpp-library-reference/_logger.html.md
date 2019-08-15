# Logging Class

Runtime logging is accomplished using the `BONSAI_LOG` macro; `BONSAI_LOG` may be invoked
freely on log domains of your choosing, but the accompanying messages will only be printed if
a given domain has been enabled either through command line flags (see `bonsai::Config`) or
through `bonsai::logging::set_enabled`.

Consumers of `bonsai::logging` may enable a custom handler for `BONSAI_LOG` via
`logging::set_handler`. By default, logs print to `stderr`.

Additionally, use `logging::set_use_colors` to enable/disable ASCII colors in log output.

```cpp
using bonsai::logging;

class MySimulator : public Simulator {
...
    void simulate(const bonsai::InklingMessage& action,
                   bonsai::InklingMessage& state,
                   float& reward, bool& terminal) {
        ...
        BONSAI_LOG(foobar, "things (" << reward << ") stuff"); // logged
        BONSAI_LOG(barfoo, "things (" << reward << ") stuff"); // not logged
        ...
    }
...
};
int main() {
    ...
    logging::log().set_enabled(true)
    logging::log().set_enabled("foobar")
    ...
}
```

## BONSAI_LOG(domain, message)

`BONSAI_LOG` is a preprocessor macro defined in `logging.hpp`. It encapsulates various calls into
the `logging` singleton and performs the necessary domain checks and I/O procedures without the
overhead of an additional function call.

```cpp
void MySimulator::simulate() {
    ...
    BONSAI_LOG(foobar, "my hovercraft has " << val << "eels");
    ...
}
```

The log domain should not include quotes. It will be quoted into a string literal at preprocessor time.

Similarly, the message will be interpolated into an expression of the form `std::cerr << X << std::endl;`. As such, `message` can be any expression which produces syntactically correct C++  when substituted for X in an expression of that form.

## enabled()

Global switch to enable/disable logging.

```cpp
bonsai::logging::log().enabled() == false;
bonsai::logging::log().set_enabled(true);
```

Individual switches to enable/disable specific domains.

```cpp
bonsai::logging::log().enabled("foobar") == false;
bonsai::logging::log().set_enabled("foobar");
```

## enabled_all()

```cpp
bonsai::logging::log().enabled_all() == true;
bonsai::logging::log().set_enable_all(true);
```

Switch to enable/disable verbose logging.

## use_colors()

```cpp
bonsai::logging::log().use_colors() == true;
bonsai::logging::log().set_use_colors(false);
```

Switch to enable/disable ASCII colors in log output. Enabled by default.

## set_handler(function\<void(stringstream&)\>)

```cpp
bonsai::logging::log().set_handler([](std::stringstream& ss) {
    printf("%s", ss.str().c_str());
});
```

Set a custom handler for `BONSAI_LOG`. Instead of streaming directly to 
`std::cerr`, each log line will be passed to this custom handler in the form of a `std::stringstream`.

