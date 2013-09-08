# Cirgonus

Cirgonus is a go-related pun on [Circonus](http://circonus.com) and is a
metrics collector for it (and anything else that can deal with json output).

Most of the built-in collectors are linux-only for now, and probably the future
unless pull requests happen. Many plugins very likely require a 3.0 or later
kernel release due to dependence on system structs and other deep voodoo.

Cirgonus does not need to be run as root to collect its metrics.

## Querying

* GET querying the root will return all metrics.
* POST querying with a `{ "Name": "metric name" }` json object will return just that metric.

e.g., if Cirgonus is running on `ubuntu.local:8000`:

```
curl http://ubuntu.local:8000 -d '{ "Name": "tha load" }'
```

Will return just the "tha load" metric:

```json
{"Type":"load_average","Value":[0,0.01,0.05]}
```

Otherwise a plain GET to the root:

```
curl http://ubuntu.local:8000
```

Will return all the metrics (Example from `test.json` configuration):

```json
{
  "cpu_usage": {
    "Type": "cpu_usage",
    "Value": [0,4]
  },
  "echo hello": {
    "Type": "command",
    "Value": {
      "hello": 1
    }
  },
  "echo hi": {
    "Type": "command",
    "Value": {
      "hi": 1
    }
  },
  "mem_usage": {
    "Type": "mem_usage",
    "Value": {
      "Free":845040,
      "Total":1011956,
      "Used":166916
    }
  },
  "tha load": {
    "Type": "load_average",
    "Value": [0, 0.01, 0.05]
  }
}
```

## Returned Metrics Format

An object of objects, keyed by the name of the metric, that have two elements:

* Type -- the type of metrics plugin.
* Value -- an arbitrary json-formatted value, dependent on the plugin (and in
  the case of `command`, the output of the command.)

## Config File

A config file is required to run Cirgonus. Check out `test.json` for an example
of how it should look.

### Attributes

* Listen: `host:port` (host optional) designation on where the agent should
  listen.
* Plugins: An array of plugin definitions. See `Plugins`.

## Plugins

Plugins all have a type, an optional name (type and name are equivalent if only
one or the other is supplied) and an optional set of parameters which depend on
the type of metric collected.

Plugin types are below.

### load\_average

Returns a three float tuple of the load averages in classic unix form. Takes no
parameters.

### mem\_usage

Note that this plugin uses the cached/buffers values to determine ram usage. It
also (currently) disregards swap.

Returns an object with these parameters:

* Free -- the amount of free ram.
* Total -- the total amount of ram in the machine.
* Used -- the amount of used ram.

### cpu\_usage

This is a two element tuple -- the first is a decimal value which indicates how
much of that is in use, and the second is the number of cpus (as measured by
linux -- so hyperthreading cores are 2 cpus). For example:

[1.5, 2] - two cores, 150% cpu usage (1 and a half cores are in use)

### command

This is the catch-all. The command plugin runs a command, and accepts json
output from it which it builds into the result.

An example from `test.json`:

```json
{
  "echo hi": {
    "Type": "command",
    "Params": [ "echo", "{\"hi\": 1}" ]
  }
}
```

This results in this MeterResult:

```json
{ "echo hi": {"Type":"command","Value":{"hi":1}} }
```

Note the value has been injected directly into the json form so that Circonus
can treat it like a proper json value.

## License

* MIT (C) Erik Hollensbe
