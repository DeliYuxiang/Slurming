# SlurmUtils Project Memory

## Project Overview
- **Package:** `slurmutils` v0.2.3
- **Author:** Yuxiang Luo (luo.929@osu.edu), OSU Center for Weldability
- **License:** GPL-2.0
- **Python:** ^3.8; pandas ^2.2.2 (requires Python ^3.9)
- **Build:** Poetry (`pyproject.toml`)

## Structure
```
slurmutils/
├── Slurm/
│   ├── slurm.py        # get_job_dict(cluster_name) -> jobs, running, pending
│   └── shellUtils.py   # make_shell_script, make_command, make_if_statement
├── SlurmJob/
│   └── job.py          # SlurmJob class (OOP sbatch builder)
└── DecentJob/
    └── metaJob.py      # DataFrame job metadata: find_job_by_params, migrate_static, query_dataframe
```

All `__init__.py` files are empty (no public re-exports).

## Key Design Notes
- `make_shell_script` writes an sbatch shell script file directly; all sbatch directives are inlined as `#SBATCH` comments.
- `SlurmJob.write()` delegates to `make_shell_script`; supports auto-cleanup of log/script on completion.
- `LICENSE_USAGE`, `CONCURRENCY_CORE_USAGE`, `TIME_CORE_USAGE` dicts in `shellUtils.py` map core counts to OSC-specific values (likely unused in main API, kept as reference constants).
- `migrate_static` uses two-phase rename (append `_ren`, then strip) to avoid name collisions.
- README was initialized in this session (was previously empty).
