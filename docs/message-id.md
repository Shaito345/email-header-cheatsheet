# Message-ID in Email Headers and How to Look It Up

Message-ID is a unique tag that the sending mail system adds to every email. You can use it to find one exact message, match replies to the original and spot duplicates.

## What a Message-ID looks like

A Message-ID has two parts joined by an at sign and wrapped in angle brackets.

```
Message-ID: <20261117101530.4F2A81B3@mail.example.com>
```

RFC 5322 requires the value to be globally unique. It suggests a domain name on the right and a timestamp plus a local value on the left.

## Where to find it

- **Gmail.** Open the message. Next to Reply, select More, then Show original. Search the window that opens for the Message-ID line.
- **Outlook on the web and new Outlook.** Open the message, select More actions, then View, then View message details.
- **Outlook for Windows (classic).** Open the message in its own window, then File and Properties. The Internet headers box holds the Message-ID line.

## How investigators use it

- **Find one message.** Search the full ID in your mail archive to land on the exact message.
- **Match replies.** The In-Reply-To header holds the Message-ID of the message being answered. The References header holds the chain of earlier IDs, so you can rebuild a thread.
- **Spot duplicates.** The same Message-ID in two files points to one message saved twice.
- **Spot odd values.** A domain in the ID that differs from the From domain gives you a reason to look closer. It is not proof.

## Worked example

The sample file [02-forged-sender.txt](../examples/02-forged-sender.txt) carries this line.

```
Message-ID: <7f3a91c2@mailer.example.org>
```

The ID names mailer.example.org, the same system that appears in Return-Path and in the Received line. The From line shows example.net. That gap is one reason to look closer. The DMARC result in the same file shows fail.

## Limits

The sending system sets the Message-ID, so a sender can change it. It does not prove who sent the message. Check SPF, DKIM and DMARC results for that.

## Related guides

- [Email Header Cheat Sheet](../README.md)
- [Message-ID in email forensics](https://www.mailxaminer.com/blog/message-id-forensic/?utm_source=github&utm_medium=repo&utm_campaign=email-header-cheatsheet&utm_content=message-id)
- [RFC 5322, section 3.6.4](https://datatracker.ietf.org/doc/html/rfc5322#section-3.6.4)
