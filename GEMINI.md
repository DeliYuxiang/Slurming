# slurming Project Context

## Project Overview
`slurming` is a Python utility library designed to simplify the creation, management, and monitoring of Slurm jobs. It provides high-level abstractions for generating Slurm shell scripts, querying job status via `squeue`, and managing job metadata using Pandas.

- **Main Technologies:** Python (3.8+), Poetry, Pandas.
- **Target System:** Slurm Workload Manager.
- **License:** GPL2.

## Project Architecture
The project is structured into several sub-packages within `slurming/`:

- `slurming/Slurm/`: Core utilities for interacting with Slurm commands and generating shell scripts (e.g., `squeue` wrappers, script generation logic).
- `slurming/SlurmJob/`: Contains the `SlurmJob` class, which provides an object-oriented interface for defining Slurm job parameters (CPUs, GPUs, memory, time, modules, etc.).
- `slurming/DecentJob/`: Utilities for managing collections of jobs and their metadata using Pandas DataFrames and JSON files.
- `.github/workflows/`: CI/CD pipelines for linting (using `yapf`) and publishing to PyPI (official and custom `coredumped` repository).

## Building and Running

### Development Environment
The project uses [Poetry](https://python-poetry.org/) for dependency management and building.

- **Install dependencies:**
  ```bash
  poetry install
  ```
- **Activate virtual environment:**
  ```bash
  poetry shell
  ```

### Key Commands
- **Format Code:**
  ```bash
  yapf -mpi $(git ls-files '*.py')
  ```
- **Build Package:**
  ```bash
  poetry build
  ```
- **Publish (to custom repository):**
  ```bash
  ./publish.sh [major|minor|patch]
  ```

## Development Conventions

### Coding Style
- **Formatting:** The project uses `yapf` for code formatting. CI/CD will fail if code is not properly formatted.
- **Type Hinting:** Extensive use of Python type hints (`typing` module) for clarity and maintainability.

### Versioning
- Versioning is managed through `pyproject.toml`.
- Releases are triggered by pushing git tags (e.g., `v0.2.3`).

### CI/CD
- **Linting:** GitHub Actions run `yapf` on every push.
- **Publishing:** Automated publishing to PyPI occurs when a new version tag is pushed.
