# RFC 117: Handling moderate/low severity security vulnerabilities in public

* RFC: 117
* Author: Matthew Westcott
* Created: 2026-10-29
* Last Modified: 2026-10-29

## Abstract

Following [the precedent set by the `curl` project](https://daniel.haxx.se/blog/2026/10/20/do-minor-curl-vulnerabilities-in-the-open/), we propose to revise our policy on [reporting security issues](https://docs.wagtail.org/en/latest/contributing/security.html) so that moderate and low severity vulnerabilities are handled through public issues and PRs. This is in recognition of AI assistance dramatically increasing the speed and effectiveness with which researchers can find vulnerabilities, to the point that keeping details private while working on fixes is no longer effective. Additionally, most of the vulnerabilities handled by Wagtail's security team involve improper access controls on editor-level user accounts, and are not exploitable by ordinary website visitors. This is a very different risk model to the one that the policy of secrecy is intended to address; namely, opportunists learning of a new exploit and scouring the public internet for vulnerable sites. Working on minor vulnerabilities through public GitHub pull requests will allow more people to participate in assessing and fixing them, allow for our CI testing infrastructure to be used in that process, and simplify the release process for security updates.

## Details

In October 2026 [the `curl` project announced that minor security vulnerabilities would now be tracked through public GitHub issues](https://daniel.haxx.se/blog/2026/10/20/do-minor-curl-vulnerabilities-in-the-open/). We believe that a similar move would be appropriate for Wagtail.

During the most recent round of security patch releases (7.0.10, 7.4.4, 8.0.1), we observed the same vulnerabilities being reported independently by multiple researchers. In the case of [advisory GHSA-jrgm-fhgv-hmj9](https://github.com/wagtail/wagtail/security/advisories/GHSA-jrgm-fhgv-hmj9), the vulnerability was reported by five separate researchers in the space of less than a month.

Evidently, AI assistance has dramatically increased the speed and ease with which researchers can discover vulnerabilities, often by instructing agents to review recent code changes and previous security advisories in search of similar issues. If those researchers who participate in responsible disclosure are able to arrive at the same findings so frequently, we must assume that malicious actors have access to the same information, and so keeping the findings private is no longer effective.

We can also note that out of the 22 security advisories published in the last year, 18 required a user account with access to the Wagtail admin, and only 4 were potentially exploitable by an ordinary visitor to the site. The most common class of vulnerability was improper validation allowing an editor-level user to perform editor actions outside of the specific area where they have permission. For many Wagtail installations, particularly smaller ones, this is not a risk profile that they need to be concerned about: either their editors are fully trusted, or where access controls are in place, they are granted uniformly across the whole site. For those sites that do make use of the permission model to segregate access for distinct sets of users, it is questionable whether the policy of keeping vulnerabilities secret until a patch is available is productive. Traditionally, this has been done to mitigate the risk of opportunist attackers learning of an unpatched exploit and scouring the public internet for vulnerable sites, and this is not applicable to vulnerabilities that require an admin account.

### Assessing severity levels

Currently, vulnerabilities are rated according to [Django's severity level guidelines](https://docs.djangoproject.com/en/6.1/internals/security/#security-issue-severity-levels). This will be revised to better reflect the risk profile of Wagtail. High severity vulnerabilities will include:

* Remote code execution
* SQL injection
* Any other attacks that could lead to privilege escalation without any user interaction, either from an unprivileged site visitor to a user with admin access, or from a user with admin access to administrator/superuser status
* Exposure of sensitive data to unprivileged site visitors
* Cross site scripting (XSS) by an unprivileged site visitor
* Loss of integrity (e.g. unauthorized modification of website content) initiated by an unprivileged site visitor

Moderate severity vulnerabilities will include:

* Cross site scripting (XSS) by a user with admin access
* Cross site request forgery (CSRF)
* Broken authentication

Low severity vulnerabilities will include:

* Denial-of-service attacks
* Exposure of sensitive data to users with admin access
* Issues requiring an uncommon configuration

### Changes to reporting process

High severity vulnerabilities should be reported via email to `security@wagtail.org`, as they are now. Moderate or low severity vulnerabilities should be opened as public GitHub issues, unless there is a strong reason for them to be handled privately. A new issue template will be created for this purpose, which will be tuned to the needs of security reports, and will tag the issue with the "component:Security" label. Where a reporter is unsure whether a vulnerability qualifies as high severity, they should email `security@wagtail.org`.

The security team will assess the severity of all reports received by email. If an issue is determined to be moderate or low severity, they will ask the reporter to open a public issue on GitHub, or open an issue themselves if the reporter does not do so within a few weeks.

### Changes to patching and release process

High severity vulnerabilities will be handled through GitHub's security advisory and temporary private fork facilities, in collaboration with the reporter(s), as they are now.

Moderate and low severity vulnerabilities will be handled through public GitHub pull requests. A security advisory will be prepared and published through GitHub to coincide with the release, along with requesting a CVE number from GitHub.

Fixes will be backported to all release series that are currently in security support. Releases containing one or more high severity fixes will be announced one week in advance on the Security Announcements section of GitHub discussions and on Slack, with this notice period potentially being reduced for time-sensitive issues. Releases containing only medium and/or low severity fixes will be released with no advance notice, and announced on the Security Announcements section of GitHub discussions and on Slack at the time of release.

### Benefits

Handling medium and low severity vulnerabilities through public GitHub issues will allow a wider audience to participate in assessing and fixing these issues, and potentially ease the security team's workload. It will also permit the use of our CI infrastructure to validate fixes, which is not possible when using temporary private forks. Finally, it will make the process of preparing security releases simpler and less error-prone, as the maintainer will be able to merge and push branches at their leisure and make full use of the GitHub UI, rather than having to work extensively on their local machine to minimise the time between the first public push to GitHub and the final PyPI release.

### Open questions

* What fields should the issue template for security reports contain? "Steps to reproduce" is definitely one, although that's in the standard issue template anyhow. It's probably better _not_ to prompt reporters for details such as severity, as we will make our own assessment on that, and researchers have a tendency to overstate this and write more than we would like in the name of justifying it. I recall that curl has some good guidelines on writing helpful security reports that would provide a good steer here.
