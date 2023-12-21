# hedgehog-tools
 
Repo of smaller scripts for malware analysis, deobfuscation and configuration extraction

## Overview

| Target                    | Depends |   Static  | Config extraction | C2 extraction | Deobfuscation | Unpacking | Handles packed sample |
| ------------------------- | ---------------------- | --- | ----------------- | ------------- | ------------- | --------- | --------------------- |
| AgentTesla (OriginLogger) | Python, dnlib          |   ✅  | ✅                | ✅            | ⛔            | ⛔        | ⛔                    |
| AllCome                   | Python                 |   ✅  | ✅                | ✅            | ⛔            | ⛔        | ⛔                    |
| GootLoader                | JavaScript, NodeJS     |   ✅  | ✅                | ✅            | ✅            | ✅        | ✅                    |
| LimeRAT                   | Python, dnlib          |   ✅  | ✅                | ✅            | ⛔            | ⛔        | ⛔                    |
| NightHawk                 | IDAPython              |   ✅  | ⛔                | ⛔            | ✅            | ⛔        | ✅                    |

## AgentTesla Config Extractor

### Requirements

The script uses dnlib. This line of the script must point to the dnlib binary on your system:

`clr.AddReference(r"dnlib.dll")`

You need a .NET Runtime on your system. Install it and run

`pip install -r requirements.txt`

### Usage

`python AgentTesla_config_extractor.py <folder_with_samples>`

Configs will be placed or appended to a file named agenttesla_configs.txt

For sample hashes look at the extracted configs.

Works on non-packed samples, so use unpacme or similar before applying the extractor.

## AllCome Clipbanker Config Extractor

Usage:

`python allcome_extractor.py <sample>`
  
Reference: https://twitter.com/3xp0rtblog/status/1486368999919300611
  
Sample: https://bazaar.abuse.ch/sample/f234b6d1801e1d4105de18a74ecd99f64cbdd7c47d6079bb2994d38ed7b0de44/

## LimeRAT Config Extractor

### Requirements

The script uses dnlib. This line of the script must point to the dnlib binary on your system:

`clr.AddReference(r"dnlib.dll")`

You need a .NET Runtime on your system. Install it and run

`pip install -r requirements.txt`

### Usage

`python limerat_config_extractor.py <folder_with_samples>`

Configs will be placed or appended to a file named limerat_configs.txt

For sample hashes look at the extracted configs.

Works on non-packed samples, so use unpacme or similar before applying the extractor.

## GootLoader JS Unpacker and C2 Extractor

### Why

This was a project to learn AST manipulation with babel and JavaScript.

So it is likely that this is not the best code because I am a JavaScript noob.

The script is static, it does not execute any of the manipulated code.

### Requirements

Install NodeJS and npm

Execute this to install required packages

`npm.exe install -save-dev @babel/core commander`

### Usage

`node.exe gootloader_decoder.js -f <sample> --c2s <textfile>`

This will unpack the Gootloader script layers to _sample\<nr\>.js_. After that it will attempt to find C2 data. Even if some of it fails, it should serve in saving some unpacking steps.

The very first transpiled layer is the extraction of just the relevant functions which are often buried in > 6000 lines of code. To achieve that, the decoder will search for the typical structure of the entry point function and determine all matched functions as start nodes. That means there might be some false positives, but as long as the actual entry point function is included, it should not be an issue. 

From this point forward it will search for all used identifiers in that entry point function and recursively for the functions that are being called. This way a 6000 lines script can be trimmed down to 200 lines, making manual analysis of the initial code possible. In case the entry point function for the malware code turns out to be the wrong one, you can set it manually, e.g., here for the function named _iolad7_:

`node.exe gootloader_decoder.js -f <sample> -s iolad7`

Starting from the second layer the unpacker will determine the responsible decrypt function, the key and a decoding constant which is changed in every sample.

It will attempt to extract C2's at the last layer, which it currently assumes to be either the third or the 6th (as these are the samples I got).

Note: Some of the layers will be wrapped into a function named _gldr()_. This function is **not** part of the malware but the decoder. It is necessary where gootloader dynamically wraps the unpacked code into an unnamed function. Since the body contains the a return, the AST can only be parsed with this wrapped function.

### Samples 


`1bc77b013c83b5b075c3d3c403da330178477843fc2d8326d90e495a61fbb01f` --> complete, has 3 layers

`08f06fc48fe8d69e4ab964500150d1b2f5f4279fea2f76fdfcefd32266dfa1af` --> complete, has 6 layers

`320b4d99c1f5fbc3cf1dfe593494484b1d4cb1ac7ac1f6266091e85ef51b4508` --> complete, has 6 layers

`445a5c6763877994206d2b692214bb4fba04f40a07ccbd28e0422cb1c21ac95b` --> complete, has 6 layers

`cbd826f59f1041065890cfe71f046e59ae0482364f1aaf79e5242de2246fb54b` --> complete, has 6 layers

`b34bcf097ad6ab0459bc6a4a8f487ca3526b6069ec01e8088fd4b00a15420554` --> complete, has 6 layers

`1b8b2fbdff9e4109edae317c4dd8cef7bb7877d656e97a3dd0a1e8c0c9d72b0b` --> only unpacks until layer 6

### Example Output

Decoded last layer with C2 data:

![Extracted layer 6](gootloader_decoded_c2layer.png)

Output of unpacking and extraction:

![GootLoader Decoder Example Output](gootloader_decoder_output.png)

## NightHawk String Decoder

IDAPython script that adds comments with decoded string contents into the idb. Needs polishing as it misses some strings.

Tested on samples:

* https://bazaar.abuse.ch/sample/0551ca07f05c2a8278229c1dc651a2b1273a39914857231b075733753cb2b988/

* https://bazaar.abuse.ch/sample/9a57919cc5c194e28acd62719487c563a8f0ef1205b65adbe535386e34e418b8/
