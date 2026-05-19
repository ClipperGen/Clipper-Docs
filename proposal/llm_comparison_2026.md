# LLM Comparison for Clipper Gen "Brain" (April 2026)

This document provides a comparative analysis of top LLM APIs for the **LLM Refiner** component of the processing engine. The goal is to identify the most cost-effective yet stable model for analyzing transcripts and identifying viral moments.

## 1. Pricing Comparison (Per 1 Million Tokens)
Prices are estimates for standard synchronous API calls as of April 2026. 

| Model Tier | Model Name | Input Cost | Output Cost | Context Window | Best For |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Budget / Flash** | **DeepSeek V4 Flash** | **$0.14** | **$0.28** | 1M | High-volume SaaS (Lowest Price) |
| | **Gemini 2.5 Flash-Lite** | $0.10 | $0.40 | 1M | Video/Audio analysis (Free tier available) |
| | **GPT-4.1 Nano** | $0.10 | $0.40 | 1M | Standard simple tasks |
| | **GPT-4o mini** | $0.15 | $0.60 | 128K | Legacy support |
| **Mid-Tier** | **DeepSeek V4 Pro** | $1.74 | $3.48 | 1M | Complex viral logic (Cost-efficient) |
| | **Gemini 3.1 Pro** | $2.00 | $12.00 | 2M | Very long videos (Massive context) |
| | **Claude 3.7 Sonnet** | $3.00 | $15.00 | 200K | High-quality reasoning & formatting |
| **Flagship** | **Claude 3.7 Opus** | $5.00 | $25.00 | 1M | Ultra-high stakes / Research |
| | **Llama 4 Behemoth** | $0.90 | $0.90 | 1M | Self-hosted or speed (via Groq) |

---

## 2. Strategic Analysis for Clipper Gen

### **Recommendation 1: DeepSeek V4 Flash (Winner for Volume)**
*   **Performance:** Matches high-end models in logic but costs ~1/10th of the competition.
*   **Stability:** High performance in early 2026, though occasionally has peak-hour latency.
*   **Verdict:** Use this as the **Primary Engine** for transcript analysis to keep user costs as low as possible.

### **Recommendation 2: Gemini 2.5 Flash-Lite (Winner for Multimodal)**
*   **Performance:** Deep integration with Google's video infrastructure.
*   **Advantage:** Extremely generous **free tier** via Google AI Studio for development and early testing.
*   **Verdict:** Best for **Prototyping** and when we want to analyze video frames directly (not just text).

### **Recommendation 3: Claude 3.7 Sonnet (Winner for Quality)**
*   **Performance:** The current industry leader in instruction following and returning perfect JSON.
*   **Verdict:** Use as a **Fallback** or "Premium" tier option where users pay more for high-accuracy viral analysis.

---

## 3. Comparison Summary Matrix

| Metric | DeepSeek V4 Flash | Gemini 2.5 Flash-Lite | Claude 3.7 Sonnet |
| :--- | :--- | :--- | :--- |
| **Cost** | ⭐️ High (Cheapest) | ⭐️⭐️ High | ⭐️⭐️⭐️ Medium |
| **Reasoning** | ⭐️⭐️ Good | ⭐️⭐️ Good | ⭐️⭐️⭐️ Excellent |
| **Context** | 1M Tokens | 1M Tokens | 200K Tokens |
| **JSON Reliability**| ⭐️⭐️ Good | ⭐️⭐️ Good | ⭐️⭐️⭐️ Perfect |
| **Development** | OpenAI Compatible API | Generous Free Tier | Best Instruction Following |

## 4. Final Recommendation
For **Clipper Gen**, we should implement the LLM Refiner using an **OpenAI-compatible wrapper** (e.g., using the `openai` Python library with custom base URLs). 

1. Start with **Gemini 2.5 Flash-Lite** for development (it's free).
2. Switch to **DeepSeek V4 Flash** for production launch to maximize profit margins while maintaining stable reasoning performance.
