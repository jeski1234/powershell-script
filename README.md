Set-Location HKLM:
New-Item -Path \SOFTWARE\Policies\Microsoft -Name FVE
New-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\FVE -Name UseAdvancedStartup -Value 1 
New-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\FVE -Name EnableBDEWithNoTPM -Value 1 
Write-Host "Hey..!! I am here to compare the password you are entering..."

$drive = Read-Host "Enter drive to encrypt ('C:', 'D:', etc.): "
$pwd1 = Read-Host -AsSecureString "Passowrd Required for Startup"
$pwd2 = Read-Host -AsSecureString "Re-enter Passowrd Required for Startup"

$pwd1_text = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pwd1))
$pwd2_text = [Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pwd2))

if ($pwd1_text -ceq $pwd2_text){
    Write-Host 'Passwords matched, starting encryption.'
} else {
    # exit script with a message when passwords do not match
    Write-Host 'Passwords differ, stopping encryption'
    exit 
}

# $black_home - to remove output form CLI
$black_home = Enable-BitLocker -MountPoint $drive -EncryptionMethod Aes256 -UsedSpaceOnly -PasswordProtector -Password $pwd1 -SkipHardwareTest


# get initial starting point of encryption, since by the time you start showing progress some of disk will already be encrypted
$encryption_progress = (Get-BitLockerVolume -MountPoint $drive | select EncryptionPercentage).EncryptionPercentage

# use while instead of for, and no need for double progress IMO
while ($encryption_progress -ne 100) {
    Start-Sleep -Milliseconds 1
    Write-Progress -Id 1 -Activity "Encryption progress" -Status "Current Count: $encryption_progress" -PercentComplete $encryption_progress -CurrentOperation "Counting ..."

    # get updated progress value each cycle
    $encryption_progress = (Get-BitLockerVolume -MountPoint $drive | select EncryptionPercentage).EncryptionPercentage
}

# give some outpot about when the progres is finished
Write-Host 'Encryption of drive' $drive 'completed.'

# you can add prompt to "press return to continue..." so the CLI doesn't close right away
pause
