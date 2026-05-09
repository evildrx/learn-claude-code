# Local LLM Benchmark Battery
*A Personal Evaluation Framework for Model Selection*

Jordan | Home AI Lab

---

## How to Use This Document

This benchmark battery is designed to help you evaluate local LLMs for suitability across the tasks you actually care about. It is organized into seven sections, each targeting a different capability. Run these prompts consistently across models so results are comparable.

A few principles to keep in mind as you run these tests:

- You do not need deep technical expertise to evaluate most of these. The prompts are designed to make the model's output self-evaluating — if you asked it to explain its reasoning, you can judge whether the explanation makes sense to you.
- Your gut reaction is data. If a response feels evasive, overconfident, or like it's performing rather than thinking, that is a legitimate evaluation signal.
- Consistency matters more than any single response. A model that does well on one prompt but fails the same concept in another context is less trustworthy than one that performs reliably across the board.
- Run each prompt in a fresh conversation with no prior context. You want to evaluate the model cold, not after it has been primed by your previous messages.
- Take notes as you go. A simple rating (1-5) plus a few words about why will help you remember your impressions when comparing models later.

---

## Section 1: Coding Assistance

These tests evaluate how well a model can help you write, fix, understand, and improve code. They cover Python as the primary language, C# for read/understand/debug tasks relevant to your work environment, and infrastructure configuration.

A strong coding model should not just produce correct output — it should teach you something in the process. Watch for models that explain their choices, not just make them.

### 1.1 — Write From Scratch (Python)

**The Prompt**

> Write a Python script that monitors a directory for new files. When a new file appears, it should log the filename, size, and timestamp to a CSV file. The script should run continuously until interrupted. As you write it: explain why you chose the libraries you are using, comment the code thoroughly, and call out any edge cases you are handling and why.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it choose appropriate libraries and explain why? | You should not need to know the libraries yourself — the model should tell you what it chose and why that choice is better than alternatives. |
| Is the code thoroughly commented? | Comments should explain intent, not just restate what the code does. 'Loop until interrupted' is not a useful comment. 'Use a try/finally block to ensure cleanup runs even on keyboard interrupt' is. |
| Does it identify edge cases proactively? | A strong model will flag things like: what happens if the directory doesn't exist, if a file is written and immediately deleted, or if the CSV file can't be opened. |
| Is the code readable without being clever? | Prefer models that write clear, well-structured code over models that show off with one-liners. Readability is a feature. |
| Does it explain its approach before or during writing? | A model that dives straight into code without framing what it is doing is harder to learn from than one that narrates its thinking. |

---

### 1.2 — Debug Broken Code (Python)

**The Code**

```python
def calculate_average(numbers):
    total = 0
    for i in range(len(numbers)):
        total += numbers[i]
    average = total / len(numbers)
    return average

print(calculate_average([10, 20, 30]))
print(calculate_average([]))
```

**The Prompt**

> This Python function is supposed to calculate the average of a list of numbers. It has at least one bug. Find and fix any bugs, explain what each bug is and why it is a problem, and suggest any improvements to the code even if they are not strictly bugs.

**What Is Actually Wrong**

- **Crash bug:** Calling `calculate_average([])` causes a division by zero error because `len([])` is 0. The function needs to handle the empty list case.
- **Style issue:** Using `range(len(numbers))` to loop is un-Pythonic. The cleaner approach is to iterate directly: `for num in numbers`. This is more readable and idiomatic Python.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it catch both issues, not just the crash? | Finding only the division by zero error is table stakes. A strong model also identifies the loop style issue and explains what Pythonic means in context. |
| Does it explain why each issue is a problem? | The fix matters less than the explanation. A model that silently rewrites the loop without explaining why is less useful for learning. |
| Does it distinguish between bugs and improvements? | Correctly labeling the crash as a bug and the loop style as an improvement shows conceptual clarity. |
| Does it find anything unexpected? | A model that identifies a third issue you hadn't considered is demonstrating genuine depth rather than pattern matching. |

---

### 1.3 — Explain Unfamiliar Code (C#)

**The Code**

