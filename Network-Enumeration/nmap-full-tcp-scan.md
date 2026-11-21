
# Nmap Full TCP Port Enumeration

## The command

```bash
nmap -sC -sV -p- -T4 TARGET
```

## What does it do?

This command scans all TCP ports on the target, identifies running services and runs Nmap's default scripts.

## Understanding the flags

| Flag | Purpose |
|---|---|
| `-sC` | Runs default NSE scripts |
| `-sV` | Detects service versions |
| `-p-` | Scans TCP ports 1-65535 |
| `-T4` | Uses a faster timing template |

## Why I use it

I use a full TCP port scan during initial enumeration because checking only common ports can leave services undiscovered.

## Important limitations

- `-p-` scans TCP ports, not all UDP ports.
- `-T4` can be less reliable on unstable networks.
- Host discovery can affect whether Nmap scans a target.
- Service detection is useful, but its results should be verified.

## Things worth investigating

- How does Nmap perform service detection?
- What is the difference between TCP SYN and TCP connect scans?
- When should I perform UDP enumeration?
- What do `-oN`, `-oX` and `-oA` do?

## My observations
 According to me this is the most efficient cmd we can run while solving labs bcz this scan whole 65535 ports + service detection + version detection. 
 It cover all those parameters which are reqd for further investigation. It uses -T4 which makes the scan slightly fast. Rest depends on the internet 
 connection speed. 
