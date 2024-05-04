# Introduction to Bash Scripting

## Bourne Again Shell

`bash` is the scripting language we use to communicate with unix-based OS and give commands to the system   
since may 2019 windows gives windows subsystem for linux that allows us to use bash in windows   
scripting vs programming = don't need to compile code to execute a scripting language   

scripting language structure: 
- I/O
- arguments, variables, and arrays
- conditionals 
- arithmetic
- loops 
- comparison operators
- functions 

to execute a script we need to specify the interpreter and tell it which script it should process: 

```bash
bash script.sh <optional arguments>
```

```bash
sh script.sh <optional arguments>
```

```bash
./script.sh <optional arguments>
```

below is a sample bash script: 

```bash
#!/bin/bash

# Check for given arguments
if [ $# -eq 0 ]
then
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
else
	domain=$1
fi

# Identify Network range for the specified IP address(es)
function network_range {
	for ip in $ipaddr
	do
		netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
		cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
		cidr_ips=$(prips $cidr)
		echo -e "\nNetRange for $ip:"
		echo -e "$netrange"
	done
}

# Ping discovered IP address(es)
function ping_host {
	hosts_up=0
	hosts_total=0
	
	echo -e "\nPinging host(s):"
	for host in $cidr_ips
	do
		stat=1
		while [ $stat -eq 1 ]
		do
			ping -c 2 $host > /dev/null 2>&1
			if [ $? -eq 0 ]
			then
				echo "$host is up."
				((stat--))
				((hosts_up++))
				((hosts_total++))
			else
				echo "$host is down."
				((stat--))
				((hosts_total++))
			fi
		done
	done
	
	echo -e "\n$hosts_up out of $hosts_total hosts are up."
}

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

echo -e "Discovered IP address:\n$hosts\n"
ipaddr=$(host $domain | grep "has address" | cut -d" " -f4 | tr "\n" " ")

# Available options
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
	"1") network_range ;;
	"2") ping_host ;;
	"3") network_range && ping_host ;;
	"*") exit 0 ;;
esac
```

some of the functionalities of this script are:   
- check for given arguments
- identify network range for specified IP addresses 
- ping discovered IP addresses 
- identify IP addresses of the specified domain 
- available options 

## Conditional Execution 

looking at the first section of our previous example: 

```bash
#!/bin/bash

# Check for a given argument
if [ $# -eq 0 ]
then 
	echo -e "You need to specify the target domain.\n"
	echo -e "Usage:"
	echo -e "\t$0 <domain>"
	exit 1
else 
	domain=$1
fi
```

the above works with these components: 
- `#!/bin/bash` - shebang
- `if-else-fi` - conditional
- `echo` - print output
- `$#`, `$0`, `$1` - special variables
- `domain` - variables 

the conditional we use will compare the values of variables using the equals comparison operator `-eq` 

### Shebang

the shebang is always at the top of the file and starts with `#!` followed by the path to the specified interpreter `/bin/bash` with which the script will be executed   
we can also use other interpreters like python, perl, and others: 

```python
#!/usr/bin/env python
```

```perl
#!/usr/bin/env perl
```

### If-else-fi

in pseudo-code, our example conditional looks like:   

```
if [ the number of given arguments equals 0 ]
then 
	print you need to specify target domain
	print empty line 
	print Usage: 
	print <name of script> <domain>
	exit script with an error
else
	the "domain" variable will be the alias for the given argument 
fi = finish the if-conditional 
```

by default, an if-else can only contain a single `if`: 

```bash
#!/bin/bash 

value=$1

if [ $value -gt "10" ]
then 
	echo "the argument is greater than 10"
fi
```

```bash
bash if-only.sh 5
```

```bash
bash if-only.sh 12
```

we can use `elif` or `else` to add alternatives to our conditional: 

```bash 
#!/bin/bash 

value=$1

if [ $value -gt "10" ]
then
	echo "argument is greater than 10"
elif [ $value -lt "10" ]
then 
	echo "argument is less than 10"
else
	echo "argument is not a number"
fi
```

we could extend our script to look for specific numbers of arguments: 

```bash 
if [ $# -eq 0 ]
then 
	... 
elif [ $# -eq 1 ]
then 
	domain=$1
else
	...
fi
```

