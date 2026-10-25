# Text processing for log analysis

Students learn `wc`, `head`, `tail`, `grep`, `cut`, `sort` and `uniq`, one at a
time, then combine them into a pipeline that ranks attacking IP addresses.

Start here: [worksheet.md](worksheet.md)

## Data

| File | What it is |
|---|---|
| `data/auth.log` | SSH log, 33 lines. Contains a brute force attack from one address and the single successful login where it worked. |
| `data/users.csv` | Ten users with address, role, department and failed login count. |

Work from the `data/` directory so the commands in the worksheet run as written:

```bash
cd text-processing/data
wc -l auth.log
```

## What the log contains

Without giving away the answers: one address is responsible for the large
majority of failed attempts, it eventually succeeds, and the final line of the
log shows what the intruder did next. Students find all of that with `grep`,
`cut`, `sort` and `uniq` alone.

Answer key and teaching notes are in [`../instructor/`](../instructor/).
