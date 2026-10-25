# Take-Home Assignment: Reading the Logs

You have a new log file: `assignment-auth.log`. It is from a different server than the one we used in class, and it has a different attack hidden in it. Your job is to investigate it using only the commands we covered today: `wc`, `head`, `tail`, `grep`, `cut`, `sort`, and `uniq`.

Setup:
```
cd ~/logclass
ls
# make sure you see assignment-auth.log
```

Write your answer AND the command you used for each question. The command matters as much as the answer. If you got the right number by guessing, you did not do the assignment.

## Part 1: Warm up

1. How many total lines are in the log?
2. How many login attempts failed? (One line equals one attempt.)
3. Show the last 3 lines of the log. What is the final action taken?

## Part 2: Filter and extract

4. Show every line where a login was accepted.
5. How many lines mention `Invalid user`?
6. Print only the usernames from `users.csv` (you still have this file from class).
7. Show every line involving the IP `45.155.205.99`.

## Part 3: Rank and count

8. Build a pipeline that ranks the attacking IPs by how many failed attempts each made. Which IP is the worst offender, and how many failed attempts did it make?
9. Count how many times each username was accepted (successful logins). Which account was accepted the most, and is that suspicious or normal?

## Part 4: The investigation

Answer these in full sentences. Use the commands to find your evidence.

10. One IP brute-forced this server, then succeeded. Find the single line where the attack worked. What time did it happen, and which account did the attacker get into?
11. What did the attacker do in the very next action after getting in?
12. In two or three sentences, describe the whole attack from start to finish, as if you were reporting it to your manager. Mention the attacker IP, the account they broke into, and what they did afterward.

## Bonus (optional, harder)

13. The compromised account had successful logins from more than one place in this log. Only one of them is the attacker. Explain how you can tell which accepted login is the malicious one and which are normal. (Hint: look at which IP each successful login came from, and compare against the IP that was doing all the failing.)

Submit your answers with the commands next session. Bring questions about anything that did not work.
