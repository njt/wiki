---
url: https://github.com/darkmentorllc/bt-re-mad-skillz
date_fetched: 2026-09-04
---

# Overview

These skills are used for reverse engineering Bluetooth Controller firmware at the HCI layer and below.

These skills are specific to Anthropic's Claude Code and OpenAI's ChatGPT (originally Codex) applications.

The core logic is the same in both, it's just that they need to invoke different CLI instances of themselves.


# Installation

See the vendor-specific README.md files for [Claude Code](./ClaudeCode/README.md) and [Codex](./ChatGPTCodex/README.md) for how to install.


# /bt-re-controller Usage

After you restart ChatGPT/Codex, you can invoke the skill like

```
/bt-re-controller <path to Ghidra file>
```

The `<path to Ghidra file>` can be either the Ghidra .gpr file, or a zip file containing the .gpr file and .rep folder.

If you provide no further information, the LLM will try to infer parameters.

However you are recommended to give the following parameters:

`CHIP_NAME` = `<Whatever you want to call it>`

`BT_SUPPORT `= `<Option>`  
Options are:  
`DUAL_MODE` (if the device supports both Bluetooth Low Energy and Bluetooth Classic (BR/EDR)).  
`BLE_ONLY` (if the device supports only Bluetooth Low Energy).  
`BTC_ONLY` (if the device supports only Bluetooth Classic).  

`STACK_MODE` = `<Option>`  
Options are:  
`BELOW_HCI` (if the firmware only implements the Controller side of the Host Controller Interface).  
`FULL_STACK` (if the firmware implements Host and Controller side of HCI).

`BT_SPEC_VERSION` = `<Version>`  
Where the Version should be a Bluetooth specification version like "4.2" or "6.0"  

E.g. the final invocation may look like:  
```
/bt-re-controller <path to Ghidra file>
CHIP_NAME = "Realtek RTL8761BU"
BT_SUPPORT = DUAL_MODE
STACK_MODE = BELOW_HCI
BT_SPEC_VERSION = 5.1
```

You can also just put all these values into a README.md next to the Ghidra path, and the skill will find them there.

In general if you're missing any prerequisites (such as the path to Ghidra), the LLM will prompt you. For much more information about the prerequisites see the deeper bt-re-controller SKILL.md for [Claude Code](./ClaudeCode/bt-re-controller/SKILL.md) or [ChatGPT/Codex](./ChatGPTCodex/bt-re-controller/SKILL.md) (but we recommend just letting it prompt you for anything it can't find.)


# /bt-re-controller-merge-ghidra-files Usage

The `/bt-re-controller-merge-ghidra-files` skill assumes you have run the `/bt-re-controller` skill with two different models and now have two different output Ghidra files which you want to merge together. It therefore invokes both models to get their opinions on what should be merged or not. By default it assumes the two models are Opus 4.8 and GPT Sol 5.6. If you used different models you should say so when you initially invoke `/bt-re-controller-merge-ghidra-files`.

You will invoke the skill in the first LLM interface (E.g. Claude Code) like this:  

```
/bt-re-controller-merge-ghidra-files <path to zip of .gpr/.rep of Ghidra file 1> <path to zip of .gpr/.rep of Ghidra file 2> <name of output merged Ghidra file>
```

E.g. `/bt-re-controller-merge-ghidra-files ~/Binary1_opus4.8_renames.zip ~/Binary1_gpt5.6_renames.zip output as ~/Desktop/Binary1_opus4.8_gpt5.6_merged.zip`  

In general if you're missing any prerequisites (such as the path to Ghidra), the LLM will prompt you. For much more information about the prerequisites see the deeper bt-re-controller-merge-ghidra-files SKILL.md for [Claude Code](./ClaudeCode/bt-re-controller-merge-ghidra-files/SKILL.md) or [ChatGPT/Codex](./ChatGPTCodex/bt-re-controller-merge-ghidra-files/SKILL.md) (but we recommend just letting it prompt you for anything it can't find.)
