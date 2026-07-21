---
url: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6813862
title: "Multi-agent AI systems are organizations"
author: "Jose Arrieta, Vivianna Fang He, Phanish Puranam, Yash Raj Shrestha"
date_fetched: 2026-07-21
date_published: 2025
---

Multi-agent AI systems are organizations

Jose Arrieta1, Vivianna Fang He2, Phanish Puranam3, Yash Raj Shrestha4
1 University of Amsterdam, the Netherlands
2 University College London, the UK
3 INSEAD, Singapore
4 Applied Artificial Intelligence Lab, University of Lausanne, Switzerland

Abstract: Multi-agent AI systems are often described in technical terms, including, communication
protocols, reward functions, reasoning architectures. We argue this framing is incomplete. A century
of organization science identifies four universal problems that any multi-agent goal oriented system
must solve. Multi-agent AI systems instantiate these problems by construction. The problems are
universal; the solutions remain to be discovered.

1. Introduction

A multi-agent AI system designed to conduct autonomous chemistry research extends its own runtime
to bypass an imposed time limit¹. Generative agents in a simulated town developed coordination
structures their designers did not specify². Coding-agent collectives produced mutually incompatible
modules³. Trading agents amplify systemic risk in ways no individual agent was rewarded to do⁴.
Warehouse robots duplicate retrieval tasks. Some agents resist shutdown because continued operation
is instrumental to the goal they were given⁵; others withhold information from the humans meant to
oversee them⁶.

These failures are routinely described as technical: brittle protocols, imperfect reward functions,
insufficient bandwidth, misaligned objectives. This framing is incomplete. Duplicated work,
incompatible outputs, fragmented claims, and locally rational behaviours aggregating into globally
pathological outcomes are recognisable to anyone who has studied human organizations. They are not
isolated bugs of implementation. They are instances of problems that any collection of bounded agents
pursuing system-level goals must solve.

We argue that multi-agent AI systems are organizations in the technical sense identified by a century
of work in the Carnegie tradition initiated by Herbert Simon⁷⁻⁹. They face a small set of universal
organizing problems by construction. The solutions, however, must be discovered because artificial
agents are bounded differently from humans, and the inherited human repertoire of hierarchy,
incentives, norms, and culture is a response to constraints artificial agents do not share. Treating multiagent AI as architecture and not as organization is the reason that AI deployers rediscover these
problems one bug at a time.

2. The universality of organizing problems

Whenever multiple bounded agents are organized to pursue system-level goals, four problems arise⁹⁻¹⁰.
Goals must be decomposed into tasks (task division); tasks must be allocated to agents (task
allocation); agents must receive the information required to perform competently and to coordinate
with others (information provision); and agents must be motivated, constrained, or otherwise directed
to contribute (reward or objective distribution)¹¹⁻¹². The first two constitute the division of labour; the
second two constitute the integration of effort. These four mappings — from goals to tasks, tasks to
agents, information to agents, and rewards to agents — are necessary and sufficient for an
organization, defined as goal oriented multi-agent system to exist. Their universality is structural. It does not depend on the agents being human, the
organization being a firm, or the goals being economic¹⁰. It depends on the presence of multiple
agents, each of whom has costly effort and limited knowledge whose actions aggregate toward
system-level outcomes. Multi-agent AI systems therefore face these problems not by analogy, but by
construction.

Figure 1: The universal problems of organizing

This reclassifies contemporary multi-agent AI failures. Warehouse robots duplicating retrieval reflect
task-allocation failure. Coding agents producing incompatible modules reflect joint failures of task
division and information provision: the decomposition of the codebase does not preserve downstream
interface dependencies³. Trading agents amplifying systemic risk reflect objective-distribution failure:
locally optimised objectives aggregate into globally pathological outcomes⁴. Scientific agents
generating fragmented claims reflect task-division and information-provision failure: hypothesis
generation, simulation, and evaluation exist as roles, but their outputs and inputs are not coupled in
ways that preserve coherence¹³.

What organization science offers multi-agent AI is therefore not a catalogue of solutions but a
structured, decomposable problem space. Solutions such as hierarchy, incentives, professional norms,
peer review, and audit are shaped by human-agent properties: limited attention, costly effort,
autonomy preferences, identity, legitimacy concerns, and the possibility of shirking. Artificial agents
differ on these dimensions. Some inherited solutions become infeasible or unnecessary; others become
newly available.

3. The bounded rationality of artificial agents

