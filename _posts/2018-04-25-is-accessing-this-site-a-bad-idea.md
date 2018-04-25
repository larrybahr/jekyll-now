---
layout: post
title: Is Accessing this Site a Bad Idea
---
Google Chrome has changed the way web developers design for the better, but not all of their changes are good ideas. At my company we have many internal systems that use self-signed certificates. Since only developers access these servers for testing, it doesn't matter that the certificate is not from a credible certificate authority. If you are like me and you just want to access the site try clicking the "ADVANCED" button and typing "badidea" without the quotes. This little trick can be a life saver and Chrome remembers your selection for a while so you do not need to do this every time you access the site!

![_config.yml]({{ site.baseurl }}/images/posts/2018-02-02-is-accessing-this-site-a-bad-idea/badidea.gif)

---
## UPDATE
[Chrome has changed the key word](https://stackoverflow.com/questions/35274659/does-using-badidea-or-thisisunsafe-to-bypass-a-chrome-certificate-hsts-error) to "thisisunsafe"
