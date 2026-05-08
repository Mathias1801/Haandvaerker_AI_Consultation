# Personalised Customer Message Generator

A Streamlit application that generates tailored outreach messages for tradespeople by combining customer demographic data, regional consumer behaviour profiles, and emotional customer type psychology — powered by Google Gemini.

---

## The Problem This Solves

Tradespeople win work through trust and personal connection, but writing individualised outreach for every potential customer is time-consuming and rarely done well. Generic messages get ignored.

This tool automates the hard part: given a customer name, address, and job description, it infers where that customer sits geographically and psychologically — and writes a message in the tradesperson's own voice that speaks directly to what motivates that specific type of person in that specific part of Denmark.

The result is a message that feels handwritten, not templated.

---

## What This Project Demonstrates

**Problem decomposition** — the core challenge (write a personalised message) is broken into three distinct subproblems: classify the customer's location, retrieve the relevant behavioural context, and generate the message. Each stage is solved with the right tool for the job rather than a single overloaded prompt.

**Two-stage LLM pipeline** — a lightweight Gemini 1.5 Flash call handles structured classification (returning clean JSON for region and settlement type), while Gemini 2.5 Flash handles the more demanding generation task. Using the right model at each stage keeps latency low and output quality high.

**Structured knowledge as prompt context** — rather than asking the LLM to reason about regional consumer behaviour from scratch, the relevant insight is looked up from an encoded knowledge base and injected as grounded context. This reduces hallucination and gives the output a factual foundation the model alone couldn't provide.

**Prompt engineering for persona and tone** — the generation prompt combines tradesperson voice, customer psychology, regional behaviour, and task description into a single coherent instruction that consistently produces short, natural, on-brand messages.

**Practical UI design** — Streamlit is used deliberately here: the goal is a tool a non-technical tradesperson could actually use, not a notebook demo. Input fields map directly to information a tradesperson naturally has about a lead.

---

## Overview

The application takes a customer profile (name, address, task, age, housing type, job title) and automatically determines which Danish region and settlement type (city, town, or rural) the customer belongs to. It maps that location to a regional consumer behaviour profile and a dominant emotional customer type, then passes all of this as structured context to Gemini to generate a short, personalised outreach message in the voice of the tradesperson.

The demo is configured for a carpenter based in Aalborg, but the tradesperson profile is fully configurable.

---

## Pipeline

```
Customer input (name, address, task, age, housing type, job title)
        │
        ▼
Gemini 1.5 Flash      Classifies address into Danish region + settlement type (JSON)
        │
        ▼
Knowledge base        Regional consumer behaviour description retrieved
lookup                Dominant emotional customer type(s) retrieved
        │
        ▼
Gemini 2.5 Flash      Generates personalised message in tradesperson voice (max 8 lines)
        │
        ▼
Streamlit UI          Message displayed for review and use
```

---

## Emotional Customer Types

The system encodes four psychological customer profiles used to tailor message tone and framing:

| Type | Core driver |
|---|---|
| Red | Seeks novelty, variety, and exciting experiences — impulsive and trend-responsive |
| Yellow | Values community, warmth, and human connection — loyal when they feel seen |
| Blue | Motivated by structure, clarity, and documented processes — needs transparency to build trust |
| Green | Seeks stability and reliability — prefers proven solutions and consistent communication |

---

## Regional Profiles

Consumer behaviour data is encoded for five Danish regions (Hovedstaden, Sjælland, Syddanmark, Midtjylland, Nordjylland), each split into three settlement categories (Storby, By, Opland). Each profile specifies a descriptive behavioural insight and the dominant emotional customer type(s) for that segment.

Region and category are resolved automatically from the customer's address and city via the Gemini classification call.

---

## Getting Started

### 1. Install dependencies

```bash
pip install streamlit google-generativeai
```

### 2. Configure your API key

Add your Gemini API key to `.streamlit/secrets.toml`:

```toml
GENMI_API_KEY = "your-key-here"
```

### 3. Run the app

```bash
streamlit run app.py
```

---

## Configuration

The tradesperson profile is defined as a dictionary at the top of `app.py`:

```python
handvaerker = {
    "navn": "Mikkel Andersen",
    "fag": "tømrer",
    "værdier": ["kvalitet", "troværdighed", "lokalt kendskab"],
    "lokation": "Aalborg",
    "stil": "lavpraktisk, ærlig og jordnær"
}
```

Replace these values to generate messages in a different tradesperson's voice.

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | Streamlit |
| LLM | Google Gemini 1.5 Flash (classification) + 2.5 Flash (generation) |
| Language | Python |
| Secrets management | Streamlit secrets (`st.secrets`) |

---

## Notes

- The regional and emotional type data is written in Danish, matching the intended output language
- Output is capped at 8 lines to keep messages practical for direct customer use
