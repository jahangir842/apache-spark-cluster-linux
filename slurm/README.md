# SLURM (Simple Linux Utility for Resource Management) 

It is a **highly popular open-source workload manager** used in **High-Performance Computing (HPC) clusters**. It’s the backbone of many supercomputers and research clusters worldwide. 

---

## 1. What is SLURM?

SLURM is a **job scheduler and resource manager**.
Think of it as the “traffic controller” of a compute cluster:

* **Resource Manager**: Keeps track of compute nodes (CPUs, GPUs, RAM, etc.).
* **Job Scheduler**: Decides which job runs on which node, and when.
* **Queue System**: Users submit jobs into queues (called *partitions*), and SLURM schedules them efficiently.

🔑 **Why SLURM?**

* Open source & widely used in HPC.
* Scales from small labs to exascale supercomputers.
* Fair-share, priority, and backfilling scheduling.
* Integrates with MPI, CUDA, containers (Singularity/Apptainer), and cloud.

---

## 2. SLURM Architecture (Core Components)

A SLURM setup has three main parts:

1. **slurmctld (Controller daemon)**

   * Runs on the **head node / login node**.
   * Manages resources and job scheduling.

2. **slurmd (Daemon)**

   * Runs on each **compute node**.
   * Executes jobs and reports back to controller.

3. **slurmdbd (Database daemon)** *(optional)*

   * Stores accounting info (job history, usage tracking).

---

## 3. Key SLURM Concepts

* **Node**: A machine (physical/virtual server).
* **Partition**: A logical group of nodes (like a queue).
* **Job**: A workload submitted to SLURM.
* **Job Step**: Subdivision of a job (e.g., MPI processes).
* **Scheduler**: The logic that decides job placement.

Example:

* Partition = `gpu` → only GPU nodes.
* Partition = `short` → small jobs, <1 hour.
* Partition = `long` → big jobs, up to 7 days.

---

## 4. Installing SLURM (Lab Setup)

You can test SLURM on a **single machine** (mini-cluster).

### On Ubuntu/Debian:

```bash
sudo apt update
sudo apt install slurm-wlm slurm-client slurmctld slurmd
```

### On CentOS/RHEL:

```bash
sudo yum install slurm slurm-slurmd slurm-slurmctld
```

Configuration files:

* `/etc/slurm/slurm.conf` → main config.
* `/etc/slurm/cgroup.conf` → resource limits.
* `/etc/slurm/` → all configs/logs.

👉 Generate a sample config at: [https://slurm.schedmd.com/configurator.html](https://slurm.schedmd.com/configurator.html)

Example minimal `slurm.conf`:

```ini
ClusterName=mycluster
ControlMachine=master
NodeName=node[1-4] CPUs=4 State=UNKNOWN
PartitionName=debug Nodes=node[1-4] Default=YES MaxTime=30:00 State=UP
```

Start services:

```bash
sudo systemctl enable slurmctld slurmd
sudo systemctl start slurmctld slurmd
```

---

## 5. Submitting Jobs

Two ways to run jobs in SLURM:

### **(a) Interactive jobs**

```bash
srun --partition=debug --ntasks=1 --time=10:00 --pty bash
```

This gives you a shell inside the allocated node.

### **(b) Batch jobs**

Create a script `job.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=test_job
#SBATCH --output=output.txt
#SBATCH --ntasks=1
#SBATCH --time=10:00
#SBATCH --mem=512M
#SBATCH --partition=debug

echo "Running on: $(hostname)"
sleep 60
```

Submit with:

```bash
sbatch job.slurm
```

Check jobs:

```bash
squeue          # shows job queue
scontrol show job <jobid>   # details
scancel <jobid> # cancel
```

---

## 6. Useful SLURM Commands

* `sinfo` → Show cluster partitions/nodes.
* `squeue` → Show running/pending jobs.
* `sbatch` → Submit a batch job.
* `srun` → Run a job step (interactive).
* `salloc` → Allocate resources interactively.
* `scancel` → Cancel a job.
* `scontrol show job <id>` → Detailed info.

Example:

```bash
sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug*       up   30:00       4   idle  node[1-4]
```

---

## 7. Example: Python Job

File: `hello.py`

```python
import socket
print("Hello from", socket.gethostname())
```

Batch script:

```bash
#!/bin/bash
#SBATCH --job-name=python_test
#SBATCH --output=python_out.txt
#SBATCH --ntasks=1
#SBATCH --time=5:00
#SBATCH --mem=256M

python3 hello.py
```

Submit:

```bash
sbatch python_job.slurm
```

---

## 8. Next Steps to Master SLURM

1. **Play locally** → Single-node cluster on VM.
2. **Learn scheduling** → Priorities, fair-share, backfill.
3. **MPI Jobs** → Run parallel jobs with `srun`.
4. **GPU Jobs** → Request GPUs with `--gres=gpu:2`.
5. **Job Arrays** → Submit 1000 similar jobs efficiently.

   ```bash
   #SBATCH --array=1-100
   ```
6. **Accounting & Monitoring** → `sacct`, `sreport`.

---

✅ **In summary**: SLURM is the cluster manager and job scheduler for HPC. You:

* Install/configure SLURM.
* Submit jobs with `sbatch` or `srun`.
* Monitor with `squeue`, `sinfo`.
* Scale from toy examples to thousands of nodes.

---

