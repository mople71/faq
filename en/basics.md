# FAQ &ndash; Security Basics

## Glossary:
- <span class="green">hacker</span> &ndash; person (mis)using their knowledge (eg. in computer security area) for personal, usually illegal, profit; this definition was originally taken from the word *cracker*
- <span class="green">malware</span> &ndash; generic term for malicious software further organized into many (more or less malicious) subgroups
- **adware** &ndash; malware containing ads and showing them to the user
- **bundleware** &ndash; superfluous (not necessarily malicious) software bundled to other software
- **foistware** &ndash; software installed without user's knowledge, usually trying to convince users that the OS isn't properly working and offers remedy for a price
- **rootkit** &ndash; code that hides presence of malware on the device and aggravates its detection
- **exploit** &ndash; code taking advantage of a vulnerability in SW (including OS) to perform a specific action
- **zero-day (0-day)** &ndash; vulnerability exploited at the day of its disclosure or prior to it
- **APT** &ndash; &bdquo;Advanced Persistent Threat&ldquo;; advanced threat usually tailored for a specific target, common users usually don't bump into these
- <span class="green">payload</span> &ndash; core piece of malware code that performs the crucial action

<br>

## Security Basics:
- Periodically backup your data.
- Don't use illegal software &ndash; the majority of cracks found on internet is infected
- Prior to any action, double check its autencity.
- Download SW only from its official website.
- Use strong passwords that are easy to remember:
<li style="list-style-type: none">![password_strength_sm](https://faq.mople71.cz/img/en/passwd.png)</li>
- Use different passwords for different services, consider using a [keyring](#basics7).
- Don't connect to unknown/unsafe networks, or at least never send any personal data through them and avoid using dangerous protocols (HTTP, FTP atc.). Ideally use a VPN.
- Pay attention to what data you enter and where you enter them &ndash; selling personal data is a profitable business.
- Don't forget the physical security aspect &ndash; lock your device, set a UEFI password, disable boot menu etc.

<br>

## Web Browsing Safely:
- Use a securely configured web browser (see other FAQ chapters).
- Consider using a separate browser for personal things like online banking etc.
- Don't visit unknown/untrusted sites and never download any files from them.
- Reduce visiting 18+ sites as it's quite common for them to serve malicious ads and code.
- Don't use social buttons outside the specific social network, these can be trivially mimicked.
- Avoid the **short URLs** &ndash; like https://bit.ly/tinyurlwiki etc. They can easily mask dangerous links.
- Avoid using unsafe network SW such as <span class="red">Flash Player</span> or <span class="red">Java</span>, its exploitation is very common. At least disable it inside your browser.
- Only open email attachments from trusted senders.
- If you suspect a file to be malicious, always scan it before opening with eg.  [VirusTotal](https://www.virustotal.com/).

![idea](https://mople71.cz/img/sm/idea.gif) The safest way to browse the web: <span class="green">securely configured live OS</span>. However, it should be pointed out that this option doesn't have to be 100% safe as well &ndash; for example the EFI can be exploited if the device was infected in the past.

<br>

## Security News Sources:
- [BleepingComputer](https://www.bleepingcomputer.com/)
- [The Hacker News](http://thehackernews.com/)
- [Threatpost](https://threatpost.com/)
- [Naked Security](https://nakedsecurity.sophos.com/)
- [LinuxSecurity](http://www.linuxsecurity.com/)

<br>

## Online File Analysis:
- [VirusTotal](https://www.virustotal.com/) &ndash; AV database
- [Metadefender](https://www.metadefender.com/) &ndash; AV database
- [Hybrid Analysis](https://www.reverse.it/) &ndash; sandbox + VT
- [Malwr](https://malwr.com/submission/) &ndash; Cuckoo sandbox

<br>

## Recommended VPN:
- [Freedome VPN](https://www.f-secure.com/en/web/home_global/freedome/) &ndash; blocks malware, trackers and ads
- [AirVPN](https://airvpn.org/) &ndash; advanced features, quite trustworthy
- [IVPN](https://www.ivpn.net/) &ndash; trustworthy
- [Cryptostorm](https://cryptostorm.is/) &ndash; trustworthy

<br>

## Recommended Keyrings:
- [1Password](https://1password.com/) &ndash; impeccalbe platform, paid
- [KeePassX](https://www.keepassx.org/) &ndash; free
- [Encryptr](https://spideroak.com/solutions/encryptr/) &ndash; free