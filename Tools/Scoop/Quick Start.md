Ross Smith II edited this page on 2 Sep 2019 · [14 revisions](https://github.com/lukesampson/scoop/wiki/Quick-Start/_history)

https://github.com/lukesampson/scoop/wiki/Quick-Start

### Prerequisites

To get to the PowerShell prompt

- "Start" --> (Search) "cmd"
- Terminal window should appear
- "powershell"
- Prompt should now start with "PS "

Make sure you have **PowerShell 5.0** or later installed. If you're on *Windows 10* or *Windows Server 2012* you should be all set, but *Windows 7* and *Windows Server 2008* might have older versions.

```
$psversiontable.psversion.major # should be >= 5.0
```

Make sure you have allowed PowerShell to execute local scripts:

```
set-executionpolicy remotesigned -scope currentuser
```

`Unrestricted` will work too, but it is less secure. So stick with `RemoteSigned` if you're not sure.

### Installing Scoop

In a PowerShell command console, run:

```
Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://get.scoop.sh')
```

or the shorter:

```
iwr -useb get.scoop.sh | iex
```

### Installing Scoop to Custom Directory

Assuming the target directory is `C:\scoop`, in a PowerShell command console, run:

```
$env:SCOOP='C:\scoop'
[environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')
iwr -useb get.scoop.sh | iex
```

Assuming you didn't see any error messages, Scoop is now ready to run.

### Installing global apps to custom directory

Assuming the target directory is `C:\apps`, in a admin-enabled PowerShell command console, run:

```
$env:SCOOP_GLOBAL='c:\apps'
[environment]::setEnvironmentVariable('SCOOP_GLOBAL',$env:SCOOP_GLOBAL,'Machine')
scoop install -g <app>
```

安装结果：

```shell
Initializing...
Downloading scoop...
Extracting...
Creating shim...
Downloading main bucket...
Extracting...
Adding D:\Scoop\shims to your path.
'lastupdate' has been set to '2020-12-13T19:57:34.4114960+08:00'
```





### Using Scoop

Although Scoop is written in PowerShell, its interface is closer to Git and Mercurial than it is to most PowerShell programs.

To get an overview of Scoop's interface, run:

```
scoop help
```

You'll see a list of commands with a brief summary of what each command does. For more detailed information on a command, run `scoop help <command>`, e.g. `scoop help install` (try it!).

Now that you have a rough idea of how Scoop commands work, let's try installing something.

```
scoop install curl
```

You'll probably see a warning about a missing hash, but you should see a final message that cURL was installed successfully. Try running it:

```
curl -L https://get.scoop.sh
```

You should see some HTML, probably with a 'document moved' message. Note that, like when you installed Scoop, you didn't need to restart your console for the program to work. Also, if you've installed cURL manually before you might have noticed that you didn't get an error about SSL—Scoop downloaded a certificate bundle for you.

##### Finding apps

Let's say you want to install the `ssh` command but you're not sure where to find it. Try running:

```
scoop search ssh
```

You'll should see a result for 'openssh'. This is an easy case because the name of the app contains 'ssh'.

You can also find apps by the name of the commands they install. For example,

```
scoop search hg
```

This shows you that the 'mercurial' app includes 'hg.exe'.

### Updating Scoop

To get the latest version of Scoop you have to run the command

```
scoop update
```

This will download the latest version of scoop and updates the local app manifests.

After you updated Scoop you can update individual apps

```
scoop update curl
```

If you want to update all your installed apps, you can run

```
scoop update *
```



- # 