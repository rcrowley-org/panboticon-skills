---
name: panboticon-audit
description: Audit and summarize human and bot user activity based on Linux kernel audit logs, systemd journal, etc. Use whenever asked to audit, review, report on, or summarize actions, activity, history, logs, or what a bot did.
allowed-tools: Agent, Bash, Glob, Grep, Write
user-invocable: true
---

# Panboticon audit

## Input data

Audit all log data available on the server unless instructed to audit only a specific time period.

1. Linux kernel audit logs from auditd in `/var/log/audit`
2. All other files in `/var/log`; glob to find files
3. The systemd journal via `journalctl`

## Principles

* Attribute records by `auid=` not `uid=` as this field survives e.g. `sudo` and `setuid` calls.
* Audit activity by non-system users. Conventionally, that's any (a)uid ≥ 1000. (a)uid 1234 and (a)uid 10101, if they exist, are of particular interest.
* Do not worry about identifying humans and bots. The reader will know which is which by name and (a)uid.
* Ignore records with `auid=4294967295`.
* Parse records directly instead of using `ausearch`.
* Do not trust `-ts` / `-te` filtering; they are unreliable.
* Always use the timestamps from the records (i.e. the log line); do not infer ordering from log filenames.
* Always use UTC; convert other timezones to UTC.
* Do not speculate about the effect or intent of a session; just summarize the facts.

## Process

1. Glob to enumerate all the log files to process, plus the systemd journal via `journalctl -o short-iso-precise --utc`.

2. For each of these files plus the systemd journal, in a sub-agent, do the following:

    a. Extract timestamps from the `audit(<timestamp>.` in audit log records, from the prefix of systemd journal entries, etc. - fall back to extracting common date formats and parsing them with the (GNU) `date -u -d` command. Convert all to RFC 3339 format.

    b. If auditing a specific time period, drop records with timestamps outside that period. In case of the systemd journal, use `journalctl --since ... --until ...` to filter.

    c. Decode full command lines from `PROCTITLE` records with (GNU) `awk`, substituting each system call number we audit one at a time for `SYSTEM_CALL_NUMBER` (`aarch64`: 221 = `execve`, 203 = `connect`; `x86_64`: 59 = `execve`, 42 = `connect`):

        # TODO(Claude): The SYSCALL record contains arch=c000003e (x86_64) / arch=c00000b7 (aarch64) — key off that instead of guessing.

        function hex2s(h,  s,i,c){ s=""; for(i=1;i<=length(h);i+=2){
          c=strtonum("0x" substr(h,i,2)); if(c==0)c=32; s=s sprintf("%c",c) } return s }
        /type=PROCTITLE/ { match($0,/:([0-9]+)\)/,a);
          match($0,/proctitle=([0-9A-F]+|"[^"]*")/,p); pt=p[1];
          if (pt ~ /^"/){ gsub(/"/,"",pt) } else { pt=hex2s(pt) }; T[a[1]]=pt }
        /type=SYSCALL/ && /syscall=SYSTEM_CALL_NUMBER / && / auid=[1-9][0-9][0-9][0-9][0-9]?[0-9]? / {
          match($0,/:([0-9]+)\)/,a); match($0,/ ses=([0-9]+)/,s);
          match($0,/audit\(([0-9]+)/,e); want[a[1]]=e[1]"\t"s[1] }
        END { for (q in want) print want[q]"\t"T[q] }

    d. Decode the `cmd=` field from hex for records with `type=USER_CMD` or `auid=[1-9][0-9][0-9][0-9][0-9]?[0-9]? ` and `uid=0`.

    e. Decode `SADDR=`, `SOCKADDR=`, and/or `laddr=`. Use `whois` to determine whether the remote IP address is owned by AWS or not.

    f. Output newline-delimited JSON with the original record or log line, the auid, the timestamp, and every decoded field, to a file.

3. Concatenate all the sub-agents' outputs into one file.

4. Sort records by their auid and timestamp, then group them into sessions when there's a gap ≥ 10 minutes between records or the `ses=` identifier changes.

5. Summarize each session in 3-5 sentences.

6. Run `hostname` to populate the hostname in the output.

## Output format

Consider individual records important if they have elevated privileges, modify files, or connect to non-AWS IP addresses.

```
# Panboticon audit of {hostname}, {earliest overall timestamp in RFC 3339} to {latest overall timestamp in RFC 3339}

## Summary

{3-5-sentence summary of all sessions}

## Sessions for {username} ({auid}) {include a section like this for every non-system user with activity}

### {one-line session summary}, {earliest session timestamp in RFC 3339} to {latest session timestamp in RFC 3339} {include a section like this for every session grouping}

{1-3-sentence summary of the session}

* {timestamp in RFC 3339} - {important record description}
* {timestamp in RFC 3339} - {important record description}
* {...}
```
