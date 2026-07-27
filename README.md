# Task Offloading Modeling

This repository provides a Lingua Franca-based framework for modeling, simulating, and deploying transformer-oriented AI inference workloads on heterogeneous platforms. The target use case is task offloading for DAG-structured inference pipelines, where each node represents a computation stage and each edge represents a dependency with communication cost.

The current codebase supports two main workflows:

- Simulation of scheduling and communication decisions in logical time.
- Federated deployment across heterogeneous devices such as Jetson, A6000, and Blackwell.

`lib/deployment/Jetson.lf` is still a work in progress. The deployment path for Jetson is therefore documented as a placeholder.

## Repository Structure

- `src/sample_simulation.lf`: end-to-end scheduling simulation using CSV task, dependency, and bandwidth inputs.
- `src/sample_deployment.lf`: federated deployment example with runtime trace collection and visualization.
- `src/LLMTest.lf`: standalone test for the Blackwell LLM path.
- `src/ViTTest.lf`: standalone test for the Blackwell vision path.
- `lib/simulation/`: logical-time device models for simulation.
- `lib/deployment/`: device implementations for federated deployment.
- `lib/tests/`: focused test reactors for deployment-side LLM and vision execution.
- `Tasks.csv`: task definitions, deadlines, execution times, and I/O sizes.
- `Dependencies.csv`: DAG structure for the workload.
- `Bandwidth.csv`: inter-device bandwidth matrix.

## Inputs and Outputs

The framework expects a workload DAG and platform description from the CSV files in the repository root:

- `Tasks.csv`: task metadata and per-device execution times.
- `Dependencies.csv`: parent-child dependency matrix.
- `Bandwidth.csv`: bandwidth matrix used to model communication cost.

The main generated artifacts are:

- `schedule_trace.png`: scheduling trace produced by the simulation workflow.
- `schedule_trace_deployment.png`: scheduling trace reconstructed from the deployment runtime trace.

## CSV Formats

### `Tasks.csv`

The scheduler reads the task file row by row. The current parser consumes the columns in this order:

| Column | Meaning |
| --- | --- |
| `Task Name` | Task identifier used in logs and visualizations |
| `Deadline` | Task deadline in milliseconds |
| `ET on A` | Execution time on device type A / Jetson in milliseconds |
| `ET on B` | Execution time on device type B / A6000 in milliseconds |
| `ET on C` | Execution time on device type C / Blackwell in milliseconds |
| `Input Size` | Input payload size |
| `Output Size` | Output payload size |
| `Source` | Present in the CSV, currently not used by the main parser |
| `Destination` | Present in the CSV, currently not used by the main parser |

Example:

```csv
,Deadline,ET on A,ET on B,ET on C,Input Size,Output Size,Source,Destination
VIT1,400,300,200,100,2000,2000,0,0
VIT2,400,300,200,100,2000,2000,0,0
LLM1,600,600,400,200,2000,100,0,0
LLM2,600,600,400,200,2000,100,0,0
```

Notes:

- The first column is treated as the task name.
- Execution times are mapped internally as Jetson, A6000, Blackwell.
- Empty task names are skipped.

### `Dependencies.csv`

`Dependencies.csv` defines the DAG adjacency matrix.

- Each row corresponds to a source task.
- Each column corresponds to a destination task.
- A value of `1` means there is a dependency edge from the row task to the column task.
- A value of `0` means no dependency.

Example:

```csv
,I1,I2,L1,L2
I1,0,0,1,0
I2,0,0,0,1
L1,0,0,0,0
L2,0,0,0,0
```

In this example:

- `I1 -> L1`
- `I2 -> L2`

The parser expects dependency values to be binary (`0` or `1`).

### `Bandwidth.csv`

`Bandwidth.csv` defines the device-to-device bandwidth matrix.

- Rows and columns are device names.
- The row order and column order must match.
- Values are interpreted as inter-device bandwidths used for communication cost modeling.
- Diagonal values are typically `0`.

