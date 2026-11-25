<h1>Investigating an AI Dependency Confusion Attack</h1>

<h2>Walk-through Description</h2>
This walk-through demonstrates how to examine your machine learning project's requirements.txt file in Linux to be sure that when you update all of the packages that the project depends upon using <b>pip-install</b>, that it is pointing to the correct, legitimate dependency packages rather than attacker-made malicious look-alikes. Attackers will register "typo-squatted" packages in the <b>PyPi.org</b> repository with names that are slightly different than the legitimate package so that if when you are listing your project's dependencies in the <b>requirements.txt</b> file, you accidentally misspell/mistype the name of the dependency, your pipeline will pull in their malicious package from <b>PyPi.org</b> instead of the one you intended to pull in. This is a walk-through of a TryHackMe lab in the Supply Chain Attack Vectors room on the AI Security pathway.

<h2>Steps of the Investigation</h2>

<b>1. Manually examine the contents of your requirements.txt file</b>

Use the <b>cat</b> command in Linux to print the contents of the requirements.txt file to screen:

<pre><b>cat [filepath to the requirements file]</b></pre>

In this lab, I used the following command: 

<pre><b>cat /opt/supply-chain/dependencies/requirements_external.txt   </b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/bdRowvI.png" height="100%" width="100%" alt="cat your requirements.txt image"/>
</p>

Here we can see three problems:

<b>numpy</b> is written with an extra <b>p</b> as <b>numppy</b>

<b>requests</b> is written with the <b>e</b> and <b>u</b> reversed as <b>reqeusts</b>

and <b>internal-ml-utils</b> has an abnormally high version number <b>99.0.0</b> and may be a dependency that we didn't even intend to be in the list at all.

These entries are all suspicious and we don't know how they got there - it could have been a typo on the part of the developer or it may have been modified on purpose during a cybersecurity incident by a malicious actor.

<b>2. By comparison look at a trusted clean version of the requirements.txt file</b>

If we have questions about what dependencies should be on our requirements.txt file and want to compare suspicious entries to a verified clean list, we can look at a back up of a last known good version and compare them. In this lab, that file is <b>requirements_internal.txt</b>:

<pre><b>cat /opt/supply-chain/dependencies/requirements_internal.txt</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/CiPhplL.png" height="100%" width="100%" alt="Verify with a trusted clean requirements.txt image"/>
</p>

Here in our verified clean list we see that <b>numpy</b> and <b>requests</b> are spelled correctly and that the questionable dependency <b>internal-ml-utils</b> is not present. This may mean that it was added by an internal developer or a malicious actor that compromised the file since the last known good version of this file was made.

<b>3. Run pip-audit on the suspicious requirements.txt file to make sure all packages are registered on PyPi.org</b>

You can also run <b>pip-audit</b> on the <b>requirements.txt</b> file with the suspicious typos and package to quickly check to see if what is in your requirements file is registered on <b>PyPi.org</b>. 

However, beware that attackers register typo-squatted malicious packages on <b>PyPi.org</b>, and in that case, your <b>pip-audit</b> would not fail and would not produce an error when looking up a typo-laden package name in your <b>requirements.txt</b> file. It would find the malicious typo-squatted package on <b>PyPi.org</b> and proceed silently through the audit. 

To run <b>pip-audit</b>, issue:

<pre><b>pip-audit -r /[filepath to your requirements.txt] 2>&1</b></pre>

For this lab I used the path to the suspicious <b>requirements_external.txt</b>:

<pre><b>pip-audit -r /opt/supply-chain/dependencies/requirements_external.txt 2>&1</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/DZgdQg1.png" height="100%" width="100%" alt="Run pip-audit image"/>
</p>

<b>pip-audit</b> returns errors for the <b>numppy</b> package because no <b>numppy</b> is registered on <b>PyPi.org</b>. It however, does not return errors for the typo-squatted <b>reqeusts</b> package or the package <b>internal-ml-utils, version 99</b>.

<h2>Summary</h2>

This walk-through demonstrates how AI developers and security professionals can inspect their package dependency lists in <b>requirements.txt</b> (or other dependency file) to prevent dependency confusion attacks that give malicious actors a foothold inside of your development pipeline. It shows how to run <b>pip-audit</b> and that automating the process of updating, installing, and auditing dependencies must be accompanied by security investigations of dependencies to be sure that malicious packages are not being pulled into your projects.


<!--
 ```diff
- text in red
+ text in green
! text in orange
