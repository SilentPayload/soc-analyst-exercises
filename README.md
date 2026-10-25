# SOC Analyst Exercises

Hands on log analysis exercises for people training into security analyst
roles. Each exercise ships with its data, so you can clone this and start
working immediately. Instructor answer keys are included.

Written for live cohorts. Students use nothing but a terminal and the standard
text processing tools: `wc`, `head`, `tail`, `grep`, `cut`, `sort`, `uniq`.

## Exercises

### [Text processing for log analysis](text-processing/)

A command by command worksheet. Each tool gets two worked examples to demo,
then tasks students complete before moving on. Builds up to a full pipeline
that ranks attacking IP addresses by failed login count.

Data: an SSH log with a brute force attack and a successful compromise hidden
in it, plus a small user table.

### [Log investigation take home](log-investigation/)

An unaided assignment on a second, different log. Students answer thirteen
questions and must submit the command they used alongside each answer, because
the command is the thing being assessed. The final questions ask them to
reconstruct the attack in prose, as if reporting it to a manager.

## Datasets

[`datasets/`](datasets/) holds two larger files for SIEM and timeline work:
a labelled attack timeline covering SQL injection through to data access, and
an enterprise event log with logins, file access, severity and geolocation.

## Instructor material

[`instructor/`](instructor/) holds the answer keys and a teaching companion
that explains the mental model behind each command, every flag used, and the
mistakes students reliably make.

**A note on publishing keys.** This repository is public, so the answer keys
are public too. That is deliberate: they demonstrate how the exercises were
built. If you intend to run these for assessment, regenerate the data or
change the questions first, because your students can read this page.

## Running the exercises

```bash
git clone https://github.com/SilentPayload/soc-analyst-exercises.git
cd soc-analyst-exercises/text-processing/data
wc -l auth.log
```

Each exercise folder has its data in `data/` alongside the worksheet. Nothing
to install.

## About the data

All of it is synthetic. The logs were written to contain a specific,
discoverable attack, not captured from a real system. Addresses are drawn from
the ranges reserved for documentation and private use in RFC 5737 and RFC 1918,
so nothing here points at a real host.

The text processing log is reconstructed to match the recorded answer key
exactly: 33 lines, 5 accepted logins, 4 invalid user attempts, and a failed
login ranking of 17, 3 and 3 across three source addresses. If you edit the
data, re-run the key and update it.

## Licence

Exercises, worksheets and keys are released under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use them, adapt
them and teach them, with attribution. See [LICENSE](LICENSE).

## Author

Praise Abu Ugbede, [SilentPayload](https://github.com/SilentPayload).
Penetration tester and security educator, Abuja, Nigeria.

Companion curriculum:
[linux-networking-foundations](https://github.com/SilentPayload/linux-networking-foundations).
