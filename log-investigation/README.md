# Log investigation take home

An unaided assignment set after the text processing worksheet. A different
server, a different attack, no new commands.

Start here: [worksheet.md](worksheet.md)

## Data

| File | What it is |
|---|---|
| `data/auth.log` | SSH log from a second server with its own attack. |
| `data/users.csv` | The same user table from the class worksheet. |

```bash
cd log-investigation/data
wc -l auth.log
```

## How it is assessed

Students submit the command alongside each answer. A correct number with no
command, or with a command that could not have produced it, does not count.
The worksheet says so explicitly.

Questions 10 to 12 move from extraction to interpretation: locate the moment
the attack succeeded, identify what happened immediately after, then describe
the whole incident in a few sentences as a report to a manager. Question 13 is
an optional harder one that asks students to distinguish the attacker's
successful login from the legitimate ones on the same account.

Answer key is in [`../instructor/`](../instructor/).
