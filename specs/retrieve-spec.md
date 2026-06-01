# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"game"` | `str` | The game name this chunk came from |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Query approach

*Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?*

```
I will use _colletion.query() to get the chunks that are the most semantically similar to the query. I will pass in the query in the form of a single element list, the number of results (which is 3), and what to include in the result ("documents", "metadatas", "distances").
```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

```
The return type will be a list of dictionaries. Each item will represent a chunk with its text, distance from query, and game it comes from. There should be at most 3 chunks ordered from closest in the semantic sense to the furthest. Here is an example of one item:

{
    "text" : "In Uno, draw two cards when an opponent plays a Draw Two.",
    "game" : "Uno",
    "distance": 0.12
}

Each of these fields come from the _collection.query that returns a nested list of query results. Since we are only doing one query, we should only have one dict. The dict contains ids of the chunks got, the text they contain, metadata that includes the game they came from, and the cosine similarities in a distance field. These values help formalize the return dictionary for the function.
```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```
I need to index into index 0 since there should only be one result for a single query. Nesting exists to parallelize x number of queries. 
```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```
I will keep the results regardless of how relevant they are. The pros is that some rules may be split between different chunks, so maybe a small percentage of a relevant rule is kept in a chunk that has a lower distance score compared to the other chunks chosen. This could still be essential to answering the question. The con is that it introduces extra noise by including irrevelant information that may affect the generated response from the model.
```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```
```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```
Query: How do you get out of jail in Monopoly?
Top result game: Monopoly
Distance score: 0.367
Does it make sense? Kinda because it explain what happens when you go to jail but not exactly what to do. The third option actually explains a bit but a lot of it (could be a chunk issue).
```

**One thing about the query results that surprised you:**

```
The queries weren't really doing well. They get similar worded chunks to the query, but some don't fully answer the question. Also, just because a chunk is more semantically similar than the third most, that doesn't mean that it answers the question better. (that is why keeping all of them works best).
```
