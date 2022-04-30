---
title: "Rotate Mail File Items by Bash Script"
date: 2021-01-23
categories: ["Code", "Bash", "Mail"]
draft: false
---
# Rotate Mail File Items by Bash Script

On a server when you install an MTA, and it can send and receives emails,
it creates a file under `/var/spool/mail` for each user.
Generally, it appends data into the end of the file when a new email received.
During the time, this file will be growing and become bigger and bigger, so
it depends on the quota of users how much data it can receive.
Normally, you can put a quota to avoid data size to be exceeded.

What if that you want to have a rotation on the mail file of a user?
I know maybe you think that it's not a useful case, but in our case it
was useful. For example, alert logs are not only be sent via a messenger, but it also
is sent by an email. So, we can rotate the email file as we know what it contains.

If you want to rotate it by a rotating tools, it's not going to work because each email
contains several lines of data and also, you need to rotate based on items' received date and not lines of data.

In the `/var/spool/mail` you can see list of files which belong to each user.
For example, all emails of `user-1` is stored in `/var/spool/mail/user-1`.

To read this file, the best way is `mail` command:

```bash
$ mail -u user-1
```

When you run the `mail` command, it puts you in an interactive environment which you need to
send commands. The first and recommended command is `help` which gives you list of available commands.

> Note: you can also use `mu` command to read and filter emails easily.

To pass a command to a `mail` you can also use `echo`:

```bash
echo 'su' | mail -u user-1 -N
```

> `su` stands for `summary`

To rotate logs we need to have a bash script to be executed to rotate emails:

```bash
#!/bin/bash

[[ "$#" -ne "2" ]] && echo "invalid argument!" && exit 64

user="$1"
max_threshold="$2"

current_items_count=$(echo 'su' | mail -u "${user}" -N | grep 'Held' | awk '{print $2}')

[[ $current_items_count -lt $max_threshold ]] && echo "current ${current_items_count} is not more than ${2}" && exit 0

remove_items=$(($max_threshold-$current_items_count))
[[ $remove_items -eq "0" ]] && remove_items=1

echo "d 1-${remove_items#-}" | mail -u "${user}" -N
```

First parameter is username, and the second parameter is maximum threshold of email items.
It removes the old emails based on threshold number.

```bash
$ chmod +x rotate-mail.sh
$ ./rotate-mail.sh user-1 100
```

Now, you can save it in `rotate-mail.sh` file in `/usr/local/bin/` and define a cronjob for it:

```
0 * * * * bash rotate-mail.sh user-1 100
```

You can use [crontab.guru](https://crontab.guru/#0_*_*_*_*) to make your own schedule time.

Good resources:
* https://mailutils.org/manual/html_section/mail.html
* https://www.commandlinux.com/man-page/man1/Mail.1.html
