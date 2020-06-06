---
title:  "Massive Hosts Discovery with SaturnV"
date:  2020-05-01 15:04:23
categories:  [NetworkPT]
tags:  [PenTest,Network,Nmap,Masscan,Discovery,Recon]
---

_SaturnV provides a fast deployable distributed port scanner and information collector infrastructure. This software was developed to provide a lightweight tool to pentesters who need to perform sporadic Network PenTest activities on big ranges of public faced IP subnets._

## Main Features
The idea behind SaturnV is simple: I need that my PC uses remote bots to perform port scans and discovery activities in a distributed way. Once bots have finished the scans I just need to grab the results on my local machine, parse them and create a "starting report" that could be useful for further manual analysis.
- Different scanning approach: since we have to optimize time, masscan is used to perform a SYN scan on all ports of every host. The results provided by masscan will be used as input for nmap and other tools that will provide more information about services (like banners or versions).
- Asynchronous operations: bots are autonomous and work without staying connected to any sort of master host. So I can use this software on my PC to start the scan on the bots and then I can turn it off, no data will be lost.
- Customizable: through the configuration file it is possible to tune your scan parameter and the tools command line. 

![](/images/2020-05-01-Massive-Hosts-Discovery-with-SaturnV/architecture.png)


## Execution Example

In order to provide a small DEMO of this tool I have used some facebook related ip subnets as targets since they are under bug bounty. The following steps will cover all the installation and execution process of the SaturnV tool providing a basic example of its behaviour.

#### 1. Clone the project from github
```
git clone https://github.com/c0mix/SaturnV.git
Cloning into 'SaturnV'...
remote: Enumerating objects: 32, done.
remote: Counting objects: 100% (32/32), done.
remote: Compressing objects: 100% (25/25), done.
remote: Total 32 (delta 10), reused 25 (delta 4), pack-reused 0
Unpacking objects: 100% (32/32), done.
```

#### 2. Install the requirement on attacker's PC
```
cd SaturnV/
sudo pip3 install -r requirements.txt 
Collecting aiocontextvars==0.2.2 (from -r requirements.txt (line 1))
Downloading https://files.pythonhosted.org/packages/db/c1/7a723e8d988de0a2e623927396e54b6831b68cb80dce468c945b849a9385/aiocontextvars-0.2.2-py2.py3-none-any.whl
Collecting bcrypt==3.1.7 (from -r requirements.txt (line 2))
Downloading https://files.pythonhosted.org/packages/8b/1d/82826443777dd4a624e38a08957b975e75df859b381ae302cfd7a30783ed/bcrypt-3.1.7-cp34-abi3-manylinux1_x86_64.whl (56kB)
100% |████████████████████████████████| 61kB 3.0MB/s 
[ . . . ]
Successfully installed PyNaCl-1.3.0 aiocontextvars-0.2.2 bcrypt-3.1.7 certifi-2020.4.5.1 cffi-1.14.0 contextvars-2.4 cryptography-2.8 idna-2.9 immutables-0.11 loguru-0.4.1 paramiko-2.7.1 pyOpenSSL-19.1.0 pycparser-2.19 python-libnmap-0.7.0 pyxattr-0.7.1 requests-2.23.0 scp-0.13.2 six-1.14.0 urllib3-1.25.9
python3 saturnV.py
***** Welcome to SaturnV *****
14:37:03 | ERROR | No argument provided!
usage: saturnV.py [-h] [-t TARGET] [-k] [-s] [-mS] [-nS] [-aS] [-gS] [-mR]
[-nR] [-aR] [-gR] [-g] [-c] [-o] [-r] [-v]
Boost your Network Discovery & Recon activity.
optional arguments:
-h, --helpshow this help message and exit
-t TARGET, --target TARGET
Takes as input a multi format target list and produces
the original_subnets.txt file
-k, --ssh-key Deploy an ssh key on bots
-s, --setupInstall all the required tools and create folder
structure on bots
-mS, --masscan-script
Create the Masscan script
-nS, --nmap-scriptCreate the Nmap script (Masscan results needed!)
-aS, --amass-scriptCreate the Amass script (Masscan results needed!)
-gS, --gobuster-script
Create the Gobuster script (Masscan results needed!)
-mR, --masscan-runSplit and run Masscan script on bots
-nR, --nmap-runSplit and run Nmap script on bots
-aR, --amass-runSplit and run Amass script on bots
-gR, --gobuster-runSplit and run Gobuster script on bots
-g, --get-results Collect outputs from all bots
-c, --check-scanCheck scan progress on each bot
-o, --osintPerform OSINT activity (Grab info from SSL certs,
HackerTarget and Bing Dork)
-r, --reportCreate the final report (at least Masscan results
needed!)
-v, --verbose Increase output verbosity
```

