# Username Enumeration via Subtly Different Responses

**Lab:** PortSwigger Web Security Academy — Authentication

## What I was testing

Same idea as the previous lab — find a valid username by watching how
the server responds — but this time the app was designed to send back
responses that look identical on the surface. No obvious length or
status code difference to lean on like last time.

## Approach

Captured the login request in Burp, sent it to Intruder, set username
as the payload position, loaded the candidate wordlist, ran the
attack.

## Where I got stuck

At first glance every response looked the same — same status code
(200), same "Invalid username or password." message. I went down a
dead end chasing a numeric ID in the page (an analytics tracking
value) that changed on every single request — assumed it might be
tied to username validity, but it turned out to be random noise
generated per-request, unrelated to the login attempt at all.

Went back to basics: sorted the results table by response Length,
same as Lab 1. One row stood out from the rest — a few bytes shorter
than everything else.

## Finding it

Opened that specific response and compared the error message text
directly against a normal one. The difference was a single missing
period: every invalid attempt returned "Invalid username or
password." (with a full stop), but the one anomalous row returned
"Invalid username or password" — no period. That's it. That one
character was the entire signal, and it was the valid username.

## Why it matters

Even a difference this small is enough to leak which usernames exist,
because it's consistent and machine-detectable at scale — a human
skimming manually would likely miss it, but an automated diff (or
even just sorting by length) catches it instantly. This is a good
reminder that "the responses look identical" isn't the same as "the
responses are byte-for-byte identical."

## Takeaways

- Don't assume a numeric/dynamic-looking value in a response (IDs,
  tokens, timestamps) is meaningful without testing whether it's
  consistent for repeated identical requests first.
- Sorting Intruder results by response Length is often faster and
  more reliable than eyeballing text for subtle differences.
- A single missing character (punctuation included) is a real,
  exploitable information leak if it's returned consistently.
