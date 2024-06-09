# Automated-Environment-Setup-for-Docker-and-WSL-with-NVIDIA-Support-
Automated Environment Setup for Docker and WSL with NVIDIA Support

README.md

🚀 Automated Environment Setup for Docker and WSL with NVIDIA Support

Welcome to the automated setup script! This guide will help you set up your environment for running Docker and WSL with NVIDIA support on your Windows machine. This setup is designed to streamline the process and ensure your system is configured correctly for optimal performance.

📋 Prerequisites

	•	Windows 10 or 11
	•	Administrator privileges
	•	Internet connection

🛠️ Setup Instructions

Follow these steps to automate your environment setup:

1. Clone the Repository

First, clone this repository to your local machine:

git clone https://github.com/yourusername/your-repo.git
cd your-repo

2. Environment Variables

Create a .env file in the root directory of your cloned repository to store environment variables:

# .env
LOG_PATH=C:\path\to\log
MAX_LOG_FILES=5

3. PowerShell Script

Create a PowerShell script named setup_environment.ps1 with the following content:

# Define log function
function Log-Message {
    param (
        [string]$message,
        [string]$type = "INFO"
    )
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "$timestamp [$type] - $message"
    
    # Ensure log directory exists
    $logDirectory = $env:LOG_PATH
    if (-not (Test-Path $logDirectory)) {
        New-Item -Path $logDirectory -ItemType Directory -Force
    }
    
    Add-Content -Path "$logDirectory\setup_log.log" -Value $logMessage
}

# Rotate logs
function Rotate-Logs {
    param (
        [string]$logPath,
        [int]$maxLogFiles = 5
    )
    $logFiles = Get-ChildItem -Path $logPath -Filter "*.log" | Sort-Object LastWriteTime -Descending
    if ($logFiles.Count -ge $maxLogFiles) {
        Remove-Item -Path $logFiles[-1].FullName
    }
}

# Error handling function
function Handle-Error {
    param (
        [string]$errorMessage
    )
    Log-Message -message $errorMessage -type "ERROR"
    throw $errorMessage
}

# Rotate logs at the start
Rotate-Logs -logPath $env:LOG_PATH

# Set execution policy
try {
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process -Force
    Log-Message -message "Execution policy set to RemoteSigned."
}
catch {
    Handle-Error -errorMessage "Failed to set execution policy. Error: $_"
}

# Check and enable WSL
function Enable-WSL {
    try {
        $wslFeature = Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
        if ($wslFeature.State -ne "Enabled") {
            Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart
            Log-Message -message "WSL enabled."
        } else {
            Log-Message -message "WSL already enabled."
        }
    }
    catch {
        Handle-Error -errorMessage "Failed to enable WSL. Error: $_"
    }
}

# Check and enable Virtual Machine Platform
function Enable-VirtualMachinePlatform {
    try {
        $vmFeature = Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
        if ($vmFeature.State -ne "Enabled") {
            Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
            Log-Message -message "Virtual Machine Platform enabled."
        } else {
            Log-Message -message "Virtual Machine Platform already enabled."
        }
    }
    catch {
        Handle-Error -errorMessage "Failed to enable Virtual Machine Platform. Error: $_"
    }
}

# Set WSL default version to 2
function Set-WSLVersion {
    try {
        wsl --set-default-version 2
        Log-Message -message "WSL default version set to 2."
    }
    catch {
        Handle-Error -errorMessage "Failed to set WSL default version to 2. Error: $_"
    }
}

# Install Ubuntu if not installed
function Install-Ubuntu {
    try {
        $ubuntuInstalled = wsl -l -v | Select-String -Pattern "Ubuntu"
        if (-not $ubuntuInstalled) {
            wsl --install -d Ubuntu
            Log-Message -message "Ubuntu installed."
        } else {
            Log-Message -message "Ubuntu already installed."
        }
    }
    catch {
        Handle-Error -errorMessage "Failed to install Ubuntu. Error: $_"
    }
}

# Check NVIDIA driver installation
function Check-NVIDIA {
    try {
        $driver = Get-WmiObject Win32_PnPSignedDriver | Where-Object { $_.DeviceClass -eq 'Display' -and $_.DeviceName -like '*NVIDIA*' }
        if ($null -eq $driver) {
            Log-Message -message "NVIDIA driver not found. Installing NVIDIA driver."
            Start-Process -FilePath "powershell" -ArgumentList "winget install --id NVIDIA.NVIDIAControlPanel" -Wait
            Log-Message -message "NVIDIA driver installed."
        } else {
            Log-Message -message "NVIDIA driver already installed: $($driver.DeviceName)"
        }
    }
    catch {
        Handle-Error -errorMessage "Failed to install or verify NVIDIA driver. Error: $_"
    }
}

# Check Docker installation and install if necessary
function Check-Docker {
    try {
        if (-not (Get-Command docker -ErrorAction SilentlyContinue)) {
            Log-Message -message "Docker not found. Installing Docker."
            winget install --id Docker.DockerDesktop
            Log-Message -message "Docker installed."
        } else {
            Log-Message -message "Docker already installed."
        }
    }
    catch {
        Handle-Error -errorMessage "Failed to install Docker. Error: $_"
    }
}

# Main script execution
Enable-WSL
Enable-VirtualMachinePlatform
Set-WSLVersion
Install-Ubuntu
Check-NVIDIA
Check-Docker

Log-Message -message "WSL, Docker, and NVIDIA driver setup completed."

4. Batch Script to Run PowerShell Script

Create a batch file (setup_environment.bat) to run the PowerShell script:

@echo off
powershell -ExecutionPolicy Bypass -File C:\path\to\setup_environment.ps1

5. Automate Execution at Startup

Use Windows Task Scheduler to run the batch file at startup:

	1.	Open Task Scheduler.
	2.	Create a new task.
	3.	Set the trigger to At startup.
	4.	Set the action to run the batch file (setup_environment.bat).
	5.	Save the task.

📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

🙌 Acknowledgements

Special thanks to all contributors and open-source projects that made this script possible.

🚀 Let’s Get Started!

By following the steps above, your environment will be ready for Docker and WSL with NVIDIA support. Happy coding! 🎉

Chief of Staff

Sign off
