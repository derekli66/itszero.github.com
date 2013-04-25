---
layout: post
title: "Two-factor Authentication for Windows"
date: 2013-04-24 15:25
comments: true
categories: RDP, two-factor authentication, Windows, MultiOTP
---

I had a remote running Windows with Remote Desktop enabled. I'm really
comfortable about just putting it online with the bare minimal Windows
authentication protection. So I went online to do a little research and I found
out that the algorithm Google Authenticator is using (it's called TOTP) could
be integrated to Windows authentication system. Here is how to do it.

The key thing here is [MultiOTP-CP], it's a credential provider for Windows to
enable two-factor authentication. Before we get into that, we will need to
install its dependency, [MultiOTP]. MultiOTP is a library to deal with one-time
passwords. You can download it from [http://www.multiotp.net]. Extract it to
`C:\multiotp`. Now, we need to tie your Windows account with a proper key for
TOTP elgorithm. Open your command prompt and switch to `C:\multiotp`. Type the
following command to setup two-factor auth.

```
multiotp -debug -create test [user name] [20-bytes(160bits) key] [4 digit pin, doesn't matter.] 6 30
```

If it says success, try you could verify it by using

```
multiotp -debug [user name] [OTP generated on your phone]
```

If you need some help with generating key, I recommend [speakeasy]. It's not
exactly a tool but a two-factor authentication library for node.js. It could
generate key and url to QR code for your phone to scan in one line. If you
don't mind go through installing node.js, try it!

Once that's done, install multiotp-cp and it's done! Next time you need to
login to Windows, you'll see an extra field for you to input one-time password.
Simple as that!

{% img /images/posts/multiotp-login.png 554 "MulitOTP-CP login screen" %}

[MultiOTP]: http://www.multiotp.net
[MultiOTP-CP]: http://code.google.com/p/multi-one-time-password--credential-provider/
[speakeasy]: https://github.com/markbao/speakeasy
