# BreachLab: Ghost - Level 21 → 22

## **Objective**

Perform Git history forensics to uncover a hardcoded secret committed to an archived project tag, bypassing the sanitized main branch timeline.

## **Why this matters in the real world**

Git history forensics is the number-one way real-world secrets leak. Every supply-chain attack in 2024-2026 either started here or passed through here. This is the only Ghost level that maps directly onto the Nexus (CI/CD) track that comes later.

## **Investigation & Solution**

1. **Repository Discovery & Baselining:** Navigated into the active `repo/` directory and reviewed the `README.md`. The documentation explicitly noted that internal snapshots were archived as tags, providing the initial pivot point for the investigation.

```bash
cd repo/
cat README.md
```


2. **Main Branch History Review:** Executed `git log` to review the linear commit history on the `main` branch. I inspected the individual commits using `git show [hash]`. The commits were sanitized, utilizing an environment variable (`${GHOST_SECRET}`) rather than a hardcoded string.

```bash
git log
git show 0560998e17e1bd2dddef346e4cafc7ffa21abca7
```


3. **Tag Forensics & Extraction:** Following the documentation hint, I enumerated the repository tags and discovered `v0.9-internal`. Inspecting this specific tag revealed an orphaned commit not present in the standard `git log` timeline. In this snapshot, the developer had temporarily hardcoded the production secret for debugging purposes, exposing the target flag.

```bash
git tag -l
git show v0.9-internal
exit
```

> 📹 **Recording:** [![asciicast](https://asciinema.org/a/1200385.svg)](https://asciinema.org/a/1200385)

## **Command Breakdown**

| **Command** | **Flag/Option** | **Purpose** |
| --- | --- | --- |
| `git log` | None | Displays the commit history for the currently active branch. |
| `git show` | `[hash/tag]` | Shows the specific diff, metadata, and changes introduced by a specific commit or tag. |
| `git tag` | `-l` | Lists all available tags in the repository, often used for version releases or snapshots. |

## **Notes & Takeaways**

**The Illusion of a Clean `main` Branch:**
A repository is a multi-dimensional database, not a flat folder. Developers frequently sanitize the `main` branch before a release, but leave the original mistakes buried elsewhere in the `.git` folder.

When conducting a forensic investigation or source code review, checking `git log` and `git tag` is just the starting point. A comprehensive review must include:

1. **`git reflog`:** Git locally records every single action (commits, hard resets, checkouts) for about 90 days. Even if a developer uses `git reset --hard` to rewrite history and "erase" a commit containing a password, `reflog` will expose the orphaned commit hash, allowing an investigator to pull the credential before the garbage collector permanently destroys it.
2. **Stale & Remote Branches (`git branch -a`):** Developers often create branches like `test-feature` or `debug-auth`, hardcode credentials to make local testing easier, merge the clean code into `main`, but forget to delete the test branch.
3. **Automated Secret Scanning:** In production environments with tens of thousands of commits, manual review is impossible. SOC and application security teams rely on automated parsers like **TruffleHog** or **Gitleaks**, which scan the entire `.git` history (every branch, tag, and commit) using regex and entropy checks to instantly flag high-value strings like AWS keys or API tokens.
