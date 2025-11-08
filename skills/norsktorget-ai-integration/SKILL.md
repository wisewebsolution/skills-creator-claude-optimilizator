---
name: norsktorget-ai-integration
description: Use when working with the Gemini AI integration in Norsktorget. This skill provides strategic guidance, best practices for prompt engineering, error handling strategies, and cost/performance optimization patterns specific to generating classified ads.
---

# AI Integration Best Practices for Norsktorget

## Overview

This skill provides the strategic "playbook" for integrating with the Gemini API in the Norsktorget project. It complements the technical documentation in `docs/features/ai-integration.md` by defining **how** and **why** we interact with the AI to achieve the best product results.

**Core Principle:** AI is a powerful but unreliable assistant. Our code must be robust, defensive, and always prioritize a good user experience, even when the AI fails.

## When to Use

- When modifying or creating prompts sent to the Gemini API.
- When handling responses or errors from the AI service.
- When implementing features that rely on AI-generated content.
- When optimizing the cost or performance of AI integrations.

## 1. The Art of Prompt Engineering for Norsktorget

A well-crafted prompt is the single most important factor for getting high-quality results from Gemini. All prompts sent to the AI **must** follow these principles.

### Principle 1: Define a Clear Role (Persona)

Always start your prompt by telling the AI what role it should play.

**✅ GOOD**

"You are an expert in the Norwegian used car market. Your task is to analyze the following image and text to create a compelling classified ad..."

**❌ BAD**

"Analyze this image and text..."

### Principle 2: Demand a Specific Format (JSON)

All prompts that expect structured data **must** require the output to be in JSON format and **must** provide a clear schema.

**✅ GOOD**

```
...Respond ONLY with a valid JSON object following this exact schema:
{
  "title": "string",
  "description": "string",
  "attributes": {
    "condition": "string (e.g., 'new', 'used')",
    "brand": "string"
  }
}
```

### Principle 3: Provide Examples (Few-Shot Prompting)

For complex tasks, provide 1-2 examples of good input and desired output. This dramatically improves accuracy.

**✅ GOOD**

```
...Here is an example:
Input Text: 'Selger min trofaste Volvo V70. God stand.'
Input Image: [Image of a Volvo V70]
Output JSON: {
  "title": "Velholdt Volvo V70 til salgs",
  "attributes": {
    "brand": "Volvo",
    "model": "V70",
    "condition": "used"
  }
}
Now, analyze the following...
```

### The "Golden Prompt" Template for Norsktorget

Use this template as a starting point for all ad analysis prompts.

```
You are an expert copywriter for the Norwegian classifieds market, specializing in [CATEGORY_NAME].
Your task is to analyze the provided user input (image and/or text) and generate a compelling ad.
Consider the context of the Norwegian market (seasons, popular brands, pricing in NOK).

Respond ONLY with a valid JSON object following this exact schema:
{
  "title": "A short, catchy, SEO-friendly title in Norwegian (max 60 chars).",
  "description": "A helpful, descriptive text in Norwegian (50-100 words).",
  "price_suggestion_nok": {
    "min": "integer",
    "max": "integer"
  },
  "attributes": {
    // List of expected attributes for the category
    "brand": "string",
    "condition": "string (new, used, slightly_used, for_parts)"
  }
}

Here is an example of a good response:
[...Provide a full, high-quality example here...]

Now, analyze the following user input and generate the JSON response.
User Text: '[USER_PROVIDED_TEXT]'
```

## 2. The AI Error Handling & Fallback Strategy

Never assume the AI will respond correctly. Our system **must** be resilient.

### The Decision Tree for Handling AI Responses

```mermaid
graph TD
    A[Call Gemini API] --> B{Response Received?};
    B -- No (Timeout > 3s) --> C[Show "AI is slow" message, enable manual mode];
    B -- Yes --> D{Is response valid JSON?};
    D -- No --> E[Attempt 1: Retry with simpler prompt];
    E --> F{Retry successful?};
    F -- No --> G[Show "AI error" message, enable manual mode];
    F -- Yes --> H[Continue with parsed data];
    D -- Yes --> H;
    H --> I{Confidence > 0.6?};
    I -- Yes --> J[Auto-fill form fields];
    I -- No --> K[Show suggestions but do not auto-fill, mark as "low confidence"];
```

### Implementation Checklist:

- **Timeout:** All AI API calls must have a strict timeout (e.g., 3-5 seconds).

- **JSON Validation:** Always wrap JSON parsing in a try-catch block.

- **Retry Logic:** On first JSON parse failure, retry the request once with a simpler prompt (e.g., asking only for the title).

- **Fallback UI:** If AI fails completely, the UI must gracefully switch to a fully manual mode. The user should never be blocked.

- **Confidence Handling:** For each suggested field, if confidence is below a threshold (e.g., 0.6), display the suggestion to the user but do not pre-fill the form field.

## 3. Cost & Performance Optimization

### Caching Strategy

To avoid redundant API calls and costs, all successful AI analysis results must be cached.

- **Cache Key:** A hash generated from the input (e.g., SHA256 of the image bytes + user text).
- **Cache Storage:** Use Hive for local, on-device caching.
- **Cache TTL (Time-To-Live):** 24 hours. A user analyzing the same item again within a day should get an instant, cached response.

### Image Pre-processing

Before sending an image to the Gemini API, it must be pre-processed to reduce token usage and cost.

- **Required Action:** Use the `ImageProcessorService`.
- **Specification:** Resize images so the longest side is max 1024px.
- **Specification:** Compress images to be under 2MB.

This is handled by `ImageProcessorService.processForAI()`, which must be called on all images before they are sent to `GeminiService`.

## Common Mistakes / Anti-Patterns

| ❌ Anti-Pattern | ✅ Required Norsktorget Pattern |
|:---|:---|
| Sending a simple, one-line prompt to the AI. | ALWAYS use the "Golden Prompt" template with a defined role, JSON schema, and examples. |
| Assuming the AI will always return valid JSON. | ALWAYS wrap JSON parsing in a try-catch block and implement the retry/fallback strategy. |
| Calling the AI API for the same image multiple times. | ALWAYS implement caching based on a hash of the input. |
| Sending full-resolution images to the API. | ALWAYS use ImageProcessorService to resize and compress images first. |