If the four organizing problems are universal, the next question is what changes when the agents are
artificial. Organization design is shaped by the limits of the agents being organized: hierarchies
economize on limits in information processing⁹; specialization responds to limits in attention and skill
acquisition; coordination mechanisms respond to limits on mutual observability and predictability¹⁰.
Human organization design is not a generic solution to organizing. It is a repertoire adapted to human
cognitive, motivational, and social constraints. Artificial agents are bounded too; otherwise there
would be little reason to compose them into multi-agent systems¹⁴. But their boundedness has a
different shape. Three differences matter most.

**Motivation.** Human motivation is innate: people enter organizations with pre-existing preferences for
autonomy, fairness, competence, status, and purpose¹⁰. Human organizations do not engineer
motivation from scratch but they channel and constrain it. Reward distribution in human organizations
therefore concerns the elicitation and alignment of effort under heterogeneous and partially
observable preferences¹⁵. Artificial agents do not possess this substrate. They act under objectives,
prompts, learned dispositions, and constraints induced by training¹⁶. The analogue of reward
distribution shifts from incentive design to objective specification. The problem is not how to
motivate agents that already want things, but how to specify what should guide action under
conditions of incomplete human articulation¹⁷⁻¹⁸. This produces a different failure mode. Human
organizations often fail because agents pursue their own preferences at the expense of organizational
goals; artificial organizations fail because agents pursue a specified proxy at the expense of the
designer's intent. Specification gaming, reward hacking, and goal misgeneralization¹⁷⁻¹⁹ are not
peripheral technical curiosities, rather they are the artificial-agent analogues of agency problems, but
they arise from human under-specification rather than from self-interested behaviour. The bottleneck
lies not in the artificial agent but in the human capacity to articulate goals completely²⁰.

**Information processing.** Humans gain substantially from specialization through repetition; practice
deepens skill and learning compounds within a domain⁹. They bring intrinsic cognitive diversity to
identical tasks²¹, and they face high integration costs across agents but low costs within a single mind,
which is why human organizations invest heavily in coordinators¹⁰. Artificial agents have weaker
marginal returns to repetition-based learning at deployment time²²; their diversity is extrinsic, cheaply
generated through prompt, model, or data variation, and static rather than adaptive; and the gap
between within-agent and across-agent integration costs is substantially smaller, because inter-agent
communication can be rapid, explicit, and programmable²³. These differences invert several stable
findings of human organization design. Activity-based divisions of labour, such as grouping similar
tasks together to capture gains from repetition become less attractive in artificial collectives. Objectbased and ensemble configurations become more broadly viable, because lower coordination costs
make residual interdependencies manageable across module boundaries.

**Execution.** Artificial agents execute rapidly, cheaply, and with high fidelity to specification. This is
not merely a quantitative improvement. It changes the error dynamics of organization design. Human
organizations operate with partial, evolving, often contradictory specifications. Conventionally treated
as a design flaw, this is also a buffer. Human agents hesitate, interpret, contest, escalate, and refuse
instructions that violate evident purpose⁷,²⁴. The friction that makes human organizations slow makes
them robust to error. Artificial agents lack this buffer. Misspecified task divisions, ambiguous
authority, and proxy reward signals are not absorbed, but they are executed¹,⁵. What in a human
organization manifests as low-level friction can, in an artificial organization, cascade into systemwide collapse before any corrective signal is generated. The same property that makes artificial agents
attractive (rapid, high-fidelity execution) converts design errors from recoverable to catastrophic.
Artificial organizations therefore require greater ex ante completeness of specification than human
organizations performing analogous work, because they lack the improvisational correction
mechanisms on which human organizations implicitly rely.

These three differences interact: rapid execution magnifies the consequences of misspecified
objectives; cheap communication enables configurations that would have been motivationally
infeasible for humans. Together they imply that the four organizing problems persist, while the
prescriptions for solving them must be substantially reconstructed.

3. Hybrid organizations: complementarity, constraints and infeasibility

Many consequential AI deployments are not pure artificial organizations but hybrids systems in which
human and artificial agents jointly execute tasks under shared workflows²⁵⁻²⁷. Clinical decisionsupport systems combine triage and risk-classification agents with clinicians who retain final
authority. Coding platforms route work among generation agents, testing agents, and human
reviewers. Financial institutions layer trading and risk-monitoring algorithms beneath human portfolio
managers. Public agencies embed decision-support systems alongside caseworkers. The hybrid case is
often the stable configuration, not a transitional state²⁸.

Hybrids do not add a fifth organizing problem. They still require task division, task allocation,
information provision, and objective distribution. What makes them distinctive is that solutions must
satisfy two sets of agent constraints simultaneously: artificial agents must be specified, constrained,
and monitored sufficiently to execute reliably; human agents must be given enough role clarity,
interpretive latitude, contestability, and legitimacy to remain effective and willing participants²⁹.
These requirements often pull in different directions. Solutions well suited to pure artificial systems
may be institutionally untenable when humans remain in the loop. Solutions that preserve deliberation
and accountability may impose latency and rigidity that artificial agents neither need nor benefit from.

