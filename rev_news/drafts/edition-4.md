---
title: Git Rev News Edition 4 (June 3rd, 2015)
layout: default
date: 2015-06-03 21:06:51 +0100
author: chriscool
categories: [news]
navbar: false
---

## Git Rev News: Edition 4 (June 3rd, 2015)

Welcome to the fourth edition of [Git Rev News](http://git.github.io/rev_news/rev_news.html),
a digest of all things Git. For our goals, the archives, the way we work, and how to contribute or to
subscribe, see [the Git Rev News page](http://git.github.io/rev_news/rev_news.html) on [git.github.io](http://git.github.io).

This edition covers what happened during the month of May 2015.

## Discussions

### General

* [submitGit for patch submission](http://thread.gmane.org/gmane.comp.version-control.git/269699/)

At the [Git Merge 2015](http://git-merge.com/) conference, during the Contributor
Summit, there were discussions about how to help people send patches
to the Git mailing list.

Properly sending patches to the mailing list is not easy in the first
place because email clients these days tend to heavily reformat the
content they send. This reformatting, which may include word-wrapping
the text, making it quoted-printable, adding MIME parts or replacing
tabs with spaces, will in most cases make inlined patches sent to
the Git mailing list impossible to apply or even review.

That's why the SubmittingPatches documentation file has [a long
explanation to help people send patches](https://github.com/git/git/blob/master/Documentation/SubmittingPatches#L137)
which starts with:

> Learn to use format-patch and send-email if possible.  These commands
> are optimized for the workflow of sending patches, avoiding many ways
> your existing e-mail client that is optimized for "multipart/*" mime
> type e-mails to corrupt and render your patches unusable.

[git send-email](http://git-scm.com/docs/git-send-email) is indeed the
best way to send emails to the mailing list once it has been properly
configured. The problem is that it is not very easy to configure to
say the least, especially on Windows.

[A recent discussion on the mailing
list](http://thread.gmane.org/gmane.comp.version-control.git/268000/)
shows how difficult it can be even for developers to find a way to
properly send a patch to the mailing list. Toward the end of the
discussion, Stefan Beller summarized the discussions at [Git Merge
2015](http://git-merge.com/) this way:

> This workflow discussion was a topic at the GitMerge2015 conference,
> and there are essentially 2 groups, those who know how to send email
> and those who complain about it. A solution was agreed on by nearly all
> of the contributors. It would be awesome to have a git-to-email proxy,
> such that you could do a git push <proxy> master:refs/for/mailinglist
> and this proxy would convert the push into sending patch series to the
> mailing list. It could even convert the following discussion back into
> comments (on Github?) but as a first step we'd want to try out a one
> way proxy.
>
> Unfortunately nobody stepped up to actually do the work, yet :(

A few days later, Roberto Tyley, who is the author of
[the BFG repo-cleaner](https://rtyley.github.io/bfg-repo-cleaner/),
replied to Stefan's email [by announcing submitGit](http://thread.gmane.org/gmane.comp.version-control.git/269699):

> Hello, I'm stepping up to do that work :) Or at least, I'm implementing a
> one-way GitHub PR -> Mailing list tool, called submitGit:
>
> https://submitgit.herokuapp.com/
>
> Here's what a user does:
>
> * create a PR on https://github.com/git/git
> * logs into https://submitgit.herokuapp.com/ with GitHub auth
> * selects their PR on https://submitgit.herokuapp.com/git/git/pulls
> * gets submitGit to email the PR as patches to themselves, in order to
> check it looks ok
> * when they're ready, get submitGit to send it to the mailing list on
> their behalf
>
> All discussion of the patch *stays* on the mailing list - I'm not
> attempting to change anything about the Git community process, other
> than make it easier for a wider group people to submit patches to the
> list.

This announcement was met with a lot of enthusiasm from the community,
especially from the [Git for Windows](https://msysgit.github.io/)
developers.

Junio Hamano, the Git maintainer, liked it a lot too and wanted to try
it, but the application, which uses GitHub authentication,
[requires too much authorization](http://thread.gmane.org/gmane.comp.version-control.git/269733),
which made it risky and made him uncomfortable to try it out.
Roberto [fixed this issue](https://github.com/rtyley/submitgit/pull/3) quickly.
It it now safe to use this service, and aspiring contributors are encouraged to try it out.

* [git-multimail resurrected!](http://thread.gmane.org/gmane.comp.version-control.git/270239) (*written by Matthieu Moy*)

  [git-multimail](https://github.com/git-multimail/git-multimail) got
  a new co-maintainer, and is active again after a long period of
  inactivity. [A summary of the recent
  activity](http://thread.gmane.org/gmane.comp.version-control.git/270239)
  was posted on the Git mailing-list. A 1.1 release is expected in
  June. Don't hesitate to join the fun and help by reviewing
  pull-requests or submitting new ones!

### Reviews

* [clean/smudge empty contents](http://thread.gmane.org/gmane.comp.version-control.git/269050) (*written by Junio C Hamano*)

Jim Hill noticed that Git issues an error message saying that copy_fd() was given a bad
file descriptor when clean/smudge filters is fed an file with empty contents, found that
the problem was caused because an in-memory contents that was empty was passed (by mistake)
as `NULL`, instead of an empty string `""` in this codepath, but the `NULL` was used as a
signal to tell Git to instead read from a given file descriptor. The fix was trivially
correct and was applied.

The new test script, however, exhibited a flaky behaviour. Sometimes it passed, sometimes
it saw EPIPE. Peff observed:

> Hmm, I thought we turned off SIGPIPE when writing to filters these days.
> Looks like we still complain if we get EPIPE, though. I feel like it
> should be the filter's business whether it wants to consume all of the
> input or not[1], and we should only be checking its exit status.
>
> [1] As a practical example, consider a file format that has a lot of
>    cruft at the end. The clean filter would want to read only to the
>    start of the cruft, and then stop for reasons of efficiency.

The discussion lead to
an [enhancement](http://thread.gmane.org/gmane.comp.version-control.git/269050/focus=269383)
to allow clean/smudge filters to quit before reading their input fully.

### Support

* [git pack protocol question: sideband responses in case of errors?](http://thread.gmane.org/gmane.comp.version-control.git/268949)

Christian Halstrick said that he sometimes gets "invalid channel 101"
errors when pushing over HTTP using a JGit client.

He had already debugged the problem and noticed it only appeared when
quotas on the filesystem prevented the Git server from storing a big
packfile. In these cases, the server sends back a packet line
"0013error: ..." to the client; but the client, thinking the sideband
communication should still be used, interprets the "e" from "error"
as a channel number. The ascii code of "e", which is 101 in
decimal, is the reason why the error is "invalid channel 101".

Christian asked a few questions to get more information about when
sideband communication should happen and how a server should respond
in case of error.

As Shawn Pearce had developed both the Smart HTTP protocol, which is
now the most commonly used HTTP protocol by Git clients and servers,
and JGit, the implementation of Git in Java, he answered those
questions with a lot of details and further nailed down the problem:

> The bug here is JGit's ReceivePack/BaseReceivePack code not setting
> up the side-band-64k early enough for this failure report to be
> wrapped in it.

And Shawn concluded with the following:

> FWIW I am glad you found this. I have been chasing this bug for
> years but couldn't really pin it down to anything. If its the "pack
> won't fit on local disk due to disk full" condition that narrows
> down the offending section of JGit considerably.

* ["git commit --date" does not behave well?](http://thread.gmane.org/gmane.comp.version-control.git/269832) (*written by Junio C Hamano*)

Bastien Traverse was having trouble specifying the date when creating a commit
with the `--date` parameter to `git commit` command.  He tried various
formats, e.g. `git commit --amend --date="2015-05-21 16∶31 +0200"` 
and got the date right but not the hours and minutes.

Peff tried to reproduce it (as the `--date=<string>` parsing was recently
corrected, there was a possibility of regression), but he couldn't. It
turns out that the input Bastien was feeding did not have the right "colon".

> Your "colon" is actually UTF-8 for code point U+2236 ("RATIO"). So git's
> date parser does not recognize it, and punts to approxidate(), which
> does all manner of crazy guessing trying to figure out what you meant.


## Releases

* Git [2.3.8](http://article.gmane.org/gmane.comp.version-control.git/268828) (final for 2.3.x series for now)
* Git [2.4.1 and 2.4.2](http://git-blame.blogspot.com/2015/05/git-241-and-242.html) maintenance releases.

  Together with Git 2.3.8, `git commit --date=now` now works correctly in timezones that honor
  daylight-saving-time, fixing a 
  breakage [Linus noticed](http://thread.gmane.org/gmane.comp.version-control.git/267183).

* [Git for Windows 2.x preview](http://article.gmane.org/gmane.comp.version-control.msysgit/21601)

We are nearing the Git 2.x release for Windows. Project maintainer, Johannes Schindelin, wrote the following:

> There are 32-bit and 64-bit versions both of regular installers and portable installers ("portable"
> meaning that they are .7z archives that can be unpacked anywhere and run in place, without any need for
> running an installer).
>
> My projected time line is to hammer out the last kinks until Friday, and then continue after a one-week
> leave, if needed, and then finally retire msysGit and start the official 2.x release cycle of Git for Windows.

Other releases:

* GitLab shipped version [7.11.4](https://about.gitlab.com/2015/05/28/gitlab-7-dot-11-dot-4-released/) on top of the major [7.11](https://about.gitlab.com/2015/05/22/gitlab-7-11-released/) release. They also announced [a new GitLab logo](https://about.gitlab.com/2015/05/18/a-new-gitlab-logo/).
* [Rugged 0.22.2](https://github.com/libgit2/rugged/releases/tag/v0.22.2) was released, bumping their libgit2 version.


## Other News

__Various__
  
* Seasoned Git contributor [Jonathan Nieder received full committer status on the JGit project](http://dev.eclipse.org/mhonarc/lists/jgit-dev/msg02895.html)
* GitHub now has an official [Engineering Blog](http://githubengineering.com)
* [GitMinutes #36: Git Merge 2015 Part
2](http://episodes.gitminutes.com/2015/05/gitminutes-36-git-merge-2015-part-2.html),
another podcast episode from the conferencere (out of a total of 5)

__Light reading__

* [A statistician's initial experiences of Git/GitHub](http://thestatsgeek.com/2015/05/16/a-statisticians-initial-experiences-of-gitgithub/), by Jonathan Bartlett
* [The power of Git subtree](https://developer.atlassian.com/blog/2015/05/the-power-of-git-subtree/), by our own Nicola Paolucci
* [Advantages of Monolithic Version Control](http://danluu.com/monorepo/), by Dan Luu
* [My Global Git Commit Template](http://ericjmritz.name/2015/05/27/my-global-git-commit-template/), by Eric James Michael Ritz

__Git tools and sites__

* [gittorrent](http://blog.printf.net/articles/2015/05/29/announcing-gittorrent-a-decentralized-github/), a decentralized GitHub approach using the BitTorrent protocol
* [Infocalypse](http://draketo.de/english/freenet/real-life-infocalypse) gives you fully decentralized Github with real anonymity, using only free software
* [Meat!](https://getmeat.io/), a new Git hosting platform challenger
* [git-hub](https://github.com/ingydotnet/git-hub), GitHub commandline interface written in bash, similar to the venerable, and identically named..
* [hub](https://hub.github.com/), which is written in [<del>Ruby</del> Go](https://github.com/github/hub/issues/475)
* [Timeglass](https://github.com/timeglass/glass), automated time tracking for Git repositories
* [multigit](https://github.com/capr/multigit), layered git repositories
* [WDX_GitCommander](https://github.com/Darthholi/WDX_GitCommander), Git plugin for Total Commander
* [git-migrate](https://github.com/afshinm/git-migrate), a simple shell script to move Git repositories from one server to another
* [git-hooks](https://github.com/juanpabloaj/git-hooks) shows useful example hooks by language and hook name. Still quite empty, but interesting
* [GitCop](https://gitcop.com/) offers automitic inspection of commit messages pushed to GitHub 
* [Gitgest](https://github.com/ccidral/gitgest), a bash script that emails git commit digests in HTML format
* [git-sham](https://bitbucket.org/tpettersen/git-sham) manipulates the `GIT_COMMITTER_DATE` of the most recent commit until the SHA matches a particular pattern :)

## Credits

This edition of Git Rev News was curated by Christian Couder &lt;<christian.couder@gmail.com>&gt;,
Thomas Ferris Nicolaisen &lt;<tfnico@gmail.com>&gt; and Nicola Paolucci &lt;<npaolucci@atlassian.com>&gt;
with help from Junio C Hamano, Matthieu Moy and Emma Jane.
