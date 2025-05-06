<h3> Procedure</h3>

<p><b>Step-1 Create a memory dump file :</b></p>
<li>Click the “MemoryDump tool” button to generate a memorydump file.Copy the filename for Step 3.</li>
<img src="images/step1.png" alt="step1">
<li>Copy the filename  highlighted in yellow color for Step 3.</li>
<img src="images/step1.2.png" alt="step1">
<p><b>Step-2 Transfer memorydump file to an Analyst machine:</b></p>
<li>To preserve integrity of the files on the victim's PC we transfer the memorydump file to an Analyst machine.</li>
<img src="images/step2.png" alt="step2">

<p><b>Step-3 Locate the image file:</b></p>
<li>We use the imageinfo plugin from the Volatility framework to look out for various Operating system related information. copy and paste  the filename  highlighted in yellow color from " Step-1:Create a Memory Dump File"</li>
<img src="images/step3.1.png" alt="step2">
<li>Copy the OS Profile string highlighted in Yellow.</li>
<img src="images/step3.2.png" alt="step2">
<li>
<p><b>Step-4 Dump the image file:</b></p>
<li> Enter the Profile name to run this plugin. Copy the Virtual Offsets of SAM and SYSTEM required for the next step</li>

<img src="images/step3.3.png" alt="step2">
<p><b>Step-5 : View the image file:</b></p>
<li>Paste the SAM and SYSTEM hive offsets in the box.</li>
<img src="images/step4.png" alt="step2">
<li>The SAM and SYSTEM hive offsets are used as input in this step to run hashdump plugin which will print the password hashes. Copy the resulting hashes for the final step.</li>
<img src="images/step5.png" alt="step2">
<p><b>Paste the hash to crack the password.</b></p>
<img src="images/step6.png" alt="step2">
<p><b>Cracked Password</b></p>
<img src="images/step7.png" alt="step2">


