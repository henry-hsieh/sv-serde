# sv-serde

A robust, Verilator-compatible Serde library for SystemVerilog with support for JSON, JSON5, and MessagePack.


## Features

- **Rust-like Serde API**: Format-specific APIs for JSON/JSON5 (`from_str`, `from_reader`, `from_file`, `to_string`, `to_string_pretty`, `to_writer`, `to_writer_pretty`) and MessagePack (`from_file`, `from_reader`, `to_file`, `to_writer`, `to_array`).
- **Format-Agnostic Traits**: `serde_serialize` and `serde_deserialize` interfaces for custom types.
- **Visitor Pattern**: Powerful architectural pattern for decoupled tree traversal.
- **Robust Error Handling**: Rust-inspired `Result#(T)` and `Option#(T)` with functional APIs (`unwrap`, `_expect`, `and_then`, `some`, `none`).
- **Verilator Compatible**: Extensively tested with Verilator 5.038.
- **Safety**: Recursion depth protection (default 1024) in both serializer and parser.


## Quick Start

```systemverilog
import common_pkg::*;
import json5_pkg::*;

// Parse JSON5 from string
Result#(json_value) res;
json_value val;
Result#(string) s;
string out;

res = serde_json5::from_str("{ key: 'value', // comments! \n }");
val = res.unwrap();

// Serialize to JSON string (outputs standard JSON)
s = serde_json5::to_string(val);
out = s.unwrap();
```


## The Core Framework

### Rust-like Serde API

The core library provides format-agnostic serialization and deserialization traits:

- **Traits**: `serde_serialize`, `serde_deserialize` - implement in your classes
- **Results**: `Result#(T)` with functional APIs (`unwrap`, `_expect`, `and_then`)
- **Options**: `Option#(T)` with `some()`, `none()`, `unwrap_or`
- **Visitor**: Decouple traversal from processing via `serde_visitor`
- **Deserializer**: `serde_deserializer` for SAX-style parsing


## Supported Formats

### JSON

Standard JSON deserialization and serialization.

**Deserializing:**
```systemverilog
import common_pkg::*;
import json_pkg::*;

Result#(json_value) res;
json_value val;

res = serde_json::from_str('{"key": "value"}');
val = res.unwrap();
```

**Serializing:**
```systemverilog
import common_pkg::*;
import json_pkg::*;

Result#(string) s;
int fd;

// Compact output
s = serde_json::to_string(my_val);

// Pretty print with default indent (2 spaces)
s = serde_json::to_string_pretty(my_val);

// Custom indent
s = serde_json::to_string_pretty_indent(my_val, "    ");

// To file
fd = $fopen("output.json", "w");
serde_json::to_writer(fd, my_val);
serde_json::to_writer_pretty(fd, my_val);
$fclose(fd);
```

### JSON5

JSON5 extends JSON with:
- Comments (`//`, `/* */`)
- Trailing commas
- Unquoted keys
- Single-quoted strings
- Hex numbers

```systemverilog
import common_pkg::*;
import json5_pkg::*;

// Parse JSON5 from string
Result#(json_value) res;
json_value val;
Result#(string) s;
string out;

res = serde_json5::from_str("{ key: 'value', // comments! \n }");
val = res.unwrap();

// Serialize to JSON string (outputs standard JSON)
s = serde_json5::to_string(val);
out = s.unwrap();
```

### MessagePack

Binary serialization format for compact storage/transmission.

**Deserializing:**
```systemverilog
import common_pkg::*;
import msgpack_pkg::*;

// Parse from byte array and file
Result#(msgpack_value) res;
msgpack_value val;

res = serde_msgpack::from_array_to_value(data);
val = res.unwrap();

res = serde_msgpack::from_file("data.msgpack");
val = res.unwrap();
```

**Serializing:**
```systemverilog
import common_pkg::*;
import msgpack_pkg::*;

// To byte array
Result#(byte_array_t) bytes;
byte_array_t data_out;
string hex;
Result#(bit) res;

bytes = serde_msgpack::to_array(my_val);
data_out = bytes.unwrap();

// To string (hex)
hex = serde_msgpack::to_string(my_val).unwrap();

// To file
res = serde_msgpack::to_file("output.msgpack", my_val);
```


## Custom Serialization and Deserialization

Implement `serde_serialize` and `serde_deserialize` in your classes:

```systemverilog
import common_pkg::*;
import serde_pkg::*;

class address implements serde_serialize, serde_deserialize;
  string city;

  virtual function Result#(bit) serialize(serde_serializer ser);
    Result#(bit) res;
    res = ser.serialize_string(city);
    return res;
  endfunction

  virtual function Result#(bit) deserialize(serde_deserializer de);
    Result#(string) res;
    res = de.deserialize_string();
    if (res.is_err()) return Result#(bit)::Err(res.unwrap_err());
    city = res.unwrap();
    return Result#(bit)::Ok(1);
  endfunction
endclass
```

## Running Tests

Requires Verilator installed.

```bash
make clean
make all
```

For targeted tests:

```bash
make help                 # List available tests and other commands
make test_<pattern_name>  # Test specified pattern
```

## License

[MIT](LICENSE)
