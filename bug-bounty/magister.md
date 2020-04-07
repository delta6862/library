# Magister
![thumbnail]

[thumbnail]: https://github.com/delta6862/library/blob/master/images/thumbnail.png?raw=true ""
## Overview
- [Introduction](#introduction)
- [Magister](#magister)
- [The bug](#the-bug)
- [Reporting](#reporting)
- [Impact](#impact)
- [Closing notes](#closing-notes)

## Introduction
This post is about a bug found in Magister which allowed a malicious acter to take over an account, before anyone gets their hopes up; the bug has been patched by now, I would not be publishing this otherwise.

## Magister
So first off, what is Magister?
> Magister is a Dutch online administrative application, used in the schooling system in secondary education. All of the students' information is on this platform. Students, parents and teachers are able to access this at all times. It is used to keep up with the management, administration and health issues of students, and users can also view their grades, timetable, homework assignments and absences. In 2012 the software had a 70% market share in the Netherlands. 
> (source: https://en.wikipedia.org/wiki/Magister_(application))
From this description there are two main takeaways;
1. If used by a school it plays a critical role in both the schools and the students school-related administration.
2. Its widely used in the Netherlands.
We can verify how widely used Magister is ourselfs aswell.
According to [onderwijsincijfers](https://www.onderwijsincijfers.nl/kengetallen/vo/instellingen-vo/aantallen-aantal-vo-scholen) there were 1450 secondary schools in the Netherlands.
Each school gets its own subdomain, using a tool like [dnsdumpster](https://dnsdumpster.com/) we can see most, if not all, of these domains. This gets us to a total of...
```bash
$wc -l subdomain-list
768 subdomain-list
```
768 subdomains, a few of these wont acctualy be for school like `files.magister.net` but for the sake of argument lets assume that 765 of them are for schools that use magister.
This means 765 out of the total 1450 secondary schools use magister. Giving us a market share of roughly 52%. It is worth pointing out that the wikipedia listing of 70% might not be inaccurate since a schooling organisation with multible locations will share a subdomain.

## The bug
In Magister, as with most platforms. There is a password reset feature. Upon activation this will send a 6 didget base-16 code to the email address listed to your account.
This code gives us a total of `16^6=16777216`combinations. Not great if we wanted to brute force it, but better than the `1.2806308e+14` or more combinations possible if we were to bruteforce a password directly.
However the email also mentions the code to be valid for a short period of time. Afther some experimentation I found that the following:
- Unlike the main login page Magister does not have a softlocking feature for the reset code, making it a valid vector of attack.
- The code lifetime is linked to the user session, generaly a new session will be given to the user afther 30 minutes
- The lifetime of a users session can be extended by sending a new request within the 30 minute window.
Combining these three things we now have a code that we can bruteforce without getting locked out, and we can make this code live for as long as we want.
This allows us to gain acces to a users account within 7-15 hours simply by resetting their password and guessing the reset code.

## Reporting
To report this to Magister I took a quick look on the platform to find a link labled [Responsible disclosure](https://www.iddinkgroup.com/responsible-disclosure-en/) following this trail of links eventualy landed me to a reporting page on a site called [Zerocopter](https://www.zerocopter.com/)
As I understand it Magister has a contract with Zerocopter in which Zerocopter is responcible for finding bugs in magister and handeling the responcible disclosure program. The images below are of the conversation between me, Magister and Zerocopter following my report. I've added some commentary between the screenshots.
![main-bug-report]
![comment-1]

At this point I was considering sending Magister a mail directly since I wanted them to be aware of the issue and was unsure if Zerocopter would accept my report.

![comment-2]

This initially really confused me since it was a fairly basic feature of Magister. Looking back im guessing the people Zerocopter assigned to this were unfamiliar with Magister as an application.

![comment-3]

![comment-4]

Afther getting no responce for a while I suspected this was the end of the reporting process however two months later I was plesantly supprised with the following.

![comment-5]

And with that the reporting process was sucessfully completed.

[main-bug-report]: https://github.com/delta6862/library/blob/master/images/main%20report.png ""
[comment-1]: https://github.com/delta6862/library/blob/master/images/comment-page-1.png?raw=true ""
[comment-2]: https://github.com/delta6862/library/blob/master/images/comment-page-2.png?raw=true ""
[comment-3]: https://github.com/delta6862/library/blob/master/images/comment-page-3.png?raw=true ""
[comment-4]: https://github.com/delta6862/library/blob/master/images/comment-page-4.png?raw=true ""
[comment-5]: https://github.com/delta6862/library/blob/master/images/comment-page-5.png?raw=true ""

## Impact
Now we have arrived at the part where I get to put down all the idea's I've come across that could have been done with this besides the obvious 'change your grade', I do want to point out that none of these ideas have been attempted using the bug described in this post.

Next to changing grades the most obvious idea is sending mail under the name of someone eltse or adding non existent events to a students agenda. However a friend of mine recently thought of a much more devastating idea, by keeping the session alive you can have a storage of valid codes and over the period of multible months have a valid code for every user on a set school system.
This could allow you to lock out all users on the network effectivly shutting down the schools administrative capabilities until it could be fixed. 

## Closing notes
- Despite how simple this bug seems I do want to compliment Magister on their security, I have spent extensive time looking at the rest of their platform without finding any bugs or other security flaws.
- This bug highlights how I feel bug hunters should approach a target, while straight away testing for xss and sqli on every user input is a valid approach (and one that has worked for me in the bast) I encourage you to do research into the feature your trying to exploit and then ask the question "How can I abuse this to my advantage?"
