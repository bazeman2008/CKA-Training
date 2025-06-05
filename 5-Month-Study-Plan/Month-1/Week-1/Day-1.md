# Day 1: Linux Setup & CKA Environment

## Overview (1.5 hours)
Set up your practice environment and essential tools that form the foundation for all CKA exam tasks.

## Why This Matters for the CKA Exam
- **60% of exam questions** require cluster setup or node management
- **Environment consistency** prevents configuration issues during practice
- **Tool familiarity** saves critical time during the 2-hour exam
- **SSH access** is essential for multi-node cluster operations

## Task 1: Install Linux on 3 VMs (45 minutes)

### What You'll Do
Set up three virtual machines to simulate a real Kubernetes cluster environment.

### Step-by-Step Instructions

#### VM Requirements
```bash
# Minimum specifications per VM:
CPU: 2 cores
RAM: 4GB (8GB recommended)
Disk: 40GB
Network: Bridged or NAT with static IPs
```

#### Installation Process
1. **Download Linux Distribution**
   ```bash
   # Recommended: Ubuntu 20.04/22.04 LTS or Rocky Linux 8
   # CKA exam uses Ubuntu/CentOS/RHEL environments
   ```

2. **VM Setup Configuration**
   ```bash
   # VM Names (for organization):
   master-node    # Control plane node
   worker-node-1  # First worker node  
   worker-node-2  # Second worker node
   ```

3. **Network Configuration**
   ```bash
   # Set static IPs (example):
   master-node:    192.168.1.10/24
   worker-node-1:  192.168.1.11/24
   worker-node-2:  192.168.1.12/24
   ```

### CKA Exam Connection
- **Cluster Architecture (25% of exam)**: Understanding multi-node setup
- **Node Management**: Practice with real cluster topology
- **Troubleshooting (30% of exam)**: Multi-node debugging scenarios

### Validation Commands
```bash
# Verify VM connectivity
ping 192.168.1.10  # From each node to others
ping 192.168.1.11
ping 192.168.1.12

# Check system resources
free -h
df -h
lscpu
```

## Task 2: Configure SSH and Essential Tools (30 minutes)

### What You'll Do
Set up secure communication between nodes and install tools required for Kubernetes operations.

### Step-by-Step Instructions

#### 1. SSH Key Generation and Distribution
```bash
# On master node, generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

# Copy public key to all nodes (including master)
ssh-copy-id user@192.168.1.10  # master
ssh-copy-id user@192.168.1.11  # worker-1
ssh-copy-id user@192.168.1.12  # worker-2

# Test passwordless SSH
ssh user@192.168.1.11 "hostname"
ssh user@192.168.1.12 "hostname"
```

#### 2. Essential Tools Installation
```bash
# Update system packages
sudo dnf update -y  # Rocky Linux 8.5

# Install essential tools
sudo dnf install -y vim tmux git curl wget htop net-tools  # Rocky Linux 8.5

# Install container runtime dependencies
sudo dnf install -y ca-certificates
# Note: redhat-lsb-core provides lsb-release functionality if needed
sudo dnf install -y redhat-lsb-core
```

### CKA Exam Connection
- **Remote Access**: SSH between nodes during troubleshooting
- **Tool Efficiency**: Familiar tools speed up problem resolution
- **Security**: Understanding secure communication between cluster components

### Validation Commands
```bash
# Verify SSH access
ssh worker-node-1 "date"
ssh worker-node-2 "date"

# Verify tools installation
vim --version
tmux -V
git --version
curl --version
```

## Task 3: Set up bash aliases (10 minutes)

### What You'll Do
Configure command shortcuts that will save significant time during the CKA exam.

### Step-by-Step Instructions

#### 1. Create Alias Configuration
```bash
# Edit bash configuration
vim ~/.bashrc

# Add CKA-specific aliases (add to end of file)
cat >> ~/.bashrc << 'EOF'

# CKA Exam Aliases - CRITICAL FOR TIME MANAGEMENT
alias k=kubectl
alias kg='kubectl get'
alias kd='kubectl describe'
alias kdel='kubectl delete'
alias kl='kubectl logs'
alias ke='kubectl edit'
alias kex='kubectl exec'
alias kaf='kubectl apply -f'
alias kcf='kubectl create -f'

# Output format shortcuts
export do='--dry-run=client -o yaml'
export now='--force --grace-period 0'

# Quick namespace switching
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kgn='kubectl get nodes'

EOF

# Apply changes
source ~/.bashrc
```

