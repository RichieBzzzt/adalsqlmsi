# adalsqlmsi
adalsql msi

```powershell

Function Install-ADAuthenticationLibraryforSQLServer {
    # from https://bzzzt.io/post/2018-05-25-horrible-adalsql-issue/
    $workingFolder = Join-Path $env:temp ([System.IO.Path]::GetRandomFileName())
    New-Item -ItemType Directory -Force -Path $workingFolder
    $uri = ""


    $splitArray = $uri -split "/"
    $fileName = $splitArray[-1]

    $Installer = Join-Path -Path $WorkingFolder -ChildPath $fileName
    try {
        Invoke-TlsWebRequest -Uri $uri -OutFile $Installer
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
        Write-Host "oh dear install did not work"
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