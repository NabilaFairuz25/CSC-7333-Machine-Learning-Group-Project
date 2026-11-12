# AIDev dataset build: quality report

## raw_rows

| item | value |
|---|---|
| pull_request | 361296 |
| pr_reviews | 541831 |
| pr_review_comments | 291248 |
| pr_commits | 1718829 |
| pr_commit_details | 12116310 |
| related_issue | 100050 |
| pr_timeline | 6515552 |

## funnel

| item | value |
|---|---|
| 1_prs_unique | 361296 |
| 2_prs_open_or_unknown_outcome_dropped | 28890 |
| 3_prs_with_final_outcome | 332406 |
| 4_prs_with_any_human_review | 89574 |
| 5_dropped_first_review_not_before_outcome | 475 |
| 6_final_cohort_prs | 87487 |

## reviews

| item | value |
|---|---|
| reviews_on_eligible_prs | 486897 |
| not_submitted_or_pending | 0 |
| bot_reviews | 225671 |
| self_reviews_by_pr_author | 60850 |
| first_review_state_DISMISSED | 1612 |

## comments

| item | value |
|---|---|
| comments_linked_to_first_review | 31075 |
| dropped_created_after_cutoff_or_missing_ts | 525 |
| dropped_bot_comments | 0 |

## leakage_flags

| item | value |
|---|---|
| first_review_comments_edited_after_cutoff | 3823 |
| prs_title_renamed_after_cutoff | 9102 |
| note_body | PR body is the final version; edits after cutoff are not tracked |

## join_checks

| item | value |
|---|---|
| pr_commit_pairs_in_pr_commits_only | 38928 |
| pr_commit_pairs_in_commit_details_only | 0 |
| pr_commit_pairs_in_both | 423415 |
| duplicate_commit_file_rows_removed | 126 |
| first_review_comments_whose_path_not_in_cutoff_files | 4006 |

## commits

| item | value |
|---|---|
| commits_with_committed_event | 461699 |
| committed_events_missing_timestamp | 461699 |
| prs_with_monotone_timeline_order | 87387 |
| prs_with_non_monotone_timeline_order | 41 |
| commits_kept_policy_strict | 260755 |

## commit_evidence_tiers

| item | value |
|---|---|
| timeline_order_before | 245567 |
| timeline_order_after | 139546 |
| ambiguous | 61178 |
| first_review_comment_ref | 15188 |
| no_evidence | 864 |

## timeline

| item | value |
|---|---|
| events_kept_before_cutoff | 1142799 |
| outcome_type_events_removed | 4712 |
| events_dropped_untimed_non_commit | 229352 |

## splits

| item | value |
|---|---|
| split_time_train | 61241 |
| split_time_val | 13123 |
| split_time_test | 13123 |
| split_repo_train | 61232 |
| split_repo_test | 13139 |
| split_repo_val | 13116 |
| repos_spanning_multiple_time_splits | 2782 |

## labels

| item | value |
|---|---|
| n_merged_1 | 78686 |
| n_closed_unmerged_0 | 8801 |
| merge_rate | 0.8994 |

## coverage

| item | value |
|---|---|
| prs_with_zero_commits_at_cutoff | 10334 |

## labels_by_agent

| item | value |
|---|---|
| Claude_Code | {"n": 36411, "merge_rate": 0.914} |
| Copilot | {"n": 39305, "merge_rate": 0.887} |
| Cursor | {"n": 2011, "merge_rate": 0.916} |
| Devin | {"n": 4447, "merge_rate": 0.899} |
| Google_Jules | {"n": 640, "merge_rate": 0.891} |
| OpenAI_Codex | {"n": 4673, "merge_rate": 0.884} |

## graph

| item | value |
|---|---|
| nodes | {"pr": 87487, "repo": 9819, "commit": 254848, "file": 808239, "review": 87487, "comment": 30550, "person": 38252} |
| relations | {"pr_repo": 87487, "pr_author": 87487, "pr_commit": 260755, "commit_author": 260748, "pr_commit_file": 1743598, "pr_review": 87487, "review_reviewer": 87487, "review_comment": 30550, "comment_file": 26544, "comment_commit": 26868, "comment_author": 30550, "pr_review_reviewer_file": 19498, "pr_actor_event": 1142614, "pr_issue_unverified": 26113} |

## settings

| item | value |
|---|---|
| selected_dir | data\selected |
| out_dir | data\processed |
| commit_policy | strict |
| allow_self_review | False |
| dismissed_policy | drop |
| issue_policy | none |
