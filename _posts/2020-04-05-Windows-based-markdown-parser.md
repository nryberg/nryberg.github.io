---
title: "Powershell: Windows Marked File Watcher"
categories:
  - Blog
tags:
  - Powershell
  - Markdown
---

I love [Brett Terpstra's](https://brettterpstra.com/) [Marked2 App](https://marked2app.com/).  It's fantastic, it's fast, it gives immediate feedback and while not free, it's absolutely worth the cost. 

Unfortunately, it's Mac Only.  :(

For those of us Windows bound, there's a Powershell method for watching files and publishing the results in HTML format.  But if you need that quick feedback, on Windows, there's a solution.

Couple of caveats:

* It uses the Github API to get the HTML
	- Good : because there's no problems parsing
	- Bad : because there's no theming yet.
* It requires installing the Convert_From_Markdown module 


### Watch the folder 
First - you have to watch the folder for changes - [Stackoverflow Article](https://stackoverflow.com/a/29067433/21275)

### Convert the files 
Second - you have to convert the changed files into HMTL. - [Craig Forrester](https://www.craigforrester.com/) and available on his [Github page](https://github.com/craigforr/ConvertFrom-Markdown).

### Combined Code
[Github](https://gist.github.com/nryberg/cb1f00067129fded3103f1b737fcb70d)

```powershell
Import-Module Convert_From_Markdown

$folder = "<enter your source folder here>"

$results = "<enter your output publish folder here>"

# Clean out existing subscriptions
foreach($sub in Get-EventSubscriber) {
    
    unregister-event -subscriptionid $sub.SubscriptionId
    }

# Get ready to register a new one
Function Register-Watcher {
    param ($folder)
    $filter = "*.MD" #all files
    $watcher = New-Object IO.FileSystemWatcher $folder, $filter -Property @{ 
        IncludeSubdirectories = $false
        EnableRaisingEvents = $true
    }

    $changeAction = [scriptblock]::Create('
        # This is the code which will be executed every time a file change is detected
		
        $path = $Event.SourceEventArgs.FullPath
        $name = $Event.SourceEventArgs.Name
        $changeType = $Event.SourceEventArgs.ChangeType
        $timeStamp = $Event.TimeGenerated
        Write-Host "The file $name was $changeType at $timeStamp"
        $outpath = $results + $name.TrimEnd(".md") + ".html"
        gc $path -Raw | ConvertFrom-Markdown | Out-File -FilePath "$outpath"
    ')
    Register-ObjectEvent $Watcher -EventName "Changed" -Action $changeAction
}

# Use the new function 
Register-Watcher $folder
```
