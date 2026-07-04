# PolicySim: Generative Agent Simulations for Public-Interest Policy Evaluation

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-research%20prototype-orange)

> A trustworthy generative social digital twin for policy experimentation: LLM-driven agents act as synthetic residents of a virtual community, reason over a policy written in plain language, interact through a social network, and surface the likely social, economic, and behavioral outcomes before a policy is deployed on real people.

---

## Overview

Public-interest organizations and local governments have to decide how to design interventions (housing assistance, transit subsidies, public-health campaigns) without a reliable way to anticipate how real, diverse residents will respond. Rule-based agent models, econometric models, and surveys capture averages or hand-coded behavior, but they say little about the reasoning, misunderstanding, distrust, and social influence that often decide whether a policy works.

PolicySim explores a different approach. Each synthetic resident is a software agent whose decisions come from a Large Language Model (LLM) reasoning over a structured profile, a memory of past events, a set of evolving beliefs, and a position in a social network. A policy is translated into simulation parameters; the agents then perceive their situation, recall what is relevant, decide, and influence one another, and the framework reports the community-level effects together with an estimate of how far each result can be trusted.

**This is a research prototype that accompanies an academic paper.** PolicySim is presented as a *proposed* framework. Its central claim, that LLM-driven agents reproduce population-level policy responses better than classical baselines, is stated as a falsifiable hypothesis with a matching experimental plan. Documented failure modes of LLM-based social simulation are foregrounded, not hidden, and the system is positioned as accountable decision support, never as automated policymaking.

**Paper:** *PolicySim: Generative Agent Simulations for Public-Interest Policy Evaluation.* Gowri Sankar Reddy Tatipathi, Jasmine Christopher, Uday Vallabhaneni. Luddy School of Informatics, Computing, and Engineering, Indiana University Bloomington. Working manuscript, 2026.

## Features

- Generative-agent simulation of a virtual community grounded in public demographic data.
- Six-layer cognitive and social architecture, from policy understanding to responsible-AI validation.
- Agents with episodic and long-term memory, reflective belief updates, goals, and hard constraints.
- Trust-weighted social network for information diffusion and social influence.
- Counterfactual comparison of policy alternatives (policy A vs. policy B).
- A seven-dimension evaluation framework covering accuracy, realism, fairness, and reliability.
- Human-in-the-loop design: every result carries a validity estimate, and a human reviews and signs off.

## Architecture

PolicySim is organized as six layers that carry a written policy through to an audited result:

1. **Policy intelligence** — parses a natural-language policy into simulation parameters.
2. **Synthetic society generation** — builds a population that matches public census distributions (no agent maps to a real person).
3. **Cognitive agent** — the per-agent reasoning core: identity, memory, beliefs, and decision-making.
4. **Social intelligence** — trust, information diffusion, and social influence across the network.
5. **Simulation intelligence** — runs the world forward and supports counterfactual policy testing.
6. **Responsible AI** — bias auditing, uncertainty estimation, explainability, and confidence calibration, running across all layers.

Technically this maps onto a decoupled frontend / backend / AI-infrastructure stack, with the LLM reasoning engine, retrieval-augmented generation (RAG) for grounding, a vector database for agent memory, and a knowledge graph for the social network.

<!-- Add an exported diagram and reference it here: ![Architecture](assets/architecture.png) -->

## Repository structure

```
.
├── src/                # framework code (agents, memory, social network, simulation engine)
├── scripts/            # entry-point scripts to run experiments
├── configs/            # experiment configurations (policies, population, seeds)
├── tests/              # unit / integration tests
├── data/
│   └── sample/         # small, licensed sample inputs (committed)
├── results/            # generated outputs (gitignored; regenerate via scripts)
├── paper/              # LaTeX source and PDF of the paper
├── assets/             # figures and diagrams
├── requirements.txt    # pinned dependencies
├── LICENSE
└── README.md
```

