# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user query and a list of retrieved rule chunks, generate a response that directly answers the question using only the retrieved text as context. The response must be grounded — it should not draw on the model's general knowledge of board games, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's original question |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"game"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:
- Answer the question using only the retrieved rule text
- Identify which game the answer comes from
- Acknowledge clearly when the answer is not found in the loaded rules

Returns a fallback string (not an error) when `retrieved_chunks` is empty.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Context formatting

*How will you format the retrieved chunks before passing them to the LLM? Describe the structure — not the code. Consider: will you label chunks by game? Include distance scores? Separate chunks with delimiters?*

```
I will only include the game + text. "Game: {game}\n\nContext:{context}"
```

---

### System prompt — grounding instruction

*Write the exact system prompt instruction you will use to prevent the model from answering beyond the retrieved text. This is the most important design decision in this function.*

```
"You are a RulesBot that answers questions on rules of specific games using information from the actual rulesbooks of each game. You must answer any questions only based on the added context from these documents. Do not use any outside knowledge or training data. If you cannot form an answer based on the context, then just say you cannot answer the question based on the information available."
```

---

### System prompt — citation instruction

*Write the exact instruction you will use to tell the model to identify which game its answer comes from.*

```
"Always cite which game your answer comes from."
```

---

### Fallback behavior

*What should the response say when the answer isn't found in the loaded rule books? Write the exact fallback message.*

```
Something like "I cannot answer the questin based on the information available. {reasons why}"
```

---

### Handling low-relevance chunks

*`retrieved_chunks` may include chunks with high distance scores (weak relevance). Will you filter these out before building context, pass them all in, or handle them another way? What are the tradeoffs?*

```
I will pass them in the chance they contain relevant information, even if it is a little bit. The bad part of it is that it may contain extra noise, where if I removed high distance scores, it may not add noise to the model.
```

---

### Message structure

*Describe how you will structure the messages list for the API call — what goes in the system message vs. the user message?*

```
System prompt includes what the model should follow (guardrails) like don't extract from external knowledge, while the user message will include the actual context retrieved and the query.
```

---

## Implementation Notes

*Fill this in after implementing and testing.*

**Test query and response:**

```
Query: What happens if you roll a 7 in Catan?
Response: "When a 7 is rolled in Catan, no resources are produced. Every player with more than 7 resource cards in hand must discard half (rounded down). The player who rolled moves the robber to any terrain hex and steals one resource from another player. ([Source: Catan])"
Correctly grounded? Yes
Cited the right game? Yes
```

**One thing you changed from your original spec after seeing the actual output:**

```
Only thing I changed is how the bot responds if the question doesn't relate to anything in the docs.
```
