## Creating PTC Windchill powershell jobs / utilities
This is a post to record making simple utility tools to help automation of ptc windhcill tasks




## Objective
Run a task that queries a certain ptc query at some random interval, get a list of issues and progress the issue based on different criterias automatically.

Used a while loop to run the task infinitely with some random sleep time
Used a simple stack(array) instead of a database to record the completed issues


## Functions

### Main loop
```
  while(1){
      Compare-AndAlert $config $option
      Start-Sleep -Seconds (Get-Random -Minimum 20 -Maximum 200)
  }
```

### Config file
```
$jsonContent = Get-Content -Path "config.json" | ConvertFrom-Json
$config = @{}
$jsonContent.PSObject.Properties | ForEach-Object {
    $config[$_.Name] = $_.Value
}
```
Used json file to manage different cases

### Queries

Save the queried result (txt)
Use -match and (regex) $_.Contains("") to match criteria

#### IM queries
```
$res = & im issues --query="" --sortField="modified date" | Select-Object -First 5 | Out-String
$text = im viewissue $at
```
####  Format text (Split)
```
$type = ($type -split "type: ")[-1]
$summary.Contains((($project -split "/")[-1] -split "_")[-1])
```

####  Match using Contains
```
if ($summary.Contains("SUP8")){
#do SUP8
}
```

#### Match using regex
```
$permitted_types = @("Modify", "Change")
if ($type -in $permitted_types -and ($summary -match '\[\w+\]')){
    $stage = $Matches[0]
}
```


### Toast
-Notify new issues using Windows.UI.Notifications.ToastNotificationManager


## Conclusion
Probably a better alternative is to actually run an AI bot to completely manage and deploy,
but there are security issues for now.

Powershell doesn't require third party apps, additional installation on a windows operating system so it is conveniant to script simple tools
without additional frameworks

also has various support with the .net framework and powershell modules such as windows excel and stuff to write reports etc..
