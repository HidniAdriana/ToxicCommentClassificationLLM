# Reflection — Toxic Comment Classification (LLM Project)

**MSc Business Analytics | Large Language Models | University of Edinburgh Business School**

---

This was a group project, and I want to be upfront about that. My role was Project Manager, and I personally owned the EDA and data sampling work. The prompt engineering notebooks (M1–M4) and BERT baselines were built primarily by my teammates — I reviewed everything and was involved in every major decision, but I didn't write that code.

What I did own fully was the foundation the whole project ran on.

---

## What I actually did

When we started, the plan was straightforward: sample the data, send it to BERT and GPT, measure results. Our first runs came back with poor performance and the team's first instinct was to blame the models. I wasn't convinced.

I went back to the EDA and looked at the class distribution properly. The Jigsaw dataset is heavily imbalanced — `threat` and `identity_hate` have less than 0.3% representation. When you apply standard stratified sampling to something like that, you end up with a test set where the rarest categories are barely present. The model technically "sees" them, but not enough to learn anything meaningful. And when it gets tested, it fails on those exact categories — which then drags down every metric.

So I redesigned the sampling from scratch. Instead of stratified, we built a stress test: take every available `threat` and `identity_hate` comment, reserve some for the few-shot pool, and put the rest into the evaluation set. Pad the remainder with standard toxic and clean comments to hit 8,000 rows with a 50/50 split. The result was a benchmark that actually challenged the models on the hard cases, not just the common ones.

That decision — and making sure the few-shot pool had zero overlap with the eval set — is what made the rest of the project's results meaningful. If we'd gone ahead with stratified sampling, we'd have been measuring performance on easy cases and calling it a day.

The other thing I did was keep the team together when things got complicated. RAG was new to all of us. When the prompt engineering work got deep into vector embeddings and retrieval logic, half the group wasn't following anymore. Rather than letting people disengage, I translated what was happening — mapped out the full pipeline in plain terms, explained what the 151k pool was actually for, why vectorising the data mattered, what the cosine similarity retrieval was doing. That conversation ended up shaping how we decided which methods to keep for the final submission.

---

## What surprised me

The ROI of RAG surprised me. M1 (zero-shot, simplest method, cheapest) achieved a micro-F1 of 0.821. M4 (HA-RAG, most engineered, most effort) achieved 0.829. That's eight thousandths of a point of improvement for a significant increase in complexity and cost.

I came into this project assuming more sophisticated = better. What the results actually showed is that GPT-4.1 already knows a lot about toxicity. The zero-shot prompt, well designed, gets you most of the way there. More examples don't automatically help — M2's 41 static examples actually *hurt* performance because they introduced example bias and prompt overload. What mattered was the *quality* of the examples and how well they matched each specific input, not the quantity.

That's a useful thing to understand if you're ever making a real build-vs-buy decision about whether to invest in a RAG pipeline. The answer isn't always yes.

The other thing that surprised me was how hard `severe_toxic` was for everything. No method got above F1 = 0.577 on that label. The report attributes this partly to annotation subjectivity in the original dataset — where the line sits between "toxic" and "severely toxic" isn't always clear, and the ground truth labels reflect that ambiguity. That's a ceiling that no amount of prompt engineering was going to break through.

---

## What I'd do differently

If I were starting this again, I'd push to test the RAG methods on an identical held-out set. In our project, M4 was evaluated on a 4,001-row dev/test split while M1–M3 ran on the full 8,000 rows. That makes direct numerical comparison tricky. It wasn't a flaw we could easily fix mid-project, but it's the kind of methodological decision I'd flag earlier.

I'd also spend more time at the start establishing a shared mental model of what we were building before anyone wrote code. We lost time early because people were working with different assumptions about what the pipeline even was. Getting everyone to draw it out on a whiteboard first would have saved a few confusing meetings.

---

## What I took away

Coming from a BI background, I'm used to working with structured data and well-defined outputs. This project pushed me into territory where the "data" was model behaviour and prompt design, and where marginal improvements required careful experimentation rather than another SQL join.

The PM role here was different from what I'd done before. In BI work, I'm usually the one with the most context on the data side. Here, I was deliberately staying out of the way of the prompt engineers while making sure the data foundation was solid and the team stayed coordinated. That balance — knowing when to lead, when to translate, and when to let other people run — is something I'll carry forward.

The EDA and sampling work is the part of this project I'm most proud of. It wasn't glamorous, but it was the decision that made everything else work.
