# Intern passordknekking: HelseCERT webinar

Nyttige linker:
- Teknisk oversikt om passord i Windows - https://docs.microsoft.com/en-us/windows-server/security/kerberos/passwords-technical-overview
- Databaseverktøy for AD ntdsutil - https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343(v=ws.11)
- Dekryptering av AD database med impacket - https://www.secureauth.com/labs/open-source-tools/impacket/
- Om passord ligger i "klartekst" i AD, ("uhashet" bak kryptering") - https://www.stigviewer.com/stig/windows_10/2019-01-04/finding/V-63429
- Implementering av "no NoLMHash policy" - https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/prevent-windows-store-lm-hash-password
- Passordknekking med hashcat - https://hashcat.net/hashcat/
- Azure AD Password Protection - https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-password-ban-bad

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
Alternativt kan man velge selv hvilken Linux-distribusjon man ønsker. 
For å liste ut distribusjoner som er tilgjengelig online:
```
wsl --list --online (For å liste ut tilgjengelige Linux distribusjoner)
```
Eksempel på å installere Ubuntu:
```
wsl --install -d Ubuntu
```
Installasjon av impacket på Ubuntu(Debian basert)Linux.
```
sudo apt install python3-impacket
```
Eksempel på kjøring av impacket verktøyet (Linux):
```
impacket-secretsdump -ntds path/to/ntds.dit -system path/to/SYSTEM LOCAL -outputfile certweb_ad
```
Vask og anonymisering av passordhasher (Linux):
```
cat certweb_ad.ntds | cut -d : -f 4 > ntlm-hashes.txt
```

Del 2 - Revisjon av passord med Hashcat:

------
Sjekke hvilke hasher som er gjenbrukt:
```
sort ntlm-hashes.txt | uniq -c | grep -v ' 1 '
```

Hashcat 1:
`-m 1000` sier at vi skal knekke NTLM-hasher. `-a 0` betyr ordlisteangrep, med eller uten ekstra regler. `-r dive.rule` sier at vi skal bruke regelfila dive.rule. Hver regel i fila brukes på hvert ord i ordlista. En regel kan f.eks. være å gjøre legge til noen tall eller spesialtegn på slutten av ordet. Regelfila følger med hashcat, men kan også lastes ned her: https://github.com/hashcat/hashcat/blob/master/rules/dive.rule

Vær obs på at denne første kommandoen vil kunne ta atskillig lengre tid enn i demoen på webinaret, siden vi jukset litt for demoens skyld. Trykk 's' mens hashcat kjører for å se tid brukt og forventet tid gjenstående.

```
hashcat -m 1000 ntlm-hashes.txt -a 0 ordliste.txt -r dive.rule
```

Hashcat 2:
`-m 1000` sier at vi skal knekke NTLM-hasher. `-a 6` betyr ordlisteangrep med tegn lagt til på slutten. `?d?d?d?d` sier at vi skal legge til fire siffer på slutten av ordet. `-i` står for incremental, og gjør at man også prøver å legge til 1-3 siffer etter ordet. `-jc` gjør at første bokstav i ordet fra ordlista er stor, mens resten er små.

```
hashcat -m 1000 ntlm-hashes.txt -a 6 ordliste.txt ?d?d?d?d -i -jc
```

Hashcat 3:
Med `-a 3 ?u?l?l?l?d?d?d?d` prøver vi alle passord som begynner med en stor bokstav påfulgt av tre små bokstaver, påfulgt av fire siffer
```
hashcat -m 1000 ntlm-hashes.txt -a 3 ?u?l?l?l?d?d?d?d
```

Hashcat, vise crackede hasher og tilhørende passord:
```
hashcat -m 1000 ntlm-hashes.txt --show
```

Hashcat, vise kun cracked hasher, men ikke passord:
```
hashcat -m 1000 ntlm-hashes.txt --show | cut -d : -f 1
```

Å bruke hashcat effektivt er en egen liten kunst i seg selv. Her har vi nå gått for noen enkle kommandoer for å ta de verste passordene, men man kan alltids utvide reportoaret. Verktøyet er godt dokumentert og det finnes mange tutorials online.