## Installation

```bash
git clone https://github.com/<your-username>/policysim.git
cd policysim

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Tested on Python 3.10+.

## Configuration

PolicySim uses an LLM backend, so it expects an API key (or a local model endpoint). Copy the example file and fill in your own values:

```bash
cp .env.example .env
```

| Variable | Purpose |
|----------|---------|
| `LLM_API_KEY` | key for the LLM provider used by the agents |
| `LLM_MODEL`   | model name for citizen agents |
| `LLM_MODEL_SYSTEM` | larger model for policy translation and reporting |

Never commit a real `.env`; only `.env.example` with placeholders is tracked.

## Usage

> The commands below assume the script and config names in the structure above. Adjust them to match your actual entry points.

Run a simulation from a policy configuration:

```bash
python scripts/run_experiment.py --config configs/transit_subsidy.yaml --seed 42
```

Compare two policy variants (counterfactual):

```bash
python scripts/run_experiment.py --config configs/counterfactual_A_vs_B.yaml
```

## Evaluation framework

Because the value of a social simulation depends on whether its behavior matches reality, PolicySim is evaluated along seven dimensions rather than by accuracy alone:

| Dimension | Example metrics | Question it answers |
|-----------|-----------------|---------------------|
| Predictive accuracy | MAE, RMSE, MAPE, directional accuracy | How close are predictions to reality? |
| Behavioral realism | ABFS, APCS | Do agents behave like real people? |
| Social simulation quality | SEAS, network similarity | Does a believable community emerge? |
| Policy decision | CPRA | Can it rank policy A against B correctly? |
| Responsible AI | demographic parity, disparate-impact ratio, BAS | Can the output be trusted to be fair? |
| Uncertainty and reliability | expected calibration error, seed variance | Does the system know its own limits? |
| Computational performance | runtime, tokens per agent, memory, scaling | Can a small organization run it? |

## Results and reproducibility

The paper proposes the framework and this evaluation methodology, and specifies the experiments (baseline comparison against classical ABM, memory and social-network ablations, base vs. adapted LLM, counterfactual ranking, scaling, and a responsible-AI ablation) as a validation plan.

Please keep the following honesty conventions when reporting numbers here:

- Distinguish clearly between results from the **reference simulation** and results **validated against real systems**.
- Mark any experiment not yet executed as **future work**, not as a result.
- Ensure every reported figure or table traces to a script in `scripts/`, so it can be regenerated from a clean clone.

## Known limitations

- **Simulation-reality gap:** an LLM models text about behavior; there is no guarantee this transfers to real behavior under a new policy. This is exactly what the evaluation is designed to test.
- **Computational cost:** large simulations need substantial compute, which can limit access for smaller organizations.
- **LLM weaknesses:** bias, hallucination, and sensitivity to prompt wording mean results must be reported with uncertainty and read at the population level rather than from any single agent.

## Future work

- Live-LLM validation against recorded policy outcomes.
- Real-time community digital twins updated from live public data.
- Multimodal and vision-language agents.
- Reinforcement-learning calibration against real behavioral traces.
- Federated community models that improve shared agents without pooling sensitive local data.

## Citation

```bibtex
@misc{tatipathi2026policysim,
  title  = {PolicySim: Generative Agent Simulations for Public-Interest Policy Evaluation},
  author = {Tatipathi, Gowri Sankar Reddy and Christopher, Jasmine and Vallabhaneni, Uday},
  year   = {2026},
  note   = {Working manuscript, Indiana University Bloomington}
}
```

## License

Released under the MIT License. See [LICENSE](LICENSE). (Apache-2.0 is a reasonable alternative if you want an explicit patent grant.)

## Contact

Gowri Sankar Reddy Tatipathi — gotati@iu.edu — Luddy School of Informatics, Computing, and Engineering, Indiana University Bloomington.

For questions or issues, please use the GitHub issue tracker.