#### 3. Edit the configuration file with your parameters (at least `bots` and `npt_name` must be set)

	![](/images/2020-05-01-Massive-Hosts-Discovery-with-SaturnV/config.png)


#### 4. Setup your SSH Key (in this case I'm using AWS bots so I've put my private key inside ssh_key folder)
```
ls -l ssh_key/
total 4
-r-------- 1 ubuntu ubuntu 1692 May9 14:27 lcomi.pem
```

#### 5. Setup bots
```
python3 saturnV.py --setup
***** Welcome to SaturnV *****
14:46:42 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
14:46:44 | INFO | Command: mkdir ~/saturnV Executed on Bot: 3.12.83.119
14:46:44 | INFO | Command: touch /tmp/saturnV_install_log.txt Executed on Bot: 3.12.83.119
14:46:44 | INFO | Uploaded bot_dependencies.txt to ~/saturnV/bot_dependencies.sh
14:46:44 | INFO | Command: chmod +x ~/saturnV/bot_dependencies.sh Executed on Bot: 3.12.83.119
14:46:44 | INFO | Starting dependencies installation on Bot 3.12.83.119, this might take a while...
14:46:56 | INFO | Dependencies installation on Bot 3.12.83.119 is FINISHED, please review logs/saturnV_install_log_3_12_83_119.txt log file to check if everything went well!
14:46:58 | INFO | Command: mkdir ~/saturnV Executed on Bot: 3.22.117.16
14:46:58 | INFO | Command: touch /tmp/saturnV_install_log.txt Executed on Bot: 3.22.117.16
14:46:58 | INFO | Uploaded bot_dependencies.txt to ~/saturnV/bot_dependencies.sh
14:46:58 | INFO | Command: chmod +x ~/saturnV/bot_dependencies.sh Executed on Bot: 3.22.117.16
14:46:58 | INFO | Starting dependencies installation on Bot 3.22.117.16, this might take a while...
14:47:11 | INFO | Dependencies installation on Bot 3.22.117.16 is FINISHED, please review logs/saturnV_install_log_3_22_117_16.txt log file to check if everything went well!
```

#### 6. Define your target inside a file, such as the following `my_target.txt`
![](/images/2020-05-01-Massive-Hosts-Discovery-with-SaturnV/target.png)
```
python3 saturnV.py --target my_target.txt 
***** Welcome to SaturnV *****
15:05:04 | INFO | File my_target.txt was successfully elaborated, new targets file is original_subnets.txt
wget https://raw.githubusercontent.com/c0mix/subnet_splitter/master/subnet_splitter.py && python3 subnet_splitter.py --input original_subnets.txt --output target_subnets.txt --size 28
--2020-05-09 15:07:13--https://raw.githubusercontent.com/c0mix/subnet_splitter/master/subnet_splitter.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.124.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.124.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2499 (2.4K) [text/plain]
Saving to: ‘subnet_splitter.py’
subnet_splitter.py 100%[=====================================>]2.44K--.-KB/sin 0s
2020-05-09 15:07:13 (46.8 MB/s) - ‘subnet_splitter.py’ saved [2499/2499]
cat target_subnets.txt
185.60.216.34/32
31.13.71.36/32
31.13.92.0/28
31.13.92.16/28
31.13.92.32/28
31.13.92.48/28
31.13.92.64/28
31.13.92.80/28
31.13.92.96/28
31.13.92.112/28
31.13.92.128/28
31.13.92.144/28
31.13.92.160/28
31.13.92.176/28
31.13.92.192/28
31.13.92.208/28
31.13.92.224/28
31.13.92.240/28
```

