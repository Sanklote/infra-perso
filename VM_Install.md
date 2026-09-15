# INSTALLATION DES MACHINES

## Prérequis

Installer les machines virtuelles comme suis :

| NOM         | IP            |   |   |   |
|-------------|---------------|---|---|---|
| k3s-cp      | 192.168.1.210 |   |   |   |
| k3s-worker1 | 192.168.1.211 |   |   |   |
| k3s-worker2 | 192.168.1.212 |   |   |   |
| k3s-worker3 | 192.168.1.213 |   |   |   |

### 1. Mise à jour et paquets de base

```bash
dnf update -y
dnf install -y vim wget curl tar unzip bash-completion man-db chrony
```

### 2. Paquets réseau et prérequis k3s

```bash
dnf install -y NetworkManager-tui net-tools iproute bind-utils
dnf install -y container-selinux iptables policycoreutils-python-utils
```

### 3. Désactiver le swap (obligatoire pour Kubernetes/k3s)

```bash
swapoff -a
sed -i '/swap/s/^/#/' /etc/fstab
```

### 4. Firewalld — ouvrir les ports nécessaires

Sur le control-plane :

```bash
firewall-cmd --permanent --add-port=6443/tcp
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --reload
```

Sur les workers :

```bash
firewall-cmd --permanent --add-port=8472/udp
firewall-cmd --permanent --add-port=10250/tcp
firewall-cmd --reload
```

### 5. Résolution des noms

Chaque VM doit pouvoir résoudre les autres par leur nom d'hôte (soit via DNS interne, soit via /etc/hosts sur chaque nœud). Exemple /etc/hosts à répliquer sur les 4 VM :

<IP_CP>      k3s-cp
<IP_WORKER1> k3s-worker1
<IP_WORKER2> k3s-worker2
<IP_WORKER3> k3s-worker3

6. Vérifier SELinux

```bash
getenforce
```

Doit rester en Enforcing (k3s fonctionne avec, grâce à container-selinux).

7. Hostname propre sur chaque VM

```bash
hostnamectl set-hostname k3s-cp        # à adapter par nœud
```

Une fois cette checklist appliquée sur les 4 VM, on pourra passer à l'installation de k3s (control-plane d'abord, puis les workers avec le token). Tu veux qu'on parte sur cette base ?
