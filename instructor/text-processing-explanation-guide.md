# Instructor Explanation Guide

This is your teaching companion. For each command it gives you the mental model, a breakdown of every flag and piece of syntax you will use, the mistakes students predictably make, and how the command fits into the bigger pipeline. Read the command's section before you demo it. The goal is that you can answer "but why does that work?" without hesitating.

A note on the whole lesson: these commands are small on their own. Their power is that they connect. Unix tools are built to do one job and pass their output to the next tool through a pipe (`|`). Keep returning to that idea. By the end, `grep | cut | sort | uniq -c | sort -rn` should feel like a sentence, not five separate words.

---

## The pipe: `|`

Explain this before anything else, because every interesting task depends on it.

A pipe takes the output of the command on its left and feeds it as the input to the command on the right. Nothing is saved to a file in between. The data just flows through.

Mental model: an assembly line. Raw material goes in one end. Each station does one small thing and hands the result to the next station. `grep` pulls out the lines you care about, hands them to `cut`, which pulls out one column, hands that to `sort`, and so on. No single station does everything. The line as a whole does.

Why analysts love it: you never have to create messy temporary files. You build a chain, look at the output, and if it is wrong you adjust one link.

Common student confusion: they think the pipe runs commands "at the same time." It does not run them independently. Each command's output literally becomes the next one's input. Show them by breaking a pipeline in half and running just the left side, then adding one stage at a time.

---

## 1. wc — word count

Essence: counts things. Lines, words, or characters.

Mental model: it does not read or understand the file. It just tallies. Think of it as a person flipping through pages and counting, not reading.

The flags:
- `-l` counts lines. This is the one you will use 90 percent of the time, because in a log, one line usually equals one event.
- `-w` counts words (anything separated by spaces).
- `-c` counts characters (bytes, technically).

The key teaching point: `wc -l file` counts every line in the file. But `grep -c "pattern" file` counts only the lines that match a pattern. Students conflate these. Drive the distinction home: `wc` is "how big is this?" and `grep -c` is "how many of a specific thing?"

Gotcha to demo: `wc -l users.csv` returns 11, but there are only 10 users. The eleventh line is the header row. This teaches them that raw counts often include something you did not mean to count, which is a recurring theme in data work.

Security framing: the first thing you do when handed a log during an incident is measure it. "Are we looking at 200 lines or 2 million?" changes your entire approach. `wc -l` is that first measurement.

---

## 2. head and tail — the top and the bottom

Essence: show the beginning (`head`) or end (`tail`) of a file without opening the whole thing.

Mental model: peeking. You do not want to dump a huge log to the screen. You glance at the top or bottom to get your bearings.

The flags:
- `head -n N` shows the first N lines. Default is 10 if you leave off `-n`.
- `tail -n N` shows the last N lines. Default is also 10.
- `tail -n +2` is a special form. The plus sign means "start from line 2 and go to the end." You will use this constantly to skip a CSV header. Make sure they understand `+2` means "from line 2 onward," not "the second line only."

The flag worth mentioning even if you do not demo it: `tail -f`. The `-f` means "follow." It keeps the file open and prints new lines as they are written. This is how a real analyst watches a log live during an attack. Students find this genuinely exciting, so it is worth a 60-second mention even though it is not in the tasks.

Why order matters in logs: logs are append-only and time-ordered. New events go at the bottom. That is why `tail` shows you "what just happened" and `head` shows you "how this started." Point at the last line of the sample log (the malicious wget) and ask why that being at the bottom is significant. Answer: it is the most recent thing the attacker did.

---

## 3. grep — the search command

Essence: print only the lines that match a pattern. This is the most important command in the whole lesson.

Mental model: a filter or a sieve. The whole file pours in. Only the lines matching your pattern come out the other side. Everything else is discarded.

The flags:
- `-c` counts matches instead of printing them.
- `-i` ignores case, so `root`, `Root`, and `ROOT` all match. Important because attackers and systems are inconsistent about capitalisation.
- `-v` inverts the match. It prints every line that does NOT contain the pattern. This confuses students at first. Frame it as "show me everything except this." Useful for stripping out noise you already understand so you can see what is left.
- `-o` prints only the matched portion, not the whole line. This is how you extract just an IP or just a word from a messy line. It is the bridge to being able to count specific things.
- `-E` turns on extended regular expressions, which lets you use patterns like `[0-9.]+` (one or more digits and dots, meaning an IP). Without `-E` some pattern characters do not work.

Regex, kept simple: they do not need full regular expressions today. Teach exactly three pieces. `[0-9]` means "any single digit." The `.` inside a character class `[0-9.]` means a literal dot. The `+` means "one or more of the thing before it." So `[0-9.]+` reads as "a run of digits and dots," which is close enough to an IP address for our purposes. Do not go deeper than this or you will lose the room.

The single most useful combination to show them: `grep -oE "from [0-9.]+"` pulls the word "from" plus the IP out of each line, throwing away everything else. That is the setup for counting attackers.

