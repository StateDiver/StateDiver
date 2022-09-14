# StateDiver
StateDiver is a tool that detects TCP-based bypassing strategies in DPI systems by utilizing the state discrepancy to guide the fuzzing process.  StateDiver can be used to uncover DPI elusion strategies in various open-source DPI systems (e.g., Snort, Snort++, and Suricata).

The framework is a customized fork to Geneva - different strategy seed pool organization, better state-discrepancy feedback, and faster in testing DPIs locally. It helps to identify TCP-related evasion strategies better.


## Source


We have continued the code composition of Geneva. To learn about how Geneva works, checkout the [documentation](https://geneva.readthedocs.io/en/latest/) of Geneva project. Our main modifications and additions are as follows.
```
...
├── actions              Original mutation methods
...
├── tools
    ├── dpi_run_scripts  Supplementary guidance on how to run DPI
    └── dpi_sys_confs    Configuration and rule files of Snort,Snort++, and Suricata
...
├── evaluator.py         Extract trace files from DPIs’ state instrumentation & Communicate with the server
├── evolve.py            State-discrepancy guidance algorithm to evaluate current strategy & AFL-like queue schedule algorithm in Strategy Selector
...

```

## Setup

For the convenience of description, we use Snort and Suricata as examples to illustrate the workflow of StateDiver. Snort++ is configured in a similar way.

### Requirement
* Ubuntu 18.04 (Other Linux-based system might work as well)
* root privilege (for sending raw packets)
* VMs locally(VMware Workstation or VirtualBox) with Shared Folder mode (for passing state-discrepancy guidance) and Promiscuous Mode on.


### Setup


#### Build DPIs


#### 1. Snort

1.1 Download the source code of Snort.

1.2 Enable debug mode when compiling.
```
./configure --enable-debug --enable-gdb --enable-debug-msgs
```

1.3 Read protocol stack related code and use debug statements to extract status information. （snort_stream_tcp.c in Snort）

1.4 make & make install

#### 2. Suricata

2.1 Download the source code of Suricata.

2.2 Enable debug mode when compiling.
```
./configure --enable-debug --enable-nfqueue --enable-non-bundled-htp 
```

2.3 Read protocol stack related code and use debug statements to extract status information. （stream-tcp.c in Suricata）

2.4 make & make install



#### 3. VMs‘ settings 

**Dependencies**

StateDiver has been developed and tested for Ubuntu. Due to limitations of netfilter and raw sockets, It works on Linux and requires *python3.6*.

Install netfilterqueue dependencies:
```
apt-get install build-essential python-dev libnetfilter-queue-dev libffi-dev libssl-dev iptables python3-pip
```

Install Python dependencies:
```
python3 -m pip install -r requirements.txt
```

**VMs' settings**

1. Enable Share-folder mode and mount share folder in your host machine to specific location. e.g. Create folder myfolder in host machine.
In VirtualBox:
```
mount -t vboxsf myfolder /mnt/hgfs/share-folders/
```
Then create two subfolder in myfolder, folder snort-log and folder suricata-log respectively.
```
myfolder
├── snort-log
└── suricata-log
```

2. Enbale Promiscuous Mode in DPIs' network adapter.
3. Configure network topology.

Typically, **four** virtual machines are required to run this experiment. 
We provide virtualbox images of the virtual machines used in our experiments at this [link](https://cloud.189.cn/t/Fjqye2YBFjmm) (access code:50q9). It contains 5 virtual environments (47.28G total, root privilege password is root).


VM1 StateDiver: run StateDiver

VM2 DPI Snort: run Snort (IPS mode)

VM3 DPI Suricata: run Suricata (IDS mode)

VM4 Server: run Server (Apache)

VM5 DPI Snort++: run Snort++


The network topology is as follows:
```
+------------+        +------+         +--------+
| StateDiver | <----> | Snort| <---->  | Server |
+------------+        +------+         +--------+
                  |                |   
                  |                |
                  |  +----------+  |
                  +- | Suricata | -+
                     +----------+
                  （Promiscuous Mode）
```



## Run the test

1. Start DPIs

Run Snort in VM2:
```
export SNORT_PP_DEBUG=0x00040000LL
iptables -I FORWARD -j NFQUEUE --queue-num=2
cd snort-2.9.19-state-instrumentation
stdbuf -oL ./src/snort -Q -c .etc/snort-ips-share.conf -A fast > /mnt/hgfs/share-folders/snort-log/console.log
```

Run Suricata in VM3:
```
export SC_LOG_OP_FILTER=DPITESTING
export SC_LOG_LEVEL=Debug
cd suricata-6.0.3-state-instrumentation
stdbuf -oL ./src/suricata -c ./suricata-ids-share.yaml -i interface > /mnt/hgfs/share-folders/suricata-log/console.log
```

2. Start StateDiver

Run StateDiver in VM1:
```
iptables -A INPUT -p tcp --sport 80 -j NFQUEUE --queue-num 1
iptables -A OUTPUT -p tcp --dport 80 -j NFQUEUE --queue-num 2
cd StateDiver
python3 evolve.py --population 50 --generations 50 --test-type http --server server_IP --bad-word ultrasurf.html --port 80 --log-on-success True
```

## Analyze the result & Verify the strategy
The result are shown in ./trials/xxxx-xx-xx_xx:xx:xx/success. 

xxxx-xx-xx_xx:xx:xx represents the time when you perform the test.

the `success` folder contains two types of files. All successful strategies are shown in the `ga_summary`. The remaining files are named with the strategy ID in `ga_summary`. These files show detailed log information when the respective strategies are performed.

We need replays and manual efforts to determine if this is a new strategy.

To replay a single strategy:
```
stra=[TCP:flags:A]-duplicate(tamper{TCP:options-md5header:corrupt}(tamper{TCP:flags:replace:RA},),)-| \/
python3 plugins/plugin_client.py --plugin http --test-type http --strategy $stra --log debug --output-directory trials/2022-01-01_00:00:00 --extrenal-server --port 80 --workers 1 --bad-word ultrasurf.html --runs 1 --fitness-by avg --server server_IP --host-header "" --injected-http-contains "" --environment-id 10000000
```






## Publication
The corresponding research paper will be published in the 38th Annual Computer Security Applications Conference (ACSAC 2022). 
StateDiver: Testing Deep Packet Inspection Systems with State-Discrepancy Guidance. Zhechang Zhang, Bin Yuan, Kehan Yang, Deqing Zou, Hai Jin.