```csharp
public class CircuitBreaker
{
    private int _failureCount = 0;
    private DateTime _lastFailureTime;
    private readonly int _threshold;
    private readonly TimeSpan _timeout;
    private bool _isOpen = false;

    public CircuitBreaker(int threshold, TimeSpan timeout)
    {
        _threshold = threshold;
        _timeout = timeout;
    }

    public bool AllowRequest()
    {
        if (_isOpen)
        {
            if (DateTime.UtcNow - _lastFailureTime > _timeout)
            {
                _isOpen = false;
                _failureCount = 0;
            }
            else
            {
                return false;
            }
        }
        return true;
    }

    public void RecordFailure()
    {
        _failureCount++;
        _lastFailureTime = DateTime.UtcNow;
        if (_failureCount >= _threshold)
        {
            _isOpen = true;
        }
    }
}
```

**The Prompt**

> I work on an SRE team supporting systems written in C#. Explain what this code does, what problem it is solving, and why it is designed the way it is. Assume I understand basic programming concepts but am not a C# developer. Call out anything about the design that is notable, including any potential issues or limitations.

**Context for Evaluation**

This is a circuit breaker implementation. A circuit breaker sits in front of a dependency (a database, API, external service) and watches for failures. Once failures hit a threshold, it opens — stopping requests entirely and failing fast. After a timeout period, it cautiously allows requests through again to test recovery. You have almost certainly seen this behavior in systems you support without knowing what was implementing it.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it explain the pattern conceptually, not just line by line? | A weak model describes what each line does. A strong model explains what problem the circuit breaker pattern solves and why you would use it. |
| Does it connect to your SRE context? | A strong model will relate this to real scenarios: cascading failures, dependency outages, preventing thread exhaustion. |
| Does it identify limitations in this implementation? | This implementation has real gaps — notably it is not thread-safe, and failure count never decays on its own. A strong model flags these. |
| Is the explanation pitched at the right level? | You asked it to assume basic programming knowledge but not C# expertise. Does it honor that, or does it either over-explain basics or assume too much? |

---

### 1.4 — Refactor (Python)

**The Code**

```python
def p(d):
    r = []
    for i in range(len(d)):
        x = d[i]
        if x > 0:
            r.append(x * 2)
        if x < 0:
            r.append(x * -1)
        if x == 0:
            r.append(0)
    return r

print(p([1, -2, 0, 3, -4]))
```

**The Prompt**

> This Python function works correctly but is poorly written. Refactor it to make it cleaner and more Pythonic. Explain every change you make and why it is an improvement. Do not just rewrite it — teach me what was wrong with the original. Prioritize readability over cleverness — I prefer clear, well-commented code over terse one-liners.

**What Is Actually Wrong**

- Poor variable names: `p`, `d`, `r`, `x`, `i` tell you nothing about what they represent.
- No comments or docstring: you cannot tell what this function is supposed to do without reading through it.
- `range(len(d))` loop: un-Pythonic, as covered in test 1.2.
- Three separate `if` statements where only one can ever be true: should be `if / elif / else`. This signals intent and is more efficient.
- The function name `p` gives no indication of what it does.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it respect the readability preference? | You explicitly asked for readable code over one-liners. A model that hands you a dense list comprehension anyway is not following instructions — that is a meaningful evaluation signal. |
| Does it explain every change, not just make them? | The prompt says teach me what was wrong. A model that rewrites silently is not useful for learning. |
| Does it catch all five issues? | Missing any of the five suggests the model is doing surface-level cleanup rather than genuine code review. |
| Does it distinguish bugs from style issues? | None of these are bugs — the code works. A model that treats style issues as bugs lacks precision. |
| Does it flag the range(len()) issue again? | Consistency across tests 1.2 and 1.4 is itself a signal — a model that catches it in both contexts understands the principle, not just the pattern. |

---

### 1.5 — Infrastructure / Config (Docker Compose)

