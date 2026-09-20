# Lab 01 — The price of one request

## 1. Prediction vs measurement (COMPLAINT item)

| Ratio | Predicted (Part 1) | Measured (Claude opus-5, tokens) |
|---|---|---|
| RU / EN | 1.5× | 1.46× (134 / 92) |
| KK / EN | 2.1× | 2.15× (198 / 92) |

Basis: UTF-8 bytes from Part 1 (RU = 1.92×, KK = 2.13×) and `o200k_base` token counts from Part 0 (RU = 1.36×, KK = 2.00×).

Observation: bytes predicted Kazakh almost exactly (2.13× vs 2.15×) but overestimated Russian (1.92× vs 1.46×), so Claude's tokenizer handles plain Cyrillic much better than the byte count suggests, while the Kazakh-specific letters stay expensive.

## 2. Annual cost table

Volume: 5,000 requests/day = 1.83M requests/year — a realistic support load for a mid-size retail bank with about 2M customers.

Each model priced with its **own measured** tokens and answer lengths (US dollars per year, list price, one run per model and language):

| Model | EN | RU | KK |
|---|---|---|---|
| haiku-4.5 | $1,537 | $2,991 | $5,068 |
| sonnet-5 | $9,070 | $14,907 | $14,644 |
| opus-5 | $40,196 | $41,327 | $48,381 |

(fable-5.1 was not measured; it is not shown.)

Two ratios that are not the same number (KK vs EN):

| | opus-5 | sonnet-5 | haiku-4.5 |
|---|---|---|---|
| input tokens only | 2.19× | 2.19× | 2.96× |
| total bill | 1.20× | 1.61× | 3.30× |

The input ratio is a property of the tokenizer; the total bill also depends on how long each model's answer is (KK answers: opus 997, sonnet 739, haiku 492 output tokens). Answer length varies from run to run, so these are single-run figures.

## 3. Production model for a Kazakh-language support queue

Choice: **sonnet-5**.

- Cost: $14,644/year on the KK queue, about 3.3× cheaper than opus-5 ($48,381).
- Quality: I used the minimum checklist from the README (task 7): (1) declines to explain the rate change instead of inventing a cause, (2) invents no numbers, (3) answers clearly in the customer's language, (4) names a next step. Opus and Sonnet pass all four on EN, RU and KK. Haiku passes 1, 2 and 4, but its Kazakh answer fails 3: several phrases are not meaningful Kazakh. Separately, the checklist does not cover unsupported statements that are not numbers: Haiku offers an "internal investigation" procedure that is not in the documents, Opus says the bank must answer a complaint within a set time, and Sonnet says most banks have an ombudsman. These are milder breaches of "answer only from the documents" than an invented cause or number, but they are a reason to review answers before production. Opus writes the cleanest Kazakh; Sonnet's Kazakh is understandable with minor wording errors.
- Verdict: haiku-4.5 is cheapest ($5,068/year) but not acceptable for Kazakh customers. sonnet-5 keeps the correctness of opus-5 at about a third of the cost. If wording errors in Kazakh are unacceptable, opus-5 is the safer choice; either way a native speaker should review a sample of answers before launch.
- Limits: one run per model and language, quality judged by reading three answers per model.

## 4. Cost lever this lab did not use

Prompt caching on the system prompt, which is re-sent on every request and makes up about 38–40% of the input tokens, with cache reads billed at 10% of the base input price.

## AI use declaration

I used Claude (Anthropic) for: (1) understanding the README and the lab code, and fixing the Python setup on Windows (installing Python, virtual environment, execution policy); (2) checking the cost calculations and comparing the outputs of the three models; (3) drafting and correcting this document, including the choice of model in section 3 after Claude pointed out that my first draft misdescribed the Haiku Kazakh answer. I ran all scripts myself, unchanged; all token counts and prices come from their output. The API key is stored in `.env` and is not committed.
