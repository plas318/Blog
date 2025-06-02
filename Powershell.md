List of commands, record for using powershell with PTC Windchill

## Basic Common Commands

Just some Powershell commands that might be usefull to know
```
ls / dir :: list
pwd ::  directory
New-Item (touch) :: Create
"Some text" > "text.txt" :: Save text
rm :: remove
alias, set-alias, get-alias :: use alias
sls (Select-String =.= grep)

clear :: clear screen

```
Powershell commands are in forms of cmdlets, and u can bind functions to support the same nature

## Piping

Able to pipe results sequentially, results are processed from left -> right

```
history | sls "rm"
```
history returns an object of all the previous commands, 
pipes the result to the Select-String cmdlet, which filters the results that include the string "rm"


## Boolean operators
Powershell uses
-and, -not, -or -eq, -lt -gt ... instead of &&, |, ! operators


## Looping

```
foreach ($a in $b) {
  do something
}

do {}
while ($a)

for ($i = 0; $i -lt 10; $i++) {
    # Code to be executed in each iteration
    Write-Host "Iteration number: $i"
}


etc..

```

## Switch
```
switch ($key){

"case1" {}

"case2" {}

default {

}
}

```

## Hashmap
```
$hash = @{}

$hash.GetEnumerator() | foreach {
    write "$($_.Key) $($_.Value)"
}
# $_.Key returns the collections object generator

$hash["key"]="value" # init value
foreach ($keys in $hash.Keys){
$value = $hash[keys] # retrieve values
# do something
}


```
## Array
```
$arr = @("a", "b", "cd")
parameter(
[string[]]$Lines
)
foreach ($_ in $arr) {
do something
}

```

## Match with Regex
the match cmdlet allows to match a keyword or a regex
Select-String is case sensitive, where match is not.

reference :: [https://www.pdq.com/blog/how-to-use-regular-expression-in-powershell/#regex-is-powerful-but-complex](https://www.pdq.com/blog/how-to-use-regular-expression-in-powershell/#regex-is-powerful-but-complex)

```
"turtle" -match "rt"  # True
"010-4123-1234" -match '\d{3}-\d{4}-\d{4}' # True

Use match to find substrings and such..
"[Epik] One" -match "\[\w+\]" # to parse [Epik]

Access the matched values via > $matches[0]

write-out "found $matches[0]"

```


## NewLine
powershell uses (tilda) \`n or \`r for newline instead of \n



## Function
```
# Definition
function My-Function{
param(
[string]$ID = "32"
)
write-output "$ID"
}

# Call
My-Function -ID "44"

```

#### Note 
powershell functions are seperated using spaces
```
MyFunction -id "44"
```

#### Im cmd api calls are not
```
im issues --query="Q1" --sortField="modified date" --field="somefield"="tbd"
```

## & operator
to call a cmdlet function inside a function
use the & operator
& im viewissue 1
would call the im issue command.
