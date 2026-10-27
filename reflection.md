# Lab 5 – Reflection

**1. Which issues were the easiest to fix, and which were the hardest? Why?**

The easiest were the style and housekeeping issues from Flake8 and Pylint: unused imports, trailing whitespace, missing blank lines, long lines, snake_case names and docstrings. The tool gives the exact line and the fix is mechanical. The hardest was the trojan-source bug in `isOfficer` (Bandit B613 / Pylint E2502). The line *looked* correct in the editor because the bidirectional Unicode characters reorder how the text is displayed, so I had to inspect the raw bytes to see that the comparison was against a hidden string and returned `True` for every role. The late-binding lambda in `announce()` was also tricky: the code looked fine but printed NOTA three times, and understanding it needed knowledge of how closures capture variables.

**2. Did the static analysis tools report any false positives? If so, describe one example.**

No clear false positives. The closest was Bandit's B105 hard-coded password, rated Low severity / Medium confidence, because it matches on variable names. Here it was a real secret, but the same rule would flag a harmless constant called `PASSWORD_FIELD = "password"`. Also, several real bugs were **not** reported by any tool: double voting, unregistered postal voters, `age > 18` instead of `>=`, votes accepted at 18:30, and division by zero in `turnout()`/`votePercent()`. I found these by running the program and checking the output. Static analysis catches patterns, not wrong business logic.

**3. How would you integrate static analysis tools into your actual software development workflow?**

Run Flake8 and Pylint locally through a `pre-commit` hook so style problems never reach a commit, and set the IDE to show them while I type. In CI (GitHub Actions), run all three tools on every pull request and fail the build if Bandit reports Medium/High issues, Flake8 reports anything, or the Pylint score drops below a threshold (e.g. 9.5). Settings would live in a shared config (`.pylintrc`, `setup.cfg`/`.flake8`) so the whole team uses the same rules. Unit tests would run alongside, since the tools miss logic bugs.

**4. What tangible improvements did you observe in the code quality, readability, or potential robustness after applying the fixes?**

- **Security:** Bandit went from 7 issues (2 High) to 0. Officers are checked against an allow-list, voter IDs use SHA-256, XML is parsed with `defusedxml`, receipts use `secrets`, and the admin password is no longer in source.
- **Correctness:** The program now gives the right results: an 18-year-old can register, Ben can't vote twice, "bob" counts for Bob, the 18:30 vote is rejected, the tally is `{'Alice': 1, 'Bob': 2, 'NOTA': 0}`, and announcements show each candidate correctly.
- **Robustness:** No more division by zero, no silent `except: pass`, and one shared `record_vote()` validates both in-person and postal ballots.
- **Readability:** Pylint went from 5.33 to 10.00/10 and Flake8 from 30 violations to 0. Consistent snake_case names, docstrings and PEP 8 layout make the file much easier to read and maintain.
