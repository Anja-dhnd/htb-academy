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