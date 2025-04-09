# Theory

<p>When delving into breach investigations or malware analysis, it is crucial to explore diverse computer data sources rather than solely relying on traditional disk analysis. Unlike non-volatile storage mediums, data residing in RAM is volatile. Acquiring this data from a live machine is highly valuable for forensic investigation. This experiment aims to familiarize participants with various artifacts retrievable from a memory dump, highlighting their relevance through familiar use cases in memory forensics.</p>

<p><strong>1. What is a memory dump?</strong></p>
<p>Memory-dump file is a copy of the contents inside the RAM (Random Access Memory). In other words, it is a snapshot of the computer’s memory at a specific point of time.</p>
<p><strong>Who uses it?</strong> Memory dumps are used by system administrators, developers, and security professionals, providing a wealth of information about a system's state at a specific moment in time.</p>

<p><strong>Following questions can be answered by analysing a memory dump (but not limited to this list below):</strong></p>
<ul>
  <li>Can we recover the hashed passwords?</li>
  <li>Can we recover open/cached files which were opened at the time memory dump was taken?</li>
  <li>Can we use it for gathering artifacts of malware execution?</li>
  <li>Can I use it to understand the nature and extent of a security breach?</li>
  <li>My system is crashed. Can I learn the reason behind the crash?</li>
  <li>What processes were running and did they interact with the internet?</li>
  <li>What was the configuration of the OS like when the memory dump was taken?</li>
</ul>

<p><strong>What open source tools are available on the internet to take a memory dump?</strong></p>
<ul>
  <li><strong>Linux:</strong> AVML, LiME, Linpmem</li>
  <li><strong>Windows:</strong> DumpIt, Winpmem</li>
  <li><strong>Android:</strong> LiME</li>
  <li><strong>OSX:</strong> osxpmem</li>
</ul>

<p><strong>What are the OS tools available on the internet to analyze a memory dump?</strong></p>
<p>Volatility, rekall, MEMProfFS are some of the well-known memory forensics open-source frameworks available on the internet.</p>

<p><strong>2. Why do we need to transfer an evidence file to an analyst machine?</strong></p>
<p>To preserve the integrity of the files on the victim's PC, we transfer the memory dump file to an Analyst machine.</p>

<p><strong>3. Volatility</strong></p>
<p>Volatility is a memory forensics analysis framework. It analyzes the memory dump and identifies various forensic artifacts with the help of plugins. Every plugin has a specific use case. For example, the <strong>pslist</strong> plugin parses relevant data structure (_EPROCESS) fields and extracts the running processes information (such as process name, process ID, parent process ID, etc.) from the memory dump. It supports analyzing memory dumps collected from Windows, Linux, OSX, and Android.</p>

<p><strong>a. OS Profile</strong></p>
<p>As a first step after getting the memory dump, we look out for the Operating System information like OS name, Service Pack, version information, and the architecture from which the memory dump was taken. We leverage the <strong>imageinfo</strong> plugin from the Volatility framework to achieve this task. The OS Profile string we obtain from this step is a crucial component that allows the tool to correctly interpret and analyze memory dumps from different operating systems and versions.</p>
<p>When you specify a profile like <strong>Win7SP1x64</strong> it means:</p>
<ul>
  <li><strong>OS:</strong> Windows 7</li>
  <li><strong>Architecture:</strong> 64 bit</li>
  <li><strong>Service Pack:</strong> version 1</li>
</ul>
<p>Volatility uses this information to:</p>
<ul>
  <li>Load the correct memory layout maps for Windows 7 Service Pack 1 on 64-bit architecture.</li>
  <li>Use the appropriate symbol tables for this specific Windows version.</li>
  <li>Apply the correct data structure definitions for parsing kernel structures, process lists, network connections, etc.</li>
</ul>

<p><strong>b. Registry Hives</strong></p>
<p>Microsoft Windows uses the Registry to store configuration settings related to the OS and the user applications.</p>
<p>Registry hives refer to the logical group of keys, subkeys, and values inside the registry where Windows stores the configurations required to run the OS and applications.</p>
<p>To dump the user credentials we need to access the Windows Registry. <strong>hivelist</strong> is a Volatility plugin that can be used to list the Registry hives.</p>

<p><strong>c. Hashed credentials</strong></p>
<p>The SAM database holds the hashed passwords and user account information. In the present implementation, to enhance security, the hashed passwords are encrypted and stored in the SAM database. The encryption is done using a boot key (called <strong>SYSKEY</strong>) from the SYSTEM hive.</p>
<p>The <strong>hashdump</strong> plugin reads the location (virtual offsets) of SYSTEM and SAM from the memory dump. It extracts the SYSKEY from the SYSTEM hive. Using the SYSKEY, it decrypts the encrypted password hashes stored in the SAM database. The plugin then outputs the password hashes, which can be used for further analysis. The hashed credentials are run against tools such as John-The-Ripper to obtain the crackable passwords.</p>

<p><strong>d. Scan for Open files inside memory-dump</strong></p>
<p>Volatility’s <strong>filescan</strong> plugin leverages the <strong>FILE_OBJECT</strong> data structure information to scan for files which were open at the time the memory dump was taken. This structure contains various fields that describe the state and properties of an open file. So the <strong>filescan</strong> plugin uses the <strong>_FILE_OBJECT</strong> structure members like <strong>DeviceObject, Type, Size</strong> etc., and their corresponding data type sizes: <strong>PDEVICE_OBJECT, CSHORT, CSHORT</strong>, etc., as signatures and scans the entire memory to locate open files. Running this plugin would give us a list of open files with their location information, using which we can recover them from the memory dump.</p>
