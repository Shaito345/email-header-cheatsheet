# Find the Sending IP Address in Email Headers

Email headers hold IP addresses, but not every address belongs to the sender. This guide shows which Received line to check first, which addresses to skip and why the top line is not always the sender. Examples use the sample files in this repo, which carry documentation IP addresses only.

*Last reviewed 24 November 2026.*

## Short Answer

Open the full header and find the top Received line. Your own mail server added it. RFC 5321 says the FROM clause of that line should hold the host name the sending client gave and the client IP address taken from the TCP connection.

That address belongs to the machine that handed the message to your server. It can be a mail server, a mailing list or a forwarder. It is the sender's own device only when the message came straight from that device.

## Get the Full Header

| Mail app | Where the header is | Source |
|---|---|---|
| Gmail on a computer | Open the email, select More beside Reply, then Show original | Google Help |
| Classic Outlook for Windows | Double-click the message to open it in its own window, then File then Properties, and look in the Internet headers box | Microsoft Support |
| New Outlook and Outlook on the web | Open the message, select More actions, View, then View message details | Microsoft Support |
| Yahoo Mail | Select the email, then More, then View Raw Message | Listed by Google Help. We did not check Yahoo's own help page |

Menu names change between versions. Copy the full header into a text file before you start.

## List the Received Lines

RFC 5321 says a mail server adds its Received line at the top and does not change the lines below it. The newest hop sits at the top and the oldest hop sits at the bottom.

## Find the First External Address

The first external address sits in the top Received line, the one your own server added. This is sample file `01-clean-message.txt`.

```
Received: from mail.example.org (mail.example.org [203.0.113.25])
        by mx.example.com with ESMTPS; Tue, 06 Oct 2026 09:14:02 +0000
Received: from workstation.example.net ([198.51.100.7])
        by mail.example.org with ESMTPA; Tue, 06 Oct 2026 09:13:58 +0000
```

| Line | Added by | Address shown | What it means |
|---|---|---|---|
| Top | `mx.example.com` (your server) | `203.0.113.25` | The server that handed the message to you. This is the first external address |
| Second | `mail.example.org` | `198.51.100.7` | The address `mail.example.org` recorded for the machine that sent to it. Your server did not see this connection |

A provider can leave the device address out of the second line. Check what your own header shows and do not assume it is there.

## Skip Addresses That Cannot Be the Sender

| Range | What it is | Source |
|---|---|---|
| `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | Blocks reserved for private networks | RFC 1918 |
| `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` | Blocks provided for use in documentation | RFC 5737 |
| `2001:db8::/32` | The IPv6 prefix reserved for documentation | RFC 3849 |

A private address shows an internal hop and not a public address. RFC 5737 says the documentation blocks should not appear on the public Internet. A real message that shows one deserves a closer look.

## Why the Top Line Is Not Always the Sender

Sample file `02-forged-sender.txt` shows a message that claims to come from `billing@example.net`.

```
Received: from unknown (HELO mailer.example.org) ([192.0.2.77])
        by mx.example.com with ESMTP; Wed, 07 Oct 2026 03:02:41 +0000
```

The address `192.0.2.77` is the server that delivered the message. It says nothing about who typed the From line. In the sample file the DMARC check fails for `example.net`.

Sample file `03-mailing-list-forward.txt` shows a different case.

```
Received: from lists.example.org (lists.example.org [198.51.100.40])
        by mx.example.com with ESMTPS; Thu, 08 Oct 2026 10:20:15 +0000
```

Here `198.51.100.40` is the mailing list server that sent the message again. It is not the author's mail server or device.

## List the Addresses With a Short Script

This Python script lists the IPv4 addresses in square brackets for every Received line in a file. We tested it with Python 3.13 on the three sample files.

```
import re
import sys
from email import message_from_file, policy

with open(sys.argv[1]) as f:
    msg = message_from_file(f, policy=policy.default)

for i, line in enumerate(msg.get_all("Received", []), start=1):
    flat = " ".join(str(line).split())
    ips = re.findall(r"\[(\d{1,3}(?:\.\d{1,3}){3})\]", flat)
    print(i, ips)
```

Save it as `list_ips.py` and start it with `python3 list_ips.py 01-clean-message.txt`. Line 1 is the top Received line. For the first sample it printed `1 ['203.0.113.25']` and `2 ['198.51.100.7']`.

The script finds IPv4 addresses only. It misses IPv6 addresses and addresses written without square brackets.

## Our Notes

These points are ours and are not quotes from the sources.

- An IP lookup tells you which organization holds an address block. It does not tell you who used the address at that moment.
- A VPN or a proxy shows its own address in place of the user's address.
- Lines below the top one come from systems you do not control. RFC 5322 calls trace fields strictly informational.
- Treat the address as a lead and not as proof of who wrote the message.

## Legal and Privacy Note

This note is general information and not legal advice. Check the rules in your country before you collect or share email and IP data, and review only mail that you own or are authorized to review.

Do not publish the name, address or IP of a person you think sent a message. A header can point at the wrong person. The longer note in [Can You Trace an Email Address to Its Owner](trace-email-sender.md) covers more.

## Sources

- [Google Help: Trace an email with its full header](https://support.google.com/mail/answer/29436)
- [Google Help: Full headers for other mail services](https://support.google.com/mail/answer/8156)
- [Microsoft Support: View internet message headers in Outlook](https://support.microsoft.com/office/cd039382-dc6e-4264-ac74-c048563d212c)
- [RFC 5321, section 4.4 Trace Information](https://datatracker.ietf.org/doc/html/rfc5321#section-4.4)
- [RFC 5322, section 3.6.7 Trace Fields](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.7)
- [RFC 1918: Address Allocation for Private Internets](https://datatracker.ietf.org/doc/html/rfc1918)
- [RFC 5737: IPv4 Address Blocks Reserved for Documentation](https://datatracker.ietf.org/doc/html/rfc5737)
- [RFC 3849: IPv6 Address Prefix Reserved for Documentation](https://datatracker.ietf.org/doc/html/rfc3849)

## Related Reading

[How to Trace Email Sender IP Address in Outlook? Step By Step Explained](https://www.mailxaminer.com/blog/how-to-trace-email-sender-ip-address-in-outlook/?utm_source=github&utm_medium=repo&utm_campaign=email-header-cheatsheet&utm_content=ip-from-headers) is our article on the MailXaminer site. It covers the Outlook steps and the SPF, DKIM and DMARC results, and it warns that an IP address in a header is only a starting point.

## Related Guides

- [Can You Trace an Email Address to Its Owner](trace-email-sender.md)
- [Message-ID in Email Headers and How to Look It Up](message-id.md)
- [Email Header Cheat Sheet](../README.md)
