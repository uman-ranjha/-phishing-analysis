# Phishing Email Analysis - Fake PayPal Invoice

I got this email in my Yahoo inbox at the end of September and decided to use it
as my first phishing analysis project. I went through the headers, the attachment,
and the content to figure out what it was trying to do.

## The Email

It looked like a PayPal invoice for a $374.41 coffee maker I never bought. There
wasn't a link anywhere. Instead the attached PDF said to call "customer support"
within 24 hours if I had questions about the charge.

From what I read, this is called callback phishing. The idea is you panic, call
the number, and someone pretending to be support tries to get remote access to
your computer or your bank info. 

<img width="500" alt="Invoice" src="https://github.com/user-attachments/assets/20fff9c0-dcc7-43e4-a907-1b048fd904c0" />

## Tools

- MXToolbox email header analyzer
- Yahoo's "View raw message" to get the full email source
- certutil (Windows) to get the file hash
- VirusTotal to check the hash

## Headers

| Field | Value | Notes |
|---|---|---|
| From | RODRIGUEZ Lemieux <gutmkevin14[@]gmail[.]com> | Random Gmail account, name doesn't match anything |
| Reply-To | none | |
| Return-Path | gutmkevin14[@]gmail[.]com | Same as sender |
| Originating IP | 74.125.231[.]139 | This is Google's mail server, not the scammer's actual IP |
| SPF / DKIM / DMARC | pass / pass / pass | |

At first I was confused that SPF, DKIM and DMARC all passed. But it makes sense
since the email actually was sent from a real Gmail account. Those checks only
tell you the domain is legit, not the person behind it.

A couple other things I noticed:

- The first couple hops say `gmailapi.google.com with HTTPREST`. So it was sent
  through the Gmail API, which probably means a script sending these out in bulk.
- The subject ends with a random string (`Y0W7~1BHT3J4HXXR5W`) and the body of the
  email is literally just that same string. Everything else is in the PDF. My guess
  is that's to get around spam filters that scan the email text.

<img width="650" alt="Header analysis" src="https://github.com/user-attachments/assets/31203407-cd19-4c3e-9d9a-4abbc4c0ece9" />

The red X on DMARC is MXToolbox flagging Gmail's policy (p=none), not a failed
check. Yahoo's results show dmarc=pass.

## Attachment

File: `RICKY_3542190877818569_202609.pdf`

Looking at the raw source, the PDF was made with iText 5.5.13.6 and the
author/title fields were left blank. I didn't see any links inside it either, so
the phone number is the whole scam.

SHA256: `077c462b4a62cb61404fe383a67513eec2c79a02872b371776835d51e5e37c48`

VirusTotal: No matches found. The file hadn't been submitted before, which makes
sense if each email gets a slightly different PDF.

## Red flags

- Sent from a random Gmail, not PayPal
- Has the PayPal logo but a Stripe address, and says "PayPal Stripe User" which isn't a thing
- "Call within 24 hours" to rush you
- Charge I never made
- Email body is just random characters

## IOCs

| Type | Value |
|---|---|
| Sender | gutmkevin14[@]gmail[.]com |
| Display name | RODRIGUEZ Lemieux |
| Phone | +1 840-260-8737 |
| Subject | Thank You For Your Order Y0W7~1BHT3J4HXXR5W |
| Attachment | RICKY_3542190877818569_202609.pdf |
| SHA256 | 077c462b4a62cb61404fe383a67513eec2c79a02872b371776835d51e5e37c48 |
| Sending IP | 74.125.231[.]139 (Google) |
| Received | 2026-09-30 15:42 UTC |

## Verdict

Malicious. It's a callback phishing scam pretending to be PayPal.

If this showed up at a company, I'd:
- Block the sender and search other inboxes for the same subject or attachment name
- Block the phone number
- Report the Gmail account to Google
- Check if anyone called it, and if they did, look for remote access software on their machine and reset their passwords
- Send a heads up to users about fake invoice scams