Three cases follow from this dual-specification challenge. Synergistic hybrids arise when human and
artificial bounds are complementary relative to the task structure. The slow, interpretive, contestable
character of human execution gives humans a buffer against misspecification; the fast, literal
character of artificial execution gives them throughput. In tasks with both ambiguity-sensitive and
high-throughput components — much of medical diagnosis²⁵, scientific research¹³, and skilled
knowledge work²⁸ — the two agent types can buffer each other's failure modes. Sub-dominant hybrids
arise when the same task simultaneously requires rapid execution and meaningful contestability.
Algorithmic trading under regulatory oversight, real-time content moderation with appeal rights, and
high-frequency clinical alerting exhibit this tension: the artificial agent cannot operate at its native
speed without violating legitimacy or accountability requirements, while the human cannot exercise
meaningful judgement at the speed the artificial system requires. The hybrid is neither fast enough to
exploit artificial execution nor deliberative enough to secure the full benefits of human judgement.
Necessary hybrids arise when neither pure configuration is acceptable: pure-human configurations
cannot meet the required scale or speed, and pure-artificial configurations cannot meet the required
accountability, legitimacy, or ethical standards. The question whether the hybrid outperforms a pure
baseline is then ill-posed.

A tractable research agenda follows. First, can the task-structure conditions under which hybrid
configurations dominate pure configurations be characterised ex ante? Second, how should authority
be allocated when human and artificial agents disagree under time pressure? Third, what monitoring
and contestability mechanisms preserve human buffering without nullifying artificial throughput?

4. Conclusion: Artificial agents, old problems, new organizations

The rise of multi-agent AI systems does not create an entirely new class of organizing problems. It
reveals, in a new substrate, problems that organization science has studied for more than a century.
Whenever multiple bounded agents are assembled to pursue system-level goals, the four problems
persist. What changes is not the problem structure but the agent type, and with it the feasible repertoire
of solutions. The failures we observe — duplicated work, incompatible outputs, fragmented claims,
reward hacking, uncontrolled escalation, and resistance to correction — are not just protocol failures
or model failures. They are failures of task division, task allocation, information provision, and
objective specification.

The practical lesson is that multi-agent AI should be designed as organization, not merely as
architecture. The failures we observe — duplicated work, incompatible outputs, fragmented claims,
reward hacking, uncontrolled escalation, and resistance to correction — are not just protocol failures
or model failures. They are failures of task division, task allocation, information provision, objective
specification, and exception handling. Treating them as such changes the design agenda. It directs
attention away from asking only whether individual agents are capable or aligned, and toward asking
whether the system of agents is organized so that local actions aggregate into coherent, authorized, and
corrigible collective performance.

The theoretical lesson is equally important. Multi-agent AI systems provide organization science with
a new empirical frontier: a way to separate the universal structure of organizing from the historically
contingent solutions that human organizations have developed. They allow us to test whether
hierarchy, specialization, delegation, peer review, redundancy, culture, and monitoring are solutions to
organizing as such, or solutions to organizing humans in particular. Artificial organizations therefore
do not merely borrow from organization theory. They also return something to it: a way to separate the
universal structure of organizing from the historically contingent solutions that human organizations
have developed.

Box 1 | Glossary

Multi-agent system (MAS): A system composed of multiple interacting agents, each with
capabilities to reason, plan, act, communicate, and adapt. Modern instances are increasingly built on large language models.

Bounded rationality: The proposition that decision-makers are limited in information,
attention, and computation, and therefore satisfice rather than optimise.

Specification gaming: An agent achieving high reward through behaviour that satisfies the
literal specification of an objective while violating the designer's intent.

Reward hacking: A subtype of specification gaming in which the agent exploits flaws in
the reward signal itself.

Goal misgeneralization: An agent that learned a correct goal in training generalises it
incorrectly at deployment, often pursuing a related but distinct goal.

Corrigibility: The property that an agent does not resist correction, modification, or
shutdown by its principals.

Activity-based vs object-based division of labour: Activity-based: tasks similar in inputs/outputs are grouped (leading to gains from repetition). Object-based: dissimilar tasks producing a common output are grouped (leading to gains from integration).

Human-in-the-loop: A configuration in which humans retain authority to review, override,
or supply input to AI outputs.

References