exercise script: 

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -c

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
    var=$(echo $var | base64)
done
```

create an if-else condition in the for loop that prints the number of characters of the 35th generated value of the variable var: 

```bash
#!/bin/bash
# Count number of characters in a variable:
#     echo $variable | wc -c

# Variable to encode
var="nef892na9s1p9asn2aJs71nIsm"

for counter in {1..40}
do
	var=$(echo $var | base64)
	
	if [ $counter -eq 35 ]
	then 
		echo $(echo $var | wc -c)
	fi
done
```

## Arguments, Variables, and Arrays

in bash we can always pass up to 9 arguments `$0`-`$9` to the script without assigning them to variables or setting the corresponding requirements for these   
9 arguments because `$0` is reserved for the script itself: 

```bash
./script.sh ARG1 ARG2 ARG3 ... ARG9
$0           $1   $2   $3       $9
```

these variables are called `special variables` 

make sure to set the scripts execution privileges: 

```bash
chmod +x cidr.sh
```

### Special variables

special variables use the internal field separator (IFS) to identify when an argument ends and the next begins   

some special variables are: 
- `$#` - holds the number of arguments passed into the script
- `$@` - retrieve the list of cli arguments
- `$n` - each cli argument can be retrieved using its position, ex: first argument is `$1`
- `$$` - process ID of the currently running process
- `$?` - exit status of the script, useful to determine a command's success; 0 = success and 1 = fail 

### Variables

we can see in our script that we assign the value of the first special variable to a new variable called `domain`   
assign a variable without `$`: 

```bash
domain=$1
```

the `$` is only intended to allow the variable's value to be used   
remember that there can't be spaces when assigning variables 

there are no types in bash like other languages, all contents of variables are treated as strings   
however, bash does allow arithmetic operations if only numbers are assigned   

### Arrays

arrays can store an ordered sequence of specific type values   
start at 0 like other langs  
use `()` to define array: 

```bash
#!/bin/bash

domains=(www.inlanefreight.com ftp.inlanefreight.com ...)

echo ${domains[0]}
```

note that we can use `''` or `""` to assign values as well if we need to use spaces   
also see above that we use `{}` to get the value of an indexed element  

if we have arrays.sh: 

```bash
#!/bin/bash

domains=("www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com" www2.inlanefreight.com)
echo ${domains[0]}
```

this would print out `www.inlanefreight.com ftp.inlanefreight.com vpn.inlanefreight.com` because the quotes will make it all one element, so we could then use `echo ${domain[1]}` to print out the next element 

## Comparison Operators

there are different types of comparison operators: 
- strings
- integers
- files
- booleans

### String operators

if we compare strings then we can use: 
- `==`
- `! =`
- `<`
- `>`
- `-z` = if the string is null
- `-n` = if the string is not null

in this sample script we put the argument variable `$1` in quotes to tell bash that its contents should be handled as a string, otherwise it would throw an error: 

```bash
#!/bin/bash

if [ "$1" != "HackTheBox" ]
then
	echo -e "You need to give HackTheBox argument"
	exit =
elif [ $# -gt 1 ]
then 
	echo -e "Too many arguments"
	exit 1
else
	domain=$1
	echo -e "Success!"
fi
```

string comparison operators `<` or `>` only work within double square brackets `[[ ]]`, and they go by ascii values  

### Integer operators

- `-eq`
- `-ne` - not equal
- `lt`
- `le` - less than or equal to 
- `gt`
- `ge`

```bash
#!/bin/bash

if [ $# -lt 1 ]
then
	echo -e "Less than 1 args"
	exit 1
elif [ $# -gt 1 ]
then 
	echo -e "Greater than 1 args"
	exit 1
else 
	domain=$1
	echo -e "1 arg"
fi
```

### File operators

file operators are useful to find out specific permissions or if they exist: 
- `-e` - if the file exists
- `-f` - tests if it is a file 
- `-d` - tests if it is a directory 
- `-L` - test if it is a symbolic link 
- `-N` - if was modified after it was last read 
- `-O` - if current user owns the file 
- `-G` - if the file's group ID matches the current user's 
- `-s` - if size is greater than 0 
- `-r` - if it has read permission
- `-w` - write permission
- `-x` - execute permission

