---
layout: post
title: Yubikey Authentication with Native Windows SSH Client
tags: 
- Yubikey
- SSH
---

Below are instructions how to get a Yubikey to act as the authentication for the native Windows SSH client. This documentation assumes your keys are already on your Yubikey. If not, check out the [Yubikey Guide](https://github.com/drduh/YubiKey-Guide). 

This code was run on Windows 11.

## Pre-requisites

1. Install [gpg4win]. Make sure you have installed at least version 4.2.0, which includes gpg support for native Windows SSH authetication.

```pwsh
gpg --version
```
The above shoudl output gpg (GnuPG) version 2.4.3 or newer.

2. Make sure you do not have the OpenSSH capability installed. Run the following command in an elevated prompt.

```pwsh
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

All items should return a state of "not present". If they exist, make sure to uninstall them with the command below.

```pwsh
Remove-WindowsCapability -Name OpenSSH.Client~~~~0.0.1.0 -Online
```

In this case, the capability that was present was named "OpenSSH.Client~~~~0.0.1.0". If your capability is named differently, set the name value appropriately.

3. Install the [Win32 OpenSSH client](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH-Using-MSI)

Once installed, make sure the ssh-agent service isn't running. It will interfer with the gpg authentication. Run the command below in an elevated prompt.

```pwsh
 Get-Service ssh
```

If it's running, open services.msc, stop the service, and disable it. In fact, if you accidentally installed sshd, which will show up with the same command above (I did that on purpose), and it's running, disable it also. 

These services appear as "OpenSSH Authentication Agent" and "OpenSSH SSH Server" when in services.msc

At this point, almost everything is setup correctly. It's just time to configure gpg to support the native Windows SSH client and gpg to accept connections from the ssh authentication agent to verify your keys.

## Configure

pgp4win typically installs it's configuration information at %APPDATA%/gnupg. Open this folder and create a gpg-agent.conf file if you don't already have one.

Add the following to the gpg-agent.conf file

```
debug-level 4
log-file C:\Users\<username>\AppData\Roaming\gnupg\gpg-agent.log
enable-ssh-support
enable-putty-support
enable-win32-openssh-support
use-standard-socket
```

This will enable a number of things above that are relatively self explanatory. It also enables debug logging and saves the log to the path noted above. &lt;name> should be replaced with your user name. Once you have everything confirmed working, you can comment out the debug-level and log-file. No point in generating debugging information.

In the same gnupg directory, run the following command:

```
gpg -K --with-keygrip
```

This will output your keys. You need to add the keygrip value for your authentication key to a file in %APPDATA%/gnupg/sshcontrol. The file likely already exists, but create it if not.

Below is an example of some of the output from the gpg command. You want to find the key identified by [A] (for authentication) and copy/paste the Keygrip, represented by the X's, into your sshcontrol file. Make sure to put a new line at the end of the file and save it.
```
ssb>  rsa4096 2022-02-21 [A] [expires: 2024-02-01]
      Keygrip = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

Set the SSH_AUTH_SOCK environment variable to the value shown below.

```pwsh
$env:SSH_AUTH_SOCK="\\.\pipe\openssh-ssh-agent"
```

It's a good idea to set this permanently as an environment variable once you have everything working. For now, set it in PowerShell.

Now, restart the gpg agent in a non-priviged terminal - preferably the one you just set the SSH_AUTH_SOCK variable with.

```pwsh
gpg-connect-agent killagent /bye
gpg-connect-agent /bye
```

Test to see if you can see your authentication key on the Yubikey from the ssh command below.

```
ssh-add -L
```

Try to connect to an ssh server and authenticate. 

If you can authenticate, turn off the debug logging in your gpg-agent.conf file, restart the agent with the gpg-connect-agent killagent commands listed above, and enjoy.

If not, review your log file and take a read through the[Support GPG and smartcard users](https://github.com/PowerShell/Win32-OpenSSH/issues/827) GitHub issue. It will likely have your issue.