Security framing: grep is how you answer almost every log question. "Show me failures." "Show me this IP." "Show me everything except the noise." An analyst runs grep dozens of times an hour.

---

## 4. cut — extract a column

Essence: split each line on a delimiter and keep only the field you want.

Mental model: scissors. Each line is a strip of paper with fields separated by a marker (a comma, a space). `cut` snips out the piece you asked for and drops the rest.

The flags:
- `-d` sets the delimiter, the character that separates fields. `-d,` for a comma, `-d' '` for a space, `-d:` for a colon.
- `-f` picks the field by number, starting from 1. `-f1` is the first field. `-f1,3` grabs the first and third. `-f2-4` grabs a range.

The critical limitation, and this is where you set up why `awk` exists later: `cut` only works cleanly when fields are separated by a single, consistent delimiter. A CSV is perfect because every field is separated by exactly one comma. But a log line separates fields with a single space in some places and multiple spaces in others. If you try `cut -d' '` on a log line where two spaces sit between fields, `cut` sees an empty field between them and your field numbers shift. This is a real gotcha. When a student's `cut` on the log gives weird output, this is usually why. Tell them: for clean data like CSV, `cut` is perfect. For messy, irregularly spaced data like logs, you will want `awk`, which is smarter about spacing. That plants the seed for section 9.

Security framing: logs and CSVs are columns of data. Extracting "just the IP column" or "just the username column" is step one of turning raw log text into something you can count and rank.

---

## 5. sort — put lines in order

Essence: reorder lines. Alphabetically by default, numerically if you ask.

Mental model: on its own, sort feels pointless. Emphasise to yourself and to them that sort is a setup command. It exists to make the NEXT command work. Its main partner is `uniq`, which can only count repeats if identical lines are sitting next to each other. Sort is what puts them next to each other.

The flags:
- `-n` sorts numerically. Without it, sort treats everything as text.
- `-r` reverses the order.
- `-rn` combined means "numeric, highest first," which is what you want when ranking attackers.
- `-u` removes duplicates while sorting (a shortcut that does some of what uniq does).

The gotcha you must demo, because it teaches a deep lesson: sort numbers without `-n` and watch what happens. Text sorting compares character by character. So `10` comes before `2`, because it compares the first character, sees `1` versus `2`, and `1` wins. To a beginner this looks broken. It is not broken. It is sorting as text, exactly as told. Run `sort` then `sort -n` on the same numbers side by side. This single demo teaches them that the computer does precisely what you say, not what you mean, which is the most important lesson in all of computing.

Security framing: you rarely sort for its own sake. You sort so you can then count and rank. Ranking is how you find the worst offender, the busiest port, the most-hit endpoint.

---

## 6. uniq — collapse and count repeats

Essence: `uniq -c` counts how many times each line appears. This is the command that turns a pile of events into a ranked summary.

Mental model: a tally counter. It walks down the list and every time it sees the same line as the one before, it adds to the count. When the line changes, it prints the count and the line, and starts a new tally.

The flags:
- `-c` prepends a count to each unique line. This is the one that matters.
- `-d` shows only lines that are duplicated (appear more than once).

The rule you must hammer: uniq only compares each line to the one immediately before it. It has no memory beyond that. So if identical lines are scattered through the file, uniq will not notice they are the same. This is why sort always comes first. Sort gathers all identical lines together, then uniq can count them in one pass. If a student runs `uniq -c` without sorting first and gets nonsense counts, this is always the reason.

The signature pipeline, which you should be able to explain link by link: `sort | uniq -c | sort -rn`. Read it aloud as "gather the duplicates together, count each group, then sort the counts highest first." This three-command move ranks anything: attacker IPs, targeted usernames, hit URLs, error types. It is the workhorse of log analysis. If students leave today knowing only this pattern, the class succeeded.

Security framing: "who attacked the most" is a counting-and-ranking question. So is "which account was targeted most," "which port was scanned most," "which error is most common." All of them are `sort | uniq -c | sort -rn` on the right column.

---

## 7. tr — translate characters

Essence: swap one set of characters for another, or delete characters entirely.

Mental model: find-and-replace, but only for individual characters, not words. It works one character at a time. This is its main limitation and the thing students misunderstand.

The syntax and flags:
- `tr 'ABC' 'abc'` replaces A with a, B with b, C with c. It maps the first set to the second, position by position.
- `tr 'A-Z' 'a-z'` uses ranges to convert all uppercase to lowercase.
- `tr ',' '\n'` replaces every comma with a newline. `\n` is the code for a newline character. This is how you turn a comma-separated row into a vertical list.
- `-d` deletes the listed characters instead of replacing. `tr -d '0-9'` strips out all digits.
- `-s` squeezes repeated characters down to one.

The gotcha: `tr` cannot replace words. `tr 'Failed' 'FAIL'` does not turn the word "Failed" into "FAIL." It maps F to F, a to A, i to I, l to L, and so on, character by character, which produces garbage. If a student wants to replace a whole word, that is `sed`, the next command. Be ready for this confusion, because the two commands look similar but work completely differently. `tr` is characters. `sed` is patterns and words.

