# PhysicsSimulator

PhysicsSimulator is a Java desktop application for experimenting with time-stepped
two-dimensional body simulations. It provides a Swing graphical user interface
(GUI) for loading scenarios, choosing force laws, running and stopping a
simulation, inspecting groups and bodies, and opening a trajectory viewer. The
same engine can be run non-interactively in batch mode, reading and writing JSON.

The project is an educational/coursework snapshot rather than a packaged
release. The repository currently distributes the implementation as
`EntregaPhysicsSimulator.zip`; the archive contains the Java source, tests,
resources, and the dependency JARs.

## Features

- Java Swing interface with file loading, run/stop controls, simulation-step and
  delta-time inputs, force-law selection, tables, and a viewer window.
- Batch execution that emits a JSON object containing the simulator state at
  each step.
- Two body types: moving bodies, with position, velocity, and mass, and
  stationary bodies, with position and mass.
- Three force-law implementations:
  - `nlug` — Newton's law of universal gravitation; optionally accepts `G`.
  - `mtfp` — acceleration towards a fixed point; optionally accepts `c` and
    `g`.
  - `nf` — no force.
- A model/controller/view split with factory-based construction of bodies and
  force laws from JSON.

## Repository layout

After extracting the archive, the project is rooted at `EntregaSergio/`:

```text
EntregaSergio/
├── lib/                 # Bundled json.jar and commons-cli-1.4.jar
├── resources/
│   ├── examples/        # Input scenarios and expected batch outputs
│   ├── icons/           # Swing toolbar icons
│   └── viewer/          # HTML viewer resource
├── src/
│   ├── simulator/
│   │   ├── control/     # Input/output coordination
│   │   ├── factories/   # JSON-backed object factories
│   │   ├── launcher/    # Command-line entry point
│   │   ├── model/       # Bodies, force laws, and simulation state
│   │   └── view/        # Swing UI and viewer
│   └── extra/json/      # JSON library usage example
└── tests/               # Model and factory tests
```

## Requirements and setup

You need a JDK with `javac` and `java` available on your `PATH`. No Maven or
Gradle build is included; the two runtime dependencies are already bundled in
`lib/`.

Extract the archive and compile from the extracted project directory:

```sh
unzip EntregaPhysicsSimulator.zip
cd EntregaSergio
mkdir -p bin
javac -cp "lib/*" -d bin $(find src -name '*.java')
```

On Windows, replace the classpath separator `:` with `;` where applicable.

## Running the application

### Swing GUI

Launch the GUI with the default settings:

```sh
java -cp "bin:lib/*" simulator.launcher.Main
```

The toolbar can load an input JSON file, select force laws for groups, open the
viewer, run or stop the simulation, set the number of steps, change the
delta-time, and exit the application. An input file can also be supplied when
launching the GUI:

```sh
java -cp "bin:lib/*" simulator.launcher.Main \
  --mode gui \
  --input resources/examples/input/ex1.json
```

### Batch mode

Batch mode requires an input file and writes the generated states to the output
file:

```sh
java -cp "bin:lib/*" simulator.launcher.Main \
  --mode batch \
  --input resources/examples/input/ex1.json \
  --output resources/Output.json \
  --steps 150 \
  --delta-time 2500 \
  --force-laws nlug
```

Short options are also available: `-m`, `-i`, `-o`, `-s`, `-dt`, and `-fl`.
Use `--help` to print the complete option list. Defaults are 150 steps,
`2500` seconds per step, and `nlug` when the corresponding options are omitted.
The `--force-laws` value may include inline JSON data after a colon, without
spaces; for example, `'mtfp:{"c":[1e11,0],"g":5.2}'`.

## JSON input format

An input document contains a `groups` array and a `bodies` array. A `laws`
array is optional and assigns a force-law configuration to a group:

```json
{
  "groups": ["g1"],
  "laws": [
    {
      "id": "g1",
      "laws": {
        "type": "nlug",
        "data": {}
      }
    }
  ],
  "bodies": [
    {
      "type": "mv_body",
      "data": {
        "id": "body-1",
        "gid": "g1",
        "p": [0.0, 0.0],
        "v": [1000.0, 0.0],
        "m": 5.0e24
      }
    }
  ]
}
```

Use `st_body` for a stationary body; it uses `id`, `gid`, `p`, and `m`.
Moving bodies additionally require `v`. Batch output is an object with a
`states` array. Each state records the simulation time and each group's bodies,
including position (`p`), velocity (`v`), mass (`m`), and force (`f`).

The archive includes four sample inputs under
`resources/examples/input/` and reference outputs under
`resources/examples/expected_output/`.

## Architecture

`simulator.launcher.Main` parses command-line options and selects GUI or batch
execution. `Controller` translates JSON into model objects and coordinates
loading and execution. `PhysicsSimulator` advances groups of bodies using the
selected `ForceLaws` implementation and notifies registered observers.
Factories keep JSON type tags (`mv_body`, `st_body`, `nlug`, `mtfp`, and `nf`)
separate from construction logic. Swing classes in `simulator.view` observe
the controller and present tables, controls, and the viewer.

## Limitations

- The repository does not currently include Maven/Gradle configuration,
  continuous integration, or a published binary distribution.
- The supplied dependencies are local JAR files, so they are not resolved
  automatically.
- The GUI is a desktop Swing application and requires a graphical environment.
- The simulator is intentionally small: it provides the included body types and
  force laws rather than a general-purpose physics engine.
- Numerical stability, collision handling, and unit conventions are not
  documented as production guarantees; choose the time step and scenario
  values appropriate for the experiment.

## Project status

This is a reference-oriented educational project. The current repository
contains the original source archive, example scenarios, tests, and an MIT
license, but no tagged release or roadmap. Contributions should preserve the
existing JSON formats and document any changes to the CLI or simulation model.

## Authors

- Sergio Martínez Olivera
- Daniel Roldán Serrano ([danirold](https://github.com/danirold))

## License

Distributed under the [MIT License](LICENSE).
