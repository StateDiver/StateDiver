# Notice
This project has been shifted to [CGCL-codes](https://github.com/CGCL-codes/StateDiver) and will be no longer maintained. Please check there! :)

# StateDiver
StateDiver is a tool that detects TCP-based bypassing strategies in DPI systems by utilizing the state discrepancy to guide the fuzzing process.  StateDiver can be used to uncover DPI elusion strategies in various open-source DPI systems (e.g., Snort, Snort++, and Suricata).

The framework is a customized fork to Geneva - different strategy seed pool organization, better state-discrepancy feedback, and faster in testing DPIs locally. It helps to identify TCP-related evasion strategies better.


## Publication
The corresponding research paper will be published in the 38th Annual Computer Security Applications Conference (ACSAC 2022). 
StateDiver: Testing Deep Packet Inspection Systems with State-Discrepancy Guidance. Zhechang Zhang, Bin Yuan, Kehan Yang, Deqing Zou, Hai Jin.