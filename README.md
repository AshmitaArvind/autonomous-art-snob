# 🤖 GenImageNet Agentic AI Pipeline

> An end-to-end automated workflow that analyses 103,200 human assignments from the GenImageNet dataset, runs dual-AI peer review, detects rater bias, and delivers reports to Telegram, Gmail and Notion. All on autopilot.

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square)
![OpenRouter](https://img.shields.io/badge/LLMs-OpenRouter-blue?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## 📌 What This Does

This pipeline processes the [GenImageNet dataset](https://openbigdata.org/resource/ai-generated-marketing-images-10k-human-ratings-for-quality-and-realism/), 10,320 AI-generated marketing images rated by human workers across 103,200 individual assignments, drawn from 254,400 total assessments covering **quality, realism, aesthetics, creativity, adherence to prompt and overall marketing effectiveness** on a 1-7 scale.

The workflow:
1. Fetches two CSV files (image metadata + human ratings) from Google Drive
2. Parses and merges them on `filename`
3. Computes per-model quality/realism statistics with mean and standard deviation
4. Detects biased raters using Z-score deviation analysis
5. Sends the summary to **two AI models** (primary analyst + adversarial reviewer)
6. Compares their conclusions and assigns an agreement score
7. Delivers a structured report to **Telegram**, **Gmail**, and **Notion**

---

## 🏗️ Architecture

```
Trigger
  ├── HTTP: Get Assignment CSV → Parse Assignment CSV ──┐
  └── HTTP: Get Image CSV      → Parse Image CSV      ──┴── Merge(Append)
                                                               ↓
                                                        Merge + Stats
                                                       ↙          ↘
                                         Kimi-K2 Analysis   Llama 3.3 70B Review
                                                        ↘           ↙
                                                         Merge(Append)
                                                               ↓
                                                      Compare + Report
                                                   ↙        ↓         ↘
                                            Telegram      Gmail       Notion
```

---

## 📊 Key Findings (from real run, May 2026)

> ⚠️ **Disclaimer:** These findings reflect one run of this pipeline on a specific dataset snapshot.  
> They are intended as a demonstration of the pipeline's capabilities, not as a definitive benchmark of AI model quality. Do not use these rankings for commercial model selection without independent validation.

| Model | Mean Quality | Mean Realism | Images | Assignments |
|---|---|---|---|---|
| Firefly 2 | 5.453 | 5.299 | 240 | 2,400 |
| Realistic Vision | 5.350 | 5.411 | 2,400 | 24,000 |
| Midjourney v6 | 5.349 | 5.231 | 2,400 | 24,000 |
| Imagine | 5.336 | 5.161 | 240 | 2,400 |
| Imagen 2 | 5.308 | 5.143 | 240 | 2,400 |
| DALL-E 3 | 5.221 | 4.819 | 2,400 | 24,000 |
| SDXL Turbo | 5.126 | 5.048 | 2,400 | 24,000 |

> ⚠️ Note: Firefly 2, Imagine and Imagen 2 have 10x fewer images than the other four models. Their rankings should be interpreted with caution.

---

## 🧠 Dual-AI Peer Review

The workflow deliberately uses **two different AI models** to avoid single-model bias:

- **Primary Analyst (Kimi-K2):** Produces structured findings on model rankings, bias impact, and actionable insights
- **Adversarial Reviewer (Llama 3.3 70B):** Actively challenges the findings, looks for overclaims, confounders, and missing data

An **agreement score** is calculated. If ≥ 75%, the report is marked `AUTO_PUBLISHED`. Below that threshold it is flagged `NEEDS_HUMAN_REVIEW` and waits for manual review before publishing.

> ⚠️ AI-generated analysis (Kimi-K2 and Llama 3.3 70B) is used for pattern summarisation only.  
> All LLM outputs should be treated as assistive commentary, not authoritative conclusions.  
> The statistical calculations (means, standard deviations, bias flags) are deterministic JavaScript and can be trusted as accurate, while the narrative interpretation is AI-assisted and should be treated as commentary rather than authoritative conclusion.

---

## 🔍 Bias & Variance Detection

Each rater's average deviation from the crowd mean is calculated per image. Workers with |avgDeviation| > 1.5 across ≥ 5 ratings are flagged:

- **Harsh raters:** consistently score below the crowd (negative deviation)
- **Lenient raters:** consistently score above the crowd (positive deviation)

In this dataset, **60 workers were flagged in total: 30 harsh raters and 30 lenient raters**, suggesting variation exists in both directions across the worker pool. Flagged ratings are surfaced in the report for transparency but are not automatically excluded from the analysis.

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| Workflow engine | n8n Cloud |
| Data source | Google Drive (CSV) |
| Primary LLM | Kimi-K2 via OpenRouter |
| Review LLM | Llama 3.3 70B via OpenRouter |
| Alerts | Telegram Bot API |
| Reports | Gmail |
| Logging | Notion Database |

---

## 🚀 Setup Instructions

### 1. Prerequisites
- n8n Cloud account (or self-hosted via Docker)
- OpenRouter API key (free tier sufficient)
- Google account (for Drive + Gmail)
- Telegram Bot token (from @BotFather)
- Notion integration token

### 2. Import the Workflow
1. Download `workflow.json` from this repo
2. In n8n: **Workflows → Import from file**
3. Select `workflow.json`

### 3. Configure Credentials
Replace all placeholder values in the workflow with your own:

| Placeholder | Replace With |
|---|---|
| `OPENROUTER_API_KEY` | Your `sk-or-...` key from openrouter.ai |
| `GOOGLE_DRIVE_FILE_ID_ASSIGNMENT` | Google Drive file ID for assignmentLevel CSV |
| `GOOGLE_DRIVE_FILE_ID_IMAGE` | Google Drive file ID for imageLevel CSV |
| `TELEGRAM_BOT_TOKEN` | Token from @BotFather |
| `TELEGRAM_CHAT_ID` | Your numeric chat ID from @userinfobot |
| `NOTION_DATABASE_ID` | Your Notion database ID |

### 4. Make CSV Files Publicly Accessible
In Google Drive, right-click each CSV → Share → Anyone with the link → Viewer

### 5. Run It
Click **Execute Workflow** in n8n. Full run takes ~30–45 seconds.

---

## 📁 Repository Structure

```
autonomous-art-snob/
├── workflow.json          # n8n workflow (credentials sanitised)
├── README.md              # This file
├── sample_output/
│   └── sample_report.txt  # Example report output
└── docs/
    └── flow_diagram.png   # Visual workflow diagram
```

---

## 📜 Dataset Credit

**GenImageNet** - AI-Generated Marketing Images: 10K Human Ratings for Quality and Realism  
Source: [openbigdata.org](https://openbigdata.org/resource/ai-generated-marketing-images-10k-human-ratings-for-quality-and-realism/)

> This project uses the GenImageNet dataset for research and educational purposes only.  
> Please review the dataset's original license before any commercial use.  
> All findings are based on the publicly available version of this dataset as of May 2026.

---

## 🪪 License

MIT : free to use, modify, and distribute.