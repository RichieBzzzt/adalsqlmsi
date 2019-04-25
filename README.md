# adalsqlmsi
adalsql msi

[Blog post](https://bzzzt.io/post/2019-03/2019-03-12-fixing-horrible-adalsql-issue-part-2/) that references back to this repo, but explains the purpose a bit better.

## tl;dr

If you have aladsqll issues, run script below to download an install msi.

```powershell

Function Install-ADAuthenticationLibraryforSQLServer {
    # from https://bzzzt.io/post/2019-03/2019-03-12-fixing-horrible-adalsql-issue-part-2/
    $workingFolder = Join-Path $env:temp ([System.IO.Path]::GetRandomFileName())
    New-Item -ItemType Directory -Force -Path $workingFolder
    $uri = "https://raw.githubusercontent.com/RichieBzzzt/adalsqlmsi/master/msi/adalsql.msi"


    $splitArray = $uri -split "/"
    $fileName = $splitArray[-1]

    $Installer = Join-Path -Path $WorkingFolder -ChildPath $fileName
    try {
        Invoke-WebRequest -Uri $uri -OutFile $Installer
    } catch {
        Throw $_.Exception
    }
    If (!(Test-Path $Installer)) {
        Throw "$Installer does not exist"
    }
    try {
        Write-Host "attempting to uninstall..."
        Write-Host "Running MsiExec.exe /uninstall {4EE99065-01C6-49DD-9EC6-E08AA5B13491} /quiet"
        Start-Process -FilePath "MsiExec.exe" -ArgumentList  "/uninstall {4EE99065-01C6-49DD-9EC6-E08AA5B13491} /quiet" -Wait -NoNewWindow
    } catch {
        Write-Host "oh dear uninstall did not work"
        $fail = $_.Exception
        Write-Error $fail
        Throw
    }
    try {
        $DataStamp = get-date -Format yyyyMMddTHHmmss
        $logFile = '{0}-{1}.log' -f $Installer, $DataStamp
        $MSIArguments = @(
            "/i"
            ('"{0}"' -f $Installer)
            "/qn"
            "/norestart"
            "/L*v"
            $logFile
        )
        Write-Host "Attempting to install.."
        Write-Host " Running msiexec.exe $($MSIArguments)"
        Start-Process "msiexec.exe" -ArgumentList $MSIArguments -Wait -NoNewWindow
    } catch {
        $fail = $_.Exception
        Write-Error $fail
        Throw
    }
}

```
