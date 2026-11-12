# AIDev Merge-Outcome Dataset (cleaned, first-human-review cutoff)

CSC 7333 Group 8: *Higher-Order Topological Deep Learning for Predicting Merge Outcomes of AI-Generated GitHub Pull Requests*

## 1. What this dataset is

Each example is one **pull request (PR) opened by an AI coding agent** (Copilot, Codex, Claude Code, Cursor, Devin, …) on GitHub.
The task is to predict whether the PR will be **merged (label = 1)** or **closed without merging (label = 0)**.

The prediction is made at the **cutoff**: the moment the **first human review** is submitted.
Every table here contains **only information that existed at or before that moment**. Nothing from after the cutoff
(later commits, later reviews, merge/close events) is included, which prevents data leakage.

- **Source:** AIDev dataset, Hugging Face `hao-li/AIDev`
- **Size:** **87,487 PRs**, feature table with **59 columns**



## 2. Which PRs are included

A PR is included only if all of the following hold:

1. It has a final outcome: merged, or closed without merging. Open PRs are removed.
2. It has at least one **submitted human review**: state APPROVED, CHANGES_REQUESTED or COMMENTED; account type `User`; not a bot login; not a review by the PR's own author.
3. That first human review was submitted after the PR was created and **before** it was merged or closed.
4. The first review is not in DISMISSED state. That state is set later, so it would leak future information.

Counts removed at each step: see `quality_report.md` → *funnel*.

| | Count |
|---|---|
| Merged (label 1) | **[78686]** |
| Closed without merge (label 0) | **[8801]** |
| Merge rate | **[0.8994]** |

## 3. Files

All files are Parquet. Load them with `pandas.read_parquet(...)`.

### 3.1 Main table: `pr_features.parquet` (one row per PR)

| Column group | Columns | Meaning |
|---|---|---|
| Identifiers (not predictors) | `pr_id`, `repo_id` | Used for joining and splitting only |
| Agent | `agent` | Which AI agent opened the PR (categorical) |
| Text | `title_len_chars`, `title_words`, `body_len_chars`, `body_words`, `body_has_testing_section`, `body_n_test_mentions`, `body_has_checklist` | Statistics of the PR title and description |
| Timing | `hours_to_first_review`, `cutoff_hour_utc`, `cutoff_dayofweek` | Hours from PR creation to first review; when the review happened |
| First review | `first_review_state`, `first_review_body_len_chars`, `first_review_body_words`, `n_first_review_comments`, `n_first_review_commented_files`, `first_review_comments_total_chars`, `n_bot_reviews_pre_cutoff` | The first human reviewer's verdict and how much feedback they gave |
| Commits | `n_commits`, `n_commit_authors`, `commit_stats_total_sum`, `commit_stats_additions_sum`, `commit_stats_deletions_sum` | Commits present by the cutoff (each commit counted once) |
| Files | `n_files_changed`, `file_additions_sum`, `file_deletions_sum`, `file_changes_sum`, `n_file_extensions`, `frac_test_file_rows`, `n_file_status_*` | Files changed by those commits: lines added/deleted, file types, share of test files, added/modified/removed/renamed counts |
| Timeline | `n_tl_*`, `n_tl_distinct_actors` | Number of each event type before the cutoff (labeled, assigned, commented, review_requested, …) and number of distinct people involved |
| Target | `label` | 1 = merged, 0 = closed without merging |
| Cutoff | `cutoff_ts` | Timestamp of the first human review |
| Splits | `split_time`, `split_repo` | `train` / `val` / `test` (see §5) |

To print the exact 59 column names: `print(list(pd.read_parquet("pr_features.parquet").columns))`.

**Do not use as model inputs:** `pr_id`, `repo_id`, `cutoff_ts`, `label`, `split_time`, `split_repo`.

### 3.2 Snapshot tables (the raw events behind the features)

