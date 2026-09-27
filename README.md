# Logger

[[عربي]](README.ar.md)

A logging library for Alusus. It writes every log record as a single line in `logfmt` format, which is a list of
`key=value` pairs:

```
time=2026-09-19T11:40:48+00:00 level=info message="Server started"
```

Every record has three keys:

* `time`: when the record was written, in RFC 3339 format.
* `level`: set by the function you call: `error`, `warn`, `info` or `debug`.
* `message`: the text you passed to the function.

## Adding to the Project

Import the library as follows:

```
import "Apm";
Apm.importPackage("Alusus/Logger@0.1");
```

## Functions

Each function takes the message as text (`ptr[array[Char]]`) and writes one record. The function you call
decides the value of the `level` key. The message always goes under the `message` key.

### error

Writes a record with `level=error`.

```
function error(message: ptr[array[Char]]): Void;
```

### warning

Writes a record with `level=warn`.

```
function warning(message: ptr[array[Char]]): Void;
```

### info

Writes a record with `level=info`.

```
function info(message: ptr[array[Char]]): Void;
```

### debug

Writes a record with `level=debug`.

```
function debug(message: ptr[array[Char]]): Void;
```

## Output Format

### time

The time is in RFC 3339 format.

* **UTC (default):** the time is in UTC and is printed with the offset `+00:00`.

  ```
  time=2026-09-19T11:40:48+00:00
  ```


### message

The message is always put between double quotes under the `message` key. Characters that would break the one-line
format are escaped, so every call writes exactly one line:

