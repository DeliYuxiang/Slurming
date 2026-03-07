# SlurmUtils

A Python utility library for generating and managing [Slurm](https://slurm.schedmd.com/) HPC job scripts.

**Version:** 0.2.3
**Author:** Yuxiang Luo ([luo.929@osu.edu](mailto:luo.929@osu.edu))
**License:** GPL-2.0
**Affiliation:** Center for Weldability, The Ohio State University

---

## Installation

```bash
pip install slurmutils
```

Or with Poetry:

```bash
poetry add slurmutils
```

**Requirements:** Python ^3.8, pandas ^2.2.2 (Python ^3.9+)

---

## Package Structure

```
slurmutils/
├── Slurm/
│   ├── slurm.py        # Query Slurm job queue
│   └── shellUtils.py   # Shell script generation helpers
├── SlurmJob/
│   └── job.py          # SlurmJob class (OOP job builder)
└── DecentJob/
    └── metaJob.py      # DataFrame-based job metadata utilities
```

---

## Modules

### `slurmutils.Slurm`

#### `get_job_dict(cluster_name)`

Query the Slurm queue and return job status dicts.

```python
from slurmutils.Slurm.slurm import get_job_dict

jobs, running_jobs, pending_jobs = get_job_dict("myCluster")
```

Returns:
- `jobs` — `{cluster: {job_name: {state, job_id}}}` for all visible jobs
- `running_jobs` — `{job_name: {state, job_id}}` filtered to `RUNNING`
- `pending_jobs` — `{job_name: {state, job_id}}` filtered to `PENDING` on the specified cluster

Tries `squeue --clusters=all` first; falls back to local cluster on error.

---

#### `make_shell_script(...)`

Generate an sbatch shell script file.

```python
from slurmutils.Slurm.shellUtils import make_shell_script
from pathlib import Path

make_shell_script(
    account="PAS2138",
    script_path=Path("run_job.sh"),
    content=["python train.py"],
    hours=4,
    cores=28,
    modules=["python/3.10"],
    memory=64,          # GB
    gpus=1,
    jobname="my_train",
    sbatch_log=Path("logs/job.log"),
)
```

Key parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `account` | `str` | — | Slurm account to charge |
| `script_path` | `Path` | — | Output `.sh` file path |
| `content` | `List[str]` | — | Shell commands to run |
| `hours` / `minutes` / `seconds` | `int` | `2` / `0` / `0` | Wall time |
| `cores` | `int` | `28` | CPUs per task |
| `gpus` | `int` | `0` | Number of GPUs |
| `memory` | `int` | `0` | Memory in GB (0 = unset) |
| `modules` | `List[str]` | `[]` | Modules to load |
| `module_profie` | `str\|None` | `None` | Custom module profile path |
| `python_env` | `str` | `""` | Path to Python virtualenv activate script |
| `env_vars` | `Dict[str,str]` | `{}` | Environment variables to export |
| `aliases` | `Dict[str,str]` | `{}` | Shell aliases to define |
| `paths` | `List[str]` | `[]` | Extra paths to append to `$PATH` |
| `notifies` | `List[str]` | `["FAIL"]` | Slurm mail-type events |
| `sbatch_args` | `Dict` | `{}` | Extra `#SBATCH` key-value directives |
| `license` | `Dict[str,int]` | `{}` | Software licenses (OSC format) |
| `set_flag` | `str\|None` | `"x"` | Bash `set -<flag>` option |
| `interactive` | `bool` | `False` | Use `#!/bin/bash -i` |
| `bashinit` | `List[str]` | `[]` | Lines inserted before env setup |

---

#### `make_command(head, params, params1_dict, params2_dict, stdout_redirect, connection)`

Build a shell command string from parts.

```python
from slurmutils.Slurm.shellUtils import make_command

cmd = make_command(
    "mpirun",
    params=["./solver"],
    params1_dict={"n": 28},
    params2_dict={"bind-to": "core"},
)
# => "mpirun -n=28 --bind-to=core ./solver"
```

---

#### `make_if_statement(if_st, elif_sts, else_st)`

Build a bash `if/elif/else/fi` block as a list of strings.

```python
from slurmutils.Slurm.shellUtils import make_if_statement

lines = make_if_statement(
    if_st=(["$ret -eq 0"], ["echo success"]),
    else_st=["echo failure"],
)
```

---

### `slurmutils.SlurmJob`

#### `SlurmJob`

OOP interface for constructing and writing sbatch scripts.

```python
from slurmutils.SlurmJob.job import SlurmJob
from pathlib import Path

job = SlurmJob(
    account="PAS2138",
    content=["python simulate.py"],
    licenses={},
    modules=["python/3.10"],
    env_vars={},
    notify_email=["FAIL"],
    aliases={},
    paths=[],
    sbatch_args={},
    output_storage=[],
    hours=8,
    cpus_per_task=28,
    memory=128,
    job_name="simulation",
    partition="gpu",
    gpus=1,
    python_home=Path("/home/user/.venv"),
    working_dir=Path("/scratch/user/project"),
)

job.append(["echo done"])
job.write(job_name="simulation", script_path=Path("sim.sh"))
```

Key methods:

| Method | Description |
|--------|-------------|
| `append(content)` | Append lines to the job script |
| `prepend(content)` | Prepend lines to the job script |
| `write(job_name, script_path, log_file, delete_log_on_completion, delete_script_on_completion)` | Write the sbatch script to disk |

---

### `slurmutils.DecentJob`

Utilities for managing collections of jobs tracked in a pandas DataFrame.

#### `find_job_by_params(condition_dict, df, exception, is_exact)`

Query a DataFrame for jobs matching a parameter dict.

```python
from slurmutils.DecentJob.metaJob import find_job_by_params

result = find_job_by_params(
    condition_dict={"solver": "myFoam", "cores": 28},
    df=job_df,
    is_exact=True,
)
```

#### `query_dataframe(condition_dict, df)`

Low-level DataFrame query builder.

```python
from slurmutils.DecentJob.metaJob import query_dataframe

result, query_str = query_dataframe({"cores": 28, "solver": "myFoam"}, df)
```

#### `migrate_static(new_job_df, static_dir)`

Migrate a directory of static job folders to new parameter-based indices by matching `case.json` metadata against a new DataFrame layout.

```python
from slurmutils.DecentJob.metaJob import migrate_static
from pathlib import Path

migrate_static(new_job_df=job_df, static_dir=Path("/scratch/cases"))
```

---

## License

GPL-2.0. See [LICENSE](LICENSE) for details.
