# FAQ &ndash; OS Linux

Linux se díky svému minoritnímu zastoupení na desktopech těší řádově menší pozornosti hackerů nežli Windows, většina malware pro Linux je určena serverům. Malware pro desktopové linuxové distribuce ovšem existuje &ndash; sice v mnohonásobně menším množství, ale existuje. Mimo jiné, exploit kity se poslední dobou snaží být co nejvíce multiplatformní a např. JS ransomware spolehlivě funguje i přes prohlížeč na Linuxu.

Zde se budeme věnovat pokročilejším možnostem zabezpečení (desktopového) Linuxu. Jako rukojmí použiji distribuci Arch Linux, která v základním nastavení není příliš zabezpečená, ale korektní konfigurací z ní lze vytvořit velmi bezpečnou instalaci. Kroky níže popisované jsou aplikovatelné na většinu distribucí, stačí korektně změnit syntax.

Tato sekce FAQ počítá s tím, že jste pročetli FAQ Linux pro méně pokročilé uživatele a máte znalosti ve zmíněné sekci rozebírané.

#### FAQ se dělí na několik sekcí:
- vrstvy zabezpečení
- anti-exploit mitigace
- audit
- ostatní doporučení

<br>

## Vrstvy zabezpečení:
### Firewall:
Pro běžné počítače stačí zakázat FORWARD chain a bezpečně nastavit INPUT.

Co se týče whitelistu odchozí komunikace (application FW), iptables není nejpříjemnější možností. Mnohem jednoduší je application FW implementovat skrz <abbr title="Mandatory Access Control">MAC</abbr>.

> Příklad pravidel pro běžný počítač (zadat do root konzole)

<pre><code># zamknout INPUT a FORWARD
iptables -P INPUT   DROP
iptables -P FORWARD DROP
iptables -A INPUT -p tcp -m tcp ! --tcp-flags SYN,RST,ACK SYN -m state --state NEW -j DROP

# zahodit INVALID pakety
iptables -N drop_invalid
iptables -A OUTPUT   -m state --state INVALID  -j drop_invalid
iptables -A INPUT    -m state --state INVALID  -j drop_invalid
iptables -A INPUT -p tcp -m tcp --sport 1:65535 --tcp-flags FIN,SYN,RST,PSH,ACK,URG NONE -j drop_invalid
iptables -A drop_invalid -j LOG --log-level debug --log-prefix "INVALID state -- DENY "
iptables -A drop_invalid -j DROP

# povolit ESTABLISHED a RELATED pripojeni
iptables -A INPUT  -m state --state ESTABLISHED,RELATED  -j ACCEPT

# povolit loopback
iptables -A INPUT -i lo -j ACCEPT

# logovat ostatni UDP
iptables -N In_RULE_1
iptables -A INPUT -p udp -m udp  -j In_RULE_1
iptables -A In_RULE_1  -j LOG  --log-level info --log-prefix "UDP -- DENY "
iptables -A In_RULE_1  -j DROP

# logovat ostatni TCP
iptables -N In_RULE_2
iptables -A INPUT -p tcp -m tcp  -j In_RULE_2
iptables -A In_RULE_2  -j LOG  --log-level info --log-prefix "TCP -- DENY "
iptables -A In_RULE_2  -j DROP

# logovat zbytek
iptables -N In_RULE_3
iptables -A INPUT  -j In_RULE_3
iptables -A In_RULE_3  -j LOG  --log-level info --log-prefix "XXX -- DENY "
iptables -A In_RULE_3  -j DROP

# ulozit pravidla
iptables-save > /etc/iptables/iptables.rules

# povolit iptables
systemctl enable iptables</code></pre>

<br>

### Mandatory Access Control:
<abbr title="Mandatory Access Control">MAC</abbr> se stal důležitou součástí bezpečnostního modelu linuxových distribucí.

**RBAC** je součástí grsecurity patchsetu (viz. níže), jedná se o velmi robustní implementaci MAC, nehodí se ovšem na *rolling release* model OS.

**SELinux** je velmi robustní implementace MAC, její nastavení je ovšem problematické. Využívá ji např. **<span class="fe">Fedora</span>**  a je důelžitou součástí bezpečnostního modelu OS Android.

**AppArmor** je implementace MAC poskytující nižší úroveň ochrany než SELinux (např. neumí omezit ioctl). Využívá ji např. **<span class="os">openSUSE</span>** a **<span class="ub">Ubuntu</span>**.

