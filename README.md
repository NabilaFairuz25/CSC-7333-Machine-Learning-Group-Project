# CSC-7333-Machine-Learning-Group-Project

[Dataset](https://huggingface.co/datasets/hao-li/AIDev)

### Team Responsibilities

| Member | Technical Responsibility | Report Responsibility | Video Responsibility |
|---|---|---|---|
| **Member 1** | Dataset collection, preprocessing, and label generation | Dataset and Preprocessing | Introduction and Dataset |
| **Member 2** | Combinatorial complex construction (0-, 1-, and 2-cells) | Higher-Order Representation | Topological Representation |
| **Member 3** | Logistic Regression, LightGBM, and optional GraphSAGE implementation | Baseline Models | Baselines and Experiments |
| **Member 4** | CCNN implementation, model training, and ablation studies | Proposed Model | CCNN Architecture |
| **Member 5** | Model evaluation, integration, and result visualization | Results, Discussion, Conclusion, and Final Report Editing | Results, Conclusion, and Video Editing |




## 1. Existing Nodes (0-Cells)

The `graph/` folder contains seven node tables representing entities extracted from the AIDev dataset.

| Existing Node File | Node Type | Number of Nodes |
|---|---|---:|
| `nodes_pr.parquet` | Pull Request | 87,487 |
| `nodes_repo.parquet` | Repository | 9,819 |
| `nodes_commit.parquet` | Commit | 254,848 |
| `nodes_file.parquet` | File | 808,239 |
| `nodes_review.parquet` | First Human Review | 87,487 |
| `nodes_comment.parquet` | Inline Review Comment | 30,550 |
| `nodes_person.parquet` | Person | 38,252 |