| File | One row = |
|---|---|
| `labels.parquet` | a PR's label and outcome time (`outcome_ts`). For auditing only, never a feature |
| `first_review.parquet` | the selected first human review (reviewer, state, `cutoff_ts`, text) |
| `first_review_comments.parquet` | an inline comment from that review (file path, text). `commit_id_current_UNSAFE` may point past the cutoff, so don't use it |
| `commits_at_cutoff.parquet` | a commit present by the cutoff, with an `evidence` column saying how that was verified |
| `file_changes_at_cutoff.parquet` | one file changed in one commit (`file_key = repo_id::filename`) |
| `timeline_at_cutoff.parquet` | a PR event before the cutoff |
| `related_issue.parquet` | a PR–issue link (link time unknown, so it's flagged unverified) |
| `splits.parquet` | `pr_id` → `split_time`, `split_repo` |

### 3.3 Relationship tables for the combinatorial complex: `graph/`

These keep **who/what is connected to what**, which the one-row-per-PR table cannot. They are the input for defining the
0-, 1- and 2-cells and building the incidence matrices.

**Entity tables (`nodes_*.parquet`)**: each entity gets a contiguous integer index (`*_idx`) for matrix construction.

| Entity | Meaning |
|---|---|
| `pr` | a pull request |
| `repo` | a repository |
| `commit` | a commit (by SHA) |
| `file` | a file within a repository |
| `review` | a first human review |
| `comment` | an inline review comment |
| `person` | one developer identity across all roles (PR author, reviewer, commenter, commit author, timeline actor), with role flags and `is_bot` |

**Relation tables (`rel_*.parquet`)**

| Relation | Rows | Meaning |
|---|---:|---|
| `pr_repo` | 87,487 | PR belongs to repository |
| `pr_author` | 87,487 | PR opened by person |
| `pr_review` | 87,487 | PR has its first review |
| `review_reviewer` | 87,487 | review written by person |
| `pr_commit` | 260,755 | PR contains commit |
| `commit_author` | 260,748 | commit authored by person |
| `pr_commit_file` | 1,743,598 | **(PR, commit, file)**: commit in this PR changed this file, with lines added/deleted |
| `review_comment` | 30,550 | review contains inline comment |
| `comment_author` | 30,550 | comment written by person |
| `comment_file` | 26,544 | comment is on this file |
| `comment_commit` | 26,868 | comment originally posted on this commit |
| `pr_review_reviewer_file` | 19,498 | **(PR, review, reviewer, file)**: reviewer commented on this file in the first review |
| `pr_actor_event` | 1,142,614 | person triggered a timeline event on the PR before the cutoff |
| `pr_issue_unverified` | 26,113 | PR linked to issue (timing unverified, not used by default) |

The two bold tables are candidate groupings for higher-order (2-) cells. The final cell design is decided in the CC modeling step.

## 4. Leakage rules applied

- Outcome fields (`state`, `merged_at`, `closed_at`) are used **only** to make the label and select PRs.
- Only the **first** human review is kept. Later reviews and their comments are dropped.
- Inline comments are kept only if created **at or before** the cutoff.
- Commits are kept only with evidence they existed by the cutoff (`evidence` column):
  - `timeline_ts`: a timeline commit event at or before the cutoff
  - `first_review_comment_ref`: the first review's comment was originally posted on that commit
  - `timeline_order_before`: the event has no timestamp, but it appears before a pre-cutoff event in the PR's ordered timeline
- Timeline events are kept only if strictly before the cutoff. Merge/close/reopen events are always removed.
- `python 03_validate.py` re-checks all of the above.

## 5. Train / validation / test splits

- `split_time` (default): PRs sorted by cutoff time, then 70% train / 15% validation / 15% test. The model is always tested on later PRs than it trained on.
- `split_repo`: repositories never shared between splits. This tests generalization to unseen projects.

Fit all preprocessing (scaling, imputation, encoding, feature selection) on the **train split only**.




