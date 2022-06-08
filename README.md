# Intern passordknekking: HelseCERT webinar

Nyttige linker:
- Teknisk oversikt om passord i Windows - https://docs.microsoft.com/en-us/windows-server/security/kerberos/passwords-technical-overview
- Databaseverktøy for AD ntdsutil - https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343(v=ws.11)
- Dekryptering av AD database - https://www.secureauth.com/labs/open-source-tools/impacket/
- Om passord ligger i "klartekst" i AD, ("uhashet" bak kryptering") - https://www.stigviewer.com/stig/windows_10/2019-01-04/finding/V-63429
- Implementering av "no NoLMHash policy" - https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/prevent-windows-store-lm-hash-password
- Passordknekking med hashcat - https://hashcat.net/hashcat/

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
For å liste ut distrubusjoner som er tilgjengelig online:
```
wsl --list --online (For å liste ut tilgjenlige Linux distribusjoner)
```
Eksempel på og installere Ubuntu:
```
wsl --install -d Ubuntu
```
Installasjon av impacket på Ubuntu(Debian basert)Linux.
```
sudo apt install python3-impacket
```
Eksempel på kjøring av impacket verktøyet (Linux):
```
impacket-secretsdump -ntds path/to/ntds.dit -system path/to/SYSTEM -outputfile certweb_ad
```
Vask og anonymisering av passordhasher (Linux):
```
cat certweb_ad.ntds | cut -d : -f 4 > bareNTLM.txt
```

Del 2 - Revisjon av passord med Hashcat:
------
