# 🔬 Multi-Node SLURM Lab (Ubuntu VMs)

## 1. Lab Topology

* **Controller (head node)**

  * Runs `slurmctld` (scheduler/queue manager).
  * Also runs `slurmdbd` (optional, for accounting).
  * Acts as the login node for job submissions.

* **Worker(s) (compute nodes)**

  * Run `slurmd` (executes jobs).
  * Must share authentication with controller (MUNGE).

### Example

```
controller: 192.168.56.10 (hostname: master)
worker1:    192.168.56.11 (hostname: node1)
worker2:    192.168.56.12 (hostname: node2)
```

---

## 2. Prerequisites

1. Create 2+ Ubuntu VMs (20.04 or newer).
2. Ensure they can **ping each other by hostname** (use `/etc/hosts` if no DNS). Example:

   ```
   192.168.56.10 master
   192.168.56.11 node1
   192.168.56.12 node2
   ```
3. Sync system clocks (install `chrony` or `ntp`):

   ```bash
   sudo apt install chrony -y
   ```

---

## 3. Install Dependencies (on ALL nodes)

Run this on **controller + workers**:

```bash
sudo apt update
sudo apt install slurm-wlm slurm-client munge libmunge-dev -y
```

---

## 4. Configure MUNGE (Authentication)

### On Controller:

Generate key:

```bash
sudo /usr/sbin/create-munge-key
```

Copy key to all workers:

```bash
scp /etc/munge/munge.key node1:/etc/munge/
scp /etc/munge/munge.key node2:/etc/munge/
```

Set permissions on all nodes:

```bash
sudo chown munge:munge /etc/munge/munge.key
sudo chmod 400 /etc/munge/munge.key
```

Start MUNGE everywhere:

```bash
sudo systemctl enable munge
sudo systemctl start munge
```

Test:

```bash
munge -n | unmunge
remunge
```

---

## 5. Configure SLURM

Main config: `/etc/slurm/slurm.conf`

### On Controller (master)

Create config:

```bash
sudo nano /etc/slurm/slurm.conf
```

Example config (adjust hostnames & CPUs):

```ini
ClusterName=slurm_lab
ControlMachine=master

# Authentication
AuthType=auth/munge

# Logs and state
SlurmctldLogFile=/var/log/slurmctld.log
SlurmdLogFile=/var/log/slurmd.log
SlurmctldPidFile=/var/run/slurmctld.pid
SlurmdPidFile=/var/run/slurmd.pid
SlurmdSpoolDir=/var/spool/slurmd

# Node definitions
NodeName=node1 CPUs=2 State=UNKNOWN
NodeName=node2 CPUs=2 State=UNKNOWN

# Partitions (queues)
PartitionName=debug Nodes=node1,node2 Default=YES MaxTime=30:00 State=UP
```

Copy this config to **all worker nodes**:

```bash
scp /etc/slurm/slurm.conf node1:/etc/slurm/
scp /etc/slurm/slurm.conf node2:/etc/slurm/
```

---

## 6. Start SLURM Services

### On Controller:

```bash
sudo systemctl enable slurmctld
sudo systemctl start slurmctld
```

### On Workers:

```bash
sudo systemctl enable slurmd
sudo systemctl start slurmd
```

---

## 7. Verify Cluster

Run on controller:

```bash
sinfo
```

Expected:

```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug*       up   30:00       2   idle  node[1-2]
```

Check job queue:

```bash
squeue
```

---

## 8. Submit a Test Job

Create `hello.slurm`:

```bash
#!/bin/bash
#SBATCH --job-name=hello_test
#SBATCH --output=hello_out.txt
#SBATCH --ntasks=1
#SBATCH --time=2:00
#SBATCH --partition=debug

echo "Hello from $(hostname)"
```

Submit:

```bash
sbatch hello.slurm
```

Check:

```bash
squeue
cat hello_out.txt
```

You should see:

```
Hello from node1
```

(or node2, depending on scheduler choice).

---

## 9. Multi-Node Parallel Job (MPI Example)

Install MPI:

```bash
sudo apt install mpich -y
```

Batch script:

```bash
#!/bin/bash
#SBATCH --job-name=mpi_test
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=2
#SBATCH --output=mpi_out.txt
#SBATCH --time=5:00

srun hostname
```

Submit:

```bash
sbatch mpi_test.slurm
```

Output should show 4 lines (2 per node):

```
node1
node1
node2
node2
```

---

## 10. Troubleshooting

* If workers show `DOWN` in `sinfo`, check logs:

  * Controller: `/var/log/slurmctld.log`
  * Worker: `/var/log/slurmd.log`
* Ensure `/etc/hosts` is consistent on all nodes.
* Ensure MUNGE key is identical across all machines.

---

## ✅ End Result

You now have a **working 2+ node SLURM cluster**:

* Controller runs `slurmctld`.
* Workers run `slurmd`.
* Authentication with MUNGE works.
* You can submit jobs across nodes.

---

