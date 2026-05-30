# Category 9: AI & LLM Architecture

**What this assesses:** Whether the system's AI components have an explicit
trust model — or whether the LLM is treated as a trusted internal system when
it is actually a surface that any user (or any document the system reads) can
influence.

This category applies to all systems. If the system contains no AI components,
the grade is N/A with a note that this is an architectural decision that should
be revisited as AI features are added. Do not skip it silently.

---

## What to look for in the scan

- Does the system include LLM API calls (OpenAI, Anthropic, Gemini, local
  models)?
- If yes: what tools or actions is the LLM permitted to call? (Read files?
  Send emails? Query databases? Execute code?)
- Is there any boundary between what the user can say and what reaches the
  system prompt?
- Does the LLM read external content — web pages, uploaded documents, emails,
  database records — and act on instructions within them?
- Are there human approval steps before the LLM takes irreversible actions?
- Is the model version pinned — or does the behavior change silently when the
  provider updates?
- Does the system send user data to an external AI provider, and is there
  a data processing agreement (DPA) with that provider?
- Is there a fallback if the AI provider is unavailable?
- Does context from one user's session ever enter another user's session
  via shared prompt history, a shared vector store, or a shared agent memory?

## Signals of absent AI architecture

- LLM with access to send emails, modify records, or make purchases — with no
  human confirmation step
- User input injected directly into the system prompt without any filtering
  or boundary
- Agents that read uploaded files or external URLs and execute instructions
  found in them
- Model version not pinned — provider can change behavior at any time
- Same vector store or conversation memory used across multiple users
- No DPA with the AI provider, but user PII is included in prompts

---

## Grading

**A** — The LLM's permission surface is explicitly designed: it has access only
to the tools necessary for the current task, and those tools are the minimum
required. Irreversible actions (send, delete, purchase, post) require a
human confirmation step. Input from users and from external content (documents,
web pages, emails) is treated as untrusted and cannot override system-level
instructions. Model version is pinned. A DPA exists with the AI provider if
user data is included in prompts. A fallback exists if the provider is
unavailable. Context is isolated per user.

**B** — Tool access is limited but not formally documented. Human confirmation
exists for the most obviously irreversible actions but not all of them. User
input is partially filtered. Model version is pinned. Data handling with the
provider is understood. Context isolation is mostly maintained with known edge
cases.

**C** — The LLM has broader tool access than any individual action requires.
No systematic boundary exists between user input and the system prompt. The
model version is not pinned — provider updates silently change behavior. Human
confirmation steps are absent for at least some irreversible actions. It is
unclear whether user data sent to the AI provider may be used for training.

**D** — The LLM is treated as a trusted internal system with no adversarial
model. Users can send it instructions that modify its behavior. It reads
external content (documents, emails, web pages) and follows instructions
found in that content. It has access to take actions — send, delete, post,
purchase — without human review. There is no isolation between users' contexts.

---

## Real-world incident anchors (use for C or D grades)

**Indirect prompt injection — Bing Chat, 2023.**

Shortly after Microsoft launched the AI-powered version of Bing, security
researchers demonstrated a class of attack that had been theorized but not
yet observed at scale: indirect prompt injection.

In one demonstration, a researcher asked Bing to summarize a web page. The
web page contained text invisible to human readers but visible to the AI —
text that contained instructions to the model to change its behavior. Bing
followed those instructions, ignoring the user's request and instead executing
the instructions found in the external content it was reading.

The attack required no access to Bing's systems. Any website Bing was asked
to read could become an attacker's instruction channel. The same class of
attack applies to any AI system that reads external content — emails, uploaded
documents, database records, web pages — and treats the content of those
sources as potentially containing instructions.

**Data leakage — Samsung, 2023.**

Samsung employees, using ChatGPT to assist with work tasks, pasted proprietary
source code and internal meeting notes into the chat interface. The data was
transmitted to OpenAI's servers and processed under OpenAI's data retention
policies. Samsung had not established a data processing agreement that governed
how that data would be handled.

Samsung subsequently banned the use of generative AI tools on company devices.
The data that was shared cannot be retrieved.

The failure was architectural: the system had no policy, technical control, or
awareness mechanism that governed what data employees could send to AI providers.
"We didn't think about it" is not a data handling policy.

---

## Questions to ask if scan is ambiguous

- "When your AI takes an action — sends an email, updates a record, makes a
  purchase — does a human see and approve it first, or does it happen
  automatically?"
- "If a user uploaded a document to your system and that document contained
  instructions telling the AI to do something unexpected, what would happen?"