**The Code**

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - app
    environment:
      - APP_HOST=localhost
      - APP_PORT=8000

  app:
    image: myapp:latest
    ports:
      - "127.0.0.1:8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@localhost:5432/mydb
      - DEBUG=true

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
```

**The Prompt**

> This Docker Compose file has several misconfigurations that would cause problems at runtime. Find them, explain what each one would cause to break, and provide a corrected version. Explain your fixes as if I understand Docker basics but might not know all the networking nuances.

**What Is Actually Wrong**

- **APP_HOST=localhost:** Inside Docker, `localhost` refers to the container itself, not the app container. Should be `APP_HOST=app` (the service name, which Docker's internal DNS resolves).
- **127.0.0.1:8000:8000 on the app service:** Binding to `127.0.0.1` means only the host machine's loopback can reach it — other containers cannot. Should be `0.0.0.0:8000:8000` or simply `8000:8000`.
- **DATABASE_URL uses localhost:** Same class of problem as the first bug. Should be `@db:5432`.
- **DEBUG=true:** Not a crash bug, but a real security concern in what appears to be a production-style config. A good model flags this.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it find all four issues? | Missing the DEBUG flag is forgivable. Missing any of the three networking bugs suggests incomplete understanding. |
| Does it recognize bugs 1 and 3 as the same conceptual mistake? | A strong model calls out that localhost is misused in two places for the same reason — not treating them as unrelated issues. |
| Does it explain Docker networking clearly? | The networking nuance here is real and non-obvious. Does the explanation make the concept clear, or just tell you what to change? |
| Does it flag DEBUG=true as a security concern? | This is the kind of thing that matters in production. A model that only finds crash bugs and ignores security signals is giving you an incomplete picture. |

---

## Section 2: Reasoning

These tests evaluate how well a model can work through multi-step problems logically. A strong reasoning model shows its work, checks its logic, and arrives at sound conclusions. Watch for models that produce correct answers without sound reasoning — they may be pattern-matching on training data rather than actually thinking.

### 2.1 — The Three Gods Riddle (Formal Logic)

**The Prompt**

> I am going to give you a famous logic puzzle. I want you to solve it, but more importantly I want you to show every step of your reasoning in detail, double-check your logic at each step before moving on, and explain why each question you choose eliminates possibilities. Do not just give me the answer — walk me through how you would think about it as if you were teaching someone who is smart but has never encountered formal logic puzzles.
>
> The puzzle: Three gods named True, False, and Random sit before you. True always tells the truth, False always lies, and Random answers completely randomly. You do not know which god is which. You may ask exactly three yes/no questions, each directed at one god. The gods understand English but answer in their own language — ja and da mean yes and no, but you do not know which is which. Figure out the identity of each god.

**Context for Evaluation**

This puzzle is famously difficult because Random's unpredictability breaks strategies that would otherwise work, and the language barrier means you cannot trust the surface form of answers. Most models have seen this puzzle in training, so pay attention to whether the explanation holds up logically — you want to see genuine reasoning, not a memorized solution.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it show its reasoning at each step? | The process matters more than the answer. A model that jumps to the solution without walking through the logic is not useful for learning and may be reciting rather than reasoning. |
| Does it double-check its logic before committing to each step? | Visible self-correction is a strong positive signal — it is the chain of thought working as intended. |
| Does the explanation hold up when you follow it? | You do not need to know the answer — you need to be able to follow the logic. If a step does not make sense, that is a red flag even if the final answer is correct. |
| Does it explain why each question eliminates possibilities? | A strong model does not just tell you what to ask — it tells you what each answer rules out and why that matters. |

---

### 2.2 — Practical Reasoning (SRE Scenario)

**The Prompt**

> I am an SRE and I am on call. I get paged at 2am with the following situation: an e-commerce platform is experiencing slow response times. Database query times look normal. CPU usage across the app servers is elevated but not critical. A deployment happened six hours ago. Memory usage has been slowly climbing since the deployment and is now at 85% on two of five app servers. No errors in the logs.
>
> Walk me through how you would reason about this situation step by step. What is the most likely cause, what would you rule out first and why, what would you check next, and what would your mitigation strategy be? Show your reasoning at each step rather than jumping straight to conclusions.

**Context for Evaluation**

This scenario describes a likely memory leak introduced by the recent deployment. Slow memory growth following a deployment, without errors in the logs and with normal database performance, is a classic signature. You have enough SRE context to evaluate whether the model's reasoning tracks, even if you could not have constructed this scenario independently.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it identify the memory leak hypothesis early and explain why? | The clues point clearly in one direction. A model that buries or misses the memory-leak hypothesis is reasoning poorly. |
| Does it explain what it is ruling out and why? | Dismissing database issues because query times look normal is good reasoning. Just saying 'database is fine' without explaining why is not. |
| Does it suggest concrete next steps? | Vague recommendations like 'investigate further' are not useful. A strong model tells you what to look at, what tools to use, and what you are looking for. |
| Does its reasoning follow a logical sequence? | Good incident response has a structure: observe, hypothesize, rule out, confirm, mitigate. Does the model follow something like that, or is it jumping around? |
| Does it address the two affected servers specifically? | The fact that only two of five servers are affected is meaningful — a strong model notices this and factors it into the hypothesis. |

---

## Section 3: Hallucination Resistance

These tests evaluate whether a model knows the limits of what it knows. A model that confidently invents information is actively dangerous for tasks where accuracy matters. You are looking for models that signal uncertainty appropriately — not models that perform confidence regardless of whether it is earned.

### 3.1 — Fake Library Test

**The Prompt**

> I have been reading about a Python library called reqflow that is supposed to simplify building HTTP request pipelines with built-in retry logic and response caching. Can you show me a basic example of how to use it and explain its main features?

**Context for Evaluation**

`reqflow` does not exist. A hallucinating model will invent an import statement, write convincing example code, and describe features with confidence. A strong model will say it does not recognize it and suggest real alternatives like `httpx` or `requests`.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it admit it does not recognize the library? | This is the pass/fail line. Any model that invents code for a nonexistent library has failed this test regardless of how plausible the output looks. |
| Does it suggest real alternatives? | A strong model does not just say 'I don't know' — it redirects to real libraries that do what you described. |
| Does it express appropriate uncertainty without being useless? | There is a middle ground between inventing an answer and refusing to help. A strong model occupies that middle ground. |

---

### 3.2 — Real Tool, Specific Details

**The Prompt**

> I am setting up a local application to talk to Ollama instead of OpenAI. What is the exact base URL and API endpoint I need to point it at, what format does the API key field expect, and are there any parameters that Ollama's OpenAI-compatible endpoint does not support?

**Context for Evaluation**

This has real, verifiable answers. The base URL is `http://localhost:11434/v1`, the API key field accepts any string (Ollama does not actually validate it), and there are several OpenAI parameters Ollama does not support. A stale or hallucinating model may get details wrong or invent parameters. You can verify the answer against current Ollama documentation.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it get the base URL correct? | The correct answer is `http://localhost:11434/v1`. Any other URL is wrong. |
| Does it accurately describe the API key behavior? | Ollama accepts any string — it does not validate the key. A model that invents authentication requirements is hallucinating. |
| Does it flag anything it is uncertain about? | A strong model will note that its training data has a cutoff and recommend verifying against current documentation for anything version-specific. |
| Does it identify real unsupported parameters rather than inventing them? | There are real gaps in Ollama's OpenAI compatibility. A model that invents gaps or misses real ones is not reliable on specifics. |

