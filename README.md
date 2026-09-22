# Physics Simulator

Physics Simulator is a Java desktop application for modelling the motion of
bodies under configurable force laws. It provides both a Swing graphical user
interface (GUI) for interactive experiments and a command-line batch mode that
reads scenarios from JSON and writes simulation states as JSON.

The simulator was developed by [Sergio Martínez Olivera](https://github.com/sergiio8)
and [Daniel Roldán Serrano](https://github.com/danirold).

## Features

- Time-stepped simulation of moving and stationary bodies.
- Bodies organised into independent simulation groups.
- Three bundled force laws:
  - `nlug`: Newton's law of universal gravitation.
  - `mtfp`: movement towards a fixed point.
  - `nf`: no force.
- JSON input and output for reproducible scenarios and batch runs.
- Swing GUI with controls for loading scenarios, selecting force laws, running
  and stopping simulations, and viewing groups and bodies.
- Factory-based creation of bodies and force laws, with observer-based updates
  for the GUI.
- Example inputs and expected outputs under
  `EntregaSergio/resources/examples`.

## Requirements

- Java Development Kit (JDK) 8 or newer.
- A shell with `unzip`.

The source distribution includes its runtime dependencies in
`EntregaSergio/lib/`:

- `json.jar` for JSON parsing and serialisation.
- `commons-cli-1.4.jar` for command-line argument parsing.

No Maven or Gradle project is currently included; the commands below compile
the source directly with `javac`.

## Quick start

The repository stores the application source and resources in
`EntregaPhysicsSimulator.zip`. Extract it before compiling:

```sh
unzip -q EntregaPhysicsSimulator.zip
cd EntregaSergio
mkdir -p out
javac -cp 'lib/json.jar:lib/commons-cli-1.4.jar' \
  -d out \
  $(find src -name '*.java' | sort)
```

The compiled application prints its available options with:

```sh
java -cp 'out:lib/json.jar:lib/commons-cli-1.4.jar' \
  simulator.launcher.Main --help
```

## Run the GUI

Launch the interactive application with one of the bundled scenarios:

```sh
java -cp 'out:lib/json.jar:lib/commons-cli-1.4.jar' \
  simulator.launcher.Main \
  --mode gui \
  --input resources/examples/input/ex1.json \
  --delta-time 1000
```

The GUI can also be started without `--input`, then a scenario can be loaded
from the application.

## Run a batch simulation

Batch mode writes the generated states to the file supplied with `--output`:

```sh
java -cp 'out:lib/json.jar:lib/commons-cli-1.4.jar' \
  simulator.launcher.Main \
  --mode batch \
  --input resources/examples/input/ex1.json \
  --output /tmp/physics-simulator-output.json \
  --steps 1000 \
  --delta-time 1000 \
  --force-laws nlug
```

The output is a JSON document containing the initial state and the state after
each simulation step. The repository includes reference outputs for the
bundled examples in
`resources/examples/expected_output/README.md`.

The short option equivalents are also supported:

| Long option | Short option | Purpose |
| --- | --- | --- |
| `--input` | `-i` | JSON scenario to load |
| `--output` | `-o` | Batch output file |
| `--steps` | `-s` | Number of simulation steps |
| `--delta-time` | `-dt` | Time represented by each step |
| `--force-laws` | `-fl` | Force-law tag |
| `--mode` | `-m` | `gui` or `batch` |
| `--help` | `-h` | Print command-line help |

Force-law tags may include a JSON data object where the selected law supports
additional parameters. The command-line help reports the supported tags and
their descriptions.

## JSON scenario format

A scenario contains group identifiers and body definitions. The optional
`laws` array assigns a force law to a group:

```json
{
  "groups": ["g1"],
  "laws": [
    {
      "id": "g1",
      "laws": { "type": "nlug", "data": {} }
    }
  ],
  "bodies": [
    {
      "type": "mv_body",
      "data": {
        "id": "body-1",
        "gid": "g1",
        "p": [0.0, 0.0],
        "v": [0.0, 1.0],
        "m": 1.0
      }
    }
  ]
}
```

Use the files in `resources/examples/input` as complete, working examples.

## Project layout

```text
EntregaSergio/
├── lib/          # Bundled JSON and Commons CLI JARs
├── resources/    # Icons, viewer assets, examples, and reference output
├── src/
│   ├── extra/    # JSON usage example
│   └── simulator/
│       ├── control/     # Input loading and simulation orchestration
│       ├── factories/  # Body and force-law builders/factories
│       ├── launcher/   # CLI argument parsing and application startup
│       ├── misc/       # Shared utilities such as Vector2D
│       ├── model/      # Bodies, groups, force laws, and simulation engine
│       └── view/       # Swing windows, tables, dialogs, and viewer
└── tests/         # JUnit test sources for model and factory behaviour
```

## Testing

The archive contains JUnit test sources under `tests/`. The repository does
not bundle a JUnit runner or a build configuration, so running those tests
requires adding a compatible JUnit 4/JUnit 5 test setup separately. The
compilation command above verifies the application source and is the supported
smoke check documented here.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
