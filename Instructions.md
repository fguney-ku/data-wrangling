Today, we are going to examine the log file of our cluster's (KUACC) login node: "kuacc.log" 

Let's say we're only interested in ai node jobs:

`cat kuacc.log | grep " ai " > ai.log`

We can use the command `sed` (stream editor) to modify a file, rather than manipulate its contents directly. 

For example, we can use it for substitution.
Since they are all ai node now, let's remove that column:

`cat ai.log | sed 's/\s*ai //'`

We used a simple regular expression; a powerful construct that lets you match text against patterns. The `s` command is written on the form: `s/REGEX/SUBSTITUTION/`, where `REGEX` is the regular expression you want to search for, and SUBSTITUTION is the text you want to substitute matching text with, nothing in that case.


** Regular Expressions **

Let’s start by looking at the regular expression we used above: 

`/\s*ai /`

Regular expressions are usually (though not always) surrounded by `/`. Most ASCII characters just carry their normal meaning, but some characters have "special" matching behavior. Exactly which characters do what vary somewhat between different implementations of regular expressions, which is a source of great frustration. Very common patterns are:

* `.` means any single character except newline
* `*` zero or more of the preceding match
* `+` one or more of the preceding match
* `[abc]` any one character of `a`, `b`, and `c`
* `(RX1|RX2)` either something that matches `RX1` or `RX2`
* `^` the start of the line
* `$` the end of the line

`sed`'s regular expressions are somewhat weird, and will require you to put a `\` before most of these to give them their special meaning. Or you can pass `-E`.

So, looking back at `/\s*ai /`, we see that it matches any text that starts with any number of space (`\s*`) characters, followed by the literal string "ai" and space: `ai_`.

Beware, regular expressions are tricky. What if there was a username which contains "ai" and space?

`*` and `+` are, by default, greedy. They will match as much text as they can. So, if there was a username which contains `ai_`, we'd remove that as well.

We can try to be a bit more specific, since we know job id is always 7 digits:

`cat ai.log | sed -E 's/^([0-9]{7})\s*ai /\1/'`

Now, we will not confuse usernames that contain `ai_`. `[0-9]` means all the digits and following `{7}` means exactly 7 of them. 

We put it in paratheses `([0-9]{7})` because we want to keep the 7-digit job id after all. For this, we can use "capture groups". Any text matched by a regex surrounded by parentheses is stored in a numbered capture group. These are available in the substitution as `\1`, `\2`, `\3`, etc. So, the following `\1` in the `sed` puts the job id back.

As you can probably imagine, you can come up with really complicated regular expressions. Regular expressions are notoriously hard to get right, but they are also very handy to have in your toolbox!


** Data Wrangling **

`sed` can do all sorts of other interesting things, like injecting text (with the `i` command), explicitly printing lines (with the `p` command), selecting lines by index, and lots of other things. Check `sed --help`!

Let's try to get all the usernames in the log file:

`cat ai.log | sed -E 's/^([0-9]{7})\s*ai\s+\w+\s+(\w+)\s*.*/\2/'`

`\w` stands for "word character". It always matches the ASCII characters `[A-Za-z0-9_]`. 

What we have now gives us a list of all the usernames on partition ai. 
Some users submit more than one job, let’s look for unique ones and counts:

`cat ai.log | sed -E 's/^([0-9]{7})\s*ai\s+\w+\s+(\w+)\s*.*/\2/' | sort | uniq -c`

`sort` will sort its input. `uniq -c` will collapse consecutive lines that are the same into a single line, prefixed with a count of the number of occurrences.

Let's say we want to sort that too and only keep the top two most common usernames:

`cat ai.log | sed -E 's/^([0-9]{7})\s*ai\s+\w+\s+(\w+)\s*.*/\2/' | sort | uniq -c | sort -nk1,1 | tail -n2`

`sort -n` will sort in numeric (instead of lexicographic) order. `-k1,1` means "sort by only the first whitespace-separated column". The `,n` part says sort until the nth field, where the default is the end of the line.

If we wanted the least common ones, we could use head instead of tail. There's also `sort -r`, which sorts in reverse order.









