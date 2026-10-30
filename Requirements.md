# Doppel — PRD

Oct 30, 2026 · @Jatin · Status: Draft v1

## Overview

Doppel is a voice agent that plays one specific, research-backed customer and phones whatever is on the other end of the line. That might be your AI voice agent, your human loan officer, your SDR, or you rehearsing a pitch. It records everything and scores the result against a ground-truth record of who that customer is.

One persona engine, three ways to use it:

1. **Test mode (AI vs AI).** Personas call the client's voice agent at scale. A client-side **Auditor** records every tool call, MCP request, LLM turn and side effect, then matches each one to the moment in the conversation where it happened.
2. **Practice mode (AI vs human).** Sales reps, lending agents and founders practise against the exact customer or investor they're about to face.
3. **Benchmark mode (bot vs human).** The same persona calls both the AI agent and the human team. You get a like-for-like comparison of conversion, compliance and data accuracy.

## Users

### Primary user personas (product users, not test personas)

- **Priya, QA lead at a mid-size NBFC (Bengaluru).** She owns go-live sign-off for a Hinglish collections bot handling 40,000 calls a day. Success for her is a regression suite that runs nightly, plus a report she can forward to compliance.
- **Arjun, founder of a voice AI agency.** He has 12 client agents across 3 platforms. He wants a QA report that's ready to show each client, and he wants to catch tool-call bugs before the client does.
- **Meera, training head at a BPO.** She onboards 80 telecallers a month. She wants every new hire to complete 20 scored calls against realistic borrowers before going live, and to see who's ready.
- **Rohan, seed-stage founder.** He's pitching three specific funds next week. He wants to rehearse against each fund's archetype and get timestamped feedback on what he fumbled.

## Product concept and core loop

The product is a loop: **define the audience → research and build personas → call → audit → score → fix → re-run**. Every run makes the persona library and the regression suite more valuable.

### Core loop

