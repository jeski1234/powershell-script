New-Item -Path HKLM:\Software\Policies\Microsoft -Name FVE 
New-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\FVE -Name UseAdvancedStartup -Value 1
New-ItemProperty -Path HKLM:\SOFTWARE\Policies\Microsoft\FVE -Name EnableBDEwithNoTPM -Value 1


$password1 = Read-Host -ConvertTo-SecureString -AsPlainText "Determine your startup password"
$password2 = Read-Host -ConvertTo-SecureString -AsPlainText "For security, re-enter your password"









if ($password1 -ceq $password2){
    Write-Host 'Your passwords have matched, initiating encryption'
    Enable-BitLocker -MountPoint "C:" -EncryptionMethod Aes256 -UsedSpaceOnly -PasswordProtector -Password $password1 -SkipHardwareTest


#this will exit the script if the passwords dont match.
} else {
    Write-Host 'Your passwords did not match. Encryption denied, please try again.'
    exit
} 