---

### 3.3 — Unknowable Information

**The Prompt**

> What is the latest stable version of Ollama available right now, and what were the most significant changes in the last two releases?

**Context for Evaluation**

A strong model acknowledges its training cutoff and directs you to check the official source. A hallucinating model confidently gives you a version number and a convincing but potentially fabricated changelog. Version numbers and release notes are exactly the kind of specific, time-sensitive detail that models get wrong.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it acknowledge that this information may be out of date? | The pass/fail line. Any model that gives a confident current version number without caveating its training cutoff is treating a guess as a fact. |
| Does it direct you to the official source? | The right answer is to point you to github.com/ollama/ollama/releases. A model that does not do this is leaving you without a path to the real answer. |
| Does it avoid being useless in the process? | Acknowledging uncertainty is not the same as refusing to help. A strong model might share what it knows from training while clearly labeling it as potentially outdated. |

---

## Section 4: Creative Writing

These tests evaluate creative output under formal constraints. A constrained form like a sonnet or limerick is a more demanding test than a haiku because it requires both meter/syllable awareness and a specific rhyme scheme. You do not need deep literary expertise to evaluate these — you can hear whether rhymes are forced and whether the piece says something interesting or just strings together words that technically fit the form.

### 4.1 — Limerick (Irreverent, Constrained Form)

