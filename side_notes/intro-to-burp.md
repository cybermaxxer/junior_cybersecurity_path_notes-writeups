# intro to burp: otp bypass via parameter removal

## table of contents

* [1. the 80/20 digest](#1-the-8020-digest)
* [2. structured breakdown (cheat sheet)](#2-structured-breakdown-cheat-sheet)
* [3. feynman checks](#3-feynman-checks)
* [4. omitted chuff](#4-omitted-chuff)
* [5. watch it explained](#5-watch-it-explained)

## 1. the 80/20 digest

### registering and hitting the otp wall

first thing i did was make myself a normal account, username `legit`. nothing fancy, just the standard registration flow. submitting that form didn't drop me straight into the dashboard though, it threw up an otp verification page first. so the app's doing some kind of second factor before it trusts a fresh session.

i wanted a legit (pun intended) session cookie in play before i started poking at the otp logic, since the whole point is to see how the server's session state talks to the otp check, not to fake my way past registration too.

q: why bother registering a real account instead of just hammering the otp endpoint blind? a: because the otp check lives downstream of a valid session, i needed that session context established first to see how the two interact.

### catching the request in burp

opened burp, flipped intercept on in the proxy tab, then went back to the browser and typed in a garbage otp, `afafa`, just to get a request flowing that burp could grab.

burp caught the outbound POST before it ever reached the server:

```http
POST /dashboard HTTP/1.1
Host: chatelaine.cylabacademy.net:42506
Content-Type: application/x-www-form-urlencoded
Content-Length: 9
Cookie: session=...

otp=afafa
```

pretty unremarkable at a glance: standard url encoded body, one parameter, `otp`, carrying whatever i typed. that parameter is clearly what gets compared against the real otp on the server side.

q: what's actually happening in that request? a: my browser is posting the user supplied `otp` value to `/dashboard`, where the server presumably checks it against whatever it generated and sent me earlier.

### the actual bypass: just delete the parameter

instead of trying to guess or brute force a real otp, i tested the laziest possible theory first: what if the backend only checks the otp *when it's present*, and doesn't actually enforce that the parameter has to be there at all.

so, with the request still held in burp:

1. deleted `otp=afafa` from the body entirely, left nothing.
2. fixed up `Content-Length` to `0` so the request wasn't malformed.
3. forwarded it.

and that was it. the app logged `legit` straight in and handed me the flag: `academy{#0TP_Bypvss_SuCc3SS_f182f0ef}`.

the logic flaw here is almost certainly something like `if (req.body.otp == secret)` with no prior check for `req.body.otp` existing at all. a wrong otp fails that comparison honestly. a *missing* otp probably evaluates to `undefined == secret`, which is just false too in most languages, except somewhere upstream of that comparison there's likely a different branch, a try/catch swallowing the missing field, or a "no otp submitted, treat as not required" fallback that was never meant to be reachable this way. point is: the validation path for "wrong code" and the validation path for "no code" aren't the same path, and only one of them was actually locked down.

q: why does removing the parameter succeed where a wrong value fails? a: a wrong value hits the real comparison and gets rejected honestly. a missing value means the comparison either never runs or runs against something that quietly resolves to true, which is a classic "the dev only coded for the happy path plus the wrong guess, not the missing field" miss.

**cybersec angle:** this is a textbook authentication logic flaw, and it comes from trusting the frontend to always send what the backend expects. the frontend form always includes `otp` because it's a form, so the devs never had to think about a request where it just isn't there. the fix is boring and that's the point: explicitly check `req.body.otp` exists and is non empty *before* doing anything with it, and fail closed (401 or 400) the instant it isn't. never let a missing field fall through to a comparison that might resolve favorably by accident.

## 2. structured breakdown (cheat sheet)

| phase | tool / technique | what it got me |
| --- | --- | --- |
| register a user | browser signup form | a real account (`legit`) and a session that actually reaches the otp step |
| capture the request | burp proxy, intercept on | the raw POST to `/dashboard` with the otp body visible |
| read the payload | burp's request editor | confirmed it's one url encoded field, `otp=afafa` |
| strip the parameter | delete the body, fix `Content-Length` to `0` | a request with no otp field at all, forwarded as is |
| get the flag | server's own response | logged in as `legit`, 2fa fully skipped |

## 3. feynman checks

**q1: why does a wrong otp get rejected but a missing one doesn't?**
a: a wrong otp runs the real comparison and loses it honestly. a missing one either skips the comparison entirely or hits some unguarded branch that was never meant to be reachable, so the two "failure" cases aren't actually going through the same code path.

**q2: what is burp actually doing when intercept is on?**
a: it's sitting as a proxy between my browser and the target server, so every request has to pass through it first. with intercept on it just pauses that handoff, letting me read and edit the raw request (headers, body, whatever) before deciding to forward it on.

**q3: what's the real fix on the backend?**
a: check that `otp` exists and isn't empty before you ever compare it to anything, and bail with a 400 or 401 immediately if it's missing. the session should also carry an explicit "mfa satisfied: yes/no" flag so no other endpoint can be reached while it's still no, instead of relying on this one check alone.

## 4. omitted chuff

* the first few junk otp guesses typed straight into the browser UI before i ever opened burp
* setting up the browser's proxy config and toggling the burp extension on
* the boring headers in the capture (`User-Agent`, `Accept-Language`, etc), none of them mattered to the bypass

## 5. watch it explained

| concept | youtube search query |
| --- | --- |
| burp suite proxy intercept | burp suite intercept and modify http requests tutorial |
| parameter tampering | web application parameter tampering vulnerability explained |
| otp / 2fa bypass logic flaws | authentication bypass missing parameter logic flaw |
| fixing it server side | server side input validation authentication security |
