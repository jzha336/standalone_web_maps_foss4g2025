# Install Debug - Caddy Web Server Setup

This document explains the debugging process for installing Caddy web server on Windows, written for beginners who are new to command line operations.

## What is Caddy?
Caddy is a web server that can serve files from your computer to a web browser. Think of it like a local website server that runs on your machine so you can test web applications.

## The Problem We Encountered

### Initial Installation Attempt
We tried to install Caddy using **Chocolatey** (a package manager for Windows, like an app store for command line tools):

```powershell
choco install caddy
```

**What happened:** The installation failed with a "lock file" error. This is like when a file is being used by another program and Windows won't let you modify it.

**Error message explained:**
```
Unable to obtain lock file access on 'C:\ProgramData\chocolatey\lib\18c1117b8756e7ae77eb1dd5672b8147e64edc92'
```
This cryptic message means: "Another process is using this file, or a previous installation crashed and left the file locked."

### Why This Happens
1. **Permissions issue** - We weren't running PowerShell as Administrator
2. **Lock file corruption** - A previous installation attempt left behind a "lock" that prevented new installations
3. **Network drive limitations** - Working from a network drive can cause permission issues

## The Solution Process

### Step 1: Try to Clear the Lock File
```powershell
Remove-Item "C:\ProgramData\chocolatey\lib\18c1117b8756e7ae77eb1dd5672b8147e64edc92" -Force -ErrorAction SilentlyContinue
```

**What this does:** Deletes the problematic lock file
- `Remove-Item` = delete a file or folder
- `-Force` = don't ask for confirmation, just do it
- `-ErrorAction SilentlyContinue` = if the file doesn't exist, don't show an error

**Result:** This didn't work - the same error occurred when we tried to install again.

### Step 2: Alternative Download Method
Since Chocolatey wasn't working, we downloaded Caddy directly from its official source:

```powershell
Invoke-WebRequest -Uri "https://github.com/caddyserver/caddy/releases/download/v2.8.4/caddy_2.8.4_windows_amd64.zip" -OutFile "caddy.zip"
```

**What this does:**
- `Invoke-WebRequest` = download a file from the internet (like right-clicking and "Save As" in a browser)
- `-Uri` = the web address to download from
- `-OutFile` = what to name the downloaded file

**Why this version:** v2.8.4 is a stable release of Caddy for Windows 64-bit systems

### Step 3: Extract the Downloaded File
```powershell
Expand-Archive -Path "caddy.zip" -DestinationPath "." -Force
```

**What this does:**
- `Expand-Archive` = unzip a compressed file
- `-Path "caddy.zip"` = the file to unzip
- `-DestinationPath "."` = extract to current folder (the dot "." means "here")
- `-Force` = overwrite existing files if they exist

**Result:** This created a `caddy.exe` file in our project folder.

### Step 4: Clean Up
```powershell
Remove-Item "caddy.zip"
```
**What this does:** Deletes the zip file since we no longer need it after extraction.

### Step 5: Handle Network Drive Execution Issues
When we tried to run caddy.exe directly from the network drive, we got:
```
Access is denied
```

**Why this happens:** Windows often blocks execution of programs from network drives for security reasons.

**Solution:** Copy the executable to a local folder:
```powershell
Copy-Item "caddy.exe" "$env:TEMP\caddy.exe" -Force
```

**What this does:**
- `Copy-Item` = copy a file
- `"$env:TEMP\caddy.exe"` = copy to the Windows temporary folder (a local drive location)
- `$env:TEMP` is a Windows environment variable that points to your temp folder (usually C:\Users\YourName\AppData\Local\Temp)

### Step 6: Test the Installation
```powershell
& "$env:TEMP\caddy.exe" version
```

**What this does:**
- `&` = run a program
- The rest tells it to run the Caddy program and show its version number

### Step 7: Start the Web Server
```powershell
& "$env:TEMP\caddy.exe" run --config Caddyfile
```

**What this does:**
- Runs Caddy with our configuration file (Caddyfile)
- `--config Caddyfile` tells Caddy to use our specific settings

## Key Lessons for Beginners

### Understanding Error Messages
- **"Access denied"** usually means permissions issues
- **"Lock file"** errors mean something is blocking the installation
- **Network path errors** often occur when running programs from shared drives

### Command Line Basics
- **Paths with spaces** need quotes: `"C:\Program Files\Something"`
- **The dot (.)** means "current folder"
- **Environment variables** like `$env:TEMP` are shortcuts to common Windows folders
- **Flags/Parameters** starting with `-` modify how commands work

### Problem-Solving Strategy
1. **Try the official method first** (Chocolatey in this case)
2. **If that fails, find alternative methods** (direct download)
3. **Work around system limitations** (copy to local drive)
4. **Test each step** to make sure it worked
5. **Clean up temporary files** when done

### Network Drive Considerations
When working on network drives:
- Some operations require local execution
- Copy executables to local drives when needed
- Be aware of permission differences between local and network storage

## Final Result
✅ Caddy web server successfully installed and running
✅ Web server accessible at http://127.0.0.1:1234
✅ Configuration working with CORS headers for web mapping applications

## Commands to Remember
To start your web server in the future:
```powershell
& "$env:TEMP\caddy.exe" run --config Caddyfile
```

To stop the server: Press `Ctrl+C` in the terminal window where Caddy is running.