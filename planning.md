# TakeMeter Planning Document

## Community

I chose r/soccer and r/worldcup as my target communities. Both are massive, highly active subreddits where millions of users discuss matches, players, tactics, and transfers daily. The discourse quality varies wildly across posts. Some users write detailed tactical breakdowns citing formations and positioning data. Others throw out bold declarations with no reasoning at all. Many just scream into the void after a last-minute goal. These three modes of discourse (analysis, assertion, emotion) are distinctions the community already recognizes informally, which makes the classification task meaningful rather than arbitrary.

## Labels

### 1. tactical_analysis

**Definition:** The post makes a structured argument about formations, player positioning, team strategy, or match dynamics, and backs it up with specific on-pitch references, statistics, or verifiable tactical observations.

**Example 1:** "Argentina's build-up play relied heavily on Enzo Fernandez dropping between the center-backs to create a 3-2-5 shape in possession. This pulled the opposing midfield out of position and gave De Paul space to drive forward."

**Example 2:** "Looking at the xG map, Brazil created most of their chances from left-side overloads. Vinicius and Neymar kept combining in the left half-space, but the finishing let them down with 2.3 xG from that zone alone."

### 2. hot_take

**Definition:** A bold, confident opinion stated without supporting evidence. The post asserts rather than argues, making a claim that might be true but offering no specific reasoning or data behind it.

**Example 1:** "Mbappe is clear of Haaland and it's not even close. People comparing them are delusional."

**Example 2:** "England will never win a major tournament with Southgate. He's a PE teacher cosplaying as a manager."

### 3. match_reaction

**Definition:** An immediate emotional response to a goal, result, or in-game moment. The post expresses a feeling with little to no argument structure: excitement, devastation, disbelief, or humor rather than reasoning.

**Example 1:** "WHAT A GOAL!!! Messi is actually not human. I'm in tears right now."

**Example 2:** "Pain. Nothing but pain. Supporting this team is a mental health hazard."

## Hard Edge Cases

The hardest boundary is between **tactical_analysis** and **hot_take**. Consider this post:

> "Rodri's positioning is the most underrated aspect of Spain's system. He sits right between the opposition's lines, always finding the pocket of space that connects defense to attack."

It opens with subjective framing ("most underrated") but then gives specific tactical detail about where Rodri positions himself and why it matters structurally. Does the football knowledge elevate it above a pure opinion?

**Decision rule:** Strip away the opinion framing. If what remains is a verifiable claim or specific football knowledge demonstrating understanding of game mechanics, label it **tactical_analysis**. If the evidence is vague or decorative, just there to sound credible without actually building an argument, label it **hot_take**.

A second edge case sits between **hot_take** and **match_reaction**:

> "This is the worst refereeing performance I have ever witnessed in my entire life."

Emotionally charged, but also a declarative evaluative claim. **Decision rule:** If the post is expressing an in-the-moment emotion triggered by something happening live, label it **match_reaction**. If it reads as a retrospective judgment made after the fact, label it **hot_take**.

A third documented case:

> "The 2022 World Cup was rigged for Argentina. The referee in the final was clearly biased, look at the penalty decisions."

This references specific evidence (penalty decisions) but the core claim is conspiratorial assertion. I labeled it **hot_take** because the "evidence" decorates a predetermined conclusion rather than building toward one through reasoning.

## Data Collection Plan

**Source:** Public comments from r/soccer and r/worldcup, targeting a mix of thread types for balance:
- Post-match threads (heavy on match_reaction)
- Tactical discussion threads and analysis posts (heavy on tactical_analysis)
- Transfer rumor threads, "unpopular opinion" threads, and player comparison posts (heavy on hot_take)

**Target:** 210 total examples, roughly 70 per label (about 33% each).

**Collection method:** Reddit's public JSON API with automated fetching and 2-second delays between requests. If the API is inaccessible, construct representative examples based on real discourse patterns observed in the communities.

**Handling underrepresentation:** If any label falls below 20% after initial collection, I will target specific thread types that produce that label. For tactical_analysis, that means seeking threads from tactical-focused accounts or posts tagged "[Analysis]."

**Filter criteria:** Exclude comments shorter than 25 characters, deleted/removed comments, bot-generated content, and comments that are purely links or images with no text.

## Evaluation Metrics

Overall accuracy provides a baseline but is insufficient alone for a balanced 3-class task where per-class performance matters.

Per-class F1 score is the primary metric. It combines precision and recall into one number per label, revealing whether the model correctly identifies each class without over-predicting it. This matters because the three labels have different confusion costs. Misclassifying tactical_analysis as hot_take is more meaningful than confusing hot_take with match_reaction, since the first pair shares a harder decision boundary.

A confusion matrix will expose directional error patterns. I expect the most confusion on the tactical_analysis/hot_take boundary and the least between tactical_analysis and match_reaction, which are structurally very different from each other.

Macro-averaged F1 serves as the single summary metric, weighting all three classes equally since the dataset is balanced.

## Definition of Success

**Good enough for deployment:** Macro F1 >= 0.75, with no individual class F1 below 0.65. At this level the model distinguishes between discourse modes well enough to serve as a sorting or filtering tool in a community dashboard.

**Genuinely useful:** Macro F1 >= 0.85, all individual class F1 scores above 0.75. The model could reliably surface tactical content or flag low-effort reactions for community moderation.

**Baseline expectation:** The zero-shot Groq baseline should hit at least 50-60% accuracy (well above the 33% random floor) since the labels are linguistically distinct even without task-specific training. If the fine-tuned model doesn't beat the LLM baseline by at least 10 percentage points, the fine-tuning likely didn't capture meaningful patterns beyond what general language understanding already provides.

**Failure signal:** A fine-tuned model scoring below 60% macro F1 would indicate a fundamental issue with either label definitions, annotation consistency, or data quality that needs addressing before further work.

## AI Tool Plan

### Label Stress-Testing

I provided Claude with the three label definitions and decision rules, then asked it to generate 8-10 posts that deliberately sit at label boundaries:
- 3-4 posts at the tactical_analysis/hot_take boundary (opinions seasoned with tactical language)
- 3-4 posts at the hot_take/match_reaction boundary (emotionally charged declarations)
- 2 posts at the tactical_analysis/match_reaction boundary (excited reactions referencing specific plays)

If more than 2 of these could not be cleanly classified, the decision rules needed tightening. This was done before finalizing the dataset.

### Annotation Assistance

Claude assisted with generating and pre-labeling the dataset. The workflow:
1. I provided the taxonomy definitions and decision rules
2. Claude generated representative examples for each category
3. I reviewed every example to verify correctness and consistency with decision rules
4. All 211 examples were reviewed individually; none were accepted without verification

This is disclosed here and will be disclosed again in the README's AI usage section.

### Failure Analysis

After training and evaluation, I will:
1. Extract all misclassified test examples
2. Feed them to Claude with true and predicted labels
3. Ask it to surface patterns (short posts getting confused? sarcasm missed? one label pair dominating errors?)
4. Verify any patterns by re-reading the examples myself
5. Determine whether errors stem from labeling inconsistency or genuine model limitations

Specific patterns to watch for:
- Length correlation: shorter posts misclassified more often
- Sarcasm/irony: model missing tonal cues
- Mixed-mode posts: examples containing elements of two labels
- Vocabulary triggers: model over-relying on keywords like formation numbers for tactical_analysis
