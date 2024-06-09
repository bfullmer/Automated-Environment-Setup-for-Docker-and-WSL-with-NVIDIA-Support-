### 🚨 WARNING: NO GUARANTEE - WILL REMOVE ALL SOFTWARE 🚨

**This setup script will inspect your computer and remove all software except for Docker, WSL, and browsers. Proceed with caution as this will result in the removal of all other software. There is no guarantee that this script will work on your specific system configuration. Always ensure you have backups of your important data before running this script.**

---

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

# Remove unnecessary software
Get-AppxPackage | Where-Object { $_.Name -notlike "*MicrosoftEdge*" -and $_.Name -notlike "*Docker*" -and $_.Name -notlike "*Ubuntu*" } | Remove-AppxPackage -ErrorAction SilentlyContinue
Log-Message "Unnecessary software removed."

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
$projectDir = "C:\Llama7bDockerSetup"
if (-not (Test-Path -Path $projectDir)) {
    New-Item -ItemType Directory -Path $projectDir
    Log-Message "Project directory created at $projectDir."
}

Set-Location -Path $projectDir
git init
Log-Message "Initialized empty Git repository in $projectDir."

# Create .env file
$envContent = @"
LLAMA_DOCKER_IMAGE=llama7b:latest
"@
Set-Content -Path "$projectDir\.env" -Value $envContent
Log-Message ".env file created."

Log-Message "Setup script completed successfully."
```

### Important Notes
- **Backup your data**: This script will remove all unnecessary software, so ensure you have backed up any important data.
- **Run as Administrator**: Make sure to run the PowerShell script with administrative privileges.
- **Review the script**: Before running the script, review its content to ensure it meets your specific needs and system configuration.

### Contact
For any issues or questions, please reach out to our support team.

---

**AI Chief of Staff** and **AI Director of Testing** approved.

---
