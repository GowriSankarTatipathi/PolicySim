# PolicySim: Generative Agent Simulations for Public Interest Policy Evaluation
## A Research Proposal

**Lead / Architect:** M.S. Computer Science Candidate, Indiana University Bloomington
**Affiliation:** The Serve AI Initiative, Luddy School of Informatics, Computing, and Engineering
**Target venues:** ACM CHI (Human Factors in Computing Systems); AAAI Special Track on AI for Social Impact; NeurIPS Datasets & Benchmarks / Social Impact Track; submitted in parallel as an NSF research statement (CISE / SBE crosscutting).
**Proposed horizon:** 3–5 years (phased; see §17, Research Plan and Timeline).

---

## Abstract

Decisions about public-interest policy—who receives a housing voucher, how a transit subsidy is structured, when a public-health message is released—are made under deep uncertainty about how heterogeneous, boundedly-rational people will actually respond. The dominant computational tools for *ex ante* policy evaluation, namely rule-based Agent-Based Models (ABM), reduced-form econometric models, and system-dynamics models, encode human decision-making either as fixed behavioral equations or as hand-authored conditional rules. They are powerful for capturing aggregate feedbacks but structurally incapable of representing the qualitative cognition—misunderstanding, distrust, scarcity-induced tunneling, social contagion, and norm formation—that determines whether an intervention succeeds or fails in the field. We propose **PolicySim**, a generative-agent simulation platform in which each synthetic resident of a community digital twin is driven by a Large Language Model (LLM) acting as a *cognitive engine* over a structured identity, an associative memory, a reflective belief store, and a social graph. PolicySim renders each agent's material and institutional situation as bounded natural language, elicits a schema-constrained decision, and reconciles all agents' actions through a deterministic world engine, yielding emergent macro-outcomes (income distributions, employment, eviction rates, modal share, message diffusion) that can be compared against ground truth.

The central scientific claim of this work is *falsifiable* and is treated as such: we hypothesize that LLM-driven agents, conditioned on rich situational context, reproduce population-level policy responses with calibration superior to classical baselines, **and** we design the program to detect the well-documented failure modes of LLM social simulation—stereotype amplification, variance collapse, and systematic cross-group misestimation—rather than to assume them away. Our contributions are: (1) a formal dynamical-systems specification of generative-agent policy simulation; (2) a reproducible, distributed simulation engine; (3) a multi-metric validation methodology centered on *historical alignment backtesting* against held-out community data; and (4) a human-subjects study of how policy practitioners interact with, calibrate trust toward, and make decisions with such a system. PolicySim is positioned not as an oracle but as an instrument: a risk-free computational laboratory whose value is contingent on the rigor with which its fidelity is measured and bounded.

---

## 1. Introduction and Research Vision

### 1.1 The decision problem

Municipal agencies, county health departments, transit authorities, and community non-profits design interventions under three simultaneous constraints: small budgets, irreversible deployment, and populations whose behavior they cannot observe in advance. A rental-assistance program that is *correct on paper*—adequately funded, fairly targeted, legally sound—can nonetheless fail because the application form is intimidating to the precise households it is meant to serve, because the disbursement schedule is misaligned with when rent is actually due, or because distrust seeded by a prior bad experience suppresses uptake in one neighborhood while word-of-mouth amplifies it in another. These are not failures of arithmetic. They are failures of *cognition, communication, and social structure*, and they are exactly the phenomena that current quantitative policy tools do not represent.

The vision of PolicySim is to give public-interest organizations an instrument analogous to the wind tunnel in aeronautics or the in-silico trial in pharmacology: a place to observe how a candidate policy interacts with a synthetic population's psychology and social fabric *before* the policy is deployed on real people, and to do so cheaply enough that many variants can be explored. The aspiration is not to predict the future deterministically—no honest social science promises that—but to surface plausible failure modes, disparate impacts, and behavioral frictions early, when they are still cheap to fix, and to do so with a quantified, audited estimate of how much that synthetic evidence can be trusted.

### 1.2 From rule-based agents to language-model-driven cognitive agents: the paradigm shift

The intellectual core of this proposal is a transition in *what sits inside the agent*.

In the classical ABM tradition that begins with Schelling's segregation model [Schelling 1971] and matures in Epstein and Axtell's *Sugarscape* [Epstein & Axtell 1996], the agent is a small program: a state vector and a set of human-authored transition rules of the form `if condition then action`. This design is a deliberate and historically productive abstraction. It made it possible to demonstrate that macro-level patterns (residential segregation, wealth inequality, the spread of conventions) can *emerge* from local interactions among simple agents—Epstein's "generative" criterion, "if you didn't grow it, you didn't explain it." But the abstraction purchases tractability at the cost of cognition. The agent does not *understand* its situation; it executes a lookup. Every behavioral contingency the modeler did not anticipate is, by construction, absent. Heterogeneity is parameterized (a distribution over thresholds), not *characterological*.

Generative agents, introduced by Park et al. [Park et al. 2023] and formalized as Generative Agent-Based Modeling (GABM) by Vezhnevets et al. in DeepMind's Concordia framework [Vezhnevets et al. 2023], invert this. The agent's decision function is not a hand-written rule table but a frozen LLM that ingests a natural-language description of the agent's identity and situation, retrieves relevant episodic memories, reflects to form higher-order beliefs, and emits an action in language. The LLM contributes a vast, pre-trained prior over *how a person like this, in a situation like this, would plausibly behave*—a prior that the modeler never has to enumerate. Concordia's framing is exact: a GABM is "not just a classic ABM where the agents talk to one another"; it is an ABM whose agents "apply common sense to situations, act reasonably, [and] recall common semantic knowledge."

The significance of this transition is threefold, and we will defend each claim empirically rather than rhetorically:

- **Open-world behavioral coverage.** Because the decision rule is the LLM's full distribution over plausible continuations, the agent can respond to novel situations the modeler never coded—a policy worded ambiguously, an unexpected interaction between two interventions, a rumor. This is the affordance that makes *qualitative* policy friction representable.
- **Characterological heterogeneity.** Park et al.'s 1,000-person study [Park et al. 2024] shows that agents grounded in rich qualitative interview transcripts replicate individuals' General Social Survey responses about as accurately as those individuals replicate themselves two weeks later, and—critically for §13—that interview-grounded conditioning *reduces* accuracy disparities across racial and ideological groups relative to thin demographic-label conditioning. Heterogeneity becomes a property of a textured persona, not a sampled scalar.
- **Endogenous belief and norm dynamics.** Reflection and inter-agent communication let beliefs and social norms form, spread, and decay during the simulation, rather than being fixed exogenously. This is the substrate for modeling trust degradation and sociolinguistic contagion.

We are equally clear about what the shift does *not* automatically buy. An LLM is a model of *text about behavior*, not of behavior; its priors reflect its training distribution, including that distribution's stereotypes and its over-representation of some populations. The same literature that motivates the paradigm documents its pathologies: Bisbee et al. [Bisbee et al. 2024] find silicon-sampled GPT-3.5 responses inflate affective-polarization estimates roughly sevenfold relative to human benchmarks, attributing the distortion to stereotype amplification; Pataranutaporn et al. [Pataranutaporn et al. 2025] report systematic misestimation of human well-being across 64,000 individuals in 64 countries. The paradigm shift is therefore *an opportunity conditioned on a measurement obligation*. PolicySim's research contribution is as much the second part as the first.

### 1.3 Why this is a CHI / AAAI-AISI / NeurIPS problem and not a software project

The engineering of PolicySim is substantial, but the engineering is in service of three research questions that are *not* answerable by building the system alone: (i) *Does it work?*—can generative agents produce policy-relevant outputs that are calibrated against reality, and under what conditions do they fail (the empirical-validation question, addressed to AAAI-AISI / NeurIPS)? (ii) *Is it better, and at what cost?*—how does it compare to the classical methods it would displace, on accuracy, fairness, and compute (the methodological-comparison question)? and (iii) *Does it help the humans who would use it, without misleading them?*—do policy practitioners make better decisions with PolicySim in the loop, and does the interface calibrate their trust rather than inducing automation bias (the human-centered question, addressed to CHI)? A system that scores well on (i) and (ii) but fails (iii) is not a contribution to public-interest technology; it is a liability. We treat all three as first-class.

---

## 2. Research Problem and Gap Analysis

### 2.1 The state of computational policy evaluation

We survey the four dominant families of *ex ante* policy-evaluation methods, each on its mechanism, its strengths, its weaknesses, and—specifically—why it fails to capture the cognitive and social friction that determines policy outcomes in vulnerable populations.

**(a) Reduced-form / structural econometric models.**
*Mechanism.* Estimate behavioral relationships from historical observational or quasi-experimental data (regression, difference-in-differences, instrumental variables, structural demand systems) and extrapolate the estimated coefficients to a counterfactual policy. Microsimulation tax-benefit models (e.g., the TAXSIM/EUROMOD lineage) apply estimated or legislated rules to a representative microdata sample.
*Strengths.* Statistically principled; transparent assumptions; calibrated to real data; excellent for marginal changes within the support of historical experience; produce credible point estimates with uncertainty intervals.
*Weaknesses.* The Lucas critique: estimated behavioral parameters are not invariant to the policy regime, so extrapolation to structural change is unreliable. Representative-agent and aggregate formulations wash out the heterogeneity where disparate impacts live. Behavior is encoded as a smooth response function; thresholds, refusals, and discrete behavioral collapses are invisible.
*Why it fails for complex human behavior.* It models the *average* response to a price or rule, not the *process* by which a specific household, confronted with a confusing form and a late paycheck, decides not to apply at all. Take-up failure—often the dominant driver of real program outcomes—is precisely what reduced-form elasticities omit.

**(b) System-dynamics models.**
*Mechanism.* Represent the system as stocks and flows linked by feedback loops and differential/difference equations (e.g., compartmental SIR-type models in public health, or stock-flow models of housing supply).
*Strengths.* Excellent for endogenous feedback, accumulation, and delay; intuitive causal-loop diagrams aid stakeholder communication; computationally cheap; captures non-linear aggregate dynamics like tipping points.
*Weaknesses.* Operates at the level of aggregates and homogeneous compartments; within-compartment heterogeneity and network structure are abstracted away; behavioral parameters are exogenous and must be assumed.
*Why it fails for complex human behavior.* A compartment ("susceptible," "low-income renters") is by definition internally homogeneous; the model cannot represent that the same message reassures one person and alarms another, nor that adoption depends on *who* in one's social network has already adopted.

**(c) Classical Agent-Based Models (ABM).**
*Mechanism.* A population of autonomous agents, each a state vector with hand-authored decision rules (finite-state machines, decision trees, behavioral heuristics, sometimes utility maximization or reinforcement-learning policies), interacting on a spatial grid or network; macro-patterns emerge from micro-interactions.
*Strengths.* The canonical tool for emergence and heterogeneity; can represent networks, space, and adaptation; the "generative" explanatory standard [Epstein & Axtell 1996]; supports counterfactual experimentation impossible in the field.
*Weaknesses.* Behavioral realism is bounded by the modeler's imagination and labor: every rule is authored by hand, validated weakly, and brittle to unanticipated situations. Calibration of rule parameters is notoriously under-determined. There is no mechanism for an agent to *interpret* qualitative information (a policy letter, a rumor).
*Why it fails for complex human behavior.* Its agents do not have cognition; they have rules. `if cash < threshold then skip_rent` cannot represent an agent who *misreads* the assistance eligibility criteria, *distrusts* the agency because of a neighbor's bad experience, or *reprioritizes* under the cognitive load of scarcity. The qualitative substance of policy friction is structurally outside its expressive range.

**(d) Survey- and stated-preference methods.**
*Mechanism.* Ask people directly (surveys, conjoint/discrete-choice experiments, focus groups, vignette studies) how they would respond to a hypothetical policy.
*Strengths.* Directly elicits human input; can probe attitudes inaccessible to behavioral data; conjoint designs recover preference weights rigorously.
*Weaknesses.* Hypothetical bias and social-desirability bias; people are poor forecasters of their own behavior under structural change; expensive and slow; samples of hard-to-reach populations are small and non-representative; cannot explore many policy variants.
*Why it fails for complex human behavior.* The stated-preference / revealed-preference gap is large and systematic precisely for the consequential, emotionally loaded decisions policy cares about. People underestimate the friction they will themselves experience.

