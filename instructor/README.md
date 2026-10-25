# Instructor material

| File | Use |
|---|---|
| `text-processing-answer-key.md` | Expected outputs for every task on the class worksheet. |
| `text-processing-explanation-guide.md` | Teaching companion. The mental model for each command, a breakdown of every flag, the mistakes students predictably make, and how each piece fits the final pipeline. Read the relevant section before demonstrating. |
| `log-investigation-answer-key.md` | Answers for the take home, including acceptable alternative pipelines. |

## Before you teach

Confirm the keys still match the data. They were verified against the files in
this repository, but if you edit a log or a CSV the counts move:

```bash
cd text-processing/data
wc -l auth.log                 # 33
grep -c "Accepted" auth.log    # 5
grep -c "Invalid user" auth.log # 4
grep -c "root" auth.log        # 13
grep "Failed password" auth.log | grep -oE "from [0-9.]+" | cut -d' ' -f2 | sort | uniq -c | sort -rn
# 17 203.0.113.42 / 3 198.51.100.7 / 3 192.0.2.88
```

## Marking the take home

The command carries the mark, not the number. A student who reaches the right
answer with a clumsy but valid pipeline has met the objective. A student who
produces the right number with no working has not.

Questions 11 and 12 are where the real signal is. Extraction can be guessed at.
Explaining an attack in order, in plain language, to a non technical reader is
the part that separates someone ready for an analyst role from someone who has
memorised flags.
