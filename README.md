# Email Header Cheat Sheet for Investigators and Students

Email headers show how a message travelled, which servers touched it and whether the sender passed authentication checks. This cheat sheet explains every common field in plain words, shows how to trace a message and lists the clues that point to a forged sender. Examples use reserved example domains and documentation IP addresses, so no real person or company appears.

*Last reviewed 20 October 2026.*

> We are the team behind MailXaminer at SysTools. Two links in the tools section point to our site and we mark them. Nothing here is legal advice.

## Contents

- [Quick Start](#quick-start)
- [Get the Full Header](#get-the-full-header)
- [Anatomy of a Header](#anatomy-of-a-header)
- [Header Field Reference](#header-field-reference)
- [Trace the Route a Message Took](#trace-the-route-a-message-took)
- [Convert Times to UTC](#convert-times-to-utc)
- [Understand SPF, DKIM, DMARC and ARC Results](#understand-spf-dkim-dmarc-and-arc-results)
- [Check the DNS Records Yourself](#check-the-dns-records-yourself)
- [Spot a Forged Sender](#spot-a-forged-sender)
- [Worked Examples](#worked-examples)
- [Keep the Header Safe as Evidence](#keep-the-header-safe-as-evidence)
- [Tools for Header Analysis](#tools-for-header-analysis)
- [FAQ](#faq)
- [Glossary](#glossary)
- [Sources](#sources)
- [Contributing](#contributing)
- [License](#license)

## Quick Start

1. Get the full header from the original message. Forwarded copies and screenshots do not carry it.
2. Start at the lowest Received line and move upward. That line is the first hop. The top line comes from your own server.
3. Check the Authentication-Results line for SPF, DKIM and DMARC.
4. Compare the From address with the Return-Path and Reply-To addresses.
5. Convert every time to UTC before you compare messages.
6. Save the raw header and hash it before you write a single note.

## Get the Full Header

Every mail client hides the header by default. Menu names change between versions, so look for words like "Show original", "Properties", "All Headers" or "View source" if your menu looks different.

| Client | Steps |
|---|---|
| Gmail on a computer | Open the email. Select More beside the Reply button, then Show original. Select Copy to clipboard. |
| Outlook (classic, Windows) | Double click the message. Select File, then Properties. Find the header text in the Internet headers box. |
| New Outlook | Open the message. Select More actions, then View, then View message details. |
| Apple Mail | Select the message. Go to View, then Message, then All Headers. Choose Default to switch back. |
| Thunderbird | Select the message. Select More, then View source. |
| Yahoo Mail | Open the email. Select the three dots in the top toolbar, then View Raw Message. |

Three habits keep the header clean.

- Work from the original message, never from a forward.
- Copy the text. Do not retype it and do not tidy it.
- Where your client lets you save the message as an EML file, save one and keep it untouched.

Google explains its steps on its own help page (<https://support.google.com/mail/answer/29436>).

## Anatomy of a Header

Headers sit as a block of lines above the message body. Every server that handles the message adds lines at the top. The oldest line sits at the bottom.

This sample is made up. The full file is in [examples/01-clean-message.txt](examples/01-clean-message.txt).

```
Return-Path: <billing@example.net>
Received: from mail.example.org (mail.example.org [203.0.113.25])
        by mx.example.com with ESMTPS; Tue, 06 Oct 2026 09:14:02 +0000
Received: from workstation.example.net ([198.51.100.7])
        by mail.example.org with ESMTPA; Tue, 06 Oct 2026 09:13:58 +0000
Authentication-Results: mx.example.com;
        spf=pass smtp.mailfrom=example.net;
        dkim=pass header.d=example.net;
        dmarc=pass header.from=example.net
From: "Accounts Team" <billing@example.net>
To: client@example.com
Message-ID: <20261006091358.1a2b3c@example.net>
Date: Tue, 06 Oct 2026 09:13:55 +0000
Subject: Invoice 4471
```

What the sample tells you.

- Hop 1 is the bottom Received line. The message left a workstation at 09:13:58 UTC.
- Receiving server mx.example.com added the top Received line. You can trust it most because your side wrote it.
- Times move forward as you go up the page. Any line that jumps backward needs an explanation.
- Mail server mx.example.com checked SPF, DKIM and DMARC and wrote down the results.

## Header Field Reference

| Field | What it holds | What to check |
|---|---|---|
| Received | One line for every server hop, with a time. Defined in RFC 5321 and RFC 5322. | Fake lines added below the first server you trust |
| Return-Path | Where bounces go. This is the envelope sender from RFC 5321. | A domain that differs from the From line |
| From | The name and address the reader sees | Anyone can type this. Check it against the authentication results. |
| Reply-To | Where a reply goes | A different domain from the From line |
| Sender | The sender when it differs from the author | A mailbox that does not match the author |
| To, Cc | Visible recipients. Bcc recipients never appear. | A missing To line means the sender hid the recipients |
| Message-ID | A unique ID made by the sending system | A domain that does not match the sender |
| Date | When the sender's system stamped the message | Wrong clocks and mixed time zones |
| Subject | The subject line | Encoded subjects start with =?UTF-8?. Decode them before you quote them. |
| MIME-Version, Content-Type | How the body and attachments are packed | Odd file types and nested parts |
| Authentication-Results | The SPF, DKIM and DMARC verdicts of the receiving server (RFC 8601) | Results written by a server you do not control |
| Received-SPF | The SPF verdict, written by some receivers | A value that disagrees with Authentication-Results |
| DKIM-Signature | The signature, the signing domain (d=) and the selector (s=) | A signing domain that differs from the From domain |
| ARC-Seal, ARC-Message-Signature, ARC-Authentication-Results | The chain that carries earlier results across a forwarder (RFC 8617) | A chain value (cv=) that says fail |
| X-Mailer, User-Agent, X-Originating-IP | Extra fields that some software adds. They are not part of the core standard. | The sender controls them. Treat them as clues and never as proof. |

## Trace the Route a Message Took

1. Copy every Received line into a note. Keep the order.
2. Number them from the bottom. The lowest Received line is hop 1.
3. Write down the server name, the IP address and the time for every hop.
4. Convert every time to UTC.
5. Work out the delay between hops. Subtract the earlier time from the later one.
6. Mark any hop that goes backward in time, jumps by hours or names a server you cannot explain.

Here is the sample from above in table form.

| Hop | Received by | From | Time (UTC) | Delay |
|---|---|---|---|---|
| 1 | mail.example.org | workstation.example.net (198.51.100.7) | 09:13:58 | Start |
| 2 | mx.example.com | mail.example.org (203.0.113.25) | 09:14:02 | 4 seconds |

Date says 09:13:55, three seconds before hop 1. That gap is normal. The sender's mail program stamps the Date and the server stamps the first Received line a moment later.

Server clocks drift. One odd time does not prove tampering. Odd times across several hops need a closer look.

## Convert Times to UTC

The last part of a date shows the offset from UTC. Subtract a plus offset and add a minus offset.

| Offset | Meaning | Example | In UTC |
|---|---|---|---|
| +0000 | UTC itself | 09:14:02 +0000 | 09:14:02 |
| +0530 | India Standard Time | 14:44:02 +0530 | 09:14:02 |
| +0100 | One hour ahead of UTC | 10:14:02 +0100 | 09:14:02 |
| -0500 | Five hours behind UTC | 04:14:02 -0500 | 09:14:02 |

RFC 5322 defines the date format and RFC 8601 is the standard for the authentication results line. Both are listed in the sources.

## Understand SPF, DKIM, DMARC and ARC Results

Four checks sit behind most of the lines you will see.

**SPF** checks the server that sent the message. The domain owner publishes a list of allowed servers in DNS. The check looks at the domain in the Return-Path line. RFC 7208 allows at most 10 DNS lookups during one check.

| SPF result | Meaning |
|---|---|
| pass | The sending server is on the domain's list |
| fail | The server is not on the list and the owner wants it rejected |
| softfail | The server is not on the list and the owner wants it treated with suspicion |
| neutral | The owner makes no claim |
| none | The domain publishes no SPF record |
| temperror | A temporary DNS problem stopped the check |
| permerror | The SPF record itself is broken |

**DKIM** checks that the message was not changed after signing. The sending domain signs selected headers and the body. The DKIM-Signature line carries the proof.

| DKIM tag | Meaning |
|---|---|
| d= | The signing domain |
| s= | The selector. It tells you which DNS record holds the public key. |
| h= | The list of signed header fields |
| bh= | The hash of the body |
| b= | The signature |
| a= | The algorithm, for example rsa-sha256 |
| c= | How the text was normalized before signing |

**DMARC** ties the first two checks to the From domain. The From domain must line up with the domain that passed SPF or with the domain in the DKIM d= tag. One aligned pass is enough. The domain owner publishes a policy.

| DMARC policy | What the owner asks the receiver to do |
|---|---|
| p=none | Report only and deliver the message |
| p=quarantine | Treat the message as suspicious, for example by sending it to spam |
| p=reject | Refuse the message |

**ARC** helps when a forwarder or mailing list changes a message. Every hop adds three lines, ARC-Authentication-Results, ARC-Message-Signature and ARC-Seal, and numbers them with i=1, i=2 and so on. The seal carries a chain value (cv=) of none, pass or fail. A receiver can check the chain to see what the earlier results were.

What a pass proves. Mail came from a server the domain approved or carried a signature the domain made.

What a pass does not prove. It does not prove who typed the words, that the account was not hacked or that the content is true.

## Check the DNS Records Yourself

You can look up the same records the receiver used. Replace example.com with the domain you are checking. Replace mail2026 with the selector from the DKIM-Signature line.

```
dig +short TXT example.com
dig +short TXT _dmarc.example.com
dig +short TXT mail2026._domainkey.example.com
```

On Windows, use nslookup.

```
nslookup -type=TXT _dmarc.example.com
```

Command one lists the SPF record, which starts with v=spf1. Command two shows the DMARC policy, which starts with v=DMARC1. Command three shows the DKIM public key.

DNS shows records as they are today. Your receiver checked them on the day the message arrived. Domains can change records later, so write down the date you looked and trust the Authentication-Results line for the original verdict.

## Spot a Forged Sender

| Clue | Why it matters | What to do |
|---|---|---|
| From and Return-Path use different domains | The visible sender is not the system that sent the message | Compare both with the SPF and DMARC results |
| dmarc=fail | The From domain did not line up with SPF or DKIM | Look up the domain's policy in DNS |
| Reply-To points to another domain | Replies go to a mailbox the From domain does not control | Record both addresses |
| Message-ID domain does not match the From domain | A different system made the message | Compare it with other mail from the same sender |
| A Received line that goes backward in time | Someone added a line or a clock is wrong | Check the hops above and below it |
| No Authentication-Results line | The receiver did not record checks, or the line was removed | Ask the mail administrator for server logs |
| The display name shows a known person and the address does not | The name is easy to fake | Check the full address, not the name |
| Look alike domains with a swapped letter | The domain is registered by someone else | Check the registration date of the domain |

## Worked Examples

Three sample files sit in the [examples](examples) folder. Open them next to this page.

**Example 1, a clean message.** [examples/01-clean-message.txt](examples/01-clean-message.txt) has two Received lines, a four second gap and a full set of passes. SPF passed for example.net, DKIM passed with d=example.net and DMARC passed because both line up with the From domain.

**Example 2, a forged sender.** [examples/02-forged-sender.txt](examples/02-forged-sender.txt) shows a From line for example.net. The Return-Path points to mailer.example.org. SPF passes, but only for mailer.example.org, so it does not line up with the From domain. DKIM is missing. DMARC fails. The Reply-To line sends replies to a different domain. The receiving server recorded the failure. What it did next depends on its own settings.

**Example 3, a mailing list.** [examples/03-mailing-list-forward.txt](examples/03-mailing-list-forward.txt) fails DKIM because the list added a footer after the sender signed the message. DMARC fails too. The ARC lines show that the message passed every check before the list touched it, and the ARC check passes. This is a legitimate message that looks broken. Check the ARC lines before you call it forged.

## Keep the Header Safe as Evidence

1. Save the raw message or the raw header as a text file. Do not open and resave it in a word processor.
2. Create a SHA-256 hash and write it in your log.
3. Work on a copy. Lock the original so nobody edits it.
4. Note the mail client, its version, the account and the date you collected the header.
5. Note the time zone of your own computer.
6. Write your findings in a separate file and quote the Message-ID so a reader can find the same message.

```
sha256sum message-header.txt
shasum -a 256 message-header.txt
Get-FileHash message-header.txt -Algorithm SHA256
```

Command one works on Linux, command two on macOS and command three in Windows PowerShell.

Do not paste a header from a live case into a web tool unless your rules allow it. Headers contain email addresses, server names and IP addresses. Use an offline tool when the case is sensitive.

## Tools for Header Analysis

- **Google Admin Toolbox Messageheader**: Web tool that reads a pasted header. It shows delivery delays and the approximate source.
  * Website: <https://toolbox.googleapps.com/apps/messageheader/>

- **Microsoft Message Header Analyzer**: Parses the raw header and shows the path every message took, with delays flagged.
  * Website: <https://learn.microsoft.com/en-us/connectivity-analyzer/message-header-analyzer>

- **emlAnalyzer**: Free command line tool that reads EML files. It shows headers and MIME structure and pulls out attachments and links.
  * Website: <https://github.com/wahlflo/eml_analyzer>

- **MailXaminer** (by the maintainers of this repo): Windows software for email evidence. It reads headers from many mailbox formats and offers search, link and timeline views.
  * Website: <https://www.mailxaminer.com/view-email-headers.html>
  * Related reading on metadata: <https://www.mailxaminer.com/blog/email-metadata-analysis/>

See also the list of email forensics tools (<https://github.com/Shaito345/email-forensic-tools>).

## FAQ

**Can an email header be faked?**
Yes. Anyone can write a From line, a Reply-To line and extra Received lines. Lines added by your own server sit at the top and are the hardest to fake. Doubt lines further down until you can verify them.

**Does dmarc=pass mean the email is safe?**
No. It means the From domain lined up with a passing SPF or DKIM check. A hacked account at that domain also passes.

**Why does a message carry two DKIM signatures?**
Different systems sign it. The sending domain signs it and the email service provider can sign it too. Check which signing domain lines up with the From domain.

**Can the X-Originating-IP line show where the sender is?**
It shows an IP address the sending software wrote. The sender controls that line and not every provider adds it. IP addresses also point to a network and not to a street address.

**Why do the times in the header disagree?**
Servers keep their own clocks and use different time zones. Convert every time to UTC and compare the gaps between hops.

**Where do I start if I have a thousand emails?**
Collect them in a mail format that keeps the full header, like a PST, an MBOX or a folder of EML files. Then use a tool that parses headers in bulk.

## Glossary

**Envelope sender.** The address in the Return-Path line. Servers use it for bounces.

**From domain.** The domain in the From line. It is what DMARC protects.

**Hop.** One server that handled the message.

**Header.** The block of lines above the body.

**MIME.** The standard that packs text, HTML and attachments into one message.

**Selector.** The name in the DKIM s= tag. It points to a DNS record.

**Alignment.** The match between the From domain and the SPF or DKIM domain.

**Received line.** The line a server adds when it takes a message.

**Message-ID.** A unique code made by the sending system.

**ARC.** A chain of lines that carries earlier authentication results across a forwarder.

**UTC.** The time standard with no offset. Use it for every comparison.

## Sources

- RFC 5322, Internet Message Format (<https://datatracker.ietf.org/doc/html/rfc5322>)
- RFC 5321, Simple Mail Transfer Protocol (<https://datatracker.ietf.org/doc/html/rfc5321>)
- RFC 7208, Sender Policy Framework (<https://datatracker.ietf.org/doc/html/rfc7208>)
- RFC 6376, DomainKeys Identified Mail Signatures (<https://datatracker.ietf.org/doc/html/rfc6376>)
- RFC 7489, DMARC (<https://datatracker.ietf.org/doc/html/rfc7489>)
- RFC 8601, Message Header Field for Indicating Message Authentication Status (<https://datatracker.ietf.org/doc/html/rfc8601>)
- RFC 8617, Authenticated Received Chain (<https://datatracker.ietf.org/doc/html/rfc8617>)
- NIST Technical Note 1945 (<https://nvlpubs.nist.gov/nistpubs/TechnicalNotes/NIST.TN.1945.pdf>)
- Google, trace an email with its full header (<https://support.google.com/mail/answer/29436>)
- Microsoft, Message Header Analyzer (<https://learn.microsoft.com/en-us/connectivity-analyzer/message-header-analyzer>)

## Contributing

Pull requests and issues are welcome. Check [CONTRIBUTING.md](CONTRIBUTING.md) first. Use only reserved example domains and documentation IP addresses in new examples.

## License

Copyright (c) 2026 alexmorgan. Licensed under the MIT License (MIT).
