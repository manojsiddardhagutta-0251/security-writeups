#Username Enumeration via Response Length

*Lab:* PortSwigger Web Security Academy — Authentication

##What I was testing

A login form that takes a username and password. The goal was to
find out if I could tell which usernames actually exist on the
system, just by looking at how the server responds — without
knowing any passwords.

##Approach

I intercepted a login request in Burp Suite and sent it to Intruder,
setting the username field as the payload position and keeping the
password fixed. I loaded a wordlist of candidate usernames and ran
the attack.

##Where I got stuck
Every single request came back with a 400 error, regardless of the
username. At first I assumed the usernames themselves were wrong,
but that didn't make sense across the entire wordlist. Turned out
the problem was Content-Length — Burp was sending the original
request's byte count instead of recalculating it for each new
payload, so the server was rejecting the request as malformed before
it ever checked the username. Fixed it by enabling "Update
Content-Length header" in Intruder's options. That one setting was
the difference between a wall of 400s and actual results.

## Finding it

Once the requests were going through cleanly, I sorted the results
by response length. Almost every response came back at 3352 bytes.
One came back at 3324 bytes — a 28-byte difference was enough to
flag it as the one worth checking manually. Opened that response and
confirmed the error message was different from the rest, which
meant the server was quietly telling me the username was valid even
though the login itself still failed.

## Why it matters

This lets an attacker build a list of confirmed-valid usernames
without guessing a single password, which turns a brute-force attack
from "guess everything" into "guess passwords only for accounts that
actually exist" — a big reduction in effort. The fix is simple:
return an identical response (length, content, timing) whether the
username is valid or not.

## Takeaways

- Content-Length mismatches can silently break an entire Intruder
  run and look like a wordlist problem when it isn't one.
- Response length is often more reliable than status code for
  spotting anomalies — every response here returned 200.
- Small, consistent differences (28 bytes here) are enough to leak
  information if you're paying attention.
