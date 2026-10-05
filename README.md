# Logger

[[عربي]](README.ar.md)

A logging library for Alusus. It writes every log record as a single line in `logfmt` format, which is a list of
`key=value` pairs:

```
time=2026-10-05T11:40:48+00:00 level=info userName=Hisham age=22 message="logged in"
```

Every record starts with two keys:

* `time`: when the record was written, in RFC 3339 format.
* `level`: the level of the record: `error`, `warn`, `info` or `debug`.

After them come the keys you add yourself, in the order you add them.

## Adding to the Project

Import the library as follows:

```
import "Apm";
Apm.importPackage("Alusus/Logger@0.1");
```

## Usage

Start a record by calling the function of the level you want, chain the fields you want to add, then call `send` to
print it:

```
Logger.info()
    .str("user", "hisham")
    .num("age", 22)
    .bool("admin", true)
    .msg("logged in")
    .send();
```

```
time=2026-10-05T11:40:48+00:00 level=info user=hisham age=22 admin=true message="logged in"
```

The record is printed only when `send` is called, so a chain that does not end with `send` prints nothing.

## Global Functions

### error

```
func error(): Entry;
```

Starts a record with `level=error`.

### warning

```
func warning(): Entry;
```

Starts a record with `level=warn`.

### info

```
func info(): Entry;
```

Starts a record with `level=info`.

### debug

```
func debug(): Entry;
```

Starts a record with `level=debug`.

### setLevel

```
func setLevel(level: Int): Void;
```

Sets the lowest level that is printed. Records below this level are dropped, and their field calls do no work.
The default is `Level.INFO`.

* `level` one of the values of [Level](#level). Any other value is ignored and the current level stays as it is.

```
Logger.setLevel(Logger.Level.WARNING);
Logger.info().msg("not printed").send();
Logger.warning().msg("printed").send();
```

### useLocalTime

```
func useLocalTime(enabled: Bool): Void;
```

Chooses the time zone of the `time` key. The setting applies to every record started after the call.

* `enabled` when `true`, the time is printed in local time with its offset. When `false` (the default), it is
  printed in UTC with the offset `+00:00`.

```
time=2026-10-05T11:40:48+00:00     // UTC (default)
time=2026-10-05T14:40:48+03:00     // local time
```

## Level

```
module Level {
    def ERROR: 40;
    def WARNING: 30;
    def INFO: 20;
    def DEBUG: 10;
}
```

The log levels, from the most important to the least. They are used with `setLevel`.

## Entry

```
class Entry {
    handler this.str(key: ptr[array[Char]], val: ptr[array[Char]]): ref[Entry];
    handler this.num(key: ptr[array[Char]], val: Int[64]): ref[Entry];
    handler this.num(key: ptr[array[Char]], val: Float[64]): ref[Entry];
    handler this.bool(key: ptr[array[Char]], val: Bool): ref[Entry];
    handler this.msg(message: ptr[array[Char]]): ref[Entry];
    handler this.send(): SrdRef[Error];
}
```

A single log record. You get it from `error`, `warning`, `info` or `debug`, which fill in the `time` and `level`
keys. Every field function returns a reference to the same record, so calls can be chained. The record is meant to
be used within a single statement that ends with `send`.

### str

```
handler this.str(key: ptr[array[Char]], val: ptr[array[Char]]): ref[Entry];
```

Adds a text field. See [Values](#values) for when the value is put between quotes.

* `key` the name of the field. See [Keys](#keys).
* `val` the value of the field.

### num

```
handler this.num(key: ptr[array[Char]], val: Int[64]): ref[Entry];
handler this.num(key: ptr[array[Char]], val: Float[64]): ref[Entry];
```

Adds a number field. Integers are printed as they are. Floats are printed with 6 decimal places, so `0.5` is printed
as `0.500000`.

* `key` the name of the field. See [Keys](#keys).
* `val` the number.

### bool

```
handler this.bool(key: ptr[array[Char]], val: Bool): ref[Entry];
```

Adds a field whose value is `true` or `false`.

* `key` the name of the field. See [Keys](#keys).
* `val` the value.

### msg

```
handler this.msg(message: ptr[array[Char]]): ref[Entry];
```

Adds the main message of the record under the key `message`. It is the same as `str("message", message)`.

### send

```
handler this.send(): SrdRef[Error];
```

Prints the record as one line. Returns the first problem found in the keys of the record, or a null reference if
all the keys were valid. The record is printed in both cases. See [Keys](#keys).

```
def err: SrdRef[Error] = Logger.info().str("user name", "ali").send();
if not err.isNull() Logger.error.msg(err.getMessage().buf).send();
```

```
time=2026-10-05T11:40:48+00:00 level=info user_name=ali
Logger: invalid chars in key replaced by '_'
```

The error has the code `LOGGER_INVALID_KEY`. If the record was dropped by `setLevel`, its keys are not checked and
`send` returns a null reference.

## Output Format

### Keys

A key must not contain spaces, control characters, `=` or `"`. If it does, each of these characters is replaced by
`_`, and `send` returns an error. An empty key is printed as `_` and also makes `send` return an error.

### Values

A text value is printed as it is, unless it is empty or contains a space, `=`, `"`, `\` or a control character. In
that case it is put between double quotes, and the characters that would break the one-line format are escaped, so
every record is exactly one line.

```
Logger.info().str("path", "/tmp/my file").str("empty", "").str("quote", "say \"hi\"").send();
```

```
time=2026-10-05T11:40:48+00:00 level=info path="/tmp/my file" empty="" quote="say \"hi\""
```
