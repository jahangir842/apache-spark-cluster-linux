## 1. Stop SLURM Services

Before uninstalling, stop the daemons:

```bash
sudo systemctl stop slurmctld
sudo systemctl stop slurmd
sudo systemctl stop slurmdbd  # only if you installed accounting
sudo systemctl stop munge
```

---

## 2. Remove SLURM Packages

### On **Ubuntu/Debian**

```bash
sudo apt purge --autoremove slurm-wlm slurm-client slurmctld slurmd munge libmunge-dev -y
```

### On **CentOS/RHEL**

```bash
sudo yum remove slurm slurm-slurmd slurm-slurmctld munge munge-libs -y
```

---

## 3. Clean Up Configuration & Logs

Remove leftover config, spool, and log files:

```bash
sudo rm -rf /etc/slurm
sudo rm -rf /var/spool/slurmd
sudo rm -rf /var/log/slurm*
sudo rm -rf /var/run/slurm*
sudo rm -rf /var/lib/slurm
```

If you had MUNGE:

```bash
sudo rm -rf /etc/munge /var/log/munge /var/lib/munge /var/run/munge
```

---

## 4. Verify Cleanup

Check if anything related is still running:

```bash
ps aux | grep slurm
```

Check packages:

```bash
dpkg -l | grep slurm   # Ubuntu/Debian
rpm -qa | grep slurm   # CentOS/RHEL
```

If nothing shows up, SLURM is fully removed ✅

---

## 5. Optional: Reinstall Fresh

If you plan to reinstall:

```bash
sudo apt update
sudo apt install slurm-wlm munge -y
```

---