**TOMOYO Linux** je velmi solidní implementace MAC poskytující vyšší úroveň ochrany než AppArmor a zároveň nabízí mnohem jednodušší konfiguraci nežli SELinux.

Na Arch Linux není problém provozovat RBAC, TOMOYO nebo AppArmor. SELinux je o něco problematičtější. Kernel **linux-grsec** má ve výchozím nastavení povolený TOMOYO Linux a RBAC. Pro použití AppArmor je nutné kernel zkompilovat.

> Instalace TOMOYO Linux

![idea](https://mople71.cz/sm/idea.gif) TOMOYO Linux není příliš rozšířený MAC a velmi těžko někde naleznete profily pro aplikace. Budete si je tedy muset sami vytvořit (příp. přepsat z AppArmor profilů &ndash; ty jsou všude).

![idea](https://mople71.cz/sm/idea.gif) Dokumentaci k TOMOYO Linux naleznete <a href="http://tomoyo.osdn.jp/2.5/index.html.en" target="_blank">zde</a>.

- Nepoužíváte-li **linux-grsec** kernel, musíte si zkompilovat vlastní. Návod na kompilaci jádra naleznete v sekci <a href="#lnx2.1">Grsecurity</a>. Případně můžete využít předkompilovaný kernel z <a href="https://aur.archlinux.org/packages/linux-lts-tomoyo/" target="_blank">AUR</a>.
- Povolte TOMOYO Linux v GRUB:
<li style="list-style-type: none"><pre><code>/etc/default/grub
-----------------------------------

GRUB_CMDLINE_LINUX_DEFAULT="quiet security=tomoyo TOMOYO_trigger=/usr/lib/systemd/systemd"</code></pre></li>
- Aktualizujte GRUB (grub-mkconfig -o /boot/grub/grub.cfg).
- Nainstalujte **tomoyo-tools**:
<li style="list-style-type: none"><pre><code>sudo pacman -S gnupg
gpg --recv-keys 43C83369623D7AD3A96C2FC7425F128D0C64F52A
git clone https://aur.archlinux.org/tomoyo-tools.git
cd tomoyo-tools
gedit PKGBUILD</code></pre></li>
- Pokud nepoužíváte kernel z AUR, odstraňte závislost na balíčku **linux-tomoyo**.
- Uložte a spusťte instalaci:
<li style="list-style-type: none"><pre><code>makepkg -si</code></pre></li>
- Restartujte OS.

> Použití TOMOYO Linux jako aplikační FW

- Po úspěšné instalaci a restartu OS inicializujte TOMOYO:
<li style="list-style-type: none"><pre><code>sudo /usr/lib/tomoyo/init_policy</code></pre></li>
- Následně upravte pravidla TOMOYO:
<li style="list-style-type: none"><pre><code>/etc/tomoyo/policy/current/profile.conf
-----------------------------------

PROFILE_VERSION=20110903
0-COMMENT=-----Disabled Mode-----
0-PREFERENCE={ max_audit_log=1024 max_learning_entry=2048 }
0-CONFIG={ mode=disabled grant_log=no reject_log=yes }
0-CONFIG::network::unix_stream_bind={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_stream_listen={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_stream_connect={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_dgram_bind={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_dgram_send={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_seqpacket_bind={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_seqpacket_listen={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network::unix_seqpacket_connect={ mode=disabled grant_log=no reject_log=no }
0-CONFIG::network={ mode=enforcing grant_log=no reject_log=yes }
1-COMMENT=-----Disabled Mode (net access)-----
1-PREFERENCE={ max_audit_log=1024 max_learning_entry=2048 }
1-CONFIG={ mode=disabled grant_log=no reject_log=no }
2-COMMENT=-----Learning Mode-----
2-PREFERENCE={ max_audit_log=1024 max_learning_entry=2048 }
2-CONFIG={ mode=learning grant_log=no reject_log=yes }
3-COMMENT=-----Permissive Mode-----
3-PREFERENCE={ max_audit_log=1024 max_learning_entry=2048 }
3-CONFIG={ mode=permissive grant_log=no reject_log=yes }
4-COMMENT=-----Enforcing Mode-----
4-PREFERENCE={ max_audit_log=1024 max_learning_entry=2048 }
4-CONFIG={ mode=enforcing grant_log=no reject_log=yes }</code></pre></li>
- Restartujte OS.
- Otevřete konfiguraci TOMOYO pro aplikace:
<li style="list-style-type: none"><pre><code>sudo tomoyo-editpolicy</code></pre></li>
- Šipkami se posunujete mezi aplikacemi. Profil aplikace změníte klávesou <span class="red">S</span>, zadáním čísla profilu a stisknutím **Enter**.
<li style="list-style-type: none">![exclaim](https://mople71.cz/sm/exclaim.gif) TOMOYO detekuje pouze aplikace, které byly od jeho aktivace alespoň 1x spuštěny.</li>
<li style="list-style-type: none"><pre><code>0     #bez přístupu k internetu
1     #s přístupem k internetu
</code></pre></li>
- Klávesou <span class="red">Q</span> konfiguraci ukončíte.
- Po dokončení konfigurace ji následně uložte:
<li style="list-style-type: none"><pre><code>sudo tomoyo-savepolicy</code></pre></li>

> RBAC

![idea](https://mople71.cz/sm/idea.gif) Více informací naleznete <a href="https://wiki.archlinux.org/index.php/Grsecurity#RBAC" target="_blank">zde</a>.

- Pro ovládání RBAC potřebujete následující balíček:
<li style="list-style-type: none"><pre><code>pacman -S gradm</code></pre></li>
- Zvolte si heslo a RBAC povolte. (-P, -E)
- Máte-li zájem o "autolearn" mód, kdy RBAC bude pozorovat aktivity OS a nakonec dle nich vytvoří svá pravidla, můžete jej zapnout následovně:
<li style="list-style-type: none"><pre><code>gradm -F -L /etc/grsec/learning.log</code></pre></li>
- Po minimálně 3 dnech aktivního používání můžete vytvořit pravidla na základě akcí v OS.
<li style="list-style-type: none"><pre><code>gradm -D
gradm -F -L /etc/grsec/learning.log -O /etc/grsec/learning.roles
# zauditujte /etc/grsec/learning.roles, jestli jsou vsechna pravidla v poradku a nic duleziteho nechybi
mv /etc/grsec/learning.roles /etc/grsec/policy
chmod 0600 /etc/grsec/policy
gradm -E</code></pre></li>

<br>

### Virtualizace:
Virtualizace může být velmi bezpečný způsob ochrany před malware (záleží na způsobu implementace), jelikož odděluje požadovanou část OS od fyzického OS.

Sandbox nativně integrovaný v aplikaci je nejúčinnější možností implementace sandboxu, jelikož je nastaven přesně na míru dané aplikaci.

Externí sandbox není zdaleka tak účinný jako sandbox integrovaný v aplikaci a ponechává výrazně větší prostor pro exploitaci, ale stále je mnohonásobně lepší, než žádný sandbox. Jsou případy, kdy lze špatně implementovaný externí sandbox prolomit přes *PulseAudio*...

**Flatpak** i **firejail** jsou rozebírány v FAQ pro méně pokročilé.

Pro pokročilou virtualizaci za použití **KVM** se podívejte <a href="https://wiki.archlinux.org/index.php/QEMU" target="_blank">zde</a>.

<br><br><hr><br>

## Anti-exploit mitigace:
### Grsecurity:
Grsecurity je bezpečnostní patchset pro kernel nabízející řadu mitigací proti exploitům. Obsahuje také vestavěný MAC &ndash; RBAC. Kompletní výpis funkcí naleznete na <a href="https://grsecurity.net/features.php" target="_blank">oficiálním webu</a>.

Kernel s grsecurity nabízí mnoho distribucí &ndash; Alpine, Hardened Gentoo, Subgraph, <a href="https://www.archlinux.org/packages/community/x86_64/linux-grsec/" target="_blank">Arch Linux</a>, <a href="https://packages.debian.org/search?keywords=linux-image-grsec" target="_blank">Debian</a>,...

<span class="red">Bohužel, grsecurity patchsety již nebudou zdarma, bezplatně dostupné patchsety končí s LTS verzí kernelu 4.9.</span>

![arrow](https://mople71.cz/sm/arrow.gif) Jakmile zmíněný kernel dosáhne EOL, zřejmě bude nutné si za patchsety platit...

> Instalace Grsecurity

- Otevřete root konzoli a zadejte:
<li style="list-style-type: none"><pre><code>pacman -S linux-grsec</code></pre></li>
- Po úspěšné instalaci aktualizujte bootloader, pokud máte GRUB, stačí jednoduchý příkaz:
<li style="list-style-type: none"><pre><code>grub-mkconfig -o /boot/grub/grub.cfg</code></pre></li>
- Restartujte OS a vyzkoušejte, zdali pro vás výchozí nastavení funguje.

> Paxd

- Pro výchozí aktivaci mitigací proti *memory corruption* exploitům potřebujete následující balíček:
<li style="list-style-type: none"><pre><code>pacman -S paxd</code></pre></li>
- Problematickým binárkám poté můžete nastavit výjimku následujícím způsobem (např.):
<li style="list-style-type: none"><pre><code>setfattr -n user.pax.flags -v "m" /home/user/.local/share/Steam/ubuntu12_32/steam</code></pre></li>

> Ruční konfigurace a kompilace kernelu s Grsecurity

![idea](https://mople71.cz/sm/idea.gif) Konfigurace patchsetu v předkompilovaném balíčku může být příliš striktní a nemusí se vám podařit nabootovat. V takovém případě je třeba identifikovat problém a kernel si následně zkompilovat ručně. Arch Linux to umožňuje velmi snadno díky *ABS*.

- Nainstalujte si ABS a GPG:
<li style="list-style-type: none"><pre><code>sudo pacman -S abs gnupg
sudo abs
cp -r /var/abs/community/linux-grsec ~/linux-grsec
cd ~/linux-grsec
gedit PKGBUILD    #nahradte vami pouzivanym editorem</code></pre></li>
- V PKGBUILD nalezněte <span class="green">#make menuconfig</span>, příkaz odkomentujte, uložte a zavřete.
<li style="list-style-type: none"><pre><code>gpg --recv-keys 79BE3E4300411886
gpg --recv-keys 38DBBDC86092693E
gpg --recv-keys 44D1C0F82525FE49
makepkg -s</code></pre></li>
- Po chvíli se zobrazí menu s možnostmi kernelu. Zvolte, co potřebujete, a zrušte, co nepotřebujete. Celkově je dobrý nápad z kernelu vyházet věci, které nepotřebujete, docílíte tím snížení prostoru pro exploitaci.
- 2x ESC a uložte konfiguraci. Tím započnete kompilaci kernelu.
- Po kompilaci kernelu balíček nainstalujte:
<li style="list-style-type: none"><pre><code>sudo pacman -U linux-grsec-*.pkg.tar.xz
sudo pacman -U linux-grsec-headers-*.pkg.tar.xz</code></pre></li>
- Aktualizujte GRUB:
<li style="list-style-type: none"><pre><code>sudo grub-mkconfig -o /boot/grub/grub.cfg</code></pre></li>
- Nastavte ignorování aktualizací pro linux-grsec (a linux-grsec-headers):
<li style="list-style-type: none"><pre><code>/etc/pacman.conf
-----------------------------------

# Pacman won't upgrade packages listed in IgnorePkg and members of IgnoreGroup
IgnorePkg   = linux-grsec linux-grsec-headers
#IgnoreGroup =</code></pre></li>
- Tento proces budete muset zopakovat při každé aktualizaci kernelu, pacman vás na aktualizaci při *-Syu* upozorní.

<br>

### Hardening aplikací:
Balíčky mohou být kompilovány s *memory corruption* mitigacemi (ASLR, PIE, RELRO,...), které následně významně ztěžují jejich exploitaci.

Jediná distribuce, která má balíčky velmi vysoké úrovně s  důležitými *memory corruption* mitigacemi, je <span class="fe">Fedora</span> (+ RHEL, CentOS).

Pro plnou funkčnost ASLR musí být všechny běžící procesy zkompilovány jako **PIE**. Poté se bude jednat o velmi robustní implementaci &ndash; alespoň tedy na platformě *x86_64*. Na 32-bit OS není problém ASLR prolomit pomocí brute-force.

Balíčky neobsahující zmíněné mitigace je tedy nutné zkompilovat ručně. Repozitáře Arch Linux obsahují **hardening-wrapper**, který automaticky nastaví kompilátor a veškeré kompilované balíčky po jeho instalaci budou kompilovány s dostupnými mitigacemi.

> Audit mitigací běžících procesů

![idea](https://mople71.cz/sm/idea.gif) Pro audit použijeme skript **checksec**, který je podrobněji rozebírán níže.

<pre><code>checksec --proc-all</code></pre>

> Instalace hardening-wrapper

<pre><code>pacman -S hardening-wrapper</code></pre>

> Kompilace aplikace s mitigacemi:

- Nainstalujte si ABS a GPG:
<li style="list-style-type: none"><pre><code>sudo pacman -S abs gnupg
sudo abs
cp -r /var/abs/extra/networkmanager ~/networkmanager
cd ~/networkmanager</code></pre></li>
- Zahajte kompilaci a následně automatickou instalaci:
<li style="list-style-type: none"><pre><code>makepkg -si</code></pre></li>
- Po úspěšné instalaci restartujte OS.

> Automatizace kompilace problémových balíčků:

- Použijte **srcpac**.
<li style="list-style-type: none"><pre><code>sudo pacman -S srcpac
man srcpac</code></pre></li>

<br><br><hr><br>

## Audit:
### Rootkit Hunter:
Rootkit Hunter je on-demand skener rootkitů, který umí zachytávat změny v důležitých souborech OS atd. Jedná se o vcelku solidní aplikaci určenou primárně pro servery, neztratí se ovšem ani na desktopu.

> Návod

- Instalace a spuštění auditu:
<li style="list-style-type: none"><pre><code>pacman -S rkhunter
rkhunter --versioncheck
rkhunter --propupd
rkhunter -c --enable all --disable none --rwo</code></pre></li>
- Rkhunter vypíše veškerá svá varování, pravděpodobně false positives. Ty je potřeba opravit, aby se již v budoucnu nezobrazovaly.
- Příklad mých úprav:
<li style="list-style-type: none"><pre><code>gedit /etc/rkhunter.conf
-----------------------------------
#
# FP Fix
SCRIPTWHITELIST="/usr/bin/egrep"
SCRIPTWHITELIST="/usr/bin/fgrep"
SCRIPTWHITELIST="/usr/bin/ldd"
SCRIPTWHITELIST="/usr/bin/vendor_perl/GET"
ALLOWDEVFILE="/dev/shm/pulse-shm-*"
ALLOWDEVFILE="/dev/shm/user-Shm_*"
ALLOWDEVFILE="/dev/shm/user-Valve*"
ALLOWHIDDENFILE="/etc/.updated"
ALLOWHIDDENFILE="/usr/share/man/man5/.k5identity.5.gz"
ALLOWHIDDENFILE="/usr/share/man/man5/.k5login.5.gz"</code></pre></li>
- Nyní byste již měli být bez false positive varování a rkhunter by měl být plně funkční. Má uloženou databázi kritických souborů OS, takže v případě jakékoli změny (např. pomocí malware) zobrazí varování. Sken můžete dát do cronu.
- Více informací naleznete na <a href="http://rkhunter.sourceforge.net/" target="_blank">oficiálních stránkách</a> a <a href="https://www.digitalocean.com/community/tutorials/how-to-use-rkhunter-to-guard-against-rootkits-on-an-ubuntu-vps" target="_blank">zde</a>.

<br>

### Lynis:
Lynis je špičková aplikace umožňující audit mnoha OS založených na UNIXu, včetně Linuxu. Provádí velmi detailní audit OS a po dokončení auditu zobrazí doporučení pro zvýšení bezpečnosti.

> Návod

- Instalace a spuštění auditu:
<li style="list-style-type: none"><pre><code>pacman -S lynis
lynis update info
lynic -c</code></pre></li>

<br>

### Checksec:
Checksec je skript určený pro kontrolu nastavení kernelu a zobrazení *memory corruption* mitigací spustitelných souborů.

> Návod

- Instalace:
<li style="list-style-type: none"><pre><code>pacman -S checksec</code></pre></li>
- Můžete auditovat bezpečnost nastavení kernelu:
<li style="list-style-type: none"><pre><code>checksec --kernel</code></pre></li>
- Dále můžete auditovat bezpečnostní funkce balíčků:
<li style="list-style-type: none"><pre><code>checksec --proc-all</code></pre></li>
- Více informací naleznete na <a href="https://github.com/slimm609/checksec.sh" target="_blank">oficiálních stránkách</a>.

<br><br><hr><br>

## Ostatní doporučení:
- používejte Wayland, implementace X.org nejsou bezpečné (jedině Xenocara v OpenBSD)
- používejte Flatpak
- na běžné prohlížení webu používejte Chromium, na bankovnictví např. Epiphany
- Firefox používejte na prohlížení PDF souborů (pdf.js je velmi bezpečný způsob prohlížení)
- ...

<br><br><hr>

<h3 class="nocol">To je vše. Stay safe! ![smile](https://mople71.cz/sm/smile.gif)</h3>