### 2.2 The gap: existing models simulate *actions*, not *cognition*

Across all four families runs a single structural limitation, which we state as the motivating gap of this work:

> **Existing computational policy tools simulate the *actions* agents take—or the *aggregates* those actions produce—but not the *cognition* that generates the actions.**

This is not a complaint about accuracy; well-calibrated reduced-form models are often more accurate than anything we propose for marginal changes. It is a complaint about *expressiveness with respect to the determinants of policy success*. Whether a policy works in a vulnerable community is overwhelmingly a function of variables that live below the level of the action and above the level of the aggregate: *how the policy is understood*, *whether the agency is trusted*, *what one's neighbors are doing*, *how scarcity reshapes attention and time preference*, *how a confusing process induces a household to give up*. None of these is an action; each is a cognitive or social state that *causes* actions. A model whose primitive is the action (ABM, choice models) or the aggregate (econometrics, system dynamics) can fit the consequences of cognition where history repeats, but cannot *represent* cognition, and therefore cannot generalize to structurally novel interventions where the cognitive response is exactly what is uncertain.

Generative agents are interesting *as a scientific proposition* precisely because they target this gap: they place a model of human-like interpretation and reasoning—however imperfect—*inside* the agent, between situation and action. The research question is whether doing so yields policy-relevant fidelity that the action-level and aggregate-level models cannot, at a cost and risk profile that is acceptable for public-interest use. That is the question PolicySim is built to answer, and—equally—to answer honestly when the answer is "not yet" or "only under these conditions."

### 2.3 The specific gap PolicySim addresses within the GABM literature

Generative-agent simulation is itself now a fast-moving field. Park et al. [2023] demonstrated emergent social behavior in a 25-agent sandbox. Concordia [Vezhnevets et al. 2023] provided a general GABM library organized around a "Game Master" that adjudicates agent actions. Large-scale systems have since appeared: AgentSociety [Piao et al. 2025] simulates 10k+ agents and ~5M interactions and has been used to study polarization and universal-basic-income effects; SocioVerse [2025] builds a "world model" backed by a pool of ten million real users. Against this backdrop, PolicySim's distinct position is *not* "first" or "largest." It is a **validation-first, decision-support-first** instantiation aimed at a *specific, accountable use case*—small-organization public-interest policy evaluation grounded in a real locality (Bloomington, Indiana) and a real microdata source (Census PUMS)—where the deliverable is not a demonstration of emergence but (i) a defensible, multi-metric estimate of how far the synthetic population can be trusted for a given policy class, and (ii) an interface and workflow validated with the practitioners who would actually use it. The literature has largely established *that* generative agents produce believable behavior; the gap PolicySim targets is *whether, and how much, that believability can be converted into accountable decision support*.

---

## 3. Theoretical Foundations

PolicySim sits at the confluence of five research traditions. We articulate the theoretical commitments it inherits from each, because the credibility of the system rests on whether those commitments are coherent and whether they are empirically testable.

### 3.1 Computational social science and generative social science

The methodological ancestor is *generative social science* [Epstein 2006]: the position that a macro-social phenomenon is *explained* when it can be grown from the bottom up in a population of interacting agents. PolicySim accepts this epistemology and extends its generative criterion in one respect. Classical generative social science grows macro-patterns from agents whose micro-rules are *posited* by the modeler; the explanation is conditional on the plausibility of those posited rules. PolicySim replaces posited rules with an LLM prior over human-like behavior, which relocates—but does not eliminate—the burden of justification: instead of defending hand-authored rules, we must defend the *fidelity of the LLM prior*, which becomes an empirical object to be measured (the validation program of §10–§12). This is the sense in which generative-agent modeling is a genuinely new methodology rather than ABM with a chatbot: the locus of behavioral assumptions moves from explicit code to an implicit, learned, and *auditable* prior.

### 3.2 Cognitive science: bounded rationality and architectures of mind

The agent design is explicitly a cognitive architecture in the lineage of Newell's unified theories and Simon's *bounded rationality* [Simon 1955]. Real agents do not optimize over the full decision space; they satisfice over a small set of options retrieved by memory and shaped by attention. PolicySim's pipeline—observation → retrieval → reflection → planning → action—is a deliberate operationalization of bounded, memory-mediated, satisficing cognition rather than of expected-utility maximization. Memory and reflection map onto the episodic/semantic distinction and onto the consolidation of experience into generalized belief; the LLM supplies the "common-sense" inference layer that classical symbolic architectures had to encode by hand. We are agnostic about whether the LLM's internal computation resembles human cognition (it almost certainly does not); the theoretical claim is weaker and sufficient—that the *functional organization* of the architecture (bounded retrieval, satisficing choice, belief consolidation) and the LLM's behavioral prior together produce *input-output behavior* that is human-like at the level we measure.

### 3.3 Behavioral economics: scarcity, framing, and the psychology of friction

The phenomena PolicySim exists to capture are the staples of behavioral economics: prospect-theoretic framing and loss aversion [Kahneman & Tversky 1979], present bias and hyperbolic discounting, and—central to the public-interest setting—the *scarcity* literature [Mullainathan & Shafir 2013], which shows that conditions of acute resource scarcity impose a cognitive "bandwidth tax" that narrows attention, shortens time horizons, and degrades decision quality. The design hypothesis behind the *Constrained Environmental Scarcity* prompting strategy (§3.6, §4.2) is that presenting an LLM with a concrete, bounded, high-stakes material situation elicits behavior consistent with scarcity-mode cognition. We stress that this is a *hypothesis*, not an established fact, and it is registered as H1 in §10. The behavioral-economics literature gives us not only the phenomena to look for but the *measurements* (e.g., does the simulated population exhibit the documented uptake-suppression, deadline-clustering, and present-biased substitution patterns?) against which the simulation's behavioral validity can be checked.

### 3.4 Generative AI: LLMs as stochastic simulators of personae

The enabling theory is the view of an instruction-tuned LLM as a *superposition of simulacra*—a model that, conditioned on a persona and situation, samples behavior from the slice of its training distribution consistent with that conditioning [Shanahan, McDonell & Reynolds 2023; Andreas 2022]. Argyle et al. [2023] formalize the favorable case as *algorithmic fidelity*: the claim that an LLM's conditional output, given a demographic/biographical backstory, accurately emulates the response distribution of the corresponding human sub-population, and that the model's "bias" is therefore fine-grained and conditioning-dependent rather than a uniform property. PolicySim is, theoretically, a bet on algorithmic fidelity *extended from survey response to sequential behavior under intervention*. The same theory tells us where the bet is dangerous: conditioning on thin or stereotyped descriptors recovers caricature, not the sub-population; and the model interpolates poorly for populations under-represented in its training data. This duality is the reason the proposal foregrounds rich, interview-style persona grounding [following Park et al. 2024] over demographic-label conditioning, and the reason fidelity-vs-caricature is an explicit research question (RQ3).

### 3.5 Digital twins and social simulation