#### 7. Prepare and launch masscan test 
```
python3 saturnV.py --masscan-script --masscan-run
***** Welcome to SaturnV *****
15:08:52 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
15:08:52 | INFO | Successfully parsed and added 18 targets subnets
15:08:54 | INFO | Uploaded scripts/masscan_scan_script_1.sh to ~/saturnV/scripts/
15:08:54 | INFO | Command: chmod +x ~/saturnV/scripts/masscan_scan_script_1.sh Executed on Bot: 3.12.83.119
15:08:54 | INFO | Command: cd ~/saturnV/ && tmux new -d -s masscan "scripts/masscan_scan_script_1.sh" Executed on Bot: 3.12.83.119
15:08:55 | INFO | Uploaded scripts/masscan_scan_script_2.sh to ~/saturnV/scripts/
15:08:55 | INFO | Command: chmod +x ~/saturnV/scripts/masscan_scan_script_2.sh Executed on Bot: 3.22.117.16
15:08:55 | INFO | Command: cd ~/saturnV/ && tmux new -d -s masscan "scripts/masscan_scan_script_2.sh" Executed on Bot: 3.22.117.16
python3 saturnV.py --check
***** Welcome to SaturnV *****
15:09:02 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
15:09:03 | WARNING | masscan execution STARTED on Bot: 3.12.83.119
15:09:04 | WARNING | masscan execution STARTED on Bot: 3.22.117.16
```

#### 8. Wait until the scan is finished on all your bots, the grab the results.
```
python3 saturnV.py --check
***** Welcome to SaturnV *****
16:51:03 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
16:51:04 | INFO | masscan execution 100% completed on Bot: 3.12.83.119
16:51:05 | INFO | masscan execution 100% completed on Bot: 3.22.117.16
python3 saturnV.py --get-results
***** Welcome to SaturnV *****
17:01:20 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
17:01:21 | INFO | Transferring outputs/masscan from bot 3.12.83.119 to local outputs/ folder
17:02:05 | INFO | Transferring outputs/masscan from bot 3.22.117.16 to local outputs/ folder
```

#### 9. Prepare and launch other scans
```
python3 saturnV.py --nmap-script --amass-script --gobuster-script --nmap-run --amass-run --gobuster-run
***** Welcome to SaturnV *****
17:03:27 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
17:03:27 | INFO | Parsing Masscan scan results
17:03:27 | INFO | Masscan discovery has found 103 open services on 52 different hosts
17:03:27 | INFO | Nmap script created: scripts/nmap_scan_script.sh
17:03:27 | INFO | Amass script created: scripts/amass_scan_script.sh
17:03:27 | INFO | Checking web service presence on each open service, this might take a while...
17:04:11 | INFO | Gobuster script created: scripts/gobuster_scan_script.sh
17:04:12 | INFO | Uploaded scripts/nmap_scan_script_1.sh to ~/saturnV/scripts/
17:04:12 | INFO | Command: chmod +x ~/saturnV/scripts/nmap_scan_script_1.sh Executed on Bot: 3.12.83.119
17:04:12 | INFO | Command: cd ~/saturnV/ && tmux new -d -s nmap "scripts/nmap_scan_script_1.sh" Executed on Bot: 3.12.83.119
17:04:14 | INFO | Uploaded scripts/nmap_scan_script_2.sh to ~/saturnV/scripts/
17:04:14 | INFO | Command: chmod +x ~/saturnV/scripts/nmap_scan_script_2.sh Executed on Bot: 3.22.117.16
17:04:14 | INFO | Command: cd ~/saturnV/ && tmux new -d -s nmap "scripts/nmap_scan_script_2.sh" Executed on Bot: 3.22.117.16
17:04:14 | INFO | Uploaded scripts/amass_scan_script_1.sh to ~/saturnV/scripts/
17:04:14 | INFO | Command: chmod +x ~/saturnV/scripts/amass_scan_script_1.sh Executed on Bot: 3.12.83.119
17:04:14 | INFO | Command: cd ~/saturnV/ && tmux new -d -s amass "scripts/amass_scan_script_1.sh" Executed on Bot: 3.12.83.119
17:04:14 | INFO | Uploaded scripts/amass_scan_script_2.sh to ~/saturnV/scripts/
17:04:14 | INFO | Command: chmod +x ~/saturnV/scripts/amass_scan_script_2.sh Executed on Bot: 3.22.117.16
17:04:15 | INFO | Command: cd ~/saturnV/ && tmux new -d -s amass "scripts/amass_scan_script_2.sh" Executed on Bot: 3.22.117.16
17:04:15 | INFO | Uploaded scripts/gobuster_scan_script_1.sh to ~/saturnV/scripts/
17:04:15 | INFO | Command: chmod +x ~/saturnV/scripts/gobuster_scan_script_1.sh Executed on Bot: 3.12.83.119
17:04:15 | INFO | Command: cd ~/saturnV/ && tmux new -d -s gobuster "scripts/gobuster_scan_script_1.sh" Executed on Bot: 3.12.83.119
17:04:15 | INFO | Uploaded scripts/gobuster_scan_script_2.sh to ~/saturnV/scripts/
17:04:15 | INFO | Command: chmod +x ~/saturnV/scripts/gobuster_scan_script_2.sh Executed on Bot: 3.22.117.16
17:04:15 | INFO | Command: cd ~/saturnV/ && tmux new -d -s gobuster "scripts/gobuster_scan_script_2.sh" Executed on Bot: 3.22.117.16
```

