# Intern passordknekking: HelseCERT webinar

Nyttige linker:
Teknisk oversikt om passord i Windows - https://docs.microsoft.com/en-us/windows-server/security/kerberos/passwords-technical-overview
ntdsutil - https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343(v=ws.11)
impacket - https://www.secureauth.com/labs/open-source-tools/impacket/ - Kan slå ut på antivirus
Om passord ligger i "klartekst" i AD, "uhashet" bak kryptering - https://www.stigviewer.com/stig/windows_10/2019-01-04/finding/V-63429
Implementer no "NoLMHash policy - https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/prevent-windows-store-lm-hash-password


Del 1 - Eksport av passordhasher fra Active Directory (AD):
------

"One-liner" for snapshot av databasen for AD:
```
ntdsutil "activate instance ntds" ifm "create full c:\revisjon" quit quit
```
Installasjon av Windows Subsystem for Linux (Ubuntu installert som default)

Anbefales at dette ikke installeres på domene kontroller, men en egen dedikert maskin.
```
wsl --install
```
Alternativt kan man velge selv hvilken Linux distribusjon man ønsker.
```
wsl --list --online (Om man ønsker og installere målrettet pentest verktøy, feks kali linux)
wsl --install -d %DISTRO%
```
Installasjon av pip og impacket på Linux.
```
sudo apt install python3-pip
pip install python3-impacket
```
Eksempel på kjøring av impacket verktøyet
```
impacket-secretsdump -ntds path/to/ntds.dit -system path/to/SYSTEM -outputfile certweb_ad
```
Vask og anonymisering av passordhasher:
```
cat certweb_ad.ntds | cut -d : -f 4 > JustTheHashes.txt
```
