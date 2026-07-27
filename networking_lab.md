# HTTP Header Spoofing: Bypassing a User-Agent Check

## Background

Before this box, I learned you can open raw TCP connections with `netcat`, and that HTTP relies on headers to communicate. A header is just a piece of metadata: information meant to stay invisible to the end user but visible to the machine, so two devices can agree on how they're going to talk to each other.

## Step 1: Nmap Scan

First, I scanned the target to see what was running:

```bash
nmap -sV -sC <target-ip> | grep http
```

- `-sV` → detect service **v**ersions
- `-sC` → run default **s**cripts (**s**ervice **c**heck)

Output:

```
80/tcp open http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
```

This confirms HTTP is running on the standard port, 80.

## Step 2: Curl (Recon)

Next, not strictly necessary, but useful to get a feel for what we're attacking and what the header we're about to spoof should look like, I checked the HTTP version with:

```bash
curl -v <target-ip>
```

- `-v` → **v**erbose, shows the full request/response instead of just the body

Output:

```
┌─[eu-academy-4]─[10.10.15.102]─[htb-ac-2397331@htb-xu262g31tk]─[~]
└──╼ [★]$ curl -v 10.129.233.197

* Trying 10.129.233.197:80...
* Connected to 10.129.233.197 (10.129.233.197) port 80
* using HTTP/1.x
GET / HTTP/1.1
Host: 10.129.233.197
User-Agent: curl/8.14.1
Accept: */*
* Request completely sent off
* Recv failure: Connection reset by peer
* closing connection #0
curl: (56) Recv failure: Connection reset by peer
```

### Breaking down the request

| Line | Meaning |
|---|---|
| `GET / HTTP/1.1` | `GET` = get request, `/` = root/index page, `HTTP/1.1` = protocol version (almost always 1.1) |
| `Host:` | which server we're talking to |
| `User-Agent:` | what software is making the request |
| `Accept: */*` | "I'll accept whatever content type you send back" (usually `html`, `txt`, etc.) |

The connection gets reset, which tells us something in our request is triggering a rejection.

## Step 3: Spoofing the User-Agent with Netcat

Given the reset, and knowing the service fingerprint from the Nmap scan, the next move was to manually craft the request and swap out the `User-Agent`.

```bash
nc -v <target-ip> 80
```

- `-v` → verbose, so nc tells us when the connection opens

Once connected, netcat just sits there waiting. Note: it does not send anything until you hit enter on an *empty* line. I lost some time here assuming a single `[enter]` after each line would trigger the send. It won't. HTTP requests need a blank line at the end to signal "request finished," so you have to press enter one extra time after the last header.

Then I typed the request manually:

```
GET / HTTP/1.1
Host: 10.129.233.197
User-Agent: Server Administrator

```

(the blank line at the end is a deliberate double enter)

That's the fix: the target was rejecting the default `curl` User-Agent, and spoofing it to something that looks like an internal/admin client got the request through.