Security framing: lighter than the others, but real. Normalising case with `tr` before matching means you do not miss `FAILED` when you searched for `failed`. And reshaping data (comma-separated to line-separated) is a common cleanup step before feeding data to another tool.

---

## 8. sed — stream editor

Essence: edit text as it flows past, most commonly by find-and-replace on patterns and words.

Mental model: a robot proofreader standing in the stream. Every line flows past it. For each line, it applies your edit rule, then lets the line continue on. It never opens the file in an editor. It transforms the text in transit.

The syntax you will use:
- `s/old/new/` is the substitute command. The `s` means substitute. The slashes separate the three parts: the `s`, the thing to find, and the thing to replace it with. So `s/Failed/FAIL/` finds "Failed" and replaces it with "FAIL."
- Add `g` at the end, `s/old/new/g`, to replace every occurrence on a line, not just the first. Without `g`, sed only replaces the first match on each line. Demo both so they see the difference.
- `-n` combined with `p` prints only matching lines. `sed -n '/Accepted/p'` prints only lines containing "Accepted." The `-n` silences the normal "print everything" behaviour, and the `p` re-enables printing just for matches. Point out this behaves almost exactly like grep, which teaches them that tools overlap and there is often more than one way to do a thing.
- `d` deletes matching lines. `sed '/noise/d'` drops every line containing "noise."

The gotcha that always bites: dots in patterns. In sed's pattern language, a bare `.` means "any single character," not a literal dot. So the pattern `192.0.2.88` would also match `192x0y2z88`. To match a real dot you escape it with a backslash: `192\.0\.2\.88`. Students will forget this. When their IP redaction matches too much or behaves oddly, this is why. The rule: inside a sed pattern, a dot you actually want as a dot needs a backslash in front.

Security framing: this is your redaction and sanitisation tool. Before you share a log with a third party, you `sed` out the internal IPs, hostnames, or usernames. It is also used to clean data into a consistent format before analysis. Both are daily analyst tasks.

---

## 9. awk — the field-aware power tool

Essence: a small programming language for text. It reads each line, automatically splits it into fields, and lets you print specific fields, filter by conditions, and do arithmetic. It can do in one step what several earlier commands did together.

Mental model: awk runs your instructions once for every line in the file, top to bottom. For each line, it has already split the line into numbered fields for you. You just say what to do with them.

The core concepts, which are the real content of this section:
- `$1`, `$2`, `$3` are the fields, numbered from the left. `$1` is the first field, `$2` the second, and so on.
- `$0` is the entire line, unsplit.
- `NF` is the Number of Fields on the current line. So `$NF` is a shortcut for "the last field," and `$(NF-1)` is "second to last." This matters for logs, where you often want a field near the end and counting from the right is easier.
- `NR` is the Number of the current Record, meaning the line number as awk works through the file. `NR>1` means "only on lines after the first," which is how you skip a CSV header. This is the awk way to do what `tail -n +2` did earlier.
- `-F` sets the field separator, the same idea as `cut`'s `-d`. `-F,` splits on commas.

The structure of an awk program: `pattern { action }`. For each line, if the pattern is true, run the action. Either part can be left out.
- `awk '/Failed/ {print $1}'` means "on lines matching Failed, print field 1."
- `awk -F, 'NR>1 {print $1, $5}'` means "on lines after the first, print fields 1 and 5."
- `awk -F, 'NR>1 && $5>2 {print $1}'` adds a condition with `&&` meaning "and." So "after the header, AND where field 5 is greater than 2, print field 1." This is filtering and column selection in a single command, which is the moment students see why awk is powerful.

The END block, for the hardest task: `awk '{ s+=$5 } END { print s/n }'` style code runs the main action for every line, accumulating a total, then runs the END block once after all lines are done. This is how you compute a sum or an average. It is genuinely a small program. Do not be surprised if this task is a stretch. It is meant to be the ceiling of the lesson.

Why awk succeeds where cut struggled: remember the cut gotcha with irregular spacing in logs. awk handles that automatically. By default it treats any run of whitespace as a single separator, so messy log spacing does not shift your field numbers. This is the payoff of the seed you planted in section 4. When a student asks "why not just use cut," this is the answer: cut breaks on messy spacing, awk does not.

Security framing: awk is the analyst's Swiss army knife for turning log lines into exactly the data you want, filtered by exactly the condition you care about, in one command. "Show me the time and source IP of every failed login" is a single awk. Once they are comfortable here, they can process almost any structured log.

A caution for you as the instructor: awk is not really a Foundations-level command. It is a programming language wearing a command-line disguise. Keep today's treatment narrow: fields, printing, one condition, and if the group is strong, the END block. Resist the urge to teach awk's full capability. That is a lesson of its own.

---

## Closing note for the instructor

The arc of this lesson is small tools combining into real capability. If you find yourself running short on time, protect the pipeline demo in section 6 above everything else. A student who leaves understanding `grep | cut | sort | uniq -c | sort -rn` has learned the actual shape of log analysis. Everything else is detail they can pick up later. The pipeline is the lesson.