```bash
#!/bin/bash

if [ -e "$1" ]
then 
	echo -e "The file exists"
	exit 0
else 
	echo -e "File does not exist"
	exit 2
fi
```

### Boolean and logical operators 

boolean values are `false` or `true` 

```bash
#!/bin/bash

# check if the string is null 
if [[ -z $1 ]]
then 
	echo -e "String is null"
	exit 1
elif [[ $# > 1 ]]
then 
	echo -e "Number of arguments is greater than 1"
	exit 1
else 
	domain=$1
	echo -e "1 argument passed"
fi
```

### Logical operators

- `!` - NOT
- `&&` - AND
- `||` - OR 

```bash
#!/bin/bash

if [[ -e "$1" && -r "$1" ]]
then 
	echo -e "we can read the specified file"
	exit 0
elif [[ ! -e "$1" ]]
then 
	echo -e "Specified file does not exist"
elif [[ -e "$1" && ! -r "$1" ]]
then 
	echo -e "we don't have read permission for this file"
	exit 1 
else 
	echo "error"
	exit 5
fi
```

exercise script: 

```bash
#!/bin/bash

var="8dm7KsjU28B7v621Jls"
value="ERmFRMVZ0U2paTlJYTkxDZz09Cg"

for i in {1..40}
do
	var=$(echo $var | base64)
	
	#<---- If condition here:
done
```

create a conditional in the for loop that checks if var contains the contents of the value variable   
also the variable var must have more than 113,450 characters   
if these conditions are met then print the last 20 characters of the variable var 

```bash
#!/bin/bash

var="8dm7KsjU28B7v621Jls"
value="ERmFRMVZ0U2paTlJYTkxDZz09Cg"

for i in {1..40}
do
	var=$(echo $var | base64)
	
	if [ $(echo $var | wc -c) -gt 113450 ] && [[ $var == *"$value"* ]]
	then 
		echo $(echo $var | tail -c 20)
	fi
done
```

notice in the above that we combine `[[]]` and `[]` using logical operators, we use `$var == *"$value"*` to check if a string exists within another one, and we use `tail -c 20` to get the last 20 characters of the output 

## Arithmetic 

arithmetic operators: 
- `+`
- `-`
- `*`
- `/`
- `%`
- `i++`
- `i--`

```bash
#!/bin/bash

increase=1
decrease=2

echo "10 + 10 = $((10 + 10))"
echo "10 - 10 = $((10 - 10))"
echo "10 * 10 = $((10 * 10))"
echo "10 / 10 = $((10 / 10))"
echo "10 % 10 = $((10 % 10))"

((increase++))
((decrease--))
```

we can also calculate the length of a variable using `${#variable}`

## Input and Output

we might get results from any sent requests and executed commands, which we have to decide manually how to proceed  
we need a way to get a running script to wait for our instructions   

```bash
# Available options
<SNIP>
echo -e "Additional options available:"
echo -e "\t1) Identify the corresponding network range of target domain."
echo -e "\t2) Ping discovered hosts."
echo -e "\t3) All checks."
echo -e "\t*) Exit.\n"

read -p "Select your option: " opt

case $opt in
	"1") network_range ;;
	"2") ping_host ;;
	"3") network_range && ping_host ;;
	"*") exit 0 ;;
esac
```

we can see that with `read -p` we get a line of input   

for output we can avoid waiting for our scripts results by using `tee`   
this will ensure that we get our results immediately and that they are stored in the corresponding files  

```bash
<SNIP>

# Identify Network range for the specified IP address(es)
function network_range {
	for ip in $ipaddr
	do
		netrange=$(whois $ip | grep "NetRange\|CIDR" | tee -a CIDR.txt)
		cidr=$(whois $ip | grep "CIDR" | awk '{print $2}')
		cidr_ips=$(prips $cidr)
		echo -e "\nNetRange for $ip:"
		echo -e "$netrange"
	done
}

<SNIP>

# Identify IP address of the specified domain
hosts=$(host $domain | grep "has address" | cut -d" " -f4 | tee discovered_hosts.txt)

<SNIP>
```

we use `tee -a CIDR.txt` to ensure that the file is appended 