Example:

```csv
,Jetson, A6000, Blackwell
Jetson, 0, 30, 20
A6000, 30, 0, 25
Blackwell, 20, 25, 0
```

The main parser strips whitespace from row and column names before validation.

## Environment Setup

### General

You need:

- Python 3
- Lingua Franca compiler (`lfc`)

<!-- Optional GUI support for the configuration helper:

```bash
sudo apt install python3-tk
``` -->

### Python Environment

First, create a virtual environment:

```bash
python3 -m venv my_env
```

Activate it:

```bash
source my_env/bin/activate
```

You can then install one of the dependency sets below. If you prefer not to activate the environment, replace `pip` with `./my_env/bin/pip` or `python` with `./my_env/bin/python` in the commands that follow.

### Python Dependencies

Three dependency sets are provided:

- `requirements-simulation.txt`: simulation-only dependencies.
- `requirements-deployment-server.txt`: deployment dependencies for server-class GPUs such as A6000 and Blackwell.
- `requirements-deployment-jetson.txt`: Jetson deployment dependencies (WiP).

Example installation:

```bash
pip install -r requirements-simulation.txt
```

```bash
pip install -r requirements-deployment-server.txt
```

## Simulation Workflow

The simulation path emulates execution and communication in logical time. It is intended for studying scheduling behavior before running on actual hardware.

Compile:

```bash
lfc src/sample_simulation.lf
```

Run:

```bash
./bin/sample_simulation
```

Expected result:

- The program reads `Tasks.csv`, `Dependencies.csv`, and `Bandwidth.csv`.
- It schedules DAG tasks across Jetson, A6000, and Blackwell device models.
- It writes `schedule_trace.png` to the repository root.

## Deployment Workflow

The deployment path uses federated Lingua Franca execution and exchanges packets among device-specific reactors. The current main example is:

- `src/sample_deployment.lf`

Compile:

```bash
lfc src/sample_deployment.lf
```

Run the generated federates from `fed-gen/sample_deployment/`. The generated folder includes the RTI and per-federate executables.

Before running a distributed deployment, update the `federated reactor at ...` host in your deployment file (for example `src/sample_deployment.lf` or `src/sample_deployment_deterministic.lf`):

- The repository default is `localhost` (safe/local placeholder).
- Replace `localhost` with the IP address of the machine where the RTI process will run.
- Use the same RTI host IP for all participating federates.

Expected result:

- The deployment run reads the workload DAG and network configuration.
- It executes the scheduled task chain across the participating federates.
- It writes `schedule_trace_deployment.png` to the repository root.

### A6000 and Blackwell

For server deployment on A6000 and Blackwell, install:

```bash
pip install -r requirements-deployment-server.txt
```

Before running the generated executable, export `PYTHONPATH` so the executable can locate the packages inside the virtual environment:

```bash
export PYTHONPATH="$PWD/my_env/lib/python3.10/site-packages:$PYTHONPATH"
```

If your virtual environment uses a different Python minor version, replace `python3.10` with the matching directory name under `my_env/lib/`.

### Jetson

Jetson deployment support is still under development.

- `lib/deployment/Jetson.lf` is currently WiP.
- `requirements-deployment-jetson.txt` is a placeholder.
- Detailed setup and execution steps for Jetson will be added once the deployment path stabilizes.

## Focused Model Tests

Two focused deployment-side tests are included for validating model execution on Blackwell:

### LLM Test

Compile:

```bash
lfc src/LLMTest.lf
```

Run the generated federates from `fed-gen/LLMTest/`.

### Vision Test

Compile:

```bash
lfc src/ViTTest.lf
```

Run the generated federates from `fed-gen/ViTTest/`.

The current vision test uses an image from `dataset128/coco128/images/train2017/`.

# ToDo Lists
 - [] Deterministic execution for deployment
