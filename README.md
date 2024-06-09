
# 🎉 Llama7b Docker Setup on Windows 🎉

Welcome to the Llama7b setup guide! This guide will walk you through the process of setting up a Docker environment on an old laptop or PC running Windows, tailored for running Llama7b. 

## Prerequisites
Ensure you have the following installed:
- Docker
- WSL2 (Windows Subsystem for Linux 2)
- NVIDIA Drivers (if you have an NVIDIA GPU)

## Setup Script

### Step 1: Set Environment Variables

Create a `.env` file in your project directory with the necessary environment variables:

```env
# .env file
LLAMA_DOCKER_IMAGE=llama7b:latest
```

### Step 2: PowerShell Setup Script

Copy and paste the following PowerShell script into a file named `setup.ps1` and run it as an Administrator. This script will:
- Inspect your computer
- Remove unnecessary software (except for Docker, WSL, and browsers)
- Set up Docker, WSL2, and NVIDIA drivers
- Create a project directory with a repository

```powershell
# setup.ps1

# Function to log messages
function Log-Message {
    param (
        [string]$message
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    Add-Content -Path "setup.log" -Value "$timestamp: $message"
}

# Rotate logs
function Rotate-Logs {
    if (Test-Path "setup.log") {
        $timestamp = Get-Date -Format "yyyyMMddHHmmss"
        Rename-Item -Path "setup.log" -NewName "setup_$timestamp.log"
    }
}

# Initial log rotation
Rotate-Logs

# Check and enable WSL if not already enabled
if ((Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux).State -ne "Enabled") {
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
    Log-Message "WSL enabled."
}

# Check and enable Virtual Machine Platform if not already enabled
if ((Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform).State -ne "Enabled") {
    Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
    Log-Message "Virtual Machine Platform enabled."
}

# Set WSL default version to 2
wsl --set-default-version 2
Log-Message "WSL default version set to 2."

# Install Ubuntu distribution if not already installed
if (-not (wsl -l -v | Select-String -Pattern "Ubuntu")) {
    wsl --install -d Ubuntu
    Log-Message "Ubuntu installed."
}

# Reboot to apply changes if required
if (Test-Path "C:\Windows\System32\shutdown.exe") {
    & "C:\Windows\System32\shutdown.exe" /r /t 0
    Log-Message "System rebooted to apply changes."
}

# Check NVIDIA driver installation
$driver = Get-WmiObject Win32_PnPSignedDriver | Where-Object { $_.DeviceClass -eq 'Display' -and $_.DeviceName -like '*NVIDIA*' }
if ($null -eq $driver) {
    Log-Message "NVIDIA driver not found. Installing NVIDIA driver."
    Start-Process -FilePath "powershell" -ArgumentList "winget install --id NVIDIA.NVIDIAControlPanel" -Wait
    Log-Message "NVIDIA driver installed."
} else {
    Log-Message "NVIDIA driver already installed: $($driver.DeviceName)"
}

Log-Message "WSL and NVIDIA driver setup completed."

# Create project directory and initialize repository
$projectPath = "$env:USERPROFILE\Llama7b-Docker"
if (-not (Test-Path $projectPath)) {
    New-Item -Path $projectPath -ItemType Directory
    Log-Message "Project directory created at $projectPath."
}

Set-Location -Path $projectPath
git init
Log-Message "Git repository initialized in $projectPath."

# Copy .env file
Copy-Item -Path "$PSScriptRoot\.env" -Destination $projectPath
Log-Message ".env file copied to $projectPath."

Log-Message "Setup script completed."
```

### Step 3: Running the Script

1. Open PowerShell as an Administrator.
2. Navigate to the directory where you saved `setup.ps1`.
3. Execute the script:
    ```powershell
    .\setup.ps1
    ```

### Step 4: Entering WSL Ubuntu

To enter the WSL Ubuntu environment, use the following command:
```sh
wsl -d Ubuntu
```

### Step 5: Install and Run Llama7b on Docker

Create a bash script `install_llama7b.sh` with the following content to automate the setup and execution:

```bash
#!/bin/bash

# Rotate logs
log_file="install_llama7b.log"
if [ -f "$log_file" ]; then
    timestamp=$(date +"%Y%m%d%H%M%S")
    mv "$log_file" "install_llama7b_$timestamp.log"
fi

exec > >(tee -a "$log_file") 2>&1

# Update package list and install prerequisites
echo "Updating package list and installing prerequisites..."
sudo apt-get update
sudo apt-get install -y docker.io docker-compose

# Pull Llama7b Docker image
echo "Pulling Llama7b Docker image..."
docker pull llama7b:latest

# Run Llama7b Docker container
echo "Running Llama7b Docker container..."
docker run --gpus all -d --name llama7b_container llama7b:latest

echo "Llama7b setup and container started successfully."
```

Make the script executable and run it:
```sh
chmod +x install_llama7b.sh
./install_llama7b.sh
```

## 🚀 You're all set! 🚀

Enjoy running Llama7b on your Docker setup!

---

© 2024 Chief of Staff
```

This README.md integrates the PowerShell setup script, instructions for entering WSL Ubuntu, and the bash script to install and run Llama7b on Docker, all presented with branding and emojis for an engaging guide.
This README.md integrates the PowerShell setup script, instructions for entering WSL Ubuntu, and the bash script to install and run Llama7b on Docker, all presented with branding and emojis for an engaging guide.