# CVE-2024-3094 - XZ Linux Backdoor Analysis


## Intro
Hello folks👋, I will be talking today about the recent incdient of the XZ Utils backdoor that compromised a lot of linux distributions across the world. This incident made the headlines. 

We are going to dive deep into the intricacies and about how this sophisticated attack came to be. This type of attack falls under the type of supply chain attack targeting the XZ Utils project, aimed at embedding a backdoor within linux systems. 

We&#39;ll explore the sequence of events leading up to the attack&#39;s discovery. Discuss the mechanisms the attacker used, assess the potential repercussions had the attack remained undetected, and reflect on the critical lessons this incident teaches us about cyber security in open-source projects.

Finally, I will wrap up with lessons learned, a detailed lab environment and the importance of vigilence and proactive security measures in the open-source ecosystem.

## Understanding the Attack
On Mar 29, 2024 at 12:00PM ET, Andres Freund [posted](https://www.openwall.com/lists/oss-security/2024/03/29/4) on the Openwall mailing list about a backdoor he discovered in the [_XZ Utils_](https://github.com/tukaani-project/xz)package. The backdoor targeted the [OpenSSH](https://www.openssh.com/) binary, allowing remote code execution on impacted machines. This backdoor was not located in the GitHub repository, but only in release versions of the package, which hid its presence.

Due to the reason XZ Utils had been installed (directly or indirectly) on billions of Linux systems worldwide, this discovery shocked the international Linux and infosec communities.

## Understanding the timeline of the attack
In late 2021, &#34;Jia Tan&#34;, an online identity, began a long and careful supply-chain attack with the goal of inserting a backdoor into Linux systems. Jia Tan submitted multiple contribution requests to several projects, including the XZ Utils project, and began harrasing Lasse Collin and other open source maintainers through various [sock puppet accounts](https://research.swtch.com/xz-timeline#jia_tan_arrives_on_scene_with_supporting_cast)to get their XZ contributions accepted and ultimately, by late 2022, Jia Tan became a maintainer of the XZ project.

During 2023 and early 2024, Jia Tan started making multiple changes to various open-source projects. This included [submitting a merge request](https://github.com/google/oss-fuzz/pull/10667)to [google/oss-fuzz](https://github.com/google/oss-fuzz), which developers use to validate their code. Apparently, the goal of this update was to take ownership of the XZ Utils project and disable specific features in order to hide certain errors that could reveal the planned backdoor.

Finally, in early 2024, Jia Tan released XZ Utils 5.6.0 as a tarball, which is the most common method of releasing software on GitHub. Specifically, they performed [two commits](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=cf44e4b7f5dfdbf8c78aef377c10f71e274f63c0) that added [&#34;test&#34; files](https://git.tukaani.org/?p=xz.git;a=commitdiff;h=6e636819e8f070330d835fce46289a3ff72a7b89) that included the backdoor in the released version of the software. Since these were binary files, and not plain text, the &#34;test&#34; files were more obfuscated from the community.

Following the release of version 5.6.0, Jia Tan rushed out a 5.6.1 release in an attempt to fix some failures that were occurring with the backdoor, with the hope of fixing them before they were spotted.

&gt;In total, Jia Tan made at least [450 commits](https://git.tukaani.org/?p=xz.git;a=search;pg=0;s=Jia&#43;Tan;st=author) to the XZ GitHub, beginning on June 10, 2022.

Eventually, on March 29, 2024 at 12:00PM ET, [Andres Freund](https://www.openwall.com/lists/oss-security/2024/03/29/4) noticed a delay in the operation of SSH when running unstable version of Debian. Without the discovery of this SSH delay and various [_valgrind_](https://valgrind.org/) test errors, the community may not have discovered this vulnerability until it had run its course, infecting untold thousands or millions of machines.

Despite the discovery of this vulnerability, the community is still wrestling with the potential scale and ramifications of the attack.

&gt;Russ Cox presented an excellent [detailed timeline](https://research.swtch.com/xz-timeline) of this attack, and @rheaeve posted an informative [timezone investigation](https://rheaeve.substack.com/p/xz-backdoor-times-damned-times-and), both of which served as outstanding community resources. 

## Grasping the Technical Overview of the Attack
To summarize, the widely-used XZ utils package, which contains the xz and the *liblzma* library, contained the obfuscated code that [created a backdoor](https://nvd.nist.gov/vuln/detail/CVE-2024-3094) in the OpenSSH service on many linux systems using XZ utils versions (5.6.0 -&gt; 5.6.1). This supply-chain attack was executed over trust-buildng campaign spanning two years🤯.

From a technical standpoint, the backdoor was created when the tarball was generated for release tag. In other words, the backdoor was not created if the user built the tool manually from it&#39;s git repo.

Thus, it is safe to assume the attacker was mainly targetting Debian and RPM-based systems which depend on this tarball for their package installers. Alternatively, systems that do not rely on this method of packaging, such as Arch Linux, were not impacted by this vulnerability. I love arch btw💘😉.

## Assessing the Potential Impact of the Attack
What makes this tremendously dangerous and attractive for such sophisticated attacks is that XZ utils is widely used over 3 billion machines, lightweight, easy to use and widely available. 

Given the widespead usage of this package, this attack could have impacted the security of linux systems worldwide, specially those that are running OpenSSH. The impact of this would have been devastating. Put simply, it&#39;s possible that this vulnerability could have granted Jia Tan and his allies or sponsors open access to every affected internet facing SSH server, globally, including those run by governments and high profile global entities. 

This attack had the potential to weaken the global linux security posture and could have triggered untold millions of dollars in damages, or worse, it could have affected global infrastucture or even the health and safety of countless citizens. 

## Learning from the incident
Let&#39;s review some of the potential lessons that we can learn from this incident. 

First, this high-profile attack has raised concerns about the security and trustworthiness of the open-source projects. At the same time, the open-source process of testing and collective effort worked in this case, as the vulnerability was discovered before it getting pushed from unstable to stable distributions.

In addition, the discovery of this sophisticaed attack raises questions regarding the safety of other open-source projects that may have already been struck with a similar fate.

This incident has highlighted the fact that solo open-source developers of high-profile projects are often overwhelmed. We cant simply completely rely on one person to do everything. We have to ensure that developers, maintainers and contributors are trustworthy.

We also need to develop tools, techniques and procedures tha tensure the validity and the safety of our code, especially binary objects and their components.

Finally, we must develop and implement tools, techniques and procedures that help detect not only supply-chain attacks but also the systems affected by these attacks in the event of breach.

## Testing Lab 
In this lab, I will be deploying two machines, **Kali**(backdoored package installed) and **Debian**(clean package). The Debian machine will serve as a baseline as we discuss the behavioral analysis that led to the discovery of this backdoor.

First I will add two entries to the hosts file in our VM machines.{{&lt; figure src=&#34;/i2.png&#34; title=&#34;Modified hosts file&#34; &gt;}} 


Here as you can see we have the backdoored version of the package to install on kali VM. It can be obtained from the Debian package snapshots. {{&lt; figure src=&#34;/i1.png&#34; title=&#34;Kali VM malicious XZ Package&#34; &gt;}}

After getting the vulnerable package, it&#39;s time to instasll it with
```bash
sudo apt-get install -y --allow-downgrades  ./liblzma5_5.6.1-1_amd64.deb
```

then we need to reload static library object of the vulnerable version with
```bash
sudo systemctl restart ssh
```

now our kali machine is vulnerable to *CVE-2024-3094*. Let&#39;s check it out.

First, let&#39;s detect if this version of *liblzma* contains the backdoor. We&#39;ll use Vegard Nossom&#39;s script, **detect.sh** which was provided in the  [disclosure](https://www.openwall.com/lists/oss-security/2024/03/29/4). The script is already located in the home directory. Let&#39;s use **cat** to view the script and analyze it.
```bash
┌──(abdo💀rog)-[~/Downloads]
└─$ cat detect.sh 
#! /bin/bash

set -eu

# find path to liblzma used by sshd
path=&#34;$(ldd $(which sshd) | grep liblzma | grep -o &#39;/[^ ]*&#39;)&#34;

# does it even exist?
if [ &#34;$path&#34; == &#34;&#34; ]
then
	echo probably not vulnerable
	exit
fi

# check for function signature
if hexdump -ve &#39;1/1 &#34;%.2x&#34;&#39; &#34;$path&#34; | grep -q f30f1efa554889f54c89ce5389fb81e7000000804883ec28488954241848894c2410
then
	echo probably vulnerable
else
	echo probably not vulnerable
fi
```
The script uses _ldd_ to list the shared libraries loaded by the _sshd_ binary and then searches for _liblzma__ to determine its path.

Then, the command `hexdump -ve &#39;1/1 &#34;%.2x&#34;&#39; &lt;&lt;file&gt;&gt;` will dump the _sshd_ binary in hexadecimal form, without any formatting, producing a long hex string. The script then searches for a hexadecimal pattern that serves as a signature for the backdoor.

As a result, if the script finds the signature, our binary has been backdoored.

Let&#39;s run the script on the _kali-backdoor_ machine first.

{{&lt; figure src=&#34;/i3.png&#34; title=&#34;Script showing the package is vulnerable&#34; &gt;}} 

In the next section, we&#39;ll use behavioral anlysis to identify this backdoor without needing to log into the servers.

## Confirm that the SSH Daemon is Slower than Usual
This vulnerability was discovered because of a small delay in the SSH rseponse time caused by peaks of CPU usage consumed by the sshd service. This anomaly remained unnoticed until Andres Freund ran a benchmark.

Let&#39;s check this in our small lab. We&#39;ll SSH into both machines and compare their behaviors.
{{&lt; figure src=&#34;/i4.png&#34; title=&#34;Showing the difference in timing between SSH&#34;&gt;}} 

Both commands produced a &#34;Permission denied&#34; error message. This is expected since we didn&#39;t submit valid authentication keys. However, the output of the **time** command, specifically the _real_ value, reveals the time it took the command to present the error message.

Although the real values may vary depending on many factors, the vulnerable machine clearly took longer to respond. We can repeat these exercise many times and each time, the backdoored machine will take longer to respond.

This is exactly what Andres Freund noticed, which drove him to dive deeper in his investigation.

We could write a custom scanner to test this, allowing us to remotely determine if a server has potentially been backdoored.

Until next time cyaa.👋

---

> Author: w01v3r  
> URL: https://0xd3d5ec.github.io/cyberblog/xzbackdoor/  

