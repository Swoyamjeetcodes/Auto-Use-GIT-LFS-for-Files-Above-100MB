# Git LFS Pre-Commit Hook for Files ≥ 100 MB

This guide provides step-by-step instructions to create a Git pre-commit hook that automatically tracks files larger than or equal to 100 MB using Git LFS. The script is written in PowerShell for use on Windows systems.

---

## Prerequisites

1. **Git** must be installed and added to your PATH.
2. **Git LFS** must be installed. You can install it with:
   ```bash
   git lfs install
   ```
3. **PowerShell** is required to run the hook.

---

## Steps to Set Up the Pre-Commit Hook

### 1. Navigate to the `.git/hooks` Directory
Go to the `.git/hooks` directory in your repository:
```powershell
cd .git/hooks
```

### 2. Create the `pre-commit` Script
Create a file named `pre-commit`:
```powershell
New-Item pre-commit -ItemType File
```

Open it for editing:
```powershell
notepad pre-commit
```

Add the following content to invoke the PowerShell script:
```bash
#!/bin/sh
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .git/hooks/pre-commit.ps1
```

Save and close the file.

### 3. Create the PowerShell Script
Create another file named `pre-commit.ps1` in the same `.git/hooks` directory:
```powershell
New-Item pre-commit.ps1 -ItemType File
```

Open it for editing:
```powershell
notepad pre-commit.ps1
```

Paste the following PowerShell script into the file:

```powershell
# PowerShell script for Git LFS pre-commit hook
# This script checks staged files and adds files >= 100 MB to Git LFS.

# Size threshold (100 MB in bytes)
$sizeThreshold = 100MB

# Ensure Git command is available
function Check-GitCommand {
    if (-not (Get-Command git -ErrorAction SilentlyContinue)) {
        Write-Output "Error: Git is not installed or not in PATH." | Out-Host
        exit 1
    }
}

Check-GitCommand

# Get a list of staged files
$stagedFiles = git diff --cached --name-only | ForEach-Object { $_.Trim() }

foreach ($file in $stagedFiles) {
    if (Test-Path $file) {
        # Get file size
        $fileSize = (Get-Item $file).Length
        if ($fileSize -ge $sizeThreshold) {
            Write-Output "$file is $($fileSize / 1MB) MB. Adding to Git LFS." | Out-Host
            # Track the file with Git LFS
            git lfs track $file
            # Re-add the file after LFS tracking
            git add $file
        }
    }
}

Write-Output "Pre-commit hook executed successfully." | Out-Host

# Exit successfully
exit 0
```

Save and close the file.

### 4. Make the Hook Executable
Unblock the script to ensure it can run:
```powershell
Unblock-File -Path .git/hooks/pre-commit
Unblock-File -Path .git/hooks/pre-commit.ps1
```

---

## Testing the Pre-Commit Hook

1. **Stage a Large File**  
   Add a file larger than or equal to 100 MB:
   ```powershell
   git add <large-file>
   ```

2. **Commit the Changes**  
   Commit the file to test the hook:
   ```powershell
   git commit -m "Testing LFS hook"
   ```

3. **Verify LFS Tracking**  
   Check if the file is being tracked by LFS:
   ```powershell
   git lfs ls-files
   ```

---

## Notes

- This script automatically tracks files ≥ 100 MB using Git LFS during the commit process.
- If you modify the threshold, update the `$sizeThreshold` variable in the `pre-commit.ps1` script.
- The hook works on Windows PowerShell and Git for Windows.

---

## License
This script is open-source. Feel free to use and modify it as needed.

--- 
