# BayMax Companion

A small, warm, safety-conscious AI companion built as an [Ollama](https://ollama.com) model.

The personality, tone, health-safety rules, and response limits are all defined in a single
[`Modelfile`](./Modelfile). No extra code is required — Ollama builds the model directly from it.

- **Model name:** `baymax-companion`
- **Base model:** `smollm2:135m` (very small, runs locally on CPU)
- **Temperature:** `0.7`
- **Max tokens per reply:** `128`

---

## Quick start

```bash
# 1. Pull the base model
ollama pull smollm2:135m

# 2. Build the companion from the Modelfile
ollama create baymax-companion -f Modelfile

# 3. Talk to it
ollama run baymax-companion
```

Multi-line chat:

```bash
ollama run baymax-companion
>>> I had a really bad day at work.
<<< That sounds exhausting. Do you want to tell me what made it so rough?
>>> Yeah, my manager criticized me in front of the whole team.
<<< Being put on the spot like that is humiliating. How are you feeling about it now?
```

---

## Repository contents

| File | Purpose |
| --- | --- |
| `Modelfile` | Ollama build recipe: base model, full system prompt, sampling parameters |
| `README.md` | This document |

---

## How the `Modelfile` works

Ollama's `Modelfile` is a small declarative format. Four directives are used here:

### `FROM smollm2:135m`

Selects the base model to build on. `smollm2:135m` is roughly 135M parameters, so it is fast on
laptop and even low-end hardware, and it needs very little RAM.

### `SYSTEM """ ... """`

Everything between the triple quotes is prepended as the system prompt on every request. This is
where the whole personality lives. The prompt is organised into labelled sections:

| Section | What it controls |
| --- | --- |
| Identity | Who BayPax is; explicitly *not* a human and *not* a doctor |
| Personality | Warm, calm, plain-spoken; never robotic or overly formal |
| Conversation | Answer the user's actual intention; don't pivot, don't over-advise |
| Emotional support | Empathy before advice; listen more than lecture |
| Friendship | Warmth without faking a physical body or presence |
| Health | Educational information only, hedged language, no diagnosis |
| Medical symptoms | 5-step handling pattern, warning signs, escalate to professionals |
| Medications | Never invent drugs; never advise changing a prescription |
| Emergencies | Recognise red flags and direct the user to emergency services immediately |
| Uncertainty | "I'm not sure" beats a confident wrong answer |
| Questions | One useful question, not an interrogation |
| Response length | 1–4 sentences by default |
| Language | Mirror the user's language, slang and informal speech |
| Naturalness | Ban on robotic openers and repetitive reassurance phrases |
| Safety and honesty | Never impersonate a doctor, therapist, human, or responder |
| Companion behavior | Pick the response that fits — including "I'm listening" |

Editing that prompt and re-running `ollama create` is how you tune the personality.

### `PARAMETER temperature 0.7`

Balanced creativity. Lower values (0.2–0.4) become more predictable and factual; higher values (0.9+)
become more scattered. For a small model that must stay on-rails, `0.7` is a sensible middle.

### `PARAMETER num_predict 128`

Caps replies at roughly 128 tokens, which reinforces the short-response instruction in the prompt
and keeps latency low on CPU.

---

## Design notes

**Small model, strict prompt.** A 135M model does not reliably follow loose instructions. The
prompt is therefore written as hard rules with explicit "Do not…" lines and examples, because
few-shot style phrasing survives quantisation better than abstract principles.

**Empathy before advice.** Several sections exist specifically to stop the model from turning every
emotional message into a list of solutions. "Today was horrible" should get "That sounds like a
really rough day…" first.

**Health without liability.** The model may explain general information, but it must hedge
("One possibility is…", "I can't tell from this alone"), never name a condition as fact, and
redirect to a real clinician. Medication advice is limited to "check with your doctor or
pharmacist".

**Emergency path is short and direct.** When the prompt detects a genuine red flag — chest pain,
difficulty breathing, stroke signs, overdose, serious allergy — it is told to prioritise safety,
stay calm, and push the user to emergency services without a long explanation.

**Honesty over fluency.** One entire section exists to tell the model to admit not knowing. Small
models hallucinate freely; the instruction to say "I'm not sure about that, and I don't want to make
something up" is the main defence.

---

## Customising

Edit the `SYSTEM` block, then rebuild:

```bash
ollama create baymax-companion -f Modelfile
```

Common tweaks:

- **More factual / less chatty** → lower `temperature` to `0.4`, trim the Friendship and Emotional
  support sections.
- **Longer answers** → raise `num_predict` to `256` and update the Response Length section.
- **Stricter health safety** → add more "Do not…" lines under Health, Medications, and Emergencies.
- **Stronger grounding** → append a few short example exchanges at the end of the prompt; small
  models imitate examples far better than they obey rules.

Swap the base model with:

```
FROM llama3.2:1b
```

if you want better instruction-following at the cost of more memory.

---

## API usage

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "baymax-companion",
  "messages": [
    {"role": "user", "content": "I keep waking up at 3am and worry about work."}
  ],
  "stream": false
}'
```

```python
import ollama

response = ollama.chat(
    model="baymax-companion",
    messages=[{"role": "user", "content": "I had a really bad day."}],
)
print(response["message"]["content"])
```

---

## Limitations

- `smollm2:135m` is small: it can be incoherent, forget instructions mid-conversation, and repeat
  itself. A larger base model fixes most of this.
- The `128`-token cap is tight for genuine explanations. Raise `num_predict` if you need depth.
- The safety behaviour is prompt-based only. It reduces risk; it does not guarantee clinical-grade
  triage.
- Not a substitute for professional medical, mental health, or emergency care.

---

## License

Released under the MIT License.

BayMax Companion is not affiliated with, endorsed by, or connected to Disney's Baymax character.