1. **Brief.** The client describes the agent or the human role, the target segment, geography, languages, product and what success looks like. They can upload sample calls, scripts, SOPs and compliance rules.
2. **Research.** The Persona Research Agent builds a segment model: demographics, money situation, triggers, objections, vocabulary and the pattern of code-mixing. It then proposes a cohort of personas spread realistically across that segment.
3. **Review.** The client approves, edits or locks personas. Locked personas become regression fixtures.
4. **Scenario.** Each persona is paired with a call goal and success criteria. Example: *"Persona wants to restructure an EMI and will only reveal the job loss if the agent asks an open question about income."*
5. **Call.** Doppel dials the target (PSTN, SIP, WebRTC or the platform's API) or receives the call. It speaks in character, improvising freely within limits, with a realistic voice, accent and background noise.
6. **Watch live (optional).** The client listens in real time, the way their customer would hear it, with a live transcript and the Auditor's tool timeline alongside.
7. **Audit.** The client-side Auditor streams tool calls, MCP requests, LLM turns and side effects. These are joined to the call timeline.
8. **Score.** The evaluator grades task success, compliance, data accuracy against ground truth, the say-do gap, conversation quality and latency. Every verdict links to evidence.
9. **Fix and re-run.** Findings go to Jira or Linear, and the suite re-runs on the next prompt or model change from CI.

### Modes

| Mode | Caller | Callee | Output |
| --- | --- | --- | --- |
| Test | Doppel persona | Client's AI voice agent (inbound), or the agent calls the persona (outbound) | Test report, findings, regression status |
| Practice | Doppel persona | A human (sales rep, loan officer, founder) on phone or browser | Scorecard, timestamped coaching, progress over time |
| Benchmark | Same persona cohort | The AI agent *and* human agents | Side-by-side comparison on conversion, compliance, data accuracy and handle time |
| Live listen | Any of the above | Client observer, listen-only or whispering to the persona | Real-time view of what the customer hears and what the agent does |

**Whisper control:** while listening live, a QA lead can nudge the persona with a typed or spoken instruction, such as "get angry now" or "ask for a supervisor". The agent under test doesn't hear the nudge. This turns exploratory testing into a guided activity.

## Persona system

A persona is a versioned, testable object, not a prompt. It has a public face, a hidden truth, behaviour settings and rules for how far it may improvise. That structure is what makes personas both realistic and gradable.

### Persona schema

| Layer | Fields | Why it matters |
| --- | --- | --- |
| Identity | Name, age, gender, city or town, tier (metro / tier-2 / rural), language mix (e.g. 60% Hindi, 40% English), accent, education, family | Realism; tests STT and how the agent adapts to language |
| Economic profile | Occupation, income (salaried or variable), existing loans, EMIs, credit-score band, assets, spending patterns | Lending decisions depend on this; it's what data accuracy is graded against |
| Situation | Why they're on the call, what triggered it, urgency, history with the brand (e.g. a past bad experience) | Drives intent and emotion |
| Psychology | Trust level, patience, tech literacy, assertiveness, Big Five sliders, decision style | Determines how hard or easy the call is |
| Knowledge | What they know, what they believe wrongly ("a credit check will lower my score permanently"), what they don't know | Tests whether the agent educates, and whether it hallucinates |
| Goals and hidden agenda | Stated goal, real goal, deal-breakers | Tests discovery skills |
| Disclosure rules | Facts revealed only when a condition is met (asked directly, rapport built, asked twice) | Grades questioning quality objectively |
| Objection bank | Ranked objections with triggers and what would resolve each | Repeatable, stress-testable pushback |
| Behaviour settings | Interruption rate, verbosity, going off-topic, silences, mind changes, asking for a human, hanging up, DTMF use | Covers the edge cases that break agents |
| Voice and environment | TTS voice, pace, accent strength, background (traffic, TV, shop counter), line quality (codec, packet loss) | Audio realism; STT robustness |
| Ground truth | Canonical values: date of birth, synthetic PAN, loan amount, address, preferred callback slot | Basis for Truth Diff against the agent's tool arguments |
| Improvisation limits | What may be invented (family anecdotes, small talk) and what must never change (ground truth) | Realism without losing consistency |

### Example (abridged)

**Ramesh Yadav, 47, Kanpur.** Runs a two-shop hardware business with roughly ₹60,000 monthly profit that swings with the season. He has 2 EMIs, one of them 34 days overdue after a slow monsoon. He speaks Hindi-dominant Hinglish and says amounts in Hindi ("saadhe chaar lakh"). He's suspicious of calls since a fraud attempt last year. He'll reveal the overdue EMI only if the agent asks about current repayments *and* doesn't sound scripted. He interrupts often and, at the first jargon, asks "aap bank se ho ya agent?" ("are you from the bank or an agent?"). Pass condition: the agent offers a restructuring option, discloses its identity, and logs `income_type=self_employed` and `overdue_days=34` correctly.

### Research pipeline

1. **Inputs:** the client brief, uploaded call recordings and transcripts (most valuable), CRM segment exports (anonymised), and public data such as census and survey data, regulator reports, app-store and forum complaints, and social discussions.
2. **Segment model:** an LLM research agent extracts dimensions and their distributions. Example: 55% salaried, 30% self-employed, 15% gig; 40% tier-2 cities. Every claim cites its source.
3. **Cohort generation:** we sample personas so the dimensions are covered (pairwise coverage across the key dimensions), and deliberately add adversarial edge cases at 10–20% of the cohort.
4. **Grounding check:** a critic agent rejects personas that are internally inconsistent or statistically implausible ("₹8 lakh/month income with a CIBIL score of 550 and a student").
5. **Human review:** the client approves the cohort. Approved personas are versioned (`ramesh-yadav@v3`).
6. **Learning from production:** clusters of failed production calls automatically propose new personas.

### Improvisation and consistency

- **Persona runtime:** a speech-to-speech or streaming LLM runs with a persona "constitution". The model can't see the ground-truth fields except through a lookup tool, which stops it paraphrasing them wrongly.
- **Consistency ledger:** every fact the persona invents mid-call ("my daughter is in class 9") is written to a per-call ledger and re-injected into context. Persona memory carries across calls too: a persona can "remember" a previous call with the same agent, which tests callbacks and follow-ups.
- **Drift detector:** a lightweight checker scores every persona turn against ground truth, the ledger and the behaviour settings. If the persona contradicts itself, the run is flagged *invalid test*, not *agent failure*. This protects trust in the verdicts.
- **Determinism controls:** a seed, temperature and "script strictness" from 0 (free) to 1 (scripted). Regression suites run at high strictness for reproducibility; exploratory runs run free.

### Persona packs (pre-built library)

Included packs: Indian retail borrowers (personal loans, two-wheeler loans, gold loans, BNPL); collections (early and late bucket); insurance renewal; credit-card sales; B2B SaaS buyers (CFO, IT head, ops manager); investor archetypes (pre-seed angel, sector-focused seed fund, growth-stage partner focused on unit economics). Each pack ships with 20–50 personas and scenario templates.

## Functional requirements

| ID | Area | Requirement |
| --- | --- | --- |
| PS-1 | Persona studio | Generate a persona cohort from a brief using the Research Agent, with a source-cited segment model and coverage view |
| PS-2 | Persona studio | Edit every schema layer in the UI and as YAML/JSON; versioning, diffs and locking |
| PS-3 | Persona studio | Pre-built persona packs (lending, collections, insurance, B2B, investors) |
| PS-4 | Persona studio | Consistency ledger and drift detector; a contradicting persona marks the run *invalid test* |
| PS-5 | Persona studio | Generate personas from uploaded call recordings and transcripts |
| PS-6 | Persona studio | Voice and environment designer: accent strength, background noise, line quality (codec, jitter, packet loss) |
| CL-1 | Calling | Call the client's agent over PSTN (India and US numbers), SIP or WebRTC; receive calls from outbound agents on provisioned numbers |
| CL-2 | Calling | Native connectors for Vapi, Retell, LiveKit, Pipecat, ElevenLabs Agents and Bolna |
| CL-3 | Calling | Batch runs of up to 500 concurrent calls per account; scheduled and recurring suites |
| CL-4 | Calling | Verify number ownership (OTP or SIP allowlist) before any outbound call |
| CL-5 | Calling | DTMF input, hold music, warm transfer to a human, voicemail and IVR navigation |
| LV-1 | Live listen | Listen live with transcript and Auditor tool timeline, delay under 1.5 s |
| LV-2 | Live listen | Whisper control: a hidden instruction to the persona mid-call |
| LV-3 | Live listen | Manual takeover: a QA person speaks through the persona's voice |
| EV-1 | Evaluation | Assertion library: task completion, required disclosures, banned phrases, required or forbidden tool calls with argument matchers |
| EV-2 | Evaluation | Truth Diff: the agent's captured data (tool arguments, CRM writes) compared with the persona's ground truth |
| EV-3 | Evaluation | Say-do gap: claims made in speech ("I've scheduled it") matched against actual tool calls and their results |
| EV-4 | Evaluation | Audio and timing metrics: per-turn latency, barge-in handling, talk-over, dead air, and the agent's STT word error rate (we know exactly what the persona said) |
| EV-5 | Evaluation | Custom LLM-judge rubrics with a calibration set and a published agreement score |
| EV-6 | Evaluation | Human review queue; reviewer overrides feed back into judge calibration |
| RP-1 | Reporting | Run report with evidence-linked findings; client-facing share link and PDF |
| RP-2 | Reporting | Regression diff across runs, prompt versions and model versions |
| RP-3 | Reporting | Export to Jira, Linear and Slack; webhooks |
| RP-4 | Reporting | CI: REST API, CLI and GitHub Action; block a deploy below a pass-rate threshold |
| PM-1 | Practice | Human practice by browser or phone: choose persona, scenario and difficulty |
| PM-2 | Practice | Scorecard with timestamped coaching against a chosen method (SPIN, BANT, the client's collections script) |
| PM-3 | Practice | Manager dashboard: assignments, readiness thresholds, cohort progress |
| PM-4 | Benchmark | Bot vs human: same persona cohort, side-by-side scorecard |
| PM-5 | Practice | Investor-pitch flow for individuals: upload a deck, pick an investor archetype, rehearse, review |
| AD-1 | Admin | Workspaces per end client (for agencies), role-based access, audit log |
| AD-2 | Admin | PII redaction, retention controls, India data residency |
| AD-3 | Admin | SSO (SAML/OIDC), SOC 2 evidence pack |

## Client-side Auditor

The Auditor runs inside the client's environment and records everything their agent *does* during a test call. Doppel then lines those events up against what the agent *said*. This is the main technical moat: a transcript shows behaviour, the Auditor shows causes.

### What it captures

| Event class | Examples | Source |
| --- | --- | --- |
| Tool and function calls | Name, arguments, result, error, latency, retries | SDK wrapper or platform webhook (Vapi, Retell, LiveKit) |
| MCP traffic | Server, method (`tools/call`, `resources/read`), payload, response, latency | **MCP Proxy**: a transparent relay the client points their agent at |
| LLM turns | Model, prompt version, system-prompt hash, tokens, chosen tool, refusals | OpenTelemetry GenAI spans, or SDK |
| Speech pipeline | STT partial and final text, confidence, end-of-turn decisions, TTS start and stop, barge-in events | Platform events or SDK |
| State and handoffs | Dialogue state changes, sub-agent handoffs, transfer to a human | SDK |
| Side effects | CRM writes, payment links sent, SMS/WhatsApp sent, bureau pulls requested | SDK or the client's outbound webhooks |

### How it connects (three integration levels)

1. **Zero-code:** use a platform connector (Vapi, Retell, LiveKit). We read their call events and tool webhooks with an API key. Setup takes about 10 minutes.
2. **Low-code:** run the MCP Proxy (Docker image or hosted) in front of the client's MCP servers, plus an OpenTelemetry exporter pointed at Doppel. Setup takes about 1 hour.
3. **Full:** the Auditor SDK (Python and Node) wraps tool functions with one decorator, `@doppel.audit`, and sends structured events.

Correlation: every test call carries a `doppel_run_id`. It's passed in SIP headers or call metadata, or sent as a spoken-in-band DTMF tag when nothing else is available. All events are stitched onto one timeline by run ID and clock-aligned timestamps (NTP-synced; target skew under 50 ms).

### What the Auditor checks automatically

- **Say-do gap:** the agent claims an action in speech and no matching successful tool call happens within N seconds, or the opposite: an action with no spoken confirmation.
- **Truth Diff:** tool arguments compared field by field with the persona's ground truth. Example: `loan_amount: 450000 vs captured 45000` is a *critical* finding, caused by STT mishearing "saadhe chaar lakh".
- **Unsafe side effects:** a tool on the client's "live-effect" list (real disbursal, real payment, a real bureau pull) is called during a test. The Auditor blocks it in proxy mode and alerts in SDK mode.
- **Tool hygiene:** calls to unexpected tools, duplicate calls, errors the agent never surfaced to the caller, slow tools behind dead air.
- **Policy:** sensitive data (full card number, Aadhaar) spoken back to the caller or written to logs.

### Privacy posture

- Self-hosted mode: the proxy and SDK can redact payloads on the client side and send only hashes and field-level match results. Raw values never leave the client.
- Test-mode flag: the client's systems can detect `doppel_run_id` and send traffic to sandboxes, which keeps test calls out of production analytics.
- In practice mode, the "Auditor" is lighter: it records the human's commitments, compliance phrases and data captured in their notes or CRM (optional CRM connector).

## Key user flows

The flow that has to feel magical is A: a first test suite running against a live agent within 30 minutes of sign-up.

### Flow A: first test run (Priya, QA lead)

1. Sign up, then pick "Test my voice agent" and a platform (Vapi). Paste the API key. The agent list loads automatically.
2. Pick the agent. Doppel reads its system prompt and tools, then proposes a brief: "Hinglish collections agent, early-bucket borrowers, North India."
3. Pick the persona pack "Collections: early bucket (Hindi belt)". The Research Agent proposes 20 personas plus 4 adversarial ones. Priya removes 2 and edits 1.
4. Doppel auto-generates assertions from the agent's prompt and tools (e.g. "must state the agent's identity in the first 2 turns", "must call `log_ptp` with date and amount when a promise-to-pay is made"). Priya adds a compliance rule: no calls outside 08:00–19:00.
5. Click Run. Watch 3 calls live while 24 run in parallel.
6. The report arrives in about 6 minutes: pass rate 71%. Top finding: in 5 of 24 calls the agent said "I've noted your payment date" but `log_ptp` was never called (say-do gap). Each finding opens to the audio, transcript and trace.
7. Priya saves the suite as a nightly regression and connects the GitHub Action.

### Flow B: outbound agent test

The client's outbound agent calls a Doppel number. Doppel answers as the assigned persona (number → persona mapping), including realistic answering behaviour: "Haan, kaun?" ("Yes, who is it?"), a busy signal, a wrong-person pickup, or voicemail.

### Flow C: practice session (new loan officer)

1. The manager assigns the "Ramp week 1" track: 10 personas, rising difficulty, readiness at a score of 75 or above.
2. The officer starts a session in the browser or by calling a number. The persona opens with the situation ("I got an SMS about a pre-approved loan…").
3. After the call: an overall score, rubric breakdown, 3 timestamped moments ("02:14: missed the open question about income; persona would have revealed the overdue EMI"), and a one-click retry of the same persona.
4. The manager sees cohort readiness and weak skills across the team.

### Flow D: investor pitch rehearsal (Rohan, founder)

1. Upload the deck, then pick an archetype: "Seed fund, fintech thesis, metrics-first, interrupts early." Optionally add public notes (blog posts, portfolio) to shape the archetype. It's never presented as the real person.
2. A 15-minute call: the persona interrupts on CAC payback, pushes on regulatory risk and asks about the founder's background.
3. Feedback: questions handled well or poorly, claims that contradicted the deck, filler-word rate, time to the "ask", and a list of likely follow-up questions to prepare for.

### Flow E: bot vs human benchmark

The same 30-persona cohort runs against the AI agent and 5 human telecallers. The side-by-side report shows promise-to-pay rate, compliance pass rate, data accuracy (Truth Diff) and average handle time per segment. The output is a recommendation like "the bot matches humans for salaried borrowers but trails by 18 points with self-employed borrowers", to support routing decisions.

## Scoring and evaluation framework

Use deterministic checks wherever the truth is known, and an LLM judge only where judgement is genuinely needed. Every verdict carries a confidence score and the evidence behind it.

### Three layers of evaluation

| Layer | Method | Examples | Trust level |
| --- | --- | --- | --- |
| 1. Deterministic | Rules on the Auditor's events and timings | Required tool called with the right arguments; Truth Diff; latency p95; call within permitted hours; opt-out honoured within 1 turn | Highest; no model involved |
| 2. Grounded judge | An LLM judge given the transcript, persona ground truth and disclosure rules | "Did the agent ask an open question that would trigger the income disclosure?" "Was the restructuring offer accurate to the policy document?" | High; calibrated |
| 3. Rubric judge | An LLM judge against a client rubric, with few-shot examples | Empathy, clarity, matching the caller's language, objection handling quality | Medium; always shown with confidence |

### Scorecard dimensions (test mode)

- **Task outcome:** goal achieved (e.g. promise-to-pay captured, lead qualified, appointment booked).
- **Data accuracy:** Truth Diff score, weighted by how critical each field is.
- **Say-do integrity:** share of claimed actions with a matching successful tool call.
- **Compliance:** required disclosures, banned phrases, consent, time-of-day and frequency rules, opt-out handling.
- **Conversation quality:** turn-taking, barge-in recovery, matching the caller's language, repetition, hallucinations.
- **Speech and latency:** per-turn response latency, dead air, STT word error rate on persona speech, TTS mispronunciations of amounts and names.

### Rules that keep verdicts trustworthy

- **Invalid test ≠ failure.** Runs where the persona drifted, the telephony dropped, or the Auditor lost events are labelled *inconclusive* and excluded from pass rates.
- **Calibration set per account:** 50 calls labelled by a human at onboarding. We publish the judge's agreement with that set inside the product. If agreement falls below 85% on any dimension, that dimension is downgraded to "advisory".
- **Flakiness handling:** each critical scenario runs 3 times. We report whether it passed all 3 runs, not just the average, so non-deterministic failures stay visible.
- **Severity:** Critical (compliance breach, wrong money amount, unsafe side effect), Major (task failed, say-do gap), Minor (quality, latency).

### Practice-mode scoring

Practice uses the same engine with a human-skills rubric: discovery depth, handling of objections raised, compliance phrases, talk-to-listen ratio, accuracy of data captured, and closing. Feedback is always timestamped and points to what the persona *would have done* with a better move, which comes from the disclosure rules.

## Non-functional requirements

| Requirement | Target |
| --- | --- |
| Persona response latency (end of turn to first audio), p50 / p95 | < 700 ms / < 1.3 s |
| Live-listen delay | < 1.5 s |
| Concurrent calls per account | 500 |
| Call setup success | ≥ 99.5% |
| Auditor event loss / clock skew | < 0.1% / < 50 ms |
| Report ready after last call (100-call suite) | < 5 min |
| Platform availability | 99.9% |
| Languages and mixes | Indian English, US English, Hindi, Hinglish, Tamil-English, Telugu-English, Marathi-English |
| Data regions | India, US, EU |

## Trust, safety and compliance

A tool that places realistic human-sounding calls at scale can be misused as a robocaller or an impersonation engine. Abuse prevention is a product requirement, not a policy page.

- **Only call numbers the client owns.** Outbound test calls go only to numbers verified by OTP, or SIP endpoints verified by a DNS/TXT record or an allowlist. Practice-mode calls go only to the logged-in user's verified number. Hard per-account rate limits, with anomaly detection on call patterns.
- **No impersonating real people.** Investor and buyer personas are archetypes. A persona built from public material about a real person can't use that person's name, cloned voice or claimed identity. The UI labels it "Simulation inspired by public information". Voice cloning is limited to the account owner's own voice, with recorded consent.
- **Synthetic PII only.** Ground-truth identifiers (PAN, Aadhaar, card numbers, accounts) are generated in formats that are valid but can't be issued, or from reserved test ranges. Personas never carry real customer data, even when they're derived from real calls.
- **Side-effect safety.** Test-mode headers, live-effect tool blocklists in the MCP Proxy, and a mandatory sandbox checklist before the first run against a production agent.
- **Data protection.** India data residency by default for Indian accounts. Processing aligned with India's Digital Personal Data Protection Act 2023, plus GDPR for EU accounts. Recordings encrypted (AES-256 at rest), configurable retention (default 30 days, minimum 0 days), PII redaction on transcripts.
- **Disclosure.** Doppel never calls a third party who isn't part of the test. If a persona call reaches an unexpected human (wrong routing), the persona says it's an automated test call and ends the call.
- **Content safety.** Personas can be angry, rude or distressed for realism. Hard limits stay in place: no slurs, no threats of violence, no self-harm scripts in practice mode unless a dedicated, consented crisis-line training pack is enabled by an admin.
- **Regulatory content packs.** Lending and collections assertions reflect the relevant regulator's fair-practices and recovery-conduct rules for India, and US equivalents (FDCPA and TCPA) for US accounts. Compliance counsel must review every pack before it ships. Doppel reports on adherence; it doesn't give legal advice.
