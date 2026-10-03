# luce-smtp

An SMTP submission client in [luce-base](https://github.com/dymokomi/luce-base)
(RFC 6409 / RFC 5321). It reads EHLO and its extensions, then STARTTLS (587) or
implicit TLS (465) through a [luce-tls](https://github.com/dymokomi/luce-tls)
`Stream`, then AUTH PLAIN, LOGIN or XOAUTH2. Messages go as MAIL / RCPT / DATA
with dot-stuffing, and SIZE, 8BITMIME and SMTPUTF8 are used when the server
offers them. Every reply's code and text are kept for the caller.

```luce-base
from luce_smtp import smtp

var client = try smtp.Client.open("smtp.example.com", 587, smtp.Security.starttls)
defer client.close()
try client.login("alice@example.com", password)
let recipients: str[2] = ["bob@example.org", "carol@example.net"]
let accepted = try client.send("alice@example.com", recipients, message_bytes)
for index in 0..<client.refused_count():
    print(f"refused: {try client.refused(index)}")
client.quit()
```

`send` delivers to every recipient the server accepts. Refused recipients are
listed by `refused`, and the send fails with `no_recipients` only when none
was accepted. A 4xx reply fails with `deferred` (try again later) and a 5xx with
`rejected`, each carrying the server's text. The message bytes are sent with
CRLF line ends whatever they had. Build them with
[luce-mime](https://github.com/dymokomi/luce-mime)'s `Draft`. Every read and
write has a deadline and can be cancelled from another thread.

## Test

```sh
./test.sh       # sessions against a scripted loopback server
```

`tools/live.lucb` sends one test message through a real server:
`build/live HOST PORT plain|starttls|tls FROM TO [USER PASSWORD]`.

## License

Dual-licensed under Apache-2.0 or MIT, at your option.
