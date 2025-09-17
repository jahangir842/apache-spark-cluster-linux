Let’s build a **step-by-step SLURM lab** on a single Ubuntu machine (you can later extend to multiple nodes).
This guide will take you from **installation → configuration → first job submission**.

---

# 🔬 SLURM Hands-On Lab (Single Node)

## 1. Prerequisites

* Ubuntu 20.04+ (VM or bare metal)
* At least 2 GB RAM
* User with `sudo` privileges

Optional: If you want a multi-node setup later, prepare 2+ VMs with SSH connectivity.

---

## 2. Install Required Packages

Update system and install SLURM:

```bash
sudo apt update
sudo apt install slurm-wlm slurm-client munge libmunge-dev -y
```

🔹 **What is MUNGE?**
MUNGE (MUNGE Uid 'N' Gid Emporium) provides authentication between nodes. Even for single-node, we need it.

---

## 3. Configure MUNGE

Generate a key:

```bash
sudo /usr/sbin/create-munge-key
```

Start MUNGE:

```bash
sudo systemctl enable munge
sudo systemctl start munge
sudo systemctl status munge
```

Test:

```bash
munge -n | unmunge
remunge
```

👉 If it prints a decoded authentication block, MUNGE works.

---

## 4. Configure SLURM

Main config file: `/etc/slurm/slurm.conf`
We’ll create a **minimal config** for a single node.

Generate with SLURM configurator:
🔗 [https://slurm.schedmd.com/configurator.html](https://slurm.schedmd.com/configurator.html)

Or directly create:

```bash
sudo nano /etc/slurm/slurm.conf
```

Paste:

```ini
# Basic SLURM Config (single node lab)
ClusterName=slurm_lab
ControlMachine=localhost

# Authentication
AuthType=auth/munge

# State and Log files
SlurmctldLogFile=/var/log/slurmctld.log
SlurmdLogFile=/var/log/slurmd.log
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
SlurmdSpoolDir=/var/spool/slurmd

# Nodes
NodeName=localhost CPUs=2 State=UNKNOWN
PartitionName=debug Nodes=localhost Default=YES MaxTime=30:00 State=UP
```

Save and exit.

---

## 5. Start SLURM Services

```bash
sudo systemctl enable slurmctld slurmd
sudo systemctl start slurmctld slurmd
```

Check status:

```bash
systemctl status slurmctld
systemctl status slurmd
```

👉 If errors: check logs in `/var/log/slurm*.log`.

---

## 6. Verify Setup

Check cluster info:

```bash
sinfo
```

Expected output:

```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug*       up   30:00       1   idle  localhost
```

If you see your node in **idle**, SLURM is working ✅.

---

## 7. Submit Jobs

### (a) Interactive job

```bash
srun --partition=debug --ntasks=1 --time=5:00 hostname
```

Expected output:

```
localhost
```

### (b) Batch job

Create a script `test_job.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=hello_job
#SBATCH --output=hello_out.txt
#SBATCH --ntasks=1
#SBATCH --time=5:00
#SBATCH --mem=128M

echo "Running on $(hostname)"
date
sleep 60
```

Submit:

```bash
sbatch test_job.slurm
```

Check queue:

```bash
squeue
```

After \~1 min, check output:

```bash
cat hello_out.txt
```

You should see:

```
Running on localhost
Mon Sep 17 12:45:01 UTC 2025
```

---

## 8. Run a Python Job

File: `hello.py`

```python
import socket, time
print("Hello from", socket.gethostname())
time.sleep(30)
```

Batch script: `python_job.slurm`

```bash
#!/bin/bash
#SBATCH --job-name=python_test
#SBATCH --output=python_out.txt
#SBATCH --ntasks=1
#SBATCH --time=5:00
#SBATCH --mem=128M

python3 hello.py
```

Submit:

```bash
sbatch python_job.slurm
```

Check results:

```bash
cat python_out.txt
```

---

## 9. Useful Commands Cheat Sheet

| Command                     | Description         |
| --------------------------- | ------------------- |
| `sinfo`                     | Cluster/node status |
| `squeue`                    | Job queue           |
| `sbatch job.slurm`          | Submit job script   |
| `srun <cmd>`                | Run interactively   |
| `salloc --ntasks=2`         | Allocate resources  |
| `scancel <jobid>`           | Cancel a job        |
| `scontrol show job <jobid>` | Job details         |
| `sacct` *(needs slurmdbd)*  | Job accounting      |

---

## 10. Next Experiments

1. **Job Arrays** (good for parameter sweeps)

```bash
#SBATCH --array=1-5
echo "This is task $SLURM_ARRAY_TASK_ID"
```

2. **Request resources**

```bash
#SBATCH --cpus-per-task=2
#SBATCH --mem=1G
```

3. **MPI Jobs**
   Install `mpich` or `openmpi`, then run:

```bash
srun -n 4 ./mpi_program
```

---

✅ **End Result:** You now have a working SLURM lab on a single Ubuntu VM, where you can submit batch/interactive jobs, test Python scripts, and scale up to multi-node.

---

