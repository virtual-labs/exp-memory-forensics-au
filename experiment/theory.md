# Theory

## Memory Forensic

When delving into breach investigations or malware analysis, it is crucial to explore diverse computer data sources rather than solely relying on traditional disk analysis. Unlike non-volatile storage mediums, data residing in RAM is volatile. Acquiring this data from a live machine is highly valuable for forensic investigation. This experiment aims to familiarize participants with various artifacts retrievable from a memory dump, highlighting their relevance through familiar use cases in memory forensics.

### 1. What is a Memory Dump?

A memory-dump file is a copy of the contents inside the RAM (Random Access Memory). In other words, it is a snapshot of the computer’s memory at a specific point in time.

**Who uses it?** Memory dumps are used by system administrators, developers, and security professionals, providing a wealth of information about a system's state at a specific moment in time.

Following questions can be answered by analyzing a memory dump (but not limited to this list below):

- Can we recover the hashed passwords?
- Can we recover open/cached files which were opened at the time the memory dump was taken?
- Can we use it for gathering artifacts of malware execution?
- Can I use it to understand the nature and extent of a security breach?
- My system crashed. Can I learn the reason behind the crash?
- What processes were running and did they interact with the internet?
- What was the configuration of the OS when the memory dump was taken?

**Open source tools available to take a memory dump:**

- **Linux:** AVML, LiME, Linpmem
- **Windows:** DumpIt, Winpmem
- **Android:** LiME
- **OSX:** osxpmem

**Open source tools available to analyze a memory dump:**

- Volatility
- Rekall
- MEMProfFS

### 2. Why do we need to transfer an evidence file to an analyst machine?

To preserve the integrity of the files on the victim's PC, we transfer the memory dump file to an analyst machine.

### 3. Volatility

#### a. OS Profile

As a first step after getting the memory dump, we look out for the Operating System information like OS name, service pack, version information, and the architecture from which the memory dump was taken. We leverage the `imageinfo` plugin from the Volatility framework to achieve this task. The OS Profile string we obtain from this step is a crucial component that allows the tool to correctly interpret and analyze memory dumps from different operating systems and versions.

When you specify a profile like `Win7SP1x64` it means:

- **OS:** Windows 7
- **Architecture:** 64 bit
- **Service pack:** Version 1

Volatility uses this information to:

- Load the correct memory layout maps for Windows 7 Service Pack 1 on 64-bit architecture. Each OS version organizes kernel structures, process information, and other critical data in specific memory locations. The profile provides a map of where to find this information in the memory dump.
- Use the appropriate symbol tables for this specific Windows version. These are like dictionaries that translate memory addresses to meaningful function or variable names. The profile contains the correct symbol information for the specific OS version, allowing Volatility to identify important functions and data structures.
- Apply the correct data structure definitions for parsing kernel structures, process lists, network connections, etc. The profile defines how these structures are laid out in memory for that specific OS version.

#### b. Registry Hives

Microsoft Windows uses the registry to store configuration settings related to the OS and user applications. Registry hives refer to the logical group of keys, subkeys, and values inside the registry where Windows stores the configurations required to run the OS and applications.

To dump the user credentials, we need to access the Windows Registry. `hivelist` is a Volatility plugin that can be used to list the Registry hives.

#### c. Hashed Credentials

The SAM database holds the hashed passwords and user account information. To enhance security, the hashed passwords are encrypted and stored in the SAM database. The encryption is done using a boot key (called SYSKEY) from the SYSTEM hive.

The `hashdump` plugin reads the location (virtual offsets) of SYSTEM and SAM from the memory dump. It extracts the SYSKEY from the SYSTEM hive. Using the SYSKEY, it decrypts the encrypted password hashes stored in the SAM database. The plugin then outputs the password hashes, which can be used for further analysis.

**Step 6: Crack the hashes**
The hashed credentials are run against tools such as John-The-Ripper to obtain the crack'able passwords.

**Step: Locate the Image file**
Volatility’s `filescan` plugin leverages the FILE_OBJECT data structure information to scan for files that were open at the time the memory dump was taken. Running this plugin would give us a list of open files with their location information, using which we can recover them from the memory dump.
