# Logger

[[عربي]](README.ar.md)

A logging library for Alusus. It writes every log record as a single line in `logfmt` format, which is a list of
`key=value` pairs:

```
time=2026-10-05T11:40:48+00:00 level=info message="logged in" user=hisham age=22
```

Every record starts with three keys:

* `time`: when the record was written, in RFC 3339 format.
* `level`: the level of the record: `error`, `warning`, `info` or `debug`.
* `message`: the message of the record.

After them come the [global properties](#global-properties) of the logger, then the properties you add to the record,
in the order you add them.

## Adding to the Project

Import the library as follows:

```
import "Apm";
Apm.importPackage("Alusus/Logger@0.1");
```

## Usage

Create a `Logger` object, then log through its level macros: `error`, `warning`, `info` and `debug`. Each macro takes
a message and, optionally, the properties of the record:

```
def logger: Logger;
logger.info["logged in", { user: "hisham", age: 22, admin: true }];
```

```
time=2026-10-05T11:40:48+00:00 level=info message="logged in" user=hisham age=22 admin=true
```

The record is printed immediately. Records below the logger's [log level](#loglevel) are dropped.

Every `Logger` object has its own settings (level, time zone and global properties).

### Properties

The properties argument can be written in any of the following forms:

```
logger.info["msg", { a: 1, b: "text" }];   // curly brackets
logger.info["msg", (a: 1, b: "text")];     // parentheses
logger.info["msg", a: 1];                  // a single property
```

A key is either an identifier or a string literal (for example `"my key"`). Anything else (such as `1: x`) is a
compile error. The supported value types are:

* Text (`ptr[array[Char]]`).
* Integers (`Int[64]`), printed as they are.
* Floats (`Float[64]`), printed with 6 decimal places, so `0.5` is printed as `0.500000`.
* `Bool`, printed as `true` or `false`. Note that the literals `1` and `0` are matched to `Bool`, not to integer.
* `Error` objects, which add two properties: `<key>.code` and `<key>.msg`.

```
def err: SrdRef[GenericError] = GenericError.new("E100", "something went wrong");
logger.error["failed", (error: err, extra: "value")];
```

```
time=2026-10-05T11:40:48+00:00 level=error message=failed error.code=E100 error.msg="something went wrong" extra=value
```

### Global Properties

Global properties are added to every record the logger writes afterwards. Add them with `set`, either as a function
call for a single property or as a macro that takes the same forms as the record properties:

```
logger.set("service", "billing");
logger.set[port: 8080];
logger.set[{ region: "eu", replicas: 3 }];

logger.info["started"];
```

```
time=2026-10-05T11:40:48+00:00 level=info message=started service=billing port=8080 region=eu replicas=3
```

Setting the same key twice adds it twice. To remove all the global properties, call `clear`:

```
logger.clear();
```

## Logger

```
class Logger {
    def Level: { ERROR, WARNING, INFO, DEBUG }
    def useLocalTime: Bool;
    handler this.logLevel: Int;
    handler this.logLevel = Int;
    func levelName(level: Int): CharsPtr;
    handler this.set(key: CharsPtr, val: CharsPtr);
    handler this.set(key: CharsPtr, val: Int[64]);
    handler this.set(key: CharsPtr, val: Float[64]);
    handler this.set(key: CharsPtr, val: Bool);
    macro set[this, props];
    handler this.clear();
    macro error[this, msg] / error[this, msg, props];
    macro warning[this, msg] / warning[this, msg, props];
    macro info[this, msg] / info[this, msg, props];
    macro debug[this, msg] / debug[this, msg, props];
}
```

### Level

```
def Level: {
    def ERROR: 40;
    def WARNING: 30;
    def INFO: 20;
    def DEBUG: 10;
}
```

The log levels, from the most important to the least. They are used with `logLevel`.

### logLevel

```
handler this.logLevel: Int;
handler this.logLevel = Int;
```

The lowest level that is printed. Records below this level are dropped. The default is `Level.INFO`.
Assigning a value that is not one of the values of [Level](#level) is ignored and the current level stays as it is.

```
logger.logLevel = Logger.Level.WARNING;
logger.info["not printed"];
logger.warning["printed"];
```

### useLocalTime

```
def useLocalTime: Bool;
```

Chooses the time zone of the `time` key. It applies to every record written after changing it. When `true`, the time
is printed in local time with its offset. When `false` (the default), it is printed in UTC with the offset `+00:00`.

```
time=2026-10-05T11:40:48+00:00     // UTC (default)
time=2026-10-05T14:40:48+03:00     // local time
```

### levelName

```
func levelName(level: Int): CharsPtr;
```

Returns the name of the level as it is printed in the `level` key: `error`, `warning`, `info` or `debug`.

### set

```
handler this.set(key: CharsPtr, val: CharsPtr);
handler this.set(key: CharsPtr, val: Int[64]);
handler this.set(key: CharsPtr, val: Float[64]);
handler this.set(key: CharsPtr, val: Bool);
macro set[this, props];
```

Adds a [global property](#global-properties). See [Keys](#keys) and [Values](#values) for how they are printed.

### clear

```
handler this.clear();
```

Removes all the [global properties](#global-properties) added with `set`. Records written afterwards contain only their
own properties.

### error, warning, info, debug

```
macro error[this, msg];
macro error[this, msg, props];
```

(and the same for `warning`, `info` and `debug`.) Write a record with the corresponding level.

* `msg` the message of the record.
* `props` the properties of the record. See [Properties](#properties).

## Entry

```
class Entry {
    handler this~init(logger: ref[Logger], level: Int, msg: CharsPtr);
    handler this.set(key: CharsPtr, val: CharsPtr);
    handler this.set(key: CharsPtr, val: Int[64]);
    handler this.set(key: CharsPtr, val: Float[64]);
    handler this.set(key: CharsPtr, val: Bool);
    handler this.set(key: CharsPtr, val: ref[Error]);
    handler this.send();
}
```

A single log record. The level macros create one, add the properties to it and send it, so you normally don't need
this class. You can use it directly to build a record in several steps:

```
def entry: Logger.Entry(logger, Logger.Level.INFO, "manual entry");
entry.set("n", 42);
entry.send();
```

The record is printed only when `send` is called. Creating an `Entry` starts the record in the logger's buffer, so
finish (`send`) one entry before starting another one on the same logger.

## Output Format

### Keys

A key must not contain spaces, control characters, `=` or `"`. If it does, each of these characters is replaced by
`_`. An empty key is printed as `_`.

### Values

A text value is printed as it is, unless it is empty or contains a space, `=`, `"`, `\` or a control character. In
that case it is put between double quotes, and the characters that would break the one-line format are escaped, so
every record is exactly one line.

```
logger.info["quoting", { path: "/tmp/my file", empty: "", quote: "say \"hi\"" }];
```

```
time=2026-10-05T11:40:48+00:00 level=info message=quoting path="/tmp/my file" empty="" quote="say \"hi\""
```

## Examples

See the [Examples](Examples) directory for an example that covers all the supported use cases, in English and in
Arabic. `Examples/test.alusus` runs them and compares the output with `Examples/expected_result.output`, ignoring
timestamps and file paths.