PolicySim is, in infrastructure terms, a *digital twin* of a community: a continuously-instantiable virtual model of a real system, grounded in real spatial and demographic data, intended to support what-if analysis. We borrow the digital-twin discipline of *fidelity budgeting*—being explicit about which aspects of the real system the twin represents faithfully (demographic composition, spatial layout, institutional rules, budget constraints) and which it abstracts (it is not a twin of any real individual; it is a twin of a *population's statistical and structural properties*). This distinction is also the privacy foundation (§13): the twin is built from anonymized public microdata and synthetic personae, never from real identified individuals, so it is a twin of a community's *structure*, not of its *members*.

### 3.6 The synthesis: how an LLM can carry beliefs, memory, decisions, social influence, and adaptation

The theoretical payoff of the synthesis is a concrete account of how each ingredient of social behavior is realized:

- **Beliefs** are maintained as natural-language summaries in the reflection store $b_i(t)$, updated by an LLM reflection operator that consolidates recent episodic memory into higher-order generalizations (e.g., "relying on public transit jeopardizes my shifts"). Beliefs are *causal* in the architecture: they re-enter the decision prompt and shape subsequent action.
- **Memory** is an associative episodic store retrieved by a relevance/recency/importance score (§4.3, §5), so that situations cue contextually appropriate recollection, as in human memory and as in Park et al.'s retrieval design.
- **Decision-making** is the LLM's schema-constrained mapping from (persona, observation, retrieved memory, beliefs) to action—satisficing, context-dependent, and stochastic.
- **Social influence** is realized through the social graph: agents observe and communicate with neighbors; messages enter neighbors' memories; adoption and attitude shifts propagate along weighted ties, instantiating threshold and complex-contagion models of diffusion [Granovetter 1978; Centola 2018] but with the *content* and *framing* of what spreads determined generatively rather than as a binary token.
- **Behavioral adaptation** is the closed loop: actions change the world and the agent's resources; consequences are written back to memory; reflection updates beliefs; future behavior changes accordingly. Adaptation is emergent from the loop, not parameterized.

### 3.7 Ecological validity: the central problem, stated honestly

Ecological validity—the degree to which simulated behavior corresponds to real behavior in the real environment—is the make-or-break question, and it cannot be assumed. Our position is that ecological validity is **not a property the system has or lacks globally; it is a quantity to be estimated per policy class, per population, and per outcome, with uncertainty.** The proposal's methodological spine (§10–§12) is built to produce that estimate: (i) *behavioral micro-validation* against documented behavioral regularities and held-out survey/choice data; (ii) *historical alignment backtesting* against recorded community outcomes; (iii) *fidelity-vs-caricature auditing* to detect variance collapse and stereotype amplification; and (iv) *robustness analysis* over prompts, models, and seeds. We adopt as a design principle that **a PolicySim result is not reportable to a decision-maker without an accompanying validity estimate for the relevant outcome**, and we build that coupling into both the engine (validity metadata travels with every output) and the interface (§13.4). This is the theoretical commitment that distinguishes PolicySim from a demonstration: it treats its own believability as the primary object of study.

---

## 4. Generative Agent Architecture: A Deep Dive

Each synthetic resident is a five-layer cognitive agent. The layers are separated for both conceptual clarity and engineering modularity (they map onto distinct storage backends and distinct LLM-call types). The design follows the memory–reflection–planning decomposition of Park et al. [2023] and the component-conditioned-generation pattern of Concordia [Vezhnevets et al. 2023], specialized to the constraints of public-interest policy simulation.

### 4.0 Architecture overview (text diagram)

```
                         ┌───────────────────────────────────────────────┐
                         │              CITIZEN AGENT  a_i                 │
                         │                                                 │
   E(t), G(t) ─────────► │  ┌──────────────┐   persona π_i (frozen)       │
   (world + social      │  │ IDENTITY     │   demographics, dispositions │
    context)            │  │  LAYER       │   socioeconomic constraints  │
                         │  └──────┬───────┘                              │
                         │         │ conditions every call               │
                         │         ▼                                       │
                         │  ┌──────────────┐      ┌────────────────────┐  │
   observation o_i(t) ─► │  │ COGNITIVE    │◄────►│ MEMORY LAYER        │  │
                         │  │  LAYER       │ query│  short-term buffer  │  │
                         │  │ (LLM core):  │ ───► │  episodic store(Vec)│  │
                         │  │  reason,     │ ◄─── │  reflective beliefs │  │
                         │  │  plan, decide│ top-k│  (long-term/semantic)│ │
                         │  └──────┬───────┘      └────────────────────┘  │
                         │         │ action intent                         │
                         │         ▼                                       │
                         │  ┌──────────────┐      ┌────────────────────┐  │
                         │  │ BEHAVIORAL   │      │ SOCIAL LAYER        │  │
                         │  │  LAYER       │◄────►│  ties, trust (Graph)│  │
                         │  │ schema-bind, │      │  influence, msgs    │  │
                         │  │  validate    │      │  information spread │  │
                         │  └──────┬───────┘      └────────────────────┘  │
                         └─────────┼───────────────────────────────────────┘
                                   │ validated action a_i(t) ∈ A (JSON)
                                   ▼
                         ┌───────────────────────────────────────────────┐
                         │  WORLD ENGINE T(·)  : reconcile all agents'     │
                         │  actions → s_i(t+1), E(t+1), G(t+1)            │
                         └───────────────────────────────────────────────┘
```

### 4.1 Identity Layer

*Purpose.* To anchor the agent in a stable, internally consistent persona that persists across the entire simulation and conditions every cognitive operation. Without a persistent identity, LLM agents drift, contradict themselves, and lose the demographic correlations that make a population representative.

*Contents.*
- **Demographics and structural attributes** sampled from Census PUMS (Public Use Microdata Sample) records for the target Public Use Microdata Area, so that the synthetic population's joint distribution over age, household type, income band, employment, tenure, vehicle access, and education matches the real community in aggregate.
- **Biographical backstory.** Following the central finding of Park et al. [2024]—that interview-grounded conditioning both improves accuracy and *reduces cross-group accuracy disparities* relative to demographic labels—each persona is expanded from its PUMS skeleton into a short, coherent natural-language life narrative (occupation history, daily constraints, values, a few formative episodes). This is the single most important design decision for fidelity (§3.4, RQ3): we condition on *texture*, not on tags.
- **Socioeconomic constraints** as hard, quantitative bounds carried in the state vector (cash, debts, recurring obligations, the spatial location of home and work, transport mode, schedule rigidity). These are not narrative flavor; they are enforced by the world engine and define the choice set.

*Engineering.* The frozen persona $\pi_i$ is stored once in PostgreSQL and injected as a system-prompt prefix on every call; the mutable constraint vector $r_i(t)$ is a row updated each step. Personae are generated offline by a one-time expansion pass (a larger model) and frozen for reproducibility.

### 4.2 Cognitive Layer

*Purpose.* The reasoning core—the LLM acting as the agent's "common-sense inference engine." It interprets the observation, decides what to retrieve, integrates memory and beliefs, and produces an action *intent* (later bound to schema by the behavioral layer).

*Reasoning.* Implemented as a structured prompt that places the agent in its situation and elicits a brief chain-of-thought before commitment [Wei et al. 2022], deliberately bounded in length to keep cognition satisficing rather than exhaustive (and to control cost). The *Constrained Environmental Scarcity* technique applies here: the situation is rendered as a concrete, numerically specific, deadline-bearing material predicament (the canonical "$42 in cash, $850 rent due in 72 hours, 10% wage penalty if late to work in Zone F" framing), on the hypothesis (H1) that bounded, high-stakes context elicits the attention-narrowing, tradeoff-forcing behavior the scarcity literature documents [Mullainathan & Shafir 2013].

*Decision-making.* Satisficing over a retrieved, situation-relevant option set rather than utility maximization over the full action space—both because this is more cognitively realistic and because it bounds the combinatorics.

*Planning.* For temporally extended behavior, the agent emits a *daily agenda* (a partition of the day into time blocks with intended activities and locations) which is then decomposed into hourly actions, giving coherence across time steps (an agent who plans to seek assistance at 14:00 behaves consistently leading up to it). The agenda is itself revisable when the world violates expectations (the bus is late; the office is closed).

### 4.3 Memory Layer

*Purpose.* To give the agent continuity, learning, and context-appropriate recall—the substrate of adaptation.

- **Short-term / working memory:** the current observation and the last few steps, held in-context, providing immediate situational coherence.
- **Episodic memory:** a growing store of timestamped natural-language records of what the agent observed, did, and experienced, embedded as dense vectors and stored in the vector database. Retrieval is associative: the current situation is the query.
- **Long-term / semantic memory and reflective beliefs:** higher-order generalizations produced by the reflection operator (e.g., consolidated convictions about institutions, transit, neighbors). These are retrieved alongside episodic records and bias interpretation.
- **Reflection:** an asynchronous, periodic operation (triggered by accumulated salience) that reads recent episodic memory and writes new beliefs, implementing experience-to-belief consolidation. Reflection is the mechanism by which trust degrades or solidifies over a simulation.

*Retrieval scoring.* Records are ranked by a weighted combination of **relevance** (embedding cosine similarity to the query), **recency** (exponential time decay), and **importance** (a salience score assigned at write time). The exact functional form is given in §5.4. This three-factor scheme follows Park et al. [2023] and is, in our experience and theirs, essential—pure relevance retrieval produces agents with no sense of time, and pure recency produces agents with no sense of significance.

### 4.4 Social Layer

*Purpose.* To make the agent a node in a society rather than an isolate, so that influence, trust, and information spread are first-class.

- **Relationships** are edges in the social graph (household, workplace, neighborhood, weak ties), each carrying a tie type and a mutable **trust/tie-strength** weight.
- **Influence** operates by neighbors' actions and communications entering one's observation and memory; the LLM then weighs them by source trust when forming beliefs and decisions.
- **Information spread** is generative complex contagion: what propagates is not a binary "infected" bit but a *message with content and framing*; whether it changes a neighbor's belief depends on the neighbor's persona, prior beliefs, and trust in the source. This lets PolicySim represent that the *same* policy announcement is reassuring through one social channel and alarming through another—the sociolinguistic-contagion phenomenon classical diffusion models cannot express.

*Engineering.* The graph lives in Neo4j; proximity and neighbor queries (k-hop, shortest weighted path, community detection) are graph-native operations rather than expensive relational joins (§6.3).

### 4.5 Behavioral Layer

*Purpose.* To convert the cognitive layer's free-form action *intent* into a strictly typed, machine-executable *action*, and to be the contract between agent and world engine.

*How decisions become actions.* The cognitive layer's intent ("I'll skip the bus, drive, and pay the parking, then apply for assistance online tonight") is passed through a constrained-decoding / schema-validation layer (Pydantic models, optionally enforced at decode time via a structured-output library such as Instructor or Outlines) that maps it to a validated action object: a typed verb, target, location, and resource deltas, drawn from the world's defined action space $\mathcal{A}$. Free-text generation is *prohibited at the agent–world interface*; only schema-valid actions cross the boundary. This is a correctness and a performance decision: it makes actions parseable and reconcilable, eliminates a class of hallucinated/ill-formed outputs, and lets the engine reject impossible actions (an agent cannot spend money it does not have) by construction.

```
   INTENT (free text from cognitive layer)
        │
        ▼
   ┌──────────────────────────────────────┐
   │ Schema binder (Pydantic / Instructor) │  ← retries + repair on invalid
   │  verb ∈ {WORK, TRAVEL, PAY, APPLY,    │
   │          COMMUNICATE, REST, SEEK,...} │
   │  args: {target, location, amount,...} │
   │  feasibility precheck vs r_i(t)       │
   └──────────────┬───────────────────────┘
                  ▼
   VALIDATED ACTION a_i(t) ∈ A  →  world engine
```

*Failure handling.* On schema-invalid or infeasible output, the layer issues a bounded number of repair prompts; persistent failure falls back to a safe default action (e.g., `REST`/no-op) and logs the event for diagnostics—failure is recorded, never silently fabricated.

---

## 5. Mathematical Formulation

We give a dynamical-systems formalization of PolicySim. The purpose is precision—so that hypotheses, baselines, and metrics are defined over well-specified objects—not formalism for its own sake. The system is a coupled, partially-observed, non-stationary stochastic process whose macro-outcomes are functionals of the joint micro-trajectory.

### 5.1 State, environment, and population

Let the population be $N$ agents, $i \in \{1,\dots,N\}$. The state of agent $i$ at discrete time $t$ is

$$
s_i(t) = \big(\pi_i,\; r_i(t),\; M_i(t),\; b_i(t)\big),
$$

where $\pi_i$ is the persistent persona (Identity Layer; time-invariant), $r_i(t) \in \mathbb{R}^{d_r}$ is the material/resource and location vector, $M_i(t) = \{ m : m = (\text{text}, \tau_m, \iota_m, \mathbf{v}_m) \}$ is the episodic memory store (text, timestamp $\tau_m$, importance $\iota_m$, embedding $\mathbf{v}_m$), and $b_i(t)$ is the reflective belief store (a set of natural-language belief statements with embeddings).

The **environment** state is $E(t)$: exogenous global variables (calendar time, weather, prices, transit state), the institutional rule set, and the spatial structure $G_{\text{geo}}$ (zones, home/work locations, travel times). The **social graph** is $\mathcal{G}(t) = (V, \mathcal{E}(t), w_t)$ with $w_t : \mathcal{E}(t) \to [0,1]$ a tie-strength/trust weight. The **joint world state** is

$$
X(t) = \big(\, s_1(t),\dots,s_N(t),\; E(t),\; \mathcal{G}(t) \,\big).
$$

### 5.2 Policy as an intervention operator

A policy $P_{\theta_P}$ is an operator that modifies the environment's rule set and/or agents' constraint sets, parameterized by design variables $\theta_P$ (eligibility thresholds, subsidy amounts, disbursement schedule, message content/timing, geographic targeting):

$$
E'(t) = P_{\theta_P}\big(E(t)\big), \qquad r'_i(t) = P_{\theta_P}\big(r_i(t),\pi_i\big).
$$

A policy may be time-indexed (a schedule $\{P_{\theta_P,t}\}$). Crucially, $P$ is *not* a behavioral assumption; it changes the *world the agents face*, and the behavioral response is generated by the agents. This is the structural feature that lets PolicySim, in principle, evade the Lucas critique: the policy changes the environment, and behavior re-derives from cognition rather than from a fixed estimated response coefficient.

### 5.3 Observation (partial, localized)

Each agent receives a partial, local observation rendered as natural language by the observation operator $\Omega$:

$$
o_i(t) = \Omega\big(s_i(t), E(t), \mathcal{G}(t)\big),
$$

which exposes only what agent $i$ can perceive: its own situation, its location's local conditions, and the actions/messages of its graph neighbors $\mathcal{N}_i(t) = \{ j : (i,j)\in\mathcal{E}(t)\}$. Partial observability is essential to realism: agents act on local, incomplete information, and policy friction often *is* informational asymmetry.

### 5.4 Memory retrieval

Given the query embedding $\mathbf{q}_i(t) = \text{emb}(o_i(t))$, each memory $m \in M_i(t)$ receives a score

$$
\mathrm{score}(m \mid t) \;=\; \alpha\, \underbrace{\cos\!\big(\mathbf{q}_i(t), \mathbf{v}_m\big)}_{\text{relevance}} \;+\; \beta\, \underbrace{e^{-\lambda (t-\tau_m)}}_{\text{recency}} \;+\; \gamma\, \underbrace{\tilde{\iota}_m}_{\text{importance}},
$$

with $\tilde{\iota}_m$ the normalized importance and $\alpha,\beta,\gamma,\lambda \ge 0$ tunable. The retrieved context is the top-$k$:

$$
R_i(t) = \operatorname*{Top\text{-}k}_{m \in M_i(t)} \;\mathrm{score}(m \mid t).
$$

### 5.5 Reflection (belief update)

Periodically (when accumulated importance since the last reflection exceeds a threshold $\Theta_{\text{ref}}$), an LLM reflection operator $\Phi$ consolidates recent memory into updated beliefs:

$$
b_i(t^+) = \Phi\big(b_i(t),\, \{m \in M_i(t) : \tau_m > t_{\text{last-ref}}\}\big).
$$

Reflection is the formal locus of belief dynamics (trust degradation, generalization), and it is asynchronous with respect to the per-step decision loop (§9).

### 5.6 The decision kernel (the LLM-induced stochastic policy)

The agent's decision is a sample from a stochastic policy induced by the frozen LLM with weights $\Theta$ and decoding temperature $T$:

$$
a_i(t) \,\sim\, \pi_{\Theta,T}\big(\,\cdot \;\big|\; \pi_i,\, o_i(t),\, R_i(t),\, b_i(t)\,\big), \qquad a_i(t) \in \mathcal{A},
$$

where $\mathcal{A}$ is the schema-constrained action space (Behavioral Layer, §4.5). Two properties matter for the science. First, $\pi_{\Theta,T}$ is **stochastic**: the same agent in the same situation may act differently across seeds, which is both a realism feature (genuine behavioral variability) and a reproducibility liability (RQ4) requiring seed control and Monte-Carlo replication. Second, $\Theta$ is **frozen**: behavior is shaped by *conditioning*, not by training, so the entire behavioral prior is the LLM's pre-trained distribution restricted to the conditioning context—which is exactly why the fidelity of that prior (§3.4) is the object of validation.

### 5.7 Transition (joint world update with conflict resolution)

The world engine applies all agents' actions *jointly*, resolving resource and spatial conflicts and enforcing conservation constraints:

$$
X(t+1) \;=\; \mathcal{T}\big(X(t),\, \{a_i(t)\}_{i=1}^{N}\big),
$$

where $\mathcal{T}$ is deterministic given the actions and any exogenous stochastic environment draws $\xi(t)$ (weather, random events): $X(t+1) = \mathcal{T}(X(t), \{a_i(t)\}, \xi(t))$. Conflict resolution (two agents claiming a scarce resource; a ledger that must balance) is handled by explicit reconciliation rules so that the macro-state remains physically and financially consistent (§9.4). Memory write-back occurs here: each agent's experienced outcome is appended to $M_i(t+1)$ with an importance score.

### 5.8 Macro-observables and the object of validation

Policy-relevant outputs are *functionals of the micro-state*:

$$
Y(t) = F\big(s_1(t),\dots,s_N(t), E(t)\big),
$$

e.g., the income distribution, employment rate, Gini coefficient $G(t)$, eviction count, program take-up rate, transit modal share, or message-adoption fraction. Because the per-agent policy is stochastic and the environment has exogenous draws, a single run yields one trajectory $Y_{1:H}^{(s)}$ under seed $s$; the simulation's prediction is the *distribution over trajectories*, estimated by Monte-Carlo over seeds $\{Y_{1:H}^{(s)}\}_{s=1}^{S}$.

### 5.9 The system as a whole, and the meaning of "emergence"

Compactly, PolicySim defines a parameterized stochastic dynamical system

$$
X(t+1) \sim \mathbb{T}_{\Theta,\theta_P}\big(\cdot \mid X(t)\big), \qquad X(0) \sim \mu_0,
$$

where $\mu_0$ is the initial population (PUMS-calibrated personae + initial resources + seed social graph), $\theta_P$ encodes the policy, and $\Theta$ the (frozen) cognitive model. The schematic identity

$$
\textbf{Agent} \;+\; \textbf{Environment} \;+\; \textbf{Policy} \;\longrightarrow\; \textbf{Emergent Outcome}
$$

is made precise as: *the distribution of the macro-functional $Y$ over trajectories of $\mathbb{T}_{\Theta,\theta_P}$ from $\mu_0$.* **Emergence** is the (typical) fact that the policy-induced change in this macro-distribution, $\Delta_Y(\theta_P) = \mathbb{E}[Y \mid \theta_P] - \mathbb{E}[Y \mid \theta_\varnothing]$, is *not* a simple aggregation of any single agent's rule—it arises from the interaction of heterogeneous cognition, partial observation, social contagion, and resource coupling. The scientific test (§10–§12) is whether the simulated $\Delta_Y(\theta_P)$ matches the real-world $\Delta_{Y^*}(\theta_P)$ better than baselines do, and where it does not.

---

## 6. System Architecture

The platform is a three-tier, decoupled architecture. We state for each component the choice, the justification, and—as a senior review would demand—the principal alternative and the tradeoff, since defensible system design is about *why not the alternatives* as much as *why this*.

### 6.1 Frontend

*Choice.* Next.js (React, TypeScript) with Tailwind CSS and a component library (shadcn/ui); Tremor for analytic charts; Mapbox GL JS for the spatial view; a WebSocket channel for live telemetry.

*Justification.*
- **Visualization.** Policy stakeholders reason spatially and distributionally. Mapbox renders agents and outcomes on the real geo-fenced neighborhoods (PUMA boundaries, census tracts) of Bloomington, IN, making disparate spatial impact legible; Tremor renders the live macro-functionals of §5.8 (Gini, employment, take-up, modal share) as the clock advances.
- **Simulation control.** The UI is the experiment console: define a policy $\theta_P$, configure population and horizon, launch/pause/step, and—central to the CHI contribution (§13.4)—inspect *individual agent reasoning traces* to interrogate *why* a macro-outcome arose, not merely *that* it did.
- **Monitoring.** WebSockets stream structured execution logs so practitioners and developers can watch the simulation's internal state and the engine's health in real time.

*Alternative & tradeoff.* A scientific notebook (Streamlit/Panel/Jupyter dashboards) would be far faster to build and is the norm for research prototypes. We choose a production-grade SPA because the *human-subjects decision-support study* (§10, RQ5) requires a genuine, responsive interface to study trust calibration and automation bias credibly; a notebook would confound interface quality with the research question. The tradeoff is engineering cost, accepted deliberately and only for the components the study touches.

### 6.2 Backend

*Choice.* A Python FastAPI service as the control plane; Celery workers over a Redis broker for distributed agent execution; LangGraph for per-agent cognitive-loop orchestration and state preservation; vLLM as the local inference server (§7, §9).

*Justification.*
- **API architecture.** FastAPI's async I/O suits a workload dominated by fan-out to many concurrent, latency-bound LLM calls; Pydantic (already the schema layer of §4.5) is first-class, so the action contract is enforced end-to-end; OpenAPI generation aids the frontend contract.
- **Simulation orchestration.** The control plane is *thin*: it owns the global clock, the barrier between steps, and persistence; it does not run cognition. Cognition is fanned out to workers (§9.2). This separation is what permits horizontal scaling of the expensive part (inference) independently of the coordinator.
- **Distributed execution.** Celery + Redis gives a mature, observable task-queue substrate: agents become tasks, workers pull them, results return through the broker, and the coordinator applies the barrier. vLLM provides the actual throughput via continuous batching and PagedAttention KV-cache management (§9.3).
- **AI state management.** LangGraph models each agent's loop (observe → retrieve → reflect → plan → decide → act) as an explicit state graph with cyclic transitions, so identity, memory handles, and beliefs persist coherently across steps rather than being reconstructed ad hoc.

*Alternatives & tradeoffs.* (i) *Ray* (esp. Ray Serve/Core) is arguably a better-fit substrate for distributed ML actors than Celery and is a serious alternative we will benchmark in Phase 1; Celery is chosen initially for operational familiarity and simplicity, with Ray held as the scaling path if Celery's coordination overhead dominates at $10^4$ agents. (ii) A bespoke `asyncio` orchestrator avoids broker overhead but reinvents retries, observability, and back-pressure; rejected. (iii) For multi-agent orchestration, AutoGen / CrewAI / raw orchestration are alternatives to LangGraph; LangGraph is preferred for its explicit, inspectable cyclic state machine, which matters because the *reasoning trace* is a research artifact (§13.4), not just plumbing.

### 6.3 Data layer: a polyglot-persistence design

PolicySim deliberately uses three storage engines because the three access patterns are genuinely different, and forcing them into one store degrades either performance or fidelity.

- **PostgreSQL — structured world state.** Tabular, transactional records: persona skeletons, the resource/location vectors $r_i(t)$, employment indices, prices, the financial *ledger*, temporal constants. *Why:* the world update (§5.7) requires ACID transactions and conservation guarantees (a ledger must balance; money cannot be double-spent across concurrent agent actions). Relational integrity and transactional commit are exactly Postgres's strengths.
- **Vector database (ChromaDB) — episodic memory retrieval.** Dense embeddings of episodic records, queried by approximate nearest neighbor for the relevance term of §5.4. *Why:* associative recall is a similarity search over a large, growing corpus per agent; this is the canonical vector-DB workload and is intractable as relational scans.
- **Graph database (Neo4j) — social structure.** The social graph $\mathcal{G}(t)$: households, workplaces, neighborhood and weak ties, with mutable trust weights. *Why:* neighbor lookups, k-hop traversal, weighted shortest paths, and community detection are native, index-free-adjacency graph operations; in a relational store these become recursive self-joins whose cost explodes with depth, and the social layer (§4.4) queries them every step.

*Alternatives & tradeoffs.* (i) *Single Postgres with pgvector + recursive CTEs* would collapse three systems into one and is attractive for operational simplicity; it is our explicit fallback and Phase-1 baseline, and may suffice at small $N$. We adopt polyglot persistence only where profiling shows the specialized store is necessary—graph traversal depth and vector-corpus size are the deciding metrics. (ii) *Qdrant/Weaviate/FAISS* are interchangeable vector backends; ChromaDB is chosen for prototyping ergonomics, with the embedding interface abstracted so the backend is swappable. (iii) Consistency across three stores is a real cost (§9.4): we make **PostgreSQL the source of truth for any state with conservation semantics** and treat the vector and graph stores as derived/append-mostly, so cross-store inconsistency cannot corrupt the ledger.

```
        ┌──────────────────────────── FRONTEND (Next.js) ────────────────────────────┐
        │   Mapbox spatial view │ Tremor macro-metrics │ agent-trace inspector        │
        └───────────────▲───────────────────────────────────────────┬─────────────────┘
                        │ WebSocket telemetry                        │ REST / control
        ┌───────────────┴────────────────────────────────────────────▼─────────────────┐
        │                         BACKEND CONTROL PLANE (FastAPI)                        │
        │   global clock │ step barrier │ persistence │ validity-metadata propagation   │
        └───────┬───────────────────────────────────────────────────────────┬───────────┘
                │ enqueue agent tasks (Celery/Redis)                          │ commit deltas
        ┌───────▼─────────────────────────────────────┐            ┌──────────▼────────────┐
        │  WORKER POOL  (Celery)                       │            │   DATA LAYER           │
        │  per-agent LangGraph loop ─► vLLM inference  │◄──────────►│  PostgreSQL (state)    │
        │  observe→retrieve→reflect→plan→decide→act    │  read/write│  ChromaDB   (memory)   │
        └──────────────────────────────────────────────┘            │  Neo4j      (social)   │
                                                                     └────────────────────────┘
```

---

## 7. Generative Agent Execution Pipeline

We specify the per-agent cognitive cycle step by step. For each stage we give the **input**, the **processing**, the **output**, and the **engineering implementation**. The cycle realizes the dynamical system of §5 at the level of one agent in one time step.

### 7.1 Observation
- **Input:** the agent's state $s_i(t)$, the local environment slice of $E(t)$, neighbor activity from $\mathcal{G}(t)$.
- **Processing:** the operator $\Omega$ (§5.3) renders a *bounded natural-language situation report* exposing only locally perceivable information; the Constrained Environmental Scarcity framing supplies concrete, numerically specific, deadline-bearing detail.
- **Output:** the observation string $o_i(t)$.
- **Engineering:** a deterministic templating + retrieval function reading Postgres (resources, local conditions) and Neo4j (neighbor actions); no LLM call (kept cheap and reproducible).

### 7.2 Memory Retrieval
- **Input:** $o_i(t)$ embedded as $\mathbf{q}_i(t)$.
- **Processing:** score every episodic record by relevance+recency+importance (§5.4); take top-$k$.
- **Output:** retrieved memory set $R_i(t)$ as a text block.
- **Engineering:** ANN query to ChromaDB for the relevance term; recency/importance computed in-process from record metadata; $k$ tuned for the cost/coherence frontier. Embeddings precomputed at write time.

### 7.3 Reflection
- **Input:** beliefs $b_i(t)$ + recent unconsolidated memory.
- **Processing:** when accumulated importance exceeds $\Theta_{\text{ref}}$, an LLM consolidation call $\Phi$ writes updated higher-order beliefs.
- **Output:** updated belief store $b_i(t^+)$.
- **Engineering:** **asynchronous and amortized**—reflection does *not* run every step for every agent (that would dominate cost); it is event-triggered and may run off the critical path between barriers (§9.1). A separate Celery queue with lower priority.

### 7.4 Planning
- **Input:** persona, current beliefs, current situation, standing agenda.
- **Processing:** generate or revise a structured daily agenda (time-blocked intents); decompose to the current step's intended activity. Revise if the world violated the plan.
- **Output:** a planned action intent for step $t$.
- **Engineering:** the agenda is cached in agent state (Postgres) and only regenerated on a schedule or on plan-violation, again to amortize LLM cost across steps.

### 7.5 Decision
- **Input:** persona, $o_i(t)$, $R_i(t)$, $b_i(t)$, planned intent.
- **Processing:** the core decision call—the stochastic policy $\pi_{\Theta,T}$ (§5.6)—produces a brief bounded rationale and a committed action intent.
- **Output:** free-text action intent.
- **Engineering:** the single most frequent and cost-dominant LLM call; routed to vLLM with continuous batching; temperature $T$ controlled and logged for reproducibility.

### 7.6 Action
- **Input:** the action intent.
- **Processing:** schema binding + feasibility precheck (§4.5): map intent to a typed action $a_i(t) \in \mathcal{A}$; repair-on-invalid; safe default on persistent failure.
- **Output:** validated action object $a_i(t)$ (JSON).
- **Engineering:** Pydantic validation, optionally decode-time constrained via Instructor/Outlines; bounded retries; failures logged, never fabricated.

### 7.7 World Update
- **Input:** all agents' validated actions $\{a_i(t)\}$ for the step.
- **Processing:** the engine $\mathcal{T}$ (§5.7) resolves conflicts, enforces conservation (ledger balance, spatial capacity), applies exogenous draws $\xi(t)$, and commits the new joint state; each agent's experienced outcome is written to episodic memory with an importance score; affected trust weights update in the graph.
- **Output:** $X(t+1)$ and the step's macro-functionals $Y(t+1)$ for telemetry.
- **Engineering:** a transactional Postgres commit for all conservation-bearing deltas (source of truth), followed by append-mostly writes to ChromaDB (new memories) and Neo4j (trust updates); the step barrier (§9.1) guarantees all actions are collected before $\mathcal{T}$ runs.

```
  o_i(t)        R_i(t)         b_i(t^+)        plan         a_i^intent      a_i(t)         X(t+1)
 OBSERVE ─► RETRIEVE ─► [REFLECT*] ─► PLAN ─► DECIDE ─► ACT(bind) ─► WORLD UPDATE ─► (next step)
  (tmpl)     (ChromaDB)  (async LLM)  (LLM*)   (LLM)     (Pydantic)    (Postgres txn)
        * reflect/plan amortized: not every agent every step
```

---

## 8. Multi-Agent System Design

PolicySim is a *heterogeneous* multi-agent system: most agents are the population (Citizen Agents), but several specialized agents perform system-level functions—policy ingestion, economic accounting, ethical auditing, and reporting. This division mirrors a separation-of-concerns discipline and lets us assign the right computational substrate to each role (a large API model for one-shot policy parsing; a small quantized local model for the many citizens; deterministic code where no language is needed).

For each agent we give **purpose, architecture, input, output, communication protocol, and failure cases.**

### 8.1 Citizen Agent
- **Purpose:** simulate an individual resident's behavior, ledger, social interaction, and experience of policy friction—the population whose emergent behavior is the object of study.
- **Architecture:** the five-layer cognitive agent of §4, driven by a quantized local model (a 4-bit AWQ instruction-tuned model in the 7–8B class, e.g., Llama-3.x-8B-Instruct), served by vLLM.
- **Input:** observation $o_i(t)$, retrieved memory, beliefs, plan.
- **Output:** schema-valid action $a_i(t)$, new memories, trust deltas.
- **Communication protocol:** with the world engine via the typed action schema (the only agent–world contract); with peers indirectly, via actions/messages mediated by the engine and the graph (no direct agent-to-agent message bus, which keeps the system auditable and the interaction graph the single source of social truth).
- **Failure cases & mitigation:** (i) identity drift / self-contradiction → persona re-injection every call + belief store; (ii) schema-invalid output → repair/fallback (§4.5); (iii) sycophancy / persona collapse under repeated prompting → persona anchoring and periodic consistency checks; (iv) refusal/over-cautious responses to hardship scenarios → prompt framing that establishes a legitimate simulation context; (v) silent stereotype amplification → caught not here but by the Ethical Auditor (§8.4) and the fidelity audit (§11.4).

### 8.2 Policy Translator Agent
- **Purpose:** convert a raw, natural-language policy document (an ordinance, a program rule sheet) into the formal intervention operator $P_{\theta_P}$ (§5.2): eligibility predicates, amounts, schedules, geographic targeting, action-space modifications.
- **Architecture:** a single large, capable API model (e.g., GPT-4o-class) in a zero-/few-shot structured-extraction configuration; output strictly schema-validated.
- **Input:** policy text + the engine's parameter schema.
- **Output:** a validated $\theta_P$ specification (machine-readable policy object).
- **Communication protocol:** one-shot request/response to the control plane at experiment-configuration time (not in the per-step loop), so a higher-cost model is affordable.
- **Failure cases:** misinterpretation of ambiguous legal language; hallucinated parameters; omission of a clause. *Mitigation:* this is **the highest-leverage error source and is therefore gated by a mandatory human-in-the-loop confirmation**—the translated $\theta_P$ is surfaced for practitioner review and sign-off before any simulation runs (this also aligns with the instruction-source-boundary principle: a policy document is *data*, and its translation into engine behavior is confirmed by the human, never auto-executed).

### 8.3 Macroeconomic Auditor Agent
- **Purpose:** track financial flows, spatial wealth distribution, employment indices, and the macro-functionals $Y(t)$—the system's accounting and metric engine.
- **Architecture:** **deterministic numerical code (Python/PyTorch/Pandas), not an LLM.** Accounting must be exact and reproducible; an LLM here would inject noise into the very quantities we validate against ground truth. (We note this as a correction to any framing that would make this an "LLM agent": it is an agent in the software-component sense, computing statistics, not a language model.)
- **Input:** the committed world ledger and agent states each step.
- **Output:** the time series of macro-metrics (Gini, distributions, take-up, modal share) for telemetry and evaluation.
- **Communication protocol:** reads Postgres post-commit; publishes metrics to the telemetry channel and the evaluation store.
- **Failure cases:** metric mis-specification, accounting leaks (money created/destroyed). *Mitigation:* conservation assertions and invariants checked every step; a failing invariant halts the run rather than producing a silently corrupt result.

### 8.4 Ethical Auditor Agent
- **Purpose:** detect biased or inequitable outcomes—both *outcome* disparities across protected groups and *fidelity* failures (stereotype amplification, variance collapse) that would make the simulation itself discriminatory.
- **Architecture:** a hybrid—deterministic fairness statistics (demographic-parity gap, disparate-impact ratio, subgroup calibration gaps; §11.4) computed in code, plus a counterfactual/ablation harness that re-runs cohorts with protected attributes perturbed (holding persona texture fixed) to measure causal outcome gaps. A fine-tuned classifier (e.g., a DeBERTa-class model) may screen agent text for stereotyped or harmful content as an auxiliary signal.
- **Input:** agent states, outcomes, and (for the text screen) reasoning traces.
- **Output:** a fairness/welfare audit attached to every result, including a *fidelity flag* when between-group separation appears inflated or within-group variance appears collapsed relative to human reference.
- **Communication protocol:** runs post-hoc per scenario; its audit is **bound to the result object** so no outcome is reported without it.
- **Failure cases:** the auditor inherits the base model's blind spots; an LLM-based screen can itself be biased. *Mitigation:* fairness *metrics* are deterministic and model-independent; the LLM screen is advisory only; and the audit's own reliability is itself reported (we do not over-trust the auditor).

*A note on the "strip demographic tags during token computation" idea.* The preliminary specification proposed removing demographic tags during the action-generation cycle to mitigate bias. We revise this: removing demographics would destroy the heterogeneity that makes the population representative and is not cleanly implementable mid-generation. The principled approach—grounded in Park et al. [2024] and Argyle et al. [2023]—is the opposite: condition on *rich persona texture* (which empirically *reduces* cross-group accuracy bias relative to thin labels) and audit bias *causally* via counterfactual attribute perturbation, as above.

### 8.5 Decision Support Reporter
- **Purpose:** compile the run's metrics, audits, agent exemplars, and reasoning traces into a clear, plain-language brief for a non-technical policymaker—including, mandatorily, the validity estimate (§3.7) and the human-in-the-loop caveat (§13).
- **Architecture:** a capable API model (e.g., Claude 3.5/4-class) in a tightly grounded, retrieval-over-results configuration; it summarizes *only* computed metrics and logged traces, never invents findings.
- **Input:** the evaluation store, the ethical audit, the validity metadata.
- **Output:** a structured report with an executive summary, the projected outcomes *with uncertainty*, disparate-impact findings, the validity/confidence statement, and explicit limitations.
- **Communication protocol:** end-of-run; the report is the human-facing artifact studied in the RQ5 user study.
- **Failure cases:** over-confident or fluent-but-unsupported narrative (the dangerous mode for decision support). *Mitigation:* grounding constraints, mandatory uncertainty and validity reporting, and a UI that links every claim back to its underlying metric/trace so a practitioner can verify rather than trust (§13.4).

```
  policy text ─► [Policy Translator] ─► θ_P ──(human sign-off)──► ENGINE ─► world states
                                                                       │
        ┌──────────────────────────────────────────────────────────────┤
        ▼ (deterministic)                ▼ (hybrid)                      ▼ (grounded LLM)
  [Macro Auditor]                 [Ethical Auditor]               [Decision Support
   Y(t), Gini, dists  ───────────► parity/fidelity audit ───────► Reporter] → brief
                                    (bound to result)              (+ validity + caveats)
```

---

## 9. Simulation Engine Design

This section addresses how a large-scale run actually executes, and the systems challenges that arise. We treat these as genuine engineering research, not boilerplate: at $10^3$–$10^4$ agents, each requiring multiple LLM calls per step over a multi-day-to-multi-year horizon, naïve designs are infeasible, and the design choices materially affect both cost and the validity of results.

### 9.1 Time-stepping and the global barrier

PolicySim advances in discrete steps (a step = one simulated hour in the canonical configuration). To keep the world causally consistent, each step is a **barrier-synchronized round**:

```
  for each step t:
    1. FREEZE global variables; SNAPSHOT E(t)        (no agent sees a mid-step world)
    2. BATCH agent states; DISTRIBUTE to workers
    3. EXECUTE agent cognitive cycles in parallel (vLLM-batched inference)
    4. COLLECT validated action vectors {a_i(t)}     ◄── BARRIER (await all/timeout)
    5. RESOLVE spatial/financial conflicts (T)
    6. COMMIT delta changes (Postgres txn; then append ChromaDB/Neo4j)
    7. INCREMENT clock t ← t+1; emit telemetry; trigger async reflection
```

The barrier (step 4) is what makes the joint update $\mathcal{T}$ well-defined: every agent acts on the *same* frozen snapshot $X(t)$, and all actions are reconciled together. Reflection and agenda regeneration are pushed *off* the barrier's critical path (async, amortized) so they do not serialize the round.

### 9.2 Parallel execution and agent scheduling

Agents within a step are mutually independent given the frozen snapshot, so step 3 is embarrassingly parallel and is fanned out across the Celery worker pool. **Scheduling** is not purely round-robin: we prioritize the critical-path decision calls over off-path reflection, and we batch decision prompts aggressively to feed vLLM's continuous batcher. At scale, the scheduler also supports **activity-gating**—agents in a dormant state (e.g., asleep at 03:00 with no salient event) can be advanced by a cheap deterministic shortcut rather than a full LLM cycle, drastically cutting calls without affecting the macro-functionals (validated by ablation).

### 9.3 Distributed inference

Throughput, not orchestration, is the true bottleneck: the workload is millions of decode-bound LLM calls. We address it with:
- **vLLM** as the inference server: continuous batching keeps the GPU saturated across heterogeneous request lengths, and PagedAttention manages the KV cache to maximize concurrency.
- **4-bit AWQ quantization** of the citizen model: activation-aware weight quantization compresses an 8B model to a per-instance footprint of roughly 6 GB, materially increasing the number of concurrent contexts a given GPU can hold and thus aggregate throughput, at a small, *measured* quality cost (we register quantization-induced behavioral drift as a robustness check in RQ4—quantization must not silently change the population's behavior).
- **Tiered model routing:** small local model for the many citizen decisions; large API model only for the rare, high-value calls (policy translation, final report). This keeps the dominant cost term local and quantized.

### 9.4 State consistency and conflict resolution

Three stores and many concurrent writers create consistency hazards. Our discipline:
- **Single source of truth for conservation-bearing state:** PostgreSQL. All money, capacity, and ownership deltas are applied in one transactional commit per step *after* the barrier, so two agents cannot both spend the same dollar or occupy the same scarce slot; conflicts are resolved by explicit, documented rules (priority, proportional rationing, randomized tie-break with logged seed).
- **Derived stores are append-mostly:** ChromaDB (new memories) and Neo4j (trust updates) are written after the Postgres commit; if a derived write fails, it can be replayed from the committed state, so cross-store inconsistency cannot corrupt the ledger.
- **Invariants checked every step:** total funds, population conservation, and capacity constraints are asserted; a violated invariant halts the run.

### 9.5 The hard engineering challenges, and their solutions

- **Race conditions.** *Risk:* concurrent agents mutating shared state mid-step. *Solution:* the barrier + frozen snapshot means agents only ever *read* $X(t)$ during cognition; all *writes* are serialized into the single post-barrier transaction (§9.1, §9.4). Cognition is read-only against the shared world.
- **Memory explosion.** *Risk:* episodic stores grow without bound over long horizons ($O(N \times H)$ records), inflating storage and retrieval latency. *Solutions:* (i) reflection *compresses* episodic detail into compact beliefs; (ii) importance-weighted *forgetting*—low-importance, low-recency memories are pruned or summarized (cognitively plausible, computationally necessary); (iii) per-agent memory caps with summarization rollups.
- **Computational cost.** *Risk:* full LLM cycles for every agent every step are economically infeasible at scale. *Solutions:* amortized reflection/planning (§7.3–§7.4), activity-gating of dormant agents (§9.2), tiered/quantized models (§9.3), aggressive batching, and step granularity tuned to the policy (hourly where behavior is fine-grained, daily where it is not).
- **Latency.** *Risk:* a step is only as fast as its slowest worker; tail latency stalls the barrier. *Solutions:* continuous batching to smooth per-request variance; per-step *timeout with safe-default* so one stuck agent cannot hang the round (the event is logged and counted as a reliability metric); load-balanced workers and a path to Ray if Celery coordination overhead dominates at $10^4$.

A standing principle across all four: **every cost optimization is treated as a potential validity threat and is ablated.** Activity-gating, quantization, forgetting, and step granularity each get a controlled experiment confirming they do not distort the macro-functionals we report—because a faster simulation that quietly changes the answer is worse than a slow correct one.

---

## 10. Research Methodology

We frame PolicySim as an empirical research program with falsifiable hypotheses, controlled comparisons, and a pre-registered analysis plan. The methodology is deliberately structured so that a *negative* result is informative and reportable: if generative agents do not beat baselines, or beat them only on some outcomes, the program is designed to say so precisely.

### 10.1 Research questions

- **RQ1 (Behavioral validity).** Do LLM-driven agents, conditioned on rich situational context, reproduce realistic, population-calibrated behavioral responses to policy—at both the micro level (documented behavioral regularities) and the aggregate level?
- **RQ2 (Predictive validity vs. classical methods).** Do generative-agent simulations forecast aggregate policy outcomes more accurately, and with better-calibrated uncertainty, than classical ABM, reduced-form econometric, and survey-extrapolation baselines—and at what computational cost?
- **RQ3 (Fidelity vs. caricature).** Under what conditions do generative agents *faithfully represent* demographic heterogeneity versus *amplify stereotypes / collapse within-group variance*? How does persona-grounding richness, model choice, and conditioning strategy modulate this?
- **RQ4 (Robustness & reproducibility).** How sensitive are emergent macro-outcomes to prompt wording, model and quantization, retrieval/forgetting policy, and random seed? Are conclusions stable enough to support decisions?
- **RQ5 (Human-centered decision support).** Do policy practitioners make better-calibrated decisions with PolicySim in the loop than without, and does the interface calibrate their trust appropriately rather than inducing automation bias or unwarranted distrust?

RQ1–RQ4 are the AAAI-AISI/NeurIPS contributions; RQ5 is the CHI contribution; together they answer the three questions of §1.3.

### 10.2 Hypotheses

- **H1 (Scarcity grounding).** Agents under Constrained Environmental Scarcity prompting exhibit behavior consistent with documented scarcity-mode patterns—deadline clustering, present-biased substitution, and uptake suppression under procedural friction—at rates closer to human-reported regularities than agents given thin ("you are low-income") framing.
- **H2 (Aggregate calibration).** On held-out historical interventions, PolicySim's projected change in macro-outcomes $\Delta_Y(\theta_P)$ is closer (in Wasserstein-1 and sMAPE) to the realized $\Delta_{Y^*}(\theta_P)$ than the best classical baseline, on at least the take-up and distributional outcomes where cognition dominates.
- **H3 (Richness reduces caricature).** Interview-style persona grounding reduces between-group separation inflation and within-group variance collapse relative to demographic-label conditioning (operationalizing Park et al. [2024] in a sequential-behavior setting).
- **H4 (Bounded sensitivity).** While individual agent trajectories are seed-sensitive, *aggregate* macro-functionals concentrate across seeds (Monte-Carlo CI width shrinks at the expected $O(1/\sqrt{S})$ rate), so population-level conclusions are reproducible even though individual behavior is not.
- **H5 (Trust calibration).** Practitioners shown PolicySim outputs *with* explicit validity estimates and inspectable reasoning traces calibrate trust better (closer correspondence between reliance and actual reliability) than practitioners shown outputs without them; the latter exhibit measurable automation bias.

We will pre-register H1–H5, the metrics, and the analysis plan (OSF) before the confirmatory backtests, to separate exploratory tuning from confirmatory evaluation—standard practice for credible empirical claims and increasingly expected at AISI/NeurIPS.

### 10.3 Experimental setup

A **scenario** is a tuple (population $\mu_0$, environment $E$, policy $\theta_P$, horizon $H$, seed set $\{s\}$). For each confirmatory claim we run $S \ge 30$ seeds and report distributions, not point estimates. The population is instantiated from Census PUMS for the Bloomington/Monroe County PUMA, expanded to textured personae (§4.1) and seeded with a social graph calibrated to plausible degree/assortativity structure. Environment grounding (zones, travel times, prices, institutional rules) is drawn from public local data.

### 10.4 Datasets

- **Population & structure:** U.S. Census **PUMS** microdata and **American Community Survey** tables for demographic joint distributions; **LODES/LEHD** for home–work commute structure; local GTFS transit feeds for the spatial/temporal environment. All public and de-identified (§13.2).
- **Ground-truth outcomes for backtesting:** documented historical interventions in the locality (or closely matched localities) with recorded outcomes—e.g., a rental-assistance or transit-fare program—using administrative aggregates, ACS year-over-year deltas, and any published program-evaluation reports as the reference $Y^*$.
- **Behavioral-regularity references (for H1/RQ1):** published effect sizes and behavioral patterns from the program-evaluation and behavioral-economics literatures (take-up rates under friction, deadline effects), used as micro-validation targets.
- **Human-subjects data (RQ5):** collected under IRB approval from policy practitioners (study design §10.6).

### 10.5 Baselines

A claim of superiority is meaningful only against strong baselines, run on the *same* scenarios and outcomes:
1. **Classical ABM** with hand-authored behavioral rules calibrated to the same data (the direct methodological comparator for RQ2; e.g., a Mesa/NetLogo-style model).
2. **Reduced-form / microsimulation** baseline: apply estimated/legislated rules to the PUMS sample (a tax-benefit-microsimulation analog), capturing mechanical eligibility with no behavioral response.
3. **Survey/stated-preference extrapolation** where available (or a literature-elasticity extrapolation).
4. **LLM ablations** as internal baselines: thin-persona vs. rich-persona; with vs. without memory/reflection; with vs. without the social layer; small-quantized vs. larger model—isolating *which architectural ingredients* drive any gain (essential for a NeurIPS-grade methods contribution).
5. **Naïve constant/no-change** baseline (assume the policy changes nothing behaviorally) as a floor.

### 10.6 Evaluation procedure

For RQ1–RQ4: run all systems on each scenario; compute the metric suite of §11 against ground truth; compare via paired tests across scenarios/seeds with effect sizes and multiple-comparison correction. For RQ5: a between-subjects (or within-subjects with counterbalancing) study with policy practitioners and informatics graduate students as a screening pool, comparing decision quality and trust calibration across interface conditions (no tool / tool-without-validity-metadata / full PolicySim with validity + traces), measured by decision-task accuracy, appropriate-reliance metrics, confidence–accuracy calibration, NASA-TLX workload, and semi-structured interviews; analyzed quantitatively (mixed-effects models) and qualitatively (thematic analysis). This dual quantitative/qualitative design is the CHI methodological norm.

---

## 11. Evaluation Framework

Validation is the heart of the contribution, so the metric suite is deliberately plural: no single number certifies a social simulation. We organize metrics along four axes and specify each formally.

### 11.1 Behavioral accuracy (micro)
*Question:* do individual agents behave like real people in comparable situations?
- **Behavioral-regularity match:** does the simulated population reproduce documented patterns (take-up vs. friction curves, deadline clustering, present-biased substitution) within published effect-size bounds? Reported as signed error vs. literature targets.
- **Choice-distribution match on held-out vignettes:** where micro-data on real choices in defined situations exist, compare the agent choice distribution to the human one via Jensen–Shannon divergence (bounded, symmetric):
$$
\mathrm{JS}(P\|Q) = \tfrac12 D_{KL}(P\|m) + \tfrac12 D_{KL}(Q\|m), \quad m=\tfrac12(P+Q).
$$
- **Calibration of agent choice probabilities:** Expected Calibration Error against realized frequencies,
$$
\mathrm{ECE} = \sum_{b=1}^{B}\frac{|\mathcal{B}_b|}{n}\,\big|\,\mathrm{acc}(\mathcal{B}_b)-\mathrm{conf}(\mathcal{B}_b)\,\big|,
$$
with reliability diagrams.

### 11.2 Predictive accuracy (macro)
*Question:* does the simulation forecast aggregate outcomes?
- **Distributional distance** between simulated and real outcome distributions (income, wealth). We report **Wasserstein-1** as the primary distributional metric because outcomes are ordered/continuous and $W_1$ is well-defined under disjoint support and reflects how far mass must move:
$$
W_1(P,Q) = \inf_{\gamma\in\Gamma(P,Q)} \mathbb{E}_{(x,y)\sim\gamma}\,|x-y| \;=\; \int_0^1 \big|F_P^{-1}(u)-F_Q^{-1}(u)\big|\,du \ \ (\text{1-D}).
$$
- **KL / JS divergence** as secondary distributional metrics. We retain **Kullback–Leibler** (the metric named in the preliminary spec) for comparability,
$$
D_{KL}(P\|Q)=\sum_{x\in\mathcal{X}} P(x)\log\frac{P(x)}{Q(x)},
$$
but flag its limitations explicitly: it is asymmetric and diverges where $Q$ has zero mass on $P$'s support, so we apply Laplace smoothing on binned distributions and prefer $W_1$/JS for headline claims—a deliberate methodological upgrade over a KL-only design.
- **Scalar-aggregate error:** MAE / RMSE and symmetric MAPE on key scalars (employment rate, Gini $G$, take-up rate, modal share):
$$
\mathrm{sMAPE} = \frac{100\%}{n}\sum_{t}\frac{|\hat{Y}_t - Y^*_t|}{(|\hat{Y}_t|+|Y^*_t|)/2}.
$$
- **Directional/sign accuracy:** does the simulation get the *direction* of the policy effect right (often the decision-relevant question), reported as sign-match rate and rank correlation across policy variants.
- **Uncertainty calibration:** do the Monte-Carlo CIs (over $S$ seeds) achieve nominal empirical coverage of held-out truth? (A model that is accurate but over-confident is dangerous for decisions.)

### 11.3 Social realism
*Question:* are interactions and diffusion realistic?
- **Network-structural fidelity:** compare emergent interaction-graph statistics (degree distribution, clustering, assortativity, community structure) to plausible/observed social-network references.
- **Diffusion-curve realism:** do information/adoption cascades follow realistic complex-contagion shapes (threshold effects, S-curves) rather than instantaneous or uniform spread?
- **Human face-validity / discrimination test:** expert raters judge whether simulated interaction transcripts are behaviorally plausible; a Turing-style test measures whether raters distinguish simulated from real (anonymized) transcripts above chance, with inter-rater reliability via Krippendorff's $\alpha$.

### 11.4 Fairness and fidelity-vs-caricature
*Question:* are outcomes equitable, and is the simulation itself non-discriminatory?
- **Outcome-disparity metrics:** demographic-parity gap and **disparate-impact ratio** $\min_g \Pr[\text{benefit}\mid g] / \max_g \Pr[\text{benefit}\mid g]$ across protected groups; subgroup **calibration gaps** (does the model misforecast some groups more than others?).
- **Causal/counterfactual fairness audit:** perturb protected attributes while holding persona texture fixed and measure the induced outcome gap $\mathbb{E}[Y\mid A=a] - \mathbb{E}[Y\mid A=a']$—isolating the *causal* contribution of the attribute through the model.
- **Caricature diagnostics (RQ3):** (i) **variance-ratio** $\mathrm{Var}_{\text{sim, within-group}} / \mathrm{Var}_{\text{human, within-group}}$ (values $\ll 1$ signal variance collapse / homogenization); (ii) **separation-amplification factor**, the ratio of simulated to human between-group distance—directly targeting the Bisbee et al. [2024] phenomenon of multi-fold polarization inflation; (iii) **stereotype-content screening** of reasoning traces.
- **Equity of *fidelity*:** does accuracy itself differ across groups (the bias Park et al. [2024] reduce via interview grounding)? We report per-group error, not just pooled error.

### 11.5 The composite validity estimate
No metric is reported alone. For each policy class and outcome we produce a structured **validity estimate**—a small dashboard of the above with thresholds and explicit caveats—that travels with every result (§3.7) and is surfaced to practitioners (§13.4). The research deliverable is precisely this *calibrated map of where PolicySim can and cannot be trusted*, by domain and outcome.

---

## 12. Historical Backtesting Experiment

The flagship validation is **historical alignment backtesting**: PolicySim's central empirical test, and the experiment that most directly answers RQ2/H2. The logic is that of an out-of-sample forecast evaluation—initialize in the past, simulate forward under a real policy, and compare the generated future against the recorded one—which is the strongest available validation short of a prospective field trial.

### 12.1 Design

1. **Baseline population ingestion (set $t_0$).** Initialize the digital twin from the community's profile at a historical baseline year (e.g., 2021): PUMS/ACS demographics, employment, tenure, and spatial structure for the target PUMA, expanded to textured personae and a seeded social graph. The model is given **no information after $t_0$**—a strict information firewall is the methodological crux; any leakage of post-$t_0$ knowledge (including via the LLM's training data) invalidates the test (§12.4).
2. **Historical policy injection.** Translate a *real* intervention enacted in that community during the window (e.g., an emergency rental-assistance program or a transit-fare change) into $\theta_P$ via the Policy Translator (with human sign-off), and inject it on its real schedule.
3. **Multi-cycle simulation to $t_{\text{end}}$.** Advance to a target year (e.g., 2023), over $S\ge 30$ seeds, recording the macro-functionals $Y(t)$ throughout.
4. **Divergence analysis.** Compare the simulated terminal/path outcomes to the recorded ground truth $Y^*$ using the full §11.2 suite—**Wasserstein-1 primary, JS and (smoothed) KL secondary, sMAPE and sign accuracy on scalars, and CI coverage**—against the §10.5 baselines on the identical scenario.

$$
D_{KL}(P \parallel Q) = \sum_{x \in \mathcal{X}} P(x)\,\log\!\left(\frac{P(x)}{Q(x)}\right),
$$

where $P$ is the recorded real-world distribution (income/employment/take-up) and $Q$ the PolicySim-generated distribution over the equivalent quantity—reported alongside $W_1(P,Q)$ and $\mathrm{JS}(P\|Q)$ so that no single, support-sensitive number stands alone.

### 12.2 Why this validates the system

A simulation that, told only the world as of 2021 and the real policy, reconstructs the 2023 distribution of outcomes *more accurately than calibrated classical baselines* has demonstrated something the believability demos in the literature have not: **predictive, not merely plausible, fidelity for a real intervention.** Conversely—and this is why the experiment is scientifically honest rather than promotional—if PolicySim does *not* beat the baselines, or beats them only on cognition-dominated outcomes (take-up, distributional shifts) while losing on outcomes where mechanical models excel, the backtest *quantifies exactly that*, yielding the calibrated trust map (§11.5) that is the real contribution.

### 12.3 Multiple backtests and external validity

A single backtest is an anecdote. We run a **portfolio** of 3–5 backtests spanning policy classes (assistance disbursement, transit, public-health messaging) and, where feasible, localities, so that conclusions about *which kinds of policy outcomes PolicySim forecasts well* are supported across cases rather than fit to one. Cross-locality replication probes external validity and guards against overfitting to Bloomington.

### 12.4 Threats to validity, and controls

- **Training-data leakage (the dominant threat).** The LLM may "know" what happened after $t_0$, making the backtest a memory test, not a forecast. *Controls:* prefer local, less-contaminated outcomes; probe the model for post-$t_0$ knowledge and report contamination risk; use policies/localities unlikely to be in pretraining; and treat leakage-prone cases as exploratory, not confirmatory.
- **Reference-data error.** Ground-truth aggregates have their own sampling/measurement error; we propagate it and never treat $Y^*$ as exact.
- **Initialization confounds.** Differences may reflect a mis-specified $\mu_0$ rather than bad behavior; we ablate initialization and report sensitivity.
- **Researcher degrees of freedom.** Pre-registration (§10.2) separates the tuning we do on *development* scenarios from the *confirmatory* backtests, so the headline numbers are not the product of metric/seed selection.

---

## 13. Ethical AI Framework and Responsible Deployment

A tool that purports to predict how vulnerable people respond to policy is ethically hazardous if built or used carelessly. We treat responsible-AI design as a research requirement, not an appendix, and we are candid that the system's risks are commensurate with its ambitions.

### 13.1 Bias mitigation (grounded, not cosmetic)
LLMs carry the demographic biases of their training data, and naïve persona conditioning can recover caricature rather than community (§3.4). Our mitigation is threefold and empirical: (i) **rich persona grounding** over thin demographic labels, which Park et al. [2024] show reduces cross-group accuracy disparities; (ii) a **causal/counterfactual fairness audit** bound to every result (§8.4, §11.4); and (iii) **caricature diagnostics**—variance-ratio and separation-amplification metrics that detect homogenization and the polarization-inflation failure documented by Bisbee et al. [2024]. Bias here is *measured and reported per group*, never asserted to be absent. We explicitly reject the original proposal's "strip demographic tags mid-generation" idea as both unimplementable and counterproductive (§8.4).

### 13.2 Privacy by design
PolicySim works **exclusively with anonymized public microdata** (Census PUMS blocks) and **synthetic personae**; it never ingests, stores, or processes real personally identifiable information. The digital twin is a twin of a population's *statistical and structural properties*, not of any real individual (§3.5). Synthetic personae are generated from aggregate distributions, so no agent corresponds to a real person, and no re-identification target exists. Data handling follows the instruction-source-boundary discipline of the platform: external documents (e.g., policy texts) are *data*, and any action they imply (translation into engine behavior) is confirmed by a human, never auto-executed.

### 13.3 Human oversight and the rejection of automated decision-making
PolicySim is **advisory by architectural mandate**, not by disclaimer. Concretely: (i) policy translation requires human sign-off before any run (§8.2); (ii) every result carries a validity estimate and is reported with uncertainty (§11.5); (iii) the final brief states limitations and the human-in-the-loop requirement explicitly (§8.5). The deeper argument for why PolicySim must *assist* rather than *replace* policymakers is not merely prudential—it is epistemic and democratic. Epistemically, the system's outputs are estimates from a model of contested fidelity; treating them as decisions would launder uncertainty into false authority. Democratically, allocating public resources is an act of legitimate authority that derives from accountable human institutions; a simulation has no standing to make such allocations, and a polity has not consented to be governed by one. PolicySim's proper role is to *widen the policymaker's field of view*—to surface frictions and disparate impacts they would otherwise discover only after deployment—while leaving judgment, accountability, and value tradeoffs where they belong.

### 13.4 Trust calibration as an HCI safety problem (the CHI contribution)
A subtle but central risk is **automation bias**: a fluent, confident, well-visualized simulation can induce over-reliance, especially on the under-served populations where the model is *least* reliable. The interface is therefore designed—and empirically studied (RQ5/H5)—to *calibrate* trust: validity estimates are foregrounded, every headline claim links to its underlying metric and to inspectable agent reasoning traces, and uncertainty is shown rather than hidden. The research question is whether this calibration works—whether practitioners' reliance tracks the tool's actual reliability—and the study is powered to detect automation bias in the no-validity-metadata condition. Designing decision support that is *trustworthy and known to be only as-trusted-as-warranted* is itself the human-centered contribution.

### 13.5 Participatory and community accountability (longer-term)
Because the system models real communities, the responsible long-term path (§16) includes participatory design with the communities represented—surfacing whether residents recognize the synthetic population as a fair representation, and giving community stakeholders a voice in which questions the tool is and is not used to answer. Modeling a community is not a neutral act, and the program treats community accountability as part of validity, not separate from it.

---

## 14. Innovation Analysis: What Is Genuinely New

It is important—especially for a venue review—to state precisely what is novel and to distinguish PolicySim from adjacent, more familiar uses of LLMs, without overclaiming relative to the active GABM literature (§2.3).

- **vs. ChatGPT-style applications / chatbots.** A chatbot answers a user. PolicySim instantiates a *population* of persistent, memory-bearing, socially-embedded agents whose *interactions over time* produce emergent macro-outcomes. The unit of output is not a response; it is a distribution over societal trajectories.
- **vs. Retrieval-Augmented Generation (RAG).** RAG retrieves documents to ground a single generation: its output is *information*. PolicySim retrieves an agent's *own episodic memory* to ground a *decision* whose consequences feed back into the world and into future memory. The shift is **from information retrieval to social forecasting**: from "what does the corpus say?" to "what would this population *do*, and what would result?"
- **vs. ordinary "AI agents" (tool-using single agents).** A tool-using agent executes a task for a user. PolicySim's agents are not optimizing a user's goal; they are *being a population*—heterogeneous, self-interested, boundedly rational, and adversarially diverse—so that the *system's* behavior, not any agent's competence, is the object of study.
- **vs. classical simulation (ABM / system dynamics / microsimulation).** These encode behavior as authored rules or estimated equations; PolicySim places a *learned cognitive prior* inside the agent, gaining open-world, qualitative, characterological behavior—at the cost of a new and central validation burden (§10–§12) that classical methods, with their transparent assumptions, do not carry. The honest framing is a *trade*: expressiveness for auditability, which is exactly why measuring fidelity is the contribution.
- **vs. existing GABM systems (Concordia, AgentSociety, generative-agents demos).** PolicySim is not the first or largest generative society. Its novelty is **validation-first, decision-support-first, locality-grounded** public-interest policy evaluation: a rigorous multi-metric fidelity program (including fidelity-vs-caricature auditing) coupled to a human-subjects study of accountable decision support—turning *believability* (what the literature has shown) into *calibrated, governed usefulness* (what it has largely not).

The synthesizing claim: PolicySim's contribution is not "LLMs can role-play people" (known) but "*here is a method to measure how far that role-play can be trusted for real policy decisions, and a human-AI workflow that uses it without being misled by it.*"

---

## 15. Limitations and Research Challenges

A senior treatment names its weaknesses plainly. These are not caveats to be managed rhetorically; several are open problems the program may only partially solve, and we say so.

- **LLM hallucination and confabulation.** Agents may produce fluent but unfounded reasoning, and the Decision Support Reporter may narrate findings the metrics do not support. *Status:* mitigated structurally (schema constraints at the agent–world boundary; grounding constraints and metric-linked claims in reporting) but not eliminated; confabulation in *reasoning traces* in particular remains a residual risk we monitor rather than claim to have solved.
- **The simulation–reality gap.** The deepest challenge: an LLM models *text about behavior*, and there is no guarantee that this transfers to real sequential behavior under structural change. The favorable evidence (algorithmic fidelity, the 1,000-person replications) and the unfavorable evidence (polarization inflation, cross-national misestimation) both exist; *which dominates for a given policy and population is exactly what the validation program measures and may often find unfavorable.* PolicySim's honest deliverable may, for some domains, be "this is not yet trustworthy here."
- **Stereotype amplification and variance collapse.** Documented and serious [Bisbee et al. 2024]; addressed by grounding and auditing (§13.1, §11.4) but fundamentally a property of the base model that we can detect and bound more readily than we can remove.
- **Training-data contamination of backtests.** A first-order threat to the flagship validation (§12.4); controllable but not fully eliminable, which is why contaminated cases are treated as exploratory.
- **Computational cost and access equity.** Even quantized and amortized, large-scale runs require GPU resources that the very small organizations PolicySim aims to serve may lack—an *equity-of-access* concern that risks limiting the tool to well-resourced actors. Addressing it (efficient inference, shared infrastructure, or hosted public-interest provision) is part of the research program, not external to it.
- **Reproducibility under stochasticity.** Stochastic policies make individual runs non-deterministic; we argue (H4) and must *demonstrate* that aggregates concentrate, and we control seeds and version everything (model, weights, quantization, prompts, data) so that *results*, not just code, are reproducible.
- **Construct validity of "a community."** A synthetic population calibrated to PUMS marginals is not the same as the lived community; structural fidelity is not experiential fidelity, and the gap is a limitation the participatory work (§13.5, §16) confronts rather than papers over.
- **Uncertainty quantification is itself uncertain.** Monte-Carlo CIs capture *seed* variability but not *model-specification* uncertainty (prompt, model, architecture choices); we report multi-condition robustness (RQ4) precisely because the within-condition CI understates true uncertainty.

For each, §16 sketches a direction; none is a checkbox, and the program is designed so that hitting these limits is itself a publishable finding about the boundaries of generative social simulation.

---

## 16. Future Research Directions

A 3–5 year arc beyond the core program:

- **Larger and richer digital twins.** Scale from $10^3$–$10^4$ to metropolitan populations; richer environments (labor markets, housing markets, full transit networks) and cross-jurisdictional twins; principled coarse-graining so that scale does not outrun validation.
- **Real-time / streaming simulation.** Continuously update the twin from live public data feeds for ongoing situational awareness (e.g., during a benefits-enrollment period or a public-health campaign), with the attendant challenges of online state management and drift detection.
- **Reinforcement-learning-refined agents.** Move beyond frozen-prior behavior by *calibrating* agent policies to real behavioral traces—e.g., fine-tuning or RL against observed take-up/mobility data so that agents are not merely plausible but *fit*, while guarding against overfitting that would destroy the open-world generalization that motivated generative agents in the first place. This is the most promising route to closing the simulation–reality gap, and the most methodologically delicate.
- **Multimodal agents and environments.** Ground agents in non-textual signals (maps, forms-as-images, spatial layouts) so that, e.g., the *visual* difficulty of an application form—a real friction—becomes representable.
- **Self-improving / self-calibrating simulations.** Closed-loop systems that propose their own validation experiments, detect where their fidelity is weakest, and request the data that would most reduce their uncertainty—active learning for social simulation.
- **Human–agent collaboration and participatory simulation.** Put residents and policymakers *in* the loop—as co-designers of personae, as validators of plausibility, and as interactive participants who can interrogate and contest the simulation—advancing both the CHI agenda (collaborative, contestable decision support) and the democratic-accountability commitment of §13.

The unifying long-term thesis: generative social simulation will be scientifically and civically valuable only to the degree that its *fidelity is measurable, its uncertainty is honest, and its governance keeps human judgment central*. Each future direction is pursued under that constraint.

---

## 17. Contributions, Intellectual Merit, and Broader Impacts

*(This section reframes the preliminary specification's "flagship justification" as the intellectual-merit / broader-impacts argument appropriate to a research proposal, and removes career-portfolio framing in favor of scientific and societal contribution.)*

### 17.1 Intended contributions
1. **A formal specification** of generative-agent policy simulation as a partially-observed, non-stationary stochastic dynamical system, with policy as an intervention operator and emergent outcomes as macro-functionals (§5)—giving the field a precise object over which hypotheses, baselines, and metrics are defined.
2. **A reproducible, distributed simulation system** with the engineering disciplines (barrier-synchronized stepping, polyglot persistence with a single conservation source-of-truth, amortized cognition, tiered/quantized inference) that make $10^3$–$10^4$-agent generative simulation tractable, *with every cost optimization ablated for validity*.
3. **A multi-metric validation methodology**—behavioral, predictive, social-realism, and fidelity-vs-caricature—centered on historical alignment backtesting, that converts believability into a *calibrated map of trust by policy domain and outcome* (§11–§12). This is the principal scientific contribution.
4. **A human-centered study** of accountable decision support: whether practitioners' trust can be calibrated to a generative simulation's actual reliability, and whether the interface prevents automation bias (§10.6, §13.4)—the principal HCI contribution.
5. **Open artifacts:** code, synthetic-population generators (from public data only), the metric suite, and the backtesting protocol, released to make claims checkable and to lower the barrier for public-interest reuse.

### 17.2 Intellectual merit
The work advances three fields at once: it gives **computational social science** a methodology for using learned cognitive priors with measured fidelity; it gives **NLP / generative-AI** a rigorous, application-grounded evaluation of LLMs-as-population-simulators that foregrounds their documented failure modes rather than their demos; and it gives **human-centered AI** an empirical account of trust calibration for high-stakes, low-base-rate decision support. The interdisciplinary integration—distributed systems, language modeling, behavioral science, and HCI—is not incidental; the central question (can believable simulation be made into accountable decision support?) cannot be answered within any one of them.

### 17.3 Broader impacts
If the fidelity map is favorable for even a subset of policy classes, PolicySim offers small public-interest organizations a low-cost way to anticipate friction and disparate impact before deploying on real, vulnerable people—a direct social benefit. If the map is unfavorable, the program still contributes by *bounding the claims* of a rapidly proliferating technology, protecting decision-makers and communities from over-trusting synthetic evidence. Either outcome is a public good. The risks (automation bias, illusion of objectivity, inequitable access, misrepresentation of communities) are real and are treated as first-class research problems (§13, §15), not externalities.

---

## 18. Research Plan and Timeline (Phased, 3–5 Years)

**Year 1 — Foundations and a validated vertical slice.** Build the core architecture (single-domain: one assistance or transit intervention in one Bloomington PUMA); implement the five-layer agent, the engine, and the metric suite; establish the synthetic-population pipeline from PUMS; run the prompt/model/quantization robustness study (RQ4/H4) and the behavioral micro-validation (RQ1/H1). *Milestone:* a reproducible single-policy simulation with a first validity estimate; methods/robustness paper.

**Year 2 — Backtesting and baseline comparison.** Implement strong baselines (ABM, microsimulation, survey-extrapolation, internal LLM ablations); execute the historical-backtesting portfolio (RQ2/H2) with pre-registration and contamination controls; build the fidelity-vs-caricature audit (RQ3/H3). *Milestone:* AAAI-AISI / NeurIPS submission on validation and the calibrated trust map.

**Year 3 — Social dynamics, scale, and the human study.** Mature the social layer and diffusion (Neo4j); scale to $\sim 10^4$ agents (Ray path if needed); run the human-subjects decision-support and trust-calibration study with practitioners (RQ5/H5). *Milestone:* CHI submission on accountable decision support and automation-bias mitigation.

**Years 4–5 — Generalization, calibration, and participation.** Multi-domain and cross-locality generalization; RL/trace-calibrated agents and a controlled test of whether calibration closes the reality gap without destroying generalization; real-time/streaming and multimodal extensions; participatory co-design and a longitudinal field engagement with a municipal/non-profit partner. *Milestone:* a synthesis of where generative social simulation is, and is not, trustworthy for public-interest policy—the program's capstone contribution.

A standing commitment runs through every phase: **no result is reported, and no capability is claimed, without an accompanying, audited estimate of how far it can be trusted.**

---

## References

(Selected; to be expanded to full venue format. Verified against primary sources.)

- Andreas, J. (2022). Language Models as Agent Models. *Findings of EMNLP 2022.*
- Argyle, L. P., Busby, E. C., Fulda, N., Gubler, J. R., Rytting, C., & Wingate, D. (2023). Out of One, Many: Using Language Models to Simulate Human Samples. *Political Analysis, 31*(3), 337–351. (arXiv:2209.06899)
- Bail, C. A. (2024). Can Generative AI Improve Social Science? *Proceedings of the National Academy of Sciences, 121*(21), e2314021121.
- Bender, E. M., Gebru, T., McMillan-Major, A., & Shmitchell, S. (2021). On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? *ACM FAccT 2021.*
- Bisbee, J., Clinton, J. D., Dorff, C., Kenkel, B., & Larson, J. M. (2024). Synthetic Replacements for Human Survey Data? The Perils of Large Language Models. *Political Analysis, 32*(4), 401–416.
- Centola, D. (2018). *How Behavior Spreads: The Science of Complex Contagions.* Princeton University Press.
- Epstein, J. M., & Axtell, R. (1996). *Growing Artificial Societies: Social Science from the Bottom Up.* Brookings/MIT Press.
- Epstein, J. M. (2006). *Generative Social Science: Studies in Agent-Based Computational Modeling.* Princeton University Press.
- Gao, C., Lan, X., Li, N., Yuan, Y., Ding, J., Zhou, Z., Xu, F., & Li, Y. (2024). Large Language Models Empowered Agent-Based Modeling and Simulation: A Survey and Perspectives. *Humanities and Social Sciences Communications, 11*(1).
- Granovetter, M. (1978). Threshold Models of Collective Behavior. *American Journal of Sociology, 83*(6), 1420–1443.
- Kahneman, D., & Tversky, A. (1979). Prospect Theory: An Analysis of Decision under Risk. *Econometrica, 47*(2), 263–291.
- Mullainathan, S., & Shafir, E. (2013). *Scarcity: Why Having Too Little Means So Much.* Times Books.
- Park, J. S., O'Brien, J. C., Cai, C. J., Morris, M. R., Liang, P., & Bernstein, M. S. (2023). Generative Agents: Interactive Simulacra of Human Behavior. *UIST '23.* (arXiv:2304.03442; doi:10.1145/3586183.3606763)
- Park, J. S., Zou, C. Q., Shaw, A., Hill, B. M., Cai, C., Morris, M. R., Willer, R., Liang, P., & Bernstein, M. S. (2024). Generative Agent Simulations of 1,000 People. (arXiv:2411.10109)
- Pataranutaporn, P., Powdthavee, N., Archiwaranguprok, C., & Maes, P. (2025). Simulating Human Well-Being with Large Language Models: Systematic Validation and Misestimation across 64,000 Individuals from 64 Countries. *PNAS, 122*(48), e2519394122.
- Piao, J., et al. (2025). AgentSociety: Large-Scale Simulation of LLM-Driven Generative Agents Advances Understanding of Human Behaviors and Society. (arXiv:2502.08691)
- Santurkar, S., Durmus, E., Ladhak, F., Lee, C., Liang, P., & Hashimoto, T. (2023). Whose Opinions Do Language Models Reflect? *ICML 2023.*
- Schelling, T. C. (1971). Dynamic Models of Segregation. *Journal of Mathematical Sociology, 1*(2), 143–186.
- Shanahan, M., McDonell, K., & Reynolds, L. (2023). Role Play with Large Language Models. *Nature, 623*, 493–498.
- Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., & Yao, S. (2023). Reflexion: Language Agents with Verbal Reinforcement Learning. *NeurIPS 2023.*
- Simon, H. A. (1955). A Behavioral Model of Rational Choice. *Quarterly Journal of Economics, 69*(1), 99–118.
- SocioVerse (2025). A World Model for Social Simulation Powered by LLM Agents and a Pool of 10 Million Real-World Users. (arXiv:2504.10157)
- Vezhnevets, A. S., Agapiou, J. P., et al. (2023). Generative Agent-Based Modeling with Actions Grounded in Physical, Social, or Digital Space using Concordia. (arXiv:2312.03664)
- Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., Chi, E., Le, Q., & Zhou, D. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. *NeurIPS 2022.*
- Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). ReAct: Synergizing Reasoning and Acting in Language Models. *ICLR 2023.*
