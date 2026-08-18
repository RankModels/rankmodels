# 🌟 RankModels

> **Stress-test LLMs from simple queries to ultra-nuanced prompts.**  
> RankModels is a collaborative benchmark site evaluating top AI models on realistic prompt complexities in an open-source vault.

---

### 🌐 Capabilities & Live Arena

> **🎮 Interactive Web Interface:** Browse the live side-by-side model comparisons at **`rankmodels.github.io`**

* **Side-by-Side Comparisons:** Toggle between different prompt types and view direct model outputs side by side.
* **Performance Metrics:** Examine latency graphs and token speed across different model versions.
* **Difficulty Filtering:** Browse prompts categorized by difficulty level without manually sorting through raw file structures.

---

### 🏛️ Prompt Museum & Vault

We rigorously test premier LLMs: Claude, Gemini, GPT, DeepSeek, and others. Users can submit any prompt type (ranging from straightforward code corrections to extremely complex, multi-level edge cases) and document each response from leading models.

* 🆓 **100% Free & Open Source:** No paywalls or undisclosed fine-tuning regimes. Raw model outputs are documented directly as markdown files.
* 🧪 **Edge Cases > Synthetic Benchmarks:** Community-generated edge case examples stress-test true instruction-following capabilities. Unlike standard benchmarks, real edge cases expose limitations, hallucinations, and intelligence ceilings in natural language generation tasks.

---

### 📁 Archive Format & Data Structure

Prompt logs are archived according to date format `YYYY-MM/DD`, followed by provider and model version. This is the usual branch formatting:

```text
/prompts
  └── /YYYY-MM                     # Year-Month folder (e.g., 2026-08)
        └── /DD                    # Day of test run (e.g., 17)
              ├── /01              # Prompt ID / Run ID
              │     ├── prompt.md  # Raw prompt & user constraints
              │     ├── /claude
              │     │     ├── /claude-opus-5
              │     │     │     └── response.md
              │     │     └── /claude-sonnet-5
              │     │           └── response.md
              │     ├── /gemini
              │     │     ├── /gemini-3.6-flash
              │     │     ├── /gemini-3.5-flash-lite
              │     │     └── /gemini-3.1-pro-preview
              │     ├── /openai
              │     │     ├── /gpt-5.6-sol
              │     │     └── /gpt-5.5-pro
              │     └── /deepseek
              │           └── /deepseek-v4-pro
              │
              └── /02              # Next prompt of the day
                    ├── prompt.md
                    └── ...