#### 2. Copy to All Nodes
```bash
# Copy bashrc to other nodes
scp ~/.bashrc worker-node-1:~/
scp ~/.bashrc worker-node-2:~/

# Apply on remote nodes
ssh worker-node-1 "source ~/.bashrc"
ssh worker-node-2 "source ~/.bashrc"
```

### CKA Exam Connection
- **Time Management**: Aliases can save 30-60 seconds per command
- **Efficiency**: Quick commands reduce typing errors
- **Exam Strategy**: More time for complex problem-solving

### Validation Commands
```bash
# Test aliases (after kubectl installation)
k version  # Should work same as 'kubectl version'
echo $do   # Should show '--dry-run=client -o yaml'
```

## Task 4: Enable kubectl bash completion (5 minutes)

### What You'll Do
Set up auto-completion for kubectl commands to speed up command entry and reduce errors.

### Step-by-Step Instructions

#### 1. Install bash-completion
```bash
# Install completion package
sudo dnf install -y bash-completion  # Rocky Linux 8.5
```

#### 2. Configure kubectl completion
```bash
# Add kubectl completion to bashrc
cat >> ~/.bashrc << 'EOF'

# Kubectl auto-completion
source <(kubectl completion bash)
complete -F __start_kubectl k  # For 'k' alias

EOF

# Apply changes
source ~/.bashrc
```

#### 3. Copy to All Nodes
```bash
# Copy updated bashrc to worker nodes
scp ~/.bashrc worker-node-1:~/
scp ~/.bashrc worker-node-2:~/
```

### CKA Exam Connection
- **Accuracy**: Reduces command typos under exam pressure
- **Speed**: Tab completion saves typing time
- **Discovery**: Helps remember command options quickly

### Validation Commands
```bash
# Test completion (after kubectl installation)
k get <TAB><TAB>     # Should show available resources
k get pods -<TAB>    # Should show available flags
```

## Common Issues and Solutions

### Issue 1: VM Network Connectivity
**Problem**: Cannot ping between VMs
**Solution**: 
```bash
# Check network configuration
ip addr show
ip route show

# Verify firewall rules
sudo firewall-cmd --list-all  # Rocky Linux 8.5
# To open ports if needed:
# sudo firewall-cmd --permanent --add-port=PORT/tcp
# sudo firewall-cmd --reload
```

### Issue 2: SSH Key Authentication Fails
**Problem**: Still prompted for password
**Solution**:
```bash
# Check SSH service status
sudo systemctl status ssh

# Verify key permissions
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
```

### Issue 3: Aliases Not Working
**Problem**: Commands not recognized
**Solution**:
```bash
# Reload bashrc
source ~/.bashrc

# Check if aliases are set
alias | grep kubectl
```

## Time Management Tips

### For CKA Exam Success
- **Practice Environment**: Spend extra time here to save time later
- **Muscle Memory**: Use aliases from day one to build habits  
- **Consistency**: Keep same environment setup throughout training
- **Documentation**: Note any custom configurations for exam day

### Daily Practice Goals
- [ ] Complete environment setup in 45 minutes or less
- [ ] SSH between all nodes without passwords
- [ ] All aliases working correctly
- [ ] Bash completion functioning

## Next Day Preparation

### Review Before Day 2
- [ ] All VMs accessible and running
- [ ] SSH connectivity verified
- [ ] Essential tools installed and tested
- [ ] Aliases and completion configured

### What's Coming Next
Day 2 will build on this foundation with container runtime installation, which requires:
- Working Linux environment ✓
- Network connectivity between nodes ✓  
- SSH access to all nodes ✓
- Essential tools available ✓

## Study Notes Section
_Use this space to document any custom configurations, IP addresses, or troubleshooting steps specific to your environment_

**Environment Details:**
- Master Node IP: ________________
- Worker Node 1 IP: ______________
- Worker Node 2 IP: ______________
- Linux Distribution: _____________
- Any custom configurations: ______