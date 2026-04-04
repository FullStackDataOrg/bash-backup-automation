# Teardown — Linux Backup Automation

A Bash + cron solution that detects encrypted password files modified within the last 24 hours and creates compressed, timestamped backups to a designated directory, logging every run.

---

## Stack Choices & Rationale

| Component | Decision Rationale |
|---|---|
| **Bash** | The native scripting language for Linux system administration. No runtime dependencies, available on every Linux distribution, and directly interfaces with the OS utilities that do the work. |
| **cron** | The standard Unix job scheduler. Reliable, zero-overhead, and universally available. No external orchestration tool needed for a single daily task. |
| **find** | The correct tool for time-based file discovery on Linux. The `-mtime` flag enables precise filtering of files modified within a given window without scripting custom modification time logic. |
| **tar + gzip** | Industry-standard compression and archiving combination. Produces `.tar.gz` archives that are portable, inspectable, and extractable with standard tools on any Unix system. |

---

## Key Design Decisions

- **Backups are partitioned by date** (`/backups/YYYY-MM-DD/`) rather than overwriting a single backup file. This enables point-in-time recovery and an audit trail.
- **Every run appends to `backup.log`**. Log entries include the target directory, files backed up, archive filename, destination path, and a completion timestamp.
- **The destination directory is auto-created** if missing, making the script safe to run on a fresh system without manual setup.
- **Absolute paths in crontab** — the path to `backup.sh` uses an absolute reference for reliability, as cron runs in a minimal environment without the user's PATH.

---

## Trade-offs

| Decision | Benefit | Cost |
|---|---|---|
| cron over Airflow/systemd timers | Zero dependency, universally available | No web UI, no retry logic, no alerting on failure — must be augmented with mail or log monitoring |
| tar + gzip over rsync | Single compressed archive, portable | rsync is more efficient for large-scale incremental backups; tar recreates archives from scratch each run |
| Bash over Python | No runtime required, fast, direct OS integration | Harder to unit test, less readable for complex logic, limited error handling patterns |

---

## Extensions & Real-World Use Cases

- Add **email alerting on failure** using the `mail` command or a webhook (`curl` to a Slack URL) when the archive creation exits non-zero.
- Extend to **remote backup** by replacing the local `cp` with `scp` or `rclone` to push archives to S3, GCS, or a remote SSH target.
- Replace cron with a **systemd timer** for better logging integration (journald), dependency management, and failure recovery on modern Linux systems.
- **Parameterise** the target directory and backup retention window via environment variables or a config file to make the script reusable across different backup jobs.
- Add a **retention policy**: delete backup archives older than N days to prevent unbounded disk growth.

---

## Portfolio Signal

This project demonstrates applied Linux system administration — a foundational skill that separates data engineers who can operate their own infrastructure from those who depend entirely on managed services.