#### 10. Grab again the results, perform OSINT activities and finally produce the CSV report.
```
python3 saturnV.py --get-results --osint --report
***** Welcome to SaturnV *****
17:08:51 | INFO | SSH private key found in ssh_key folder, the following key will be used: lcomi.pem
17:08:52 | INFO | Transferring outputs/masscan from bot 3.12.83.119 to local outputs/ folder
17:09:01 | INFO | Transferring outputs/amass from bot 3.12.83.119 to local outputs/ folder
17:09:11 | INFO | Transferring outputs/nmap from bot 3.12.83.119 to local outputs/ folder
17:09:36 | INFO | Transferring outputs/gobuster from bot 3.12.83.119 to local outputs/ folder
17:09:42 | INFO | Transferring outputs/masscan from bot 3.22.117.16 to local outputs/ folder
17:09:50 | INFO | Transferring outputs/amass from bot 3.22.117.16 to local outputs/ folder
17:10:00 | INFO | Transferring outputs/nmap from bot 3.22.117.16 to local outputs/ folder
17:10:25 | INFO | Transferring outputs/gobuster from bot 3.22.117.16 to local outputs/ folder
17:10:29 | INFO | Parsing Masscan scan results
17:10:29 | INFO | Nmap outputs found! Adding them to final report
17:10:29 | INFO | Parsing Nmap scan results
17:10:29 | INFO | Services found by Nmap: 103 - Services found by Masscan: 103
17:10:29 | INFO | Amass outputs found! Adding them to final report
17:10:29 | INFO | Parsing Amass scan results
17:10:29 | INFO | Gobuster outputs found! you can manually review them in outputs/gobuster folder
17:10:29 | INFO | Getting information through OSINT
17:11:48 | INFO | Web application URLs eventually discovered with Bing dork can be found here: outputs/bing/url_resources.txt
17:11:48 | INFO | Creating the final report: final_output.csv
```

![](/images/2020-05-01-Massive-Hosts-Discovery-with-SaturnV/final.png)

## Why SaturnV
The name of this tool wants to honor the magnificent NASA rocket which took men on the moon in 1969.

> As of 2020, the Saturn V remains the tallest, heaviest, and most powerful rocket ever brought to operational status.

![](https://ourplnt.com/wp-content/uploads/2018/07/Saturn-V-at-Johnson-Space-Center-NASA.jpg)

## References
- [Masscan](https://github.com/robertdavidgraham/masscan)
- [Nmap](https://nmap.org/)
- [Amass](https://github.com/OWASP/Amass)
- [Gobuster](https://github.com/OJ/gobuster)
- [HackerTargeT](https://hackertarget.com/)


## Take me immediatly to the code! 
[![](/images/2020-05-01-Massive-Hosts-Discovery-with-SaturnV/github.png)](https://github.com/c0mix/SaturnV) 

https://github.com/c0mix/SaturnV