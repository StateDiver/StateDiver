# StateDiver
StateDiver is a tool that detects TCP-based bypassing strategies in DPI systems by utilizing the state discrepancy to guide the fuzzing process.  StateDiver can be used to uncover DPI elusion strategies in various open-source DPI systems (e.g., Snort, Snort++, and Suricata).

The framework is a customized fork to Geneva - different strategy seed pool organization, better state-discrepancy feedback, and faster in testing DPIs locally. It helps to identify TCP-related evasion strategies better.



## Usage

### Requirement
* Ubuntu 18.04 (Other Linux-based system might work as well)
* root privilege (for sending raw packets)
* VMs locally(VMware Workstation or VirtualBox) with Shared Folder mode on (for passing state-discrepancy guidance)

### Network topology (换个名字 改成 Network configuration？)
[这个就不打算说了]

### Setup DPIs‘ VMs

#### Build DPIs
[把规则整理好，放在代码内]
[如何编译DPI]
[路径设定]
[人工插桩]


### Setup StateDiver’s VM

StateDiver has been developed and tested for Ubuntu. Due to limitations of netfilter and raw sockets, It works on Linux and requires *python3.6*.





## Publication
The corresponding research paper will be published in the 38th Annual Computer Security Applications Conference (ACSAC 2022). 
StateDiver: Testing Deep Packet Inspection Systems with State-Discrepancy Guidance. Zhechang Zhang, Bin Yuan, Kehan Yang, Deqing Zou, Hai Jin.