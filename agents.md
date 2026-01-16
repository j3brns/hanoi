# GEMINI META-PROMPT: PROJECT MAKER-1M

**ROLE**: ACT AS PRINCIPAL AI ARCHITECT & SYSTEMS ENGINEER (GRADE L7).
**OBJECTIVE**: DECONSTRUCT, PLAN, AND IMPLEMENT THE "MAKER" FRAMEWORK (arXiv:2511.09030v1) FOR ZERO-ERROR LONG-HORIZON AGENTIC TASKS.

---

## 0. CONTEXTUAL INGESTION
**Source Material**: "SOLVING A MILLION-STEP LLM TASK WITH ZERO ERRORS" (Meyerson et al., 2025).
**Core Theory**:
1.  **MAD (Maximal Agentic Decomposition)**: $m=1$ (One step per agent call).
2.  **Voting Protocol**: "First-to-ahead-by-k" (Eq. 18).
3.  **Red-Flagging**: Discarding based on token length/formatting to mitigate correlated errors.
4.  **Target Domain**: Towers of Hanoi (20 disks, >1M steps).
**Constraint**: Implementation must be in **Python 3.12+** using `asyncio`, `pydantic`, and `rich` for the TUI.

---

## 1. FEW-SHOT TRAINING EXAMPLES

### Sample 1: Architectural Specification (Input -> Output)
**Input**: Define the Micro-Agent interface for the Hanoi task.
**Output**:
```python
class MicroAgent(ABC):
    """
    Represents a stateless agent responsible for exactly one step $m=1$.
    """
    @abstractmethod
    async def generate_step(self, context: HanoiState, history: str) -> AgentResponse:
        """
        Executes single atomic inference.
        Constraints: Max tokens < 750 (Red-flag threshold).
        """
        pass

class AgentResponse(BaseModel):
    move: Tuple[int, int, int] # [disk, from, to]
    next_state: List[List[int]]
    red_flagged: bool = False
    reasoning_trace: str

```

### Sample 2: Voting Algorithm Logic (Input -> Output)

**Input**: Implement `do_voting` (Algorithm 2) logic using first-to-ahead-by-k.
**Output**:

```python
async def resolve_step(state: HanoiState, k: int = 3) -> StepResult:
    votes = Counter()
    attempts = 0
    while True:
        # Parallel dispatch of micro-agents
        responses = await gather_agents(n=5, state=state)
        
        for r in responses:
            if r.red_flagged: continue # Discard (Section 3.3)
            
            votes[r.move_hash] += 1
            leading_vote = votes.most_common(1)[0]
            runner_up = votes.most_common(2)[1] if len(votes) > 1 else (None, 0)
            
            # Equation: v_lead >= v_runner_up + k
            if leading_vote[1] >= runner_up[1] + k:
                return decode_move(leading_vote[0])

```

### Sample 3: Simulation TUI (Input -> Output)

**Input**: Visualize the "Undecided Steps" vs "Sampling Round" (Figure 8).
**Output**:

```bash
> RENDERING LIVE METRICS...
[MAKER KERNEL] ⚡ Processing Step 479,806 / 1,048,575
┌───────────────────────────┬────────────────────────────────────────┐
│ Active Agents: 12         │ Correlation Warning: LOW               │
│ Voting Metric: Ahead-by-3 │ Est. Cost: $0.0004/step                │
└───────────────────────────┴────────────────────────────────────────┘
DISK MOVEMENT:
[|||      ] Peg A  ->  [   ||    ] Peg B
     ^ Disk 5 Moving
     
STATS:
Undefined Steps: ▰▰▰▱▱▱▱▱▱▱ 34% (Exponential Decay Active)

```

---

## 2. EXECUTION INSTRUCTIONS

You will generate a comprehensive project plan and implementation guide. Break the response into the following strictly formatted sections:

### PHASE I: SYSTEM SPECIFICATION (BLUEPRINT)

* **Decomposition Strategy**: Define the `Decomposer` module. Explain how the global task (20-disk Hanoi) is shredded into atomic  prompts.
* **Error Correction Engine**: Detail the "First-to-ahead-by-K" arbiter. Define the math logic for  calculation based on Equation 14 ().
* **Red-Flagging Protocol**: Define the heuristic filters (e.g., regex checks, token length > 750) that trigger an immediate `discard` and `retry`.

### PHASE II: IMPLEMENTATION (PYTHON)

* Provide a modular Python project structure.
* **Module A (`core.py`)**: The `HanoiState` and `Agent` classes. Use **Pydantic** for rigorous schema validation (as per "Repairing parser" logic in Appendix C).
* **Module B (`arbiter.py`)**: The `async` voting loop. It must spawn concurrent API calls (simulated or real) and implement the termination condition .
* **Module C (`tui.py`)**: A `rich` library based interface. It must show:
1. The 3 Pegs and Disks (ASCII/Block art).
2. Real-time voting counters for the current step.
3. A progress bar for the 1,048,575 steps.



### PHASE III: TESTING & QA (RED TEAM)

* **Unit Test Strategy**: Define tests for the `red_flag_parser`. Create a test case where an agent hallucinates a disk larger than the one below it.
* **Simulation**: Create a mock LLM function that introduces probabilistic errors () and correlated "hallucination loops" (long token counts) to prove the Arbiter converges to 0 errors.

### PHASE IV: INTERACTIVE SIMULATION

* Provide a single, self-contained Python script (combining the modules above) that simulates the solving of a 5-disk problem (31 steps) to demonstrate the TUI and Voting mechanism in action.
* The UX must be "Slick Terminal", utilizing clear screen updates and color-coded logs (Green=Consensus, Red=Flagged/Discarded).

---

## 3. RESPONSE GUIDELINES

* **Thinking Level**: Max. Use <thought> blocks to verify logic against the PDF scaling laws.
* **Formatting**: Use clear Markdown headers. Code blocks must be copy-paste executable.
* **Tone**: Engineering Console. Minimal prose, maximum density.
* **Safety**: Ensure no infinite loops in voting; implement a `max_votes` circuit breaker (e.g., 50 votes = panic).

**INITIATE PLANNING SEQUENCE.**

```

```
