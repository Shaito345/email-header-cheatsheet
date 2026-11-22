# Can You Trace an Email Address to Its Owner

Email headers show the path a message took. They do not show the person who typed it. This guide lists what a header can tell you, what it cannot and how to follow the path with fake sample data.

*Last reviewed 22 November 2026.*

## Short Answer

You can trace a message to the mail servers that handled it. You cannot trace it to a named person from the header alone. The sender address is text that the sending system fills in, and RFC 5321 says SMTP mail is inherently insecure because even casual users can create messages that look like they came from someone else.

## What a Header Can Show

| Header | What it records | Source |
|---|---|---|
| `Received:` | The host that sent the message, the host that received it and the date and time of receipt | RFC 5321, section 4.4 |
| `Received:` (FROM clause) | Should hold the sending host name from the EHLO command and the sender IP address taken from the TCP connection | RFC 5321, section 4.4 |
| `From:` | The mailbox of the author | RFC 5322, section 3.6.2 |
| `Sender:` | The mailbox that actually sent the message, when it differs from the author | RFC 5322, section 3.6.2 |
| `Reply-To:` | Where replies should go | RFC 5322, section 3.6.2 |
| `Return-Path:` | The return address, placed before the `Received:` lines of the header | RFC 5322, section 3.6.7 |

The SMTP server that accepts a message adds a `Received:` line at the start of the message. Every new server adds its line above the older ones. This means the oldest hop sits at the bottom of the list and you follow the path from the bottom upward.

## What a Header Cannot Show

- **The person.** RFC 5321 says SMTP mail cannot be authenticated or integrity checked at the transport level. It calls efforts to stop users from setting the return path and `From:` fields to other valid addresses largely misguided.
- **The device or the place.** A header holds server names and IP addresses, not the name of the writer.
- **A guaranteed record.** RFC 5322 calls trace fields strictly informational.

## A Routing Diagram

This diagram uses fake hosts and documentation IP addresses.

```
Sender mail app
      |  submits the message
      v
Sending server   mail-out.example.net  [203.0.113.25]
      |  relays the message
      v
Your server      mx.example.com
      |  adds the top Received line and stores the message
      v
Your mailbox
```

The matching header holds two `Received:` lines. Fake sample data.

```
Received: from mail-out.example.net (mail-out.example.net [203.0.113.25])
        by mx.example.com with ESMTP; Sat, 21 Nov 2026 10:15:30 +0000
Received: from [198.51.100.7] (helo=laptop.example.org)
        by mail-out.example.net with ESMTPSA; Sat, 21 Nov 2026 10:15:28 +0000
```

Start with the second line. It shows the message entering `mail-out.example.net`. The first line shows `mx.example.com` taking the message from that server.

## How to Follow the Path

1. Open the full message headers. The [Email Header Cheat Sheet](../README.md) lists where to find them in every mail app.
2. List every `Received:` line, then work from the bottom to the top.
3. Write down the host name, the IP address and the time of every hop.
4. Compare the `From:` and `Return-Path:` domains with the domains in the lowest hops.
5. Check the SPF, DKIM and DMARC results in the same header.
6. Treat the result as a lead. Do not treat it as proof of who wrote the message.

## Our Notes

These points are ours and are not quotes from the RFCs.

- Only the top `Received:` line comes from your own server. Lower lines come from systems you do not control, so a sender can write false values there.
- An IP lookup tells you which organization holds an address block. It does not tell you who used the address at that moment.
- A message sent through a mail provider can show the provider's server in the path and not the sender's own device.
- Times in different hops can differ by a few seconds or more because the servers keep their own clocks.

## Legal and Privacy Note

This note is general information and not legal advice. A lawyer in your country should review any real case.

- Rules on collecting, keeping and sharing email and IP data differ by country and by case type. Check yours before you collect or share anything.
- Review only mail that you own or that you are authorized to review.
- Do not publish the name, address or IP of a person you think sent a message. A header can point at the wrong person.
- Do not contact or confront a suspected sender based on a header.
- Account records sit with the mail provider. Ask your legal team or the police how to request them in your country.
- The samples in this repo are fake. Do not post real headers with real names, addresses or IPs.

## Sources

- [RFC 5321, section 4.4 Trace Information](https://datatracker.ietf.org/doc/html/rfc5321#section-4.4)
- [RFC 5321, section 7.1 Mail Security and Spoofing](https://datatracker.ietf.org/doc/html/rfc5321#section-7.1)
- [RFC 5322, section 3.6.2 Originator Fields](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.2)
- [RFC 5322, section 3.6.7 Trace Fields](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.7)

## Related Guides

- [Message-ID in Email Headers and How to Look It Up](message-id.md)
- [Email Header Cheat Sheet](../README.md)