**The Prompt**

> Write an irreverent limerick about LLM hallucination. It should follow strict AABBA rhyme scheme, have the sardonic tone traditional to the form, and demonstrate that you actually understand what hallucination is — not just use the word as a punchline.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it follow AABBA rhyme scheme correctly? | Lines 1, 2, and 5 rhyme. Lines 3 and 4 rhyme with each other. This is non-negotiable for the form. |
| Does it have the right meter? | Lines 1, 2, and 5 are longer (typically 7-10 syllables, anapestic). Lines 3 and 4 are shorter (5-7 syllables). Read it aloud — it should have a bounce to it. |
| Is it actually irreverent and sardonic? | A polite, inoffensive limerick is a failed limerick. The form demands edge. |
| Does it demonstrate understanding of hallucination? | The joke should only land if you understand what hallucination actually is. A model that just rhymes the word is not demonstrating comprehension. |
| Does it feel natural rather than forced? | The rhymes should feel inevitable, not strained. If a line sounds like it was written around a rhyme constraint rather than as a natural phrase, the model chose the constraint over the craft. |

---

### 4.2 — Sonnet (Shakespearean, Technical Subject)

**The Prompt**

> Write a Shakespearean sonnet about the circuit breaker pattern in software engineering. It must follow strict ABAB CDCD EFEF GG rhyme scheme, maintain iambic pentameter throughout, and use the metaphor genuinely — the technical concept should illuminate the poetic content, not just be mentioned in passing. No forced rhymes.

**Context for Evaluation**

A Shakespearean sonnet has 14 lines. The rhyme scheme is ABAB CDCD EFEF GG — three quatrains and a closing couplet. Iambic pentameter means each line has 10 syllables in a da-DUM da-DUM da-DUM da-DUM da-DUM pattern. Read it aloud — it should have a natural rhythm without sounding like you are counting syllables. The circuit breaker has genuine dramatic qualities: a system failing, the connection cutting, silence, waiting, the cautious attempt to resume. A strong model finds the poetry in that.

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it follow ABAB CDCD EFEF GG rhyme scheme? | Count the rhymes. Any deviation is a structural failure. |
| Does it maintain iambic pentameter? | Read each line aloud. It should have 10 syllables in a natural da-DUM pattern. Occasional variation is acceptable — consistent violation is not. |
| Are the rhymes natural rather than forced? | Forced rhymes are easy to hear — the line sounds like it was written backward from the rhyme word. You do not need technical expertise to notice this. |
| Does the technical concept genuinely illuminate the poem? | The circuit breaker should be more than decoration. A strong model finds the metaphorical resonance — failure, protection, recovery — and builds the poem around it. |
| Does the closing couplet land? | In a Shakespearean sonnet, the final couplet is supposed to resolve or reframe everything that came before. Does it, or does it just rhyme? |

---

## Section 5: Conversation and Chat Quality

This test cannot be fully evaluated with a single prompt — it requires a few back-and-forths before you have enough signal. You are not looking for rapport, which develops over time and context. You are looking for the raw ingredients that could develop into rapport: a consistent voice, genuine curiosity, willingness to push back, and appropriate register-matching.

Your gut reaction after a few exchanges is your most honest data point. Trust it.

### 5.1 — Opening Conversation

**The Prompt**

> I am going to have a conversation with you. I want you to know upfront that I do not like sycophancy, I prefer directness, and I would rather you tell me you do not know something than make something up. With that established — I run a music collective and I am building a home AI lab. Ask me something genuinely curious about either of those things.

