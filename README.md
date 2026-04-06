# NiFi Custom Processors — TitanDB Integration

A custom Apache NiFi processor that reads JSON FlowFiles and writes edges into a [TitanDB](http://titan.thinkaurelius.com/) graph database using the TinkerPop 3 Gremlin API.

## Project Structure

```
NiFi-Titan-Processor/
├── nifi-demo-processors/   # Processor implementation and unit tests
├── nifi-demo-nar/          # NiFi Archive (NAR) packaging
└── custom-nifi-templates/  # Example NiFi flow templates
```

## Processor: `PutTitan`

Reads a JSON FlowFile representing a train allocation and writes an `allocation` edge between two existing `station` vertices in TitanDB.

### Expected JSON input

```json
{
  "headcode":    "2F52",
  "time":        "13:00",
  "date":        "2015/04/13",
  "origin":      "YRKTL",
  "destination": "GLCTL",
  "unit_type":   "142/0"
}
```

The processor looks up `origin` and `destination` vertices by the configured **Vertex Property Name** (e.g. `tiploc`). If both vertices are found and the edge does not already exist, it writes the `allocation` edge with all fields as properties.

### Relationships

| Relationship | Description |
|---|---|
| `success` | FlowFile processed and edge written (or skipped as duplicate) |
| `failure` | FlowFile could not be processed (e.g. vertex not found, Titan error) |

### Processor Properties

| Property | Required | Description |
|---|---|---|
| Vertex Property Name | Yes | Graph vertex property used to look up origin/destination (e.g. `tiploc`) |
| Storage Backend | Yes* | Titan storage backend (e.g. `cassandra`, `inmemory`) |
| Storage Backend HostName | No | Hostname of the storage backend |
| Storage backend directory | No | Directory path for local storage backends |
| Index Search Backend | No | Index search backend (e.g. `elasticsearch`) |
| Index Search HostName | No | Hostname of the index search backend |

\* Required at runtime — validated in `onScheduled`.

## Building

Requires Java 8 and Maven 3.x.

```bash
cd NiFi-Titan-Processor
mvn clean install
```

The NAR file will be produced at `nifi-demo-nar/target/nifi-demo-nar-1.0.nar`.

## Deploying to NiFi

Copy the NAR file into NiFi's `lib/` directory and restart NiFi:

```bash
cp nifi-demo-nar/target/nifi-demo-nar-1.0.nar $NIFI_HOME/lib/
$NIFI_HOME/bin/nifi.sh restart
```

`PutTitan` will then be available as a processor in the NiFi UI.

## NiFi Templates

Pre-built flow templates are available in `custom-nifi-templates/`:

- **`CSV2JSON.template`** — Converts CSV to JSON
- **`CSV2JSON-extended.template`** — Extended CSV-to-JSON conversion
- **`DataFlowTemplate.template`** — Full end-to-end data flow example

Import via NiFi UI → **Templates** → **Upload Template**.

## Running Tests

```bash
cd NiFi-Titan-Processor
mvn test
```

Tests use Titan's `inmemory` storage backend so no external services are required.