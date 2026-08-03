# Webshell / Command Execution Detection

## Purpose
Detect evidence of post-exploitation command execution via an uploaded web shell.

## Explanation
Requests containing command-style parameters, combined with references to unusual uploaded filenames, indicate the attacker may have deployed and interacted with a web shell to execute commands on the compromised server.

## Expected Output
A table of timestamped requests matching webshell-style paths/parameters, their user agent, and HTTP status.

## MITRE Mapping
T1505.003 – Server Software Component: Web Shell; T1059 – Command and Scripting Interpreter.

## SPL Query
```spl
sourcetype=web_traffic
client_ip="198.51.100.55"
AND path IN ("*bunnylock.bin*","*shell.php?cmd=*")
| table _time,path,user_agent,status
```
