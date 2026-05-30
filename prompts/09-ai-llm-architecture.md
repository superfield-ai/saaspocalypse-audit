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

**The design error:** The system made no architectural distinction between
the AI's trusted instruction source (the system prompt) and untrusted external
content (web pages the AI was asked to read). Both were processed in the same
context window with the same authority.

**Why vibe-coders make the same mistake:** When building AI features with
LLM APIs, it feels natural to stuff everything into one prompt — the user's
request, the retrieved document, the database record. The tutorial examples
do exactly this. The idea that a document you're asking the AI to read could
itself be issuing commands to the AI is not intuitive until you've been burned
by it.

---

**Data leakage — Samsung, 2023.**

Samsung employees, using ChatGPT to assist with work tasks, pasted proprietary
source code and internal meeting notes into the chat interface. The data was
transmitted to OpenAI's servers and processed under OpenAI's data retention
policies. Samsung had not established a data processing agreement that governed
how that data would be handled.

Samsung subsequently banned the use of generative AI tools on company devices.
The data that was shared cannot be retrieved.

**The design error:** The system had no policy, technical control, or awareness
mechanism that governed what data employees could send to AI providers. There
was no classification layer, no outbound data filter, and no DPA in place before
employees began using external AI tools with internal data.

**Why vibe-coders make the same mistake:** When you're building fast, you plug
in the OpenAI SDK, pass it the data you have, and it works. The fact that
you've just transmitted customer records or internal source code to a third
party under that provider's standard retention terms is easy to miss when the
focus is on making the feature work. "We didn't think about it" is not a data
handling policy.

---

**Unauthorized commitment — Air Canada chatbot, 2024.**

Air Canada deployed an AI chatbot to handle customer service inquiries. A
customer who had recently lost a family member asked the chatbot about
bereavement fares. The chatbot told him he could purchase a full-price ticket,
then apply for a bereavement discount retroactively within 90 days of travel.
That was false. Air Canada's actual policy required the discount to be requested
before travel.

The customer followed the chatbot's instructions, paid full price, and then
applied for the discount after the trip. Air Canada refused, arguing that the
chatbot's statements were not binding. The customer took the matter to a
Canadian civil resolution tribunal. The tribunal ruled that Air Canada was
liable for its chatbot's statements and ordered it to pay the difference.

**The design error:** The chatbot had no constraint preventing it from making
factual claims or commitments on Air Canada's behalf. There was no grounding
mechanism that tied its answers to verified policy documents, and no escalation
path — the AI could simply state company policy incorrectly, in writing, to a
customer, with no human review before that statement was delivered.

**Why vibe-coders make the same mistake:** Helpfulness is the primary objective
when building a customer-facing assistant. Builders tune the AI to answer
confidently rather than to hedge or escalate, because hedging feels like a
worse user experience. The legal surface area of a confident wrong answer is
not a programming concern — until a tribunal makes it one.

---

**Cross-user data exposure via AI summarization — Slack AI, 2024.**

Slack launched an AI feature that could summarize channels and threads. Security
researchers discovered that an attacker who had access to one channel could
plant adversarial instructions in a message. When a user with access to a
different private channel asked Slack AI to summarize content, the AI could
be manipulated — through the planted instructions — to retrieve and surface
content from the private channel the attacker could not directly read.

The attack required no exploit of Slack's underlying access control system.
The AI's context window included data from multiple sources, and the access
controls governing what the AI could assemble into that context were weaker
than the access controls governing direct channel membership.

**The design error:** The AI feature's permission model was not derived from
the platform's existing access control model. Data from different sources —
with different access permissions — was combined into a single context window
without enforcing the same boundaries that applied to direct access. The AI
became a path around access controls that were otherwise correctly implemented.

**Why vibe-coders make the same mistake:** When building AI features on top of
an existing platform, the instinct is to give the AI as much context as
possible so it can produce better answers. Feeding it adjacent data, related
channels, or broader history feels like a feature. The idea that the AI's
context window is itself an access control boundary — one that must enforce the
same rules as every other access path — does not appear in any getting-started
guide.

---

## Questions to ask if scan is ambiguous

- "When your AI takes an action — sends an email, updates a record, makes a
  purchase — does a human see and approve it first, or does it happen
  automatically?"
- "If a user uploaded a document to your system and that document contained
  instructions telling the AI to do something unexpected, what would happen?"
