# Trabalho realizado na Semana #3

## Identificação
- CVE-2022-28755
- The Zoom Client for Meetings (for Android, iOS, Linux, macOS, and Windows) before version 5.11.0 are susceptible to a URL parsing vulnerability.
- If a malicious Zoom meeting URL is opened, the malicious link may direct the user to connect to an arbitrary network address.
- Leads to additional attacks including the potential for remote code execution through launching executables from arbitrary paths.

## Catalogação
- Found in the 11th of August in 2022
- NVD: Base score 6.1 (medium) C:Low, I:Low, A: None
- CNA: Zoom Video Communications, Inc.; Base score 9.6 (critical) C:High, I:High, A:High
- Bug bounty: 250$ to 50.000$ 

## Exploit
- CWE-601 URL Redirection to Untrusted Site ('Open Redirect')	cwe source acceptance level NIST  
- CWE-20 Improper Input Validation

## Ataques
- There aren't any public documents that describes a successful attack using this exploit
- This attack can cause a significative amount of damage to all areas of CIA