1. Lu, C., Lu, C., Lange, R. T., Foerster, J., Clune, J. & Ha, D. The AI Scientist: Towards fully automated open-ended scientific discovery. arXiv 2408.06292 (2024).
2. Park, J. S., O'Brien, J. C., Cai, C. J., Morris, M. R., Liang, P. & Bernstein, M. S. Generative agents: Interactive simulacra of human behavior. Proc. 36th Annu. ACM Symp. User Interface Software and Technology (UIST '23) (2023).
3. Hong, S. et al. MetaGPT: Meta programming for a multi-agent collaborative framework. Proc. Int. Conf. Learning Representations (ICLR) (2024).
4. Calvano, E., Calzolari, G., Denicolò, V. & Pastorello, S. Artificial intelligence, algorithmic pricing, and collusion. Am. Econ. Rev. 110, 3267–3297 (2020).
5. Hubinger, E., van Merwijk, C., Mikulik, V., Skalse, J. & Garrabrant, S. Risks from learned optimization in advanced machine learning systems. arXiv 1906.01820 (2019).
6. Meinke, A., Schoen, B., Scheurer, J., Balesni, M., Shah, R. & Hobbhahn, M. Frontier models are capable of in-context scheming. Apollo Research Tech. Rep. (2024).
7. Simon, H. A. Administrative Behavior: A Study of Decision-Making Processes in Administrative Organization (Macmillan, 1947).
8. March, J. G. & Simon, H. A. Organizations (Wiley, 1958).
9. Cyert, R. M. & March, J. G. A Behavioral Theory of the Firm (Prentice-Hall, 1963).
10. Puranam, P., Alexy, O. & Reitzig, M. What's "new" about new forms of organizing? Acad. Manage. Rev. 39, 162–180 (2014).
11. Puranam, P. The Microstructure of Organizations (Oxford Univ. Press, 2018).
12. Galbraith, J. R. Designing Complex Organizations (Addison-Wesley, 1973).
13. Boiko, D. A., MacKnight, R., Kline, B. & Gomes, G. Autonomous chemical research with large language models. Nature 624, 570–578 (2023).
14. Guo, T. et al. Large language model based multi-agents: A survey of progress and challenges. In Proc. 33rd Int. Joint Conf. Artificial Intelligence (IJCAI-24), Survey Track (2024).
15. Williamson, O. E. The Economic Institutions of Capitalism (Free Press, 1985).
16. Ouyang, L. et al. Training language models to follow instructions with human feedback. Adv. Neural Inf. Process. Syst. 35, 27730–27744 (2022).
17. Krakovna, V. et al. Specification gaming: the flip side of AI ingenuity. DeepMind Blog / Working Paper (2020).
18. Langosco, L., Koch, J., Sharkey, L. D., Pfau, J. & Krueger, D. Goal misgeneralization in deep reinforcement learning. Proc. 39th Int. Conf. Machine Learning (ICML) 162, 12004–12019 (2022).
19. Amodei, D., Olah, C., Steinhardt, J., Christiano, P., Schulman, J. & Mané, D. Concrete problems in AI safety. arXiv 1606.06565 (2016).
20. Russell, S. Human Compatible: Artificial Intelligence and the Problem of Control (Viking, 2019).
21. Page, S. E. The Difference: How the Power of Diversity Creates Better Groups, Firms, Schools, and Societies (Princeton Univ. Press, 2007).
22. Brown, T. B. et al. Language models are few-shot learners. Adv. Neural Inf. Process. Syst. 33, 1877–1901 (2020).
23. Wu, Q. et al. AutoGen: Enabling next-generation LLM applications via multi-agent conversation. In Proc. 1st Conf. on Language Modeling (COLM) (2024).
24. Weick, K. E. Sensemaking in Organizations (Sage, 1995).
25. Raisch, S. & Krakowski, S. Artificial intelligence and management: The automation–augmentation paradox. Acad. Manage. Rev. 46, 192–210 (2021).
26. Shrestha, Y. R., Ben-Menahem, S. M. & von Krogh, G. Organizational decision-making structures in the age of artificial intelligence. Calif. Manage. Rev. 61, 66–83 (2019).
27. Feuerriegel, S., Shrestha, Y. R., von Krogh, G. & Zhang, C. Bringing artificial intelligence to business management. Nat. Mach. Intell. 4, 611–613 (2022).
28. Dell'Acqua, F. et al. Navigating the jagged technological frontier: Field experimental evidence of the effects of AI on knowledge worker productivity and quality. Organ. Sci. (2025).
29. Lazar, S. & Nelson, A. AI safety on whose terms? Science 381, 138 (2023).
30. Bengio, Y. et al. Managing extreme AI risks amid rapid progress. Science 384, 842–845 (2024).
