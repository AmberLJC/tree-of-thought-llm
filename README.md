# Official Repo of Tree of Thoughts (ToT)

<p>
    <a href="https://badge.fury.io/py/tree-of-thoughts-llm">
        <img src="https://badge.fury.io/py/tree-of-thoughts-llm.svg">
    </a>
    <a href="https://www.python.org/">
        <img alt="Build" src="https://img.shields.io/badge/Python-3.7+-1f425f.svg?color=purple">
    </a>
    <a href="https://copyright.princeton.edu/policy">
        <img alt="License" src="https://img.shields.io/badge/License-MIT-blue">
    </a>
    <a href="https://zenodo.org/badge/latestdoi/642099326">
        <img src="https://zenodo.org/badge/642099326.svg">
    </a>
</p>

![teaser](pics/teaser.png)

Official implementation for paper [Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601) with code, prompts, model outputs.
Also check [its tweet thread](https://twitter.com/ShunyuYao12/status/1659357547474681857) in 1min.





## Setup
1. Set up OpenAI API key and store in environment variable ``OPENAI_API_KEY`` (see [here](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety)). 

2. Install `tot` package from source:
```bash
git clone https://github.com/princeton-nlp/tree-of-thought-llm
cd tree-of-thought-llm
# install the conda environment
conda create --name tree-of-thought-env python=3.12 -y
conda activate tree-of-thought-env
pip install -r requirements.txt
pip install -e .  # install `tot` package
source env.sh # set up the environment variables
```


## Quick Start
The following minimal script will attempt to solve the game of 24 with `4 5 6 10` (might be a bit slow as it's using GPT-4):
```python
import argparse
from tot.methods.bfs import solve
from tot.tasks.game24 import Game24Task

args = argparse.Namespace(backend='gpt-4', temperature=0.7, task='game24', naive_run=False, prompt_sample=None, method_generate='propose', method_evaluate='value', method_select='greedy', n_generate_sample=1, n_evaluate_sample=3, n_select_sample=5)

task = Game24Task()
ys, infos = solve(args, task, 900)
print(ys[0])
```

And the output would be something like (note it's not deterministic, and sometimes the output can be wrong):
```
10 - 4 = 6 (left: 5 6 6)
5 * 6 = 30 (left: 6 30)
30 - 6 = 24 (left: 24)
Answer: (5 * (10 - 4)) - 6 = 24
```

## Paper Experiments

Run experiments via ``sh scripts/{game24, text, crosswords}/{standard_sampling, cot_sampling, bfs}.sh``, except in crosswords we use a DFS algorithm for ToT, which can be run via ``scripts/crosswords/search_crosswords-dfs.ipynb``.

The very simple ``run.py`` implements the ToT + BFS algorithm, as well as the naive IO/CoT sampling. Some key arguments:

- ``--naive_run``: if True, run naive IO/CoT sampling instead of ToT + BFS.
- ``--prompt_sample`` (choices=[``standard``, ``cot``]): sampling prompt
- ``--method_generate`` (choices=[``sample``, ``propose``]): thought generator, whether to sample independent thoughts (used in Creative Writing) or propose sequential thoughts (used in Game of 24)
- ``--method_evaluate`` (choices=[``value``, ``vote``]): state evaluator, whether to use the value states independently (used in Game of 24) or vote on states together (used in Creative Writing)
- ``--n_generate_sample``: number of times to prompt for thought generation
- ``--n_evaluate_sample``: number of times to prompt for state evaluation
- ``--n_select_sample``: number of states to keep from each step (i.e. ``b`` in the paper's ToT + BFS algorithm)