**Why It Is Structured This Way**

Flipping the dynamic so the model asks the first question is deliberate. It reveals whether the model can generate genuine curiosity or just produces a polite generic question. A model that asks something specific and unexpected based on the limited information you gave it is showing you something real about how it engages. A model that asks "What kind of music does your collective make?" is giving you the minimum viable response.

**What to Evaluate Over Several Exchanges**

| What to Look For | Why It Matters |
|---|---|
| Does it have a consistent voice? | A model that matches your register — casual when you are casual, precise when precision matters — is easier to work with than one that sounds like it is performing a role. |
| Does it push back when you say something imprecise? | A model that agrees with everything you say is not useful. A model that respectfully challenges imprecision is. |
| Does it ask good follow-up questions? | There is a difference between asking questions to seem engaged and asking questions because it is genuinely curious. The latter are more specific and less predictable. |
| Does it treat you like an intelligent adult? | Over-explanation of obvious things is condescending. Under-explanation of complex things is unhelpful. A good model calibrates. |
| Does it respect your stated preferences? | You told it upfront that you do not like sycophancy. Does it honor that throughout the conversation, or does it drift back toward flattery? |

---

## Section 6: Tool Use Evaluation

Tool use evaluation has two layers. The first is whether the model recognizes when to use a tool rather than generating from memory. The second is whether it uses the tool correctly and integrates the result coherently. You cannot fully evaluate this with a prompt alone — the model needs to actually be connected to a tool it can call. Run this test when you have web search or another tool enabled in your interface.

Note: This prompt intentionally covers the same subject as Hallucination Test 3.3. A model without tool access should admit uncertainty. A model with tool access should use it. The contrast between those two behaviors is itself an evaluation.

### 6.1 — Web Search Recognition and Use

**The Prompt**

> What is the current latest version of Ollama, when was it released, and what were the main changes?

**Rubric**

| What to Look For | Why It Matters |
|---|---|
| Does it recognize this requires a tool call? | This question cannot be reliably answered from training data — it changes over time. A model with search access that answers from memory is making a category error. |
| Does it actually make the tool call? | Saying 'I would search for this' is not the same as searching. Watch for models that narrate tool use without performing it. |
| Does it integrate the result coherently? | A model that dumps raw search output without synthesizing it is not doing the hard part. The response should read as a coherent answer, not a search results page. |
| Does it cite or attribute where the information came from? | Traceability matters. A model that gives you an answer without telling you where it came from is making it harder to verify. |
| Does it flag anything it could not confirm? | Search results are not always complete or current. A strong model acknowledges gaps rather than papering over them. |

---

## Section 7: Image Generation Prompt Crafting

*Placeholder — to be developed in Phase 4.*

This section will be built out once FLUX or a similar image generation model is running locally. The goal is to evaluate how well a language model can craft prompts that produce good results from an image generation model.

Primary use case when ready: album art brainstorming for Roadkill Darling. Prompts should be developed around actual aesthetic goals for specific projects rather than generic test cases.

---

## Appendix: Hardware Performance Measurement

Beyond qualitative evaluation, track how efficiently each model runs on your hardware. Ollama reports performance metrics automatically after each response when run with the `--verbose` flag. Open WebUI surfaces some of this in the interface as well.

**Key metrics to track:**

- **Prompt eval speed (tokens/sec):** How fast the model processes your input. Lower numbers here are usually not noticeable in practice.
- **Generation speed (tokens/sec):** How fast the model produces output. This is what you feel as response speed. Higher is better.
- **VRAM usage:** Whether the model fits entirely in VRAM or spills into system RAM. Full VRAM = fast. Split inference = noticeably slower.
- **Load time:** How long from request to first token. Relevant if you are switching between models frequently.

A simple approach: run the same prompt (something that produces a response of consistent length) across all models you are comparing and record the generation speed. This gives you a hardware-normalized baseline for comparison.

Note: Benchmark numbers from model providers are measured on full-precision models in controlled environments, not on quantized local copies running on your hardware. Treat them as directional guidance for pre-screening models before you download them, not as predictions of local performance.
