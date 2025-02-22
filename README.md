# MANDO-HGT: Heterogeneous Graph Transformers for Smart Contract Vulnerability Detection
[![python](https://img.shields.io/badge/python-3.7.12-blue)](https://www.python.org/)
[![slither](https://img.shields.io/badge/slither-0.8.0-orange)](https://github.com/crytic/slither)
[![dgl](https://img.shields.io/badge/dgl-0.6.1-green)](https://www.dgl.ai/)

<p align="center">
<img src="./assets/MANDO_HGT.png" alt="MANDO HGT Logo" width="225" height="225" class="center">
</p>


# Overview
This repository is the implementation of MANDO-HGT which should be applicable to either source code or bytecode/binary form of software programs to cater to different situations where the source code of the software may or may not be available.

<p align="center">
<img src="./assets/MANDO-HGT-overview.png" alt="MANDO HGT overview" class="center">
</p>

<p align="center">
<img src="./assets/code_snippet_source_code_cg_cfg.png" alt="code snippet source code cg cfg" class="center">
</p>

<p align="center">
<img src="./assets/code_snippet_bytecode_cfg.png" alt="code snippet bytecode cfg" width="546" height="462" class="center">
</p>

## Citation
Nguyen, H. H., Nguyen, N.M., Xie, C., Ahmadi, Z., Kudenko, D., Doan, T. N., & Jiang, L. (2023, May). *MANDO-HGT: Heterogeneous Graph Transformers for Smart Contract Vulnerability Detection*. In Proceedings of 20th International Conference on Mining Software Repositories (MSR' 23), Melbourne, Australia, 2023. [Preprint](https://hoanghnguyen.com/assets/pdf/nguyen2023msr.pdf)

```
@inproceedings{nguyen2023msr,
  author = {Nguyen, Hoang H. and Nguyen, Nhat-Minh and Xie, Chunyao and Ahmadi, Zahra and Kudenko, Daniel and Doan, Thanh-Nam and Jiang, Lingxiao},
  title = {MANDO-HGT: Heterogeneous Graph Transformers for Smart Contract Vulnerability Detection},
  year = {2023},
  month = {5},
  booktitle = {Proceedings of the 20th International Conference on Mining Software Repositories},
  numpages = {13},
  keywords = {vulnerability detection, smart contracts, source code, bytecode, heterogeneous graph learning, graph transformer},
  location = {Melbourne, Australia},
  series = {MSR '23}
}
```

# Table of contents

- [MANDO-HGT](#mando-hgt)
- [Table of contents](#table-of-contents)
- [How to train the models?](#how-to-train-the-models)
  - [Dataset](#dataset)
    - [Source code](#source-code)
    - [Byte code](#byte-code)
  - [System Description](#system-description)
  - [Install Environment](#install-environment)
  - [Inspection scripts](#inspection-scripts)
    - [Graph Classification](#graph-classification)
    - [Node Classification](#node-classification)
  - [Trainer](#trainer)
    - [Graph Classification](#graph-classification-1)
      - [Usage](#usage)
      - [Examples](#examples)
    - [Node Classification](#node-classification-1)
      - [Usage](#usage-1)
      - [Examples](#examples-1)
  - [Testing](#testing)
  - [Visuallization](#visuallization)

# How to train the models?

## Dataset

### Source code
- We prepared dataset for [experiments](experiments/ge-sc-data/source_code).

### Byte code
- We used Solc compilation of [Crytic-compile](https://github.com/crytic/crytic-compile) to compile EVM creation/runtime bytecode from the source code of smart contracts. Then we used [EtherSolve](https://github.com/SeUniVr/EtherSolve) to generate CFGs for bytecode.

## System Description

We run all experiments on 
- Ubuntu 20.04
- CUDA 11.1
- NVIDA 3080

## Install Environment

Install python required packages.
```bash
pip install -r requirements.txt -f https://download.pytorch.org/whl/lts/1.8/torch_lts.html -f https://data.pyg.org/whl/torch-1.8.0+cu111.html -f https://data.dgl.ai/wheels/repo.html
```

## Inspection scripts

We provied inspection scripts for Graph Classification and Node Classification tasks as well as their required data.

### Graph Classification

Training Phase
```bash
python -m experiments.graph_classification --epochs 50 --repeat 20
```
To show the result table

```bash
python -m experiments.graph_classification --result
```

### Node Classification

Training Phase
```bash
python -m experiments.node_classification --epochs 50 --repeat 20
```
To show the result table

```bash
python -m experiments.node_classification --result
```

- We currently supported 7 types of bug: `access_control`, `arithmetic`, `denial_of_service`, `front_running`, `reentrancy`, `time_manipulation`, `unchecked_low_level_calls`.

- Run the inspection 


## Trainer

### Graph Classification

#### Usage

```bash
usage: MANDO Graph Classifier [-h] [-s SEED] [-ld LOG_DIR]
                              [--output_models OUTPUT_MODELS]
                              [--compressed_graph COMPRESSED_GRAPH]
                              [--dataset DATASET] [--testset TESTSET]
                              [--label LABEL] [--checkpoint CHECKPOINT]
                              [--feature_extractor FEATURE_EXTRACTOR]
                              [--node_feature NODE_FEATURE]
                              [--k_folds K_FOLDS] [--test] [--non_visualize]

optional arguments:
  -h, --help            show this help message and exit
  -s SEED, --seed SEED  Random seed

Storage:
  Directories for util results

  -ld LOG_DIR, --log-dir LOG_DIR
                        Directory for saving training logs and visualization
  --output_models OUTPUT_MODELS
                        Where you want to save your models

Dataset:
  Dataset paths

  --compressed_graph COMPRESSED_GRAPH
                        Compressed graphs of dataset which was extracted by
                        graph helper tools
  --dataset DATASET     Dicrectory of all souce code files which were used to
                        extract the compressed graph
  --testset TESTSET     Dicrectory of all souce code files which is a
                        partition of the dataset for testing
  --label LABEL         Label of sources in source code storage
  --checkpoint CHECKPOINT
                        Checkpoint of trained models

Node feature:
  Define the way to get node features

  --feature_extractor FEATURE_EXTRACTOR
                        If "node_feature" is "GAE" or "LINE" or "Node2vec", we
                        need a extracted features from those models
  --node_feature NODE_FEATURE
                        Kind of node features we want to use, here is one of
                        "nodetype", "metapath2vec", "han", "gae", "line",
                        "node2vec"

Optional configures:
  Advanced options

  --k_folds K_FOLDS     Config for cross validate strategy
  --test                Set true if you only want to run test phase
  --non_visualize       Wheather you want to visualize the metrics
```

#### Examples

- We prepared some scripts for the custom MANDO structures bellow:

- Graph Classication for Heterogeous Control Flow Graphs (HCFGs) which detect vulnerabilites at the contract level.
  - GAE as node features.
```bash
python graph_classifier.py -ld ./logs/graph_classification/cfg/gae/access_control --output_models ./models/graph_classification/cfg/gae/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/ --compressed_graph ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/cfg_compressed_graphs.gpickle --label ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/graph_labels.json --node_feature gae --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_gae_dim128_of_core_graph_of_access_control_cfg_clean_57_0.pkl --seed 1
```

- Graph Classication for Heterogeous Call Graphs (HCGs) which detect vulnerabilites at the contract level.
  - LINE as node features.
```bash
python graph_classifier.py -ld ./logs/graph_classification/cg/line/access_control --output_models ./models/graph_classification/cg/line/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/ --compressed_graph ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/cg_compressed_graphs.gpickle --label ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/graph_labels.json --node_feature line --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_line_dim128_of_core_graph_of_access_control_cg_clean_57_0.pkl --seed 1
```

- Graph Classication for combination of HCFGs and HCGs and which detect vulnerabilites at the contract level.
  - node2vec as node features.
```bash
python graph_classifier.py -ld ./logs/graph_classification/cfg_cg/node2vec/access_control --output_models ./models/graph_classification/cfg_cg/node2vec/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/ --compressed_graph ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/cfg_cg_compressed_graphs.gpickle --label ./experiments/ge-sc-data/source_code/access_control/clean_57_buggy_curated_0/graph_labels.json --node_feature node2vec --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_node2vec_dim128_of_core_graph_of_access_control_cfg_cg_clean_57_0.pkl --seed 1
```

### Node Classification
- We used node classification tasks to detect vulnerabilites at the line level and function level for Heterogeneous Control flow graph (HCFGs) and Call Graphs (HCGs) in corressponding.

#### Usage
```bash
usage: MANDO Node Classifier [-h] [-s SEED] [-ld LOG_DIR]
                             [--output_models OUTPUT_MODELS]
                             [--compressed_graph COMPRESSED_GRAPH]
                             [--dataset DATASET] [--testset TESTSET]
                             [--label LABEL]
                             [--feature_compressed_graph FEATURE_COMPRESSED_GRAPH]
                             [--cfg_feature_extractor CFG_FEATURE_EXTRACTOR]
                             [--feature_extractor FEATURE_EXTRACTOR]
                             [--node_feature NODE_FEATURE] [--k_folds K_FOLDS]
                             [--test] [--non_visualize]

optional arguments:
  -h, --help            show this help message and exit
  -s SEED, --seed SEED  Random seed

Storage:
  Directories \for util results

  -ld LOG_DIR, --log-dir LOG_DIR
                        Directory for saving training logs and visualization
  --output_models OUTPUT_MODELS
                        Where you want to save your models

Dataset:
  Dataset paths

  --compressed_graph COMPRESSED_GRAPH
                        Compressed graphs of dataset which was extracted by
                        graph helper tools
  --dataset DATASET     Dicrectory of all souce code files which were used to
                        extract the compressed graph
  --testset TESTSET     Dicrectory of all souce code files which is a
                        partition of the dataset for testing
  --label LABEL

Node feature:
  Define the way to get node features

  --feature_compressed_graph FEATURE_COMPRESSED_GRAPH
                        If "node_feature" is han, you mean use 2 HAN layers.
                        The first one is HAN of CFGs as feature node for the
                        second HAN of call graph, This is the compressed
                        graphs were trained for the first HAN
  --cfg_feature_extractor CFG_FEATURE_EXTRACTOR
                        If "node_feature" is han, feature_extractor is a
                        checkpoint of the first HAN layer
  --feature_extractor FEATURE_EXTRACTOR
                        If "node_feature" is "GAE" or "LINE" or "Node2vec", we
                        need a extracted features from those models
  --node_feature NODE_FEATURE
                        Kind of node features we want to use, here is one of
                        "nodetype", "metapath2vec", "han", "gae", "line",
                        "node2vec"

Optional configures:
  Advanced options

  --k_folds K_FOLDS     Config cross validate strategy
  --test                If true you only want to run test phase
  --non_visualize       Wheather you want to visualize the metrics
```

#### Examples
We prepared some scripts for the custom MANDO structures bellow:

- Node Classication for Heterogeous Control Flow Graphs (HCFGs) which detect vulnerabilites at the line level.
  - GAE as node features for detection access_control bugs.
  ```bash
  python node_classifier.py -ld ./logs/node_classification/cfg/gae/access_control --output_models ./models/node_classification/cfg/gae/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/buggy_curated/ --compressed_graph ./experiments/ge-sc-data/source_code/access_control/buggy_curated/cfg_compressed_graphs.gpickle --node_feature gae --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_gae_dim128_of_core_graph_of_access_control_cfg_buggy_curated.pkl --testset ./experiments/ge-sc-data/source_code/access_control/curated --seed 1
    ```

- Node Classification for Heterogeous Call Graphs (HCGs) which detect vulnerabilites at the function level.
- The command lines are the same as CFG except the dataset. 
  - LINE as node features for detection access_control bugs.
  ```bash
  python node_classifier.py -ld ./logs/node_classification/cg/line/access_control --output_models ./models/node_classification/cg/line/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/buggy_curated --compressed_graph ./experiments/ge-sc-data/source_code/access_control/buggy_curated/cg_compressed_graphs.gpickle --node_feature line --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_line_dim128_of_core_graph_of_access_control_cg_buggy_curated.pkl --testset ./experiments/ge-sc-data/source_code/access_control/curated --seed 1
  ```

- Node Classication for combination of HCFGs and HCGs and which detect vulnerabilites at the line level.
  - node2vec as node features.
  ```bash
  python node_classifier.py -ld ./logs/node_classification/cfg_cg/node2vec/access_control --output_models ./models/node_classification/cfg_cg/node2vec/access_control --dataset ./experiments/ge-sc-data/source_code/access_control/buggy_curated --compressed_graph ./experiments/ge-sc-data/source_code/access_control/buggy_curated/cfg_cg_compressed_graphs.gpickle --node_feature node2vec --feature_extractor ./experiments/ge-sc-data/source_code/gesc_matrices_node_embedding/matrix_node2vec_dim128_of_core_graph_of_access_control_cfg_cg_buggy_curated.pkl --testset ./experiments/ge-sc-data/source_code/access_control/curated --seed 1
  ```


- We also stack 2 HAN layers for function-level detection. The first HAN layer is based on HCFGs used as feature for the second HAN layer based on HCGs (It will be deprecated in a future version).
```bash
python node_classifier.py -ld ./logs/node_classification/call_graph/node2vec_han/access_control --output_models ./models/node_classification/call_graph/node2vec_han/access_control --dataset ./ge-sc-data/node_classification/cg/access_control/buggy_curated --compressed_graph ./ge-sc-data/node_classification/cg/access_control/buggy_curated/compressed_graphs.gpickle --testset ./ge-sc-data/node_classification/cg/curated/access_control --seed 1  --node_feature han --feature_compressed_graph ./data/smartbugs_wild/binary_class_cfg/access_control/buggy_curated/compressed_graphs.gpickle --cfg_feature_extractor ./data/smartbugs_wild/embeddings_buggy_currated_mixed/cfg_mixed/gesc_matrices_node_embedding/matrix_node2vec_dim128_of_core_graph_of_access_control_compressed_graphs.pkl --feature_extractor ./models/node_classification/cfg/node2vec/access_control/han_fold_0.pth
```

## Testing
- We automatically run testing after training phase for now.

## Visuallization
- You also use tensorboard and take a look the trend of metrics for both training phase and testing phase.

```bash
tensorboard --logdir LOG_DIR
```
