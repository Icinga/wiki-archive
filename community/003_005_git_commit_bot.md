#GIT Commit Bot

##Source
[Simple Gitbot](https://github.com/ehamberg/simple-gitbot)

Alternative: [Gitbot Alternative](http://tools.suckless.org/ii/)
##Setup
###simble-gitbot
Put it somewhere, where it can run, and call it as ruby script, putting it into the background, but logging stuff somewhere.
Modified for usage in **`#icinga-devel`**

Simple background script

	#!/bin/bash
	ruby simple-gitbot/gitbot.rb > /dev/null &
###git hook
Place the content of post-receive in the appropriate hooks script in your GIT repository. I've modified the output a bit to match the outpur for adding Icinga GIT web url.

* If more than 5 commits, only show short log to head
* passing each line with who, where, what, gitweb url
* nops will be filtered by git --log directly

```
~$ vim hooks/post-receive-irc

#!/bin/bash

fifo="/tmp/gitbotfifo"

read line
set -- $line

max=5
num=$(git log --pretty=oneline ${1}..${2}|wc -l)
branch=${3/refs\/heads\//}
repo=$(basename `pwd`)
url="https://git.icinga.org/?p=${repo};a=commit;h="
head="https://git.icinga.org/?p=${repo};a=shortlog;h=${3}"


if [ ${num} -ge ${max} ]; then
        echo "${num} commit(s) pushed to ${repo} on branch '${branch}': ${head}" > $fifo
else
        git log --pretty=format:"%cn [${repo}|${branch}] %s | ${url}%h%n" ${1}..${2} > $fifo
fi

exit 0

```

Include the git hook.

	Test.git$ vim hooks/post-receive

	. /var/cache/git/hooks/post-receive-irc

If using multiple hook scripts, the first one eats stdin, not leaving anything for the second one. use this trick.

	FILE=mktemp
	cat - > $FILE
	
	cat $FILE | /var/cache/git/hooks/post-receive-mail
	cat $FILE | /var/cache/git/hooks/post-receive-irc
	
	rm $FILE

##Example
###Single Commit
	[17:55:50] <icinga-gitbot> Michael Friedrich [Test.git|master] foobar | https://git.icinga.org/?p=Test.git;a=commit;h=6ca9f0f
###More than 5 Commits
	[18:03:22] <icinga-gitbot> 6 commit(s) pushed to Test.git on branch 'master': https://git.icinga.org/?p=Test.git;a=shortlog;h=refs/heads/master