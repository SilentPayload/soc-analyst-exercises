# Text Processing for Log Analysis

A command-by-command worksheet. Each command has two worked examples for the instructor to demo, then a set of tasks for students to do before moving on.

Every task uses one of two files: `auth.log` (an SSH server log with a real attack hidden in it) or `users.csv` (a small user table). Students should have both files in their working directory before starting.

Setup command for students:
```
cd ~/logclass
ls
# you should see: auth.log  users.csv
```

---

## How to run this class

Teach depth-first, not breadth-first. Do not rush to cover all nine commands. If students only truly learn six, that is a win.

Core six to aim for today (they build the full attacker-hunting pipeline by the end): `wc`, `head`/`tail`, `grep`, `cut`, `sort`, `uniq`.

Stretch three for next session or fast groups: `tr`, `sed`, `awk`.

Rhythm per command: instructor demos the two examples out loud, explaining each flag, then students do the tasks on their own keyboards. Do not move on until most of the room has finished. The instructor answer key is at the bottom of this document. Do not hand that part out.

---

## 1. wc — count lines, words, characters

Why it matters: the first question in any log review is "how much am I even looking at?" Counting is how you measure the size of a problem.

Flags you will use: `-l` lines, `-w` words, `-c` characters.

Instructor example A:
```
wc -l auth.log
```
Counts how many lines the whole log has.

Instructor example B:
```
grep -c "Failed password" auth.log
```
`grep -c` counts only the matching lines. This is the fast way to answer "how many failed logins were there?"

Student tasks:
1. How many lines are in `auth.log`?
2. How many lines are in `users.csv`? (Remember one line is the header.)
3. How many lines contain the word `Accepted`?
4. How many lines contain `Invalid user`?
5. In one sentence, explain the difference between `wc -l file` and `grep -c "text" file`.

---

## 2. head and tail — look at the top and bottom

Why it matters: logs are written in time order. The newest events are at the bottom. `tail` is how an analyst checks "what just happened," and `tail -f` is how you watch a log live during an incident.

Flags you will use: `head -n N` first N lines, `tail -n N` last N lines.

Instructor example A:
```
head -n 5 auth.log
```
Shows the first five lines, the oldest events.

Instructor example B:
```
tail -n 3 auth.log
```
Shows the last three lines. Point out the final line: someone ran a `wget` of a script as root. Ask the room what that might mean before explaining.

Student tasks:
1. Show the first 10 lines of `auth.log`.
2. Show the last 5 lines of `auth.log`.
3. Show only the header row of `users.csv`.
4. What is the very last action taken in the log, and why should it worry you?
5. Explain when you would use `tail` instead of `head` on a log file.

---

## 3. grep — find lines that match

Why it matters: this is the single most used command in log analysis. It is how you pull the needles out of the haystack. Students may have met it already; here it gets a security job.

Flags you will use: `-c` count, `-i` ignore case, `-v` invert (show lines that do NOT match), `-o` show only the matched part, `-E` extended regex.

Instructor example A:
```
grep "Failed password" auth.log
```
Pulls every failed login attempt.

Instructor example B:
```
grep -v "Failed password" auth.log
```
The `-v` flips it: everything EXCEPT the failures. Useful for asking "what else happened besides the noise?"

Student tasks:
1. Show every line containing `Accepted password`.
2. Count how many lines mention the user `root`.
3. Show every line for the IP `203.0.113.42`.
4. Show every line that is NOT a `Failed password` line.
5. Find the single line where the attack finally succeeded. (Hint: what word marks a successful login, and which suspicious IP should it be paired with?)

---

## 4. cut — pull out columns

Why it matters: log lines and CSV rows are made of fields. `cut` grabs the field you want and throws away the rest. This is your first tool for turning messy text into clean data.

Flags you will use: `-d` delimiter (the character that separates fields), `-f` field number.

Instructor example A:
```
cut -d, -f1 users.csv
```
`-d,` says fields are separated by commas. `-f1` grabs the first field. This prints every username.

Instructor example B:
```
cut -d, -f1,3 users.csv
```
You can ask for more than one field. This prints username and role together.

Student tasks:
1. Print only the `ip` column from `users.csv`.
2. Print the `role` column.
3. Print username and department together.
4. Print the `failed_logins` column (the last one).
5. Harder: the header row prints too and clutters the output. Combine `cut` with `tail -n +2` using a pipe to skip the header. (`tail -n +2` means "start from line 2.")

---

## 5. sort — put lines in order

Why it matters: on its own, sorting looks boring. Its real power shows up when paired with `uniq` in the next section to count and rank things. Teach this one knowing the payoff is coming.

Flags you will use: `-n` sort numerically (not alphabetically), `-r` reverse, `-u` unique.

Instructor example A:
```
cut -d, -f3 users.csv | sort
```
Grabs the role column and sorts it alphabetically. Notice matching roles now sit next to each other. That grouping is the setup for counting.

Instructor example B:
```
cut -d, -f5 users.csv | tail -n +2 | sort -rn
```
Grabs the failed_logins numbers, skips the header, and sorts them highest first. `-rn` is reverse numeric.

Student tasks:
1. Print all usernames sorted alphabetically.
2. Print all IPs from the CSV, sorted.
3. Print the failed_logins column sorted lowest to highest.
4. Print the same column sorted highest to lowest.
5. Explain what goes wrong if you sort numbers with plain `sort` instead of `sort -n`. (Try it and see.)

---

## 6. uniq — collapse and count repeats

Why it matters: this is the command that turns "a list of attacks" into "who attacked the most." `uniq -c` counts. But it only works on sorted input, which is why sort came first. Chaining `sort | uniq -c | sort -rn` is the classic move that ranks anything.

Flags you will use: `-c` count occurrences, `-d` show only duplicates.

Instructor example A:
```
cut -d, -f3 users.csv | tail -n +2 | sort | uniq -c
```
Counts how many users have each role. Walk through it left to right: cut the column, drop the header, sort so repeats group, then count.

Instructor example B (the payoff, build it live piece by piece):
```
grep "Failed password" auth.log | grep -oE "from [0-9.]+" | cut -d' ' -f2 | sort | uniq -c | sort -rn
```
This ranks attacker IPs by number of failed attempts. Do not paste it all at once. Build it one pipe at a time and show the output growing sharper at each step. This is the moment the whole lesson pays off.

Student tasks:
1. Count how many users are in each `department` (CSV column 4).
2. Rank attacker IPs by failed-login count (rebuild the instructor pipeline yourself).
3. Which single IP made the most failed attempts, and how many?
4. Count how many times each username appears in the `Failed password` lines of `auth.log`.
5. In your own words, explain why `uniq` needs its input sorted first.

---

## 7. tr — translate or delete characters

Why it matters: a lighter tool, but handy for cleanup. It swaps one set of characters for another, or strips characters out. Common uses: change case, or turn a comma-separated line into one-item-per-line.

Flags you will use: `-d` delete characters, `-s` squeeze repeats.

Instructor example A:
```
head -n 1 users.csv | tr ',' '\n'
```
Takes the header row and turns every comma into a newline, printing each column name on its own line.

Instructor example B:
```
echo "Failed Password" | tr 'A-Z' 'a-z'
```
Converts uppercase to lowercase. Useful when you want to match text regardless of case.

Student tasks:
1. Take the first data row of `users.csv` and print each field on its own line using `tr`.
2. Convert the string `ADMIN` to lowercase with `tr`.
3. Delete every digit from the first line of `auth.log` using `tr -d`.
4. Explain one situation in log work where forcing everything to lowercase would help.

---

## 8. sed — find and replace in a stream

Why it matters: `sed` edits text as it flows past. Its most common job is search-and-replace. In security work you will use it to redact sensitive values (like masking an IP in a report) or to clean data before analysis.

Flags and syntax you will use: `s/old/new/` substitute, `-n` with `p` to print only matches, `d` to delete lines.

Instructor example A:
```
grep "203.0.113.42" auth.log | sed 's/203\.0\.113\.42/[REDACTED]/'
```
Replaces the attacker IP with `[REDACTED]`. The backslashes protect the dots so they are treated as literal dots. This is exactly how you would sanitise a log before sharing it.

Instructor example B:
```
sed -n '/Accepted/p' auth.log
```
`-n` silences normal output, and `/Accepted/p` prints only lines matching `Accepted`. This behaves like `grep`, which is worth pointing out: many tools overlap.

Student tasks:
1. Replace every occurrence of the word `Failed` with `FAIL` in the log output.
2. Redact the IP `192.0.2.88` to `[REDACTED]` in the log.
3. Use `sed -n` to print only the lines containing `Invalid user`.
4. Replace `web01` with `SERVER` everywhere it appears.
5. Explain why the dots in an IP address need a backslash in front of them inside a `sed` pattern.

---

## 9. awk — the field-aware power tool

Why it matters: this is the big one. `awk` reads each line, splits it into fields automatically, and lets you print, filter, and calculate. Everything the earlier commands did, `awk` can often do in one step. Save it for last because it rewards everything they have learned.

Key ideas: `$1`, `$2` etc are fields. `$0` is the whole line. `NF` is the number of fields. `NR` is the current line number. `-F` sets the delimiter.

Instructor example A:
```
awk -F, 'NR>1 {print $1, $5}' users.csv
```
`-F,` splits on commas. `NR>1` skips the header. It prints username and failed_logins for every user.

Instructor example B:
```
awk -F, 'NR>1 && $5>2 {print $1, $5}' users.csv
```
Adds a condition: only print users whose failed_logins (field 5) is greater than 2. This is filtering and column selection in a single command.

Student tasks:
1. Print just the username and department from `users.csv` using `awk`.
2. Print only users whose `failed_logins` is 0 (the clean accounts).
3. Print only users whose `failed_logins` is 5 or more (the suspicious accounts).
4. From `auth.log`, print the timestamp and IP of every `Failed password` line. (Hint: figure out which field numbers hold the time and the IP.)
5. Harder: calculate the average number of failed logins across all users in the CSV. (Hint: add up field 5, count the rows, divide at the end using an `END` block.)

---

## Capstone challenge

Give this after the core commands are done. Students combine everything into one investigation.

The story: someone broke into this server. Your job is to prove it with the command line.

Answer these using pipelines of the commands you learned today:

1. How many failed login attempts happened in total?
2. Which IP is the main attacker, and how many failed attempts did it make?
3. Which usernames was the attacker targeting?
4. Find the exact log line where the attacker succeeded. What time was it, and which account did they get into?
5. What did the attacker do immediately after getting in? (Look at the last line.)
6. Write one or two sentences describing the full attack, start to finish, as if reporting it to a manager.

There is no single correct pipeline. Reward any approach that gets the right answer. The thinking is the point.
