# UPDATE: The issue patched in march 2021 https://support.kaspersky.com/general/vulnerability.aspx?el=12430#300321
# Kaspersky Safe Money Local Privilege Escalation
So in the last days I discovered a lot of local privileges escalation bugs in kaspersky products, about 11 vulnerability were reported, 3 of them were patched while 8 of them are still unpatched. I ain't gonna lie firstly they were very nice to me (even in the end) but their bounty were low (I won't spend 3 days for 500$) and even in the last 2 reports they didn't rewarded them. Don't expect me to work for free, I don't have a job at cyberark or something ? I asked for acknowledgements in they're next advisory but they don't seems to be caring too much while they acknowledged some dll hijack bugs. 

So no bounty and no acknowledgments, nothing make me report this to them instead of public disclosure.

However at the time of this disclosure kaspersky is unaware of this bug, this bug affect kaspersky safe money module which comes by default installed with kaspersky internet security & kaspersky total security & kaspersky cloud security.


The bug has been tested on windows 10 20h2 with the latest security patches and on Kaspersky total security 21.2.16.590


The issue is when a user attempt to open a new page with the safe money module, i.e when you open microsoft edge the av service will create a directory in "C:\ProgramData\Kaspersky Lab\SafeBrowser\Common\<CURRENT_USER_SID>" and call SetSecurityInfo without impersonating the user and without checking if "C:\ProgramData\Kaspersky Lab\SafeBrowser\Common" is a reparse point, this behaviour only trigger when open safe money for the first time or the dir either do no exist or innacessible.


The bug is more likely exploitable because there's no ACL or self protection enforcement in "C:\ProgramData\Kaspersky Lab\SafeBrowser\Common" and will grant a standard user to access to it with GENERIC_WRITE access which is enough to set a reparse point.


However there's an enforcement by the kernel driver to prevent malicious actions in "C:\ProgramData\Kaspersky Lab\SafeBrowser\Common\<CURRENT_USER_SID>" and it also prevent any attempt to delete or move anything on it (a handle will be granted but the file ops will be filtered by the kernel driver). Luckily there's a feature in the AV itself called "File Shredder" which actually could bypass those restrictions.


But based on what I see kaspersky also applied an enforcement on the system and mark it as a suspicious behaviour, any application that attempt to create a symbolic link in "\RPC CONTROL\" will be killed then moved to the quarantine and flagged as a malware, they did that because in every single report I used to create symlinks in "\RPC CONTROL\" however this mitigation can be easily bypassed since not only "\RPC CONTROL\" is the write-able location in the object manager, while "\Sessions\<current_user_session>\AppContainerNamedObjects" and "\BaseNamedObjects\Restricted" are also write-able and we can create symbolic links on it, so the mitigation is useless.
To exploit this issue we need "C:\ProgramData\Kaspersky Lab\SafeBrowser\Common" to be empty, then we set a reparse point to "\BaseNamedObjects\Restricted" which has a symlink "\BaseNamedObjects\Restricted\<CURRENT_USER_SID>" that point to our target, this bug is a little bit impactful since SetSecurityInfo will change the security of the target either if it a file or directory so technically we can take over any directory of file that exist on the system, even files owned by TrustedInstaller are vulnerable because the kaspersky antivirus service has SeRestorePrivileges enabled which grand WRITE_DAC to any file exist on the system (idk you can even execute kernel mode code if you managed to do everything correctly)

# Demo

[![](https://img.youtube.com/vi/DbuiFs6_oTo/0.jpg)](https://www.youtube.com/watch?v=DbuiFs6_oTo "Demo")
