# FSI CVE-2020-6389

## Description

- Google Chrome Real-Time Communication (WebRTC) vulnerability allows heap corruption exploitation using as pivot crafted video streams.
- This vulnerability targets all desktop versions of Google Chrome before 80.0.3987.87.
- Since Google Chrome 80.0.3987.87 is available for all major OS, this bug affects Linux, Windows and macOS.

## Catalogue

- Firstly published on Google’s official report platform, [bugs.chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=1042933), it is also documented in the [CVE platform](https://nvd.nist.gov/vuln/detail/CVE-2020-6389).
- Natalie Silvanovich ([@natashenka](https://twitter.com/natashenka)). Member of Google Project Zero team. 16th January 2020.
- Since the bug reporter is a member of the Google team, there wasn’t bug-bounty. This bug has a high CVSS score (8.8).
- This vulnerability was fixed with the 80.0.3987.90 release of Google Chrome, which means it’s harmless in the current version.

## Exploit

- This exploit can be classified as a typical buffer overflow attack. It uses as a mean a memory leak on WebRTC.
- ```C++
    layer_info_it->second[temporal_idx] //is a size 5 vector.
    //temporal_idx is an integer [0-7].
  ```
- This allows two extra vector indexes to be exploited by an attacker to inject all sorts of malicious code.
- Google provides two example files to exploit the vulnerability and the commands to do it.

## Attacks

- All the details were found and released by the Google team, so they were able to resolve the problem internally.
- There isn't any information regarding the use of this vulnerability and its exploitation by the community.
- Since all this information was handled internally, no harm was reported.

**IMPORTANTE: Estamos a adicionar a informação sobre os CTF à posteriori tal como indicado pelo professor Manuel Barbosa, no email de dia 15/12/2021 trocado com o aluno up201704618**

## CTF #1 Resolution:

The first CTF was just a "Sanity Check". We should setup the FEUP VPN and test if we were able to access the CTF website on the IP: http://10.227.243.188/

After accessing the website we read the rules and collect the first flag:{SeOCovidFosseUmVirusDeComputadorEQueEra}.

After completing this task we score 24 points in the competition.

*FEUP 2021-12-15*



