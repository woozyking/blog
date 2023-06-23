---
title: 'Natural Language to JSON: GPT before/after 0613'
date: 2023-06-23 11:20:24
tags:
  - meta-automation
  - NLP
---

## TL;DR

- We leveraged OpenAI's June 13 update to make our web app navigational assistant **5x faster and 20x cheaper**.
- While OpenAI stepped in the right direction with the _Function Calling_ feature, we proved it has room to improve still.

![table](https://github.com/RiskThinking/work-samples/assets/2837532/0d7a2764-d81b-42f4-aa0d-db3706352472)

## Intro

At [RiskThinking.AI](https://riskthinking.ai/), we initiated a project back in April to leverage OpenAI's GPT model to build a navigational assistant for our web app. At the time, `gpt-4-0314` was the only viable model since `gpt-3.5-turbo-0301` doesn't have system steerability.

<!-- more -->

<video controls>
  <source src="https://github.com/RiskThinking/work-samples/assets/2837532/c46526a7-f96a-418c-8ffc-e4c106b3dc71" type="video/mp4">
  Your browser does not support HTML video.
</video>

> Credit: [@jegrieve](https://medium.com/@jegrieve)

One glaring problem is that it's not fast and can get excruciatingly slow!

<video controls>
  <source src="https://github.com/RiskThinking/work-samples/assets/2837532/a2cc9466-fedc-4fd3-b9bf-eccdb37f4249" type="video/mp4">
  Your browser does not support HTML video.
</video>

> Nobody has the patience to wait for 10+ seconds to spin the globe to Toronto

Until now...

Since OpenAI's [June 13 update](https://openai.com/blog/function-calling-and-other-api-updates), OpenAI enabled system steerability for it's `gpt-3.5-turbo` model, and along with that, came the [_Function Calling_](https://platform.openai.com/docs/guides/gpt/function-calling) feature in recognition of the overwhelming use cases built on top of the same simple principle, _Natural Language to JSON_ (our web app navigational assistant would be no different).

> Under the hood, functions are injected into the system message in a syntax the model has been trained on. This means functions count against the model's context limit and are billed as input tokens. If running into context limits, we suggest limiting the number of functions or the length of documentation you provide for function parameters.

We benchmarked the models before and after the June 13 update and concluded a new navigational assistant that's **5 times faster and 20 times cheaper**. On top of that, we exposed that OpenAI's _Function Calling_, while promising, can still do better!

## Benchmark Base (`gpt-4-0314`)

This is the base implementation of our original Copilot implementation, where we leverage both `<system>` and `<user>` prompt engineering to achieve the steerability we needed for stable and high precision `assistant` output of JSON object that can be used for application usage (e.g. VELO).

```
Model: "gpt-4-0314" (pre-0613)

<System>:
You are an app navigation copilot which converts user asks based on possible parameters to a JSON form. For errors, respond with {}

<User>:
Bench nikkei against tsx in the energy sector for >4 degrees scenario in 2070

Output format:
{"Horizon": 2050, "Scenario": {"id": "RT3", "name": "ALL"}, "Benchmark": {"name": "TSX 60", "index": 20, "portfolio_name": "TSX 60", "portfolio_id": 20}, "BenchmarkSector": "Index", "Portfolio": {"name": "S&P 500", "index": 22, "portfolio_name": "S&P 500", "portfolio_id": 22}, "PortfolioSector": "Index"}

Possible values for each parameter:
- Horizon: {'type': 'integer', 'description': 'parsed from user input to best match the closest next decade year', 'min': 2030, 'max': 2090, 'default': 2050}
- Scenario: {'type': 'object', 'description': 'Climate crisis temperature rise scenario', 'enums': [{'id': 'RT3', 'name': 'ALL'}, {'id': 'Paris', 'name': '<2°'}, {'id': 'NDC', 'name': '2°- 4°'}, {'id': 'Hot House', 'name': '>4°'}], 'default': {'id': 'RT3', 'name': 'ALL'}}
- Benchmark: {'type': 'object', 'description': 'The base benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}}
- BenchmarkSector: {'type': 'string', 'description': 'The base benchmark portfolio/index sector', 'enums': ['Financials', 'Materials', 'Energy', 'Consumer Staples', 'Consumer Discretionary', 'Industrials', 'Real Estate', 'Information Technology', 'Utilities'], 'default': 'Index'}
- Portfolio: {'type': 'object', 'description': 'The target benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}}
- PortfolioSector: {'type': 'string', 'description': 'The target benchmark portfolio/index sector', 'enums': ['Financials', 'Industrials', 'Consumer Discretionary', 'Health Care', 'Utilities', 'Information Technology', 'Real Estate', 'Energy', 'Materials', 'Communications', 'Consumer Staples'], 'default': 'Index'}

<Assistant>:
{
  "Horizon": 2070,
  "Scenario": {
    "id": "Hot House",
    "name": ">4°"
  },
  "Benchmark": {
    "name": "NIKKEI 225",
    "index": 23,
    "portfolio_name": "NIKKEI 225",
    "portfolio_id": 23
  },
  "BenchmarkSector": "Energy",
  "Portfolio": {
    "name": "TSX 60",
    "index": 20,
    "portfolio_name": "TSX 60",
    "portfolio_id": 20
  },
  "PortfolioSector": "Energy"
}
143.06s over 10 runs
```

Token `usage`:

```json
{
  "prompt_tokens": 1848,
  "completion_tokens": 140,
  "total_tokens": 1988
}
```

The cost per call would be `(1848 * 0.03 / 1000) + (140 * 0.06 / 1000) = $0.06384` per call.

## 0613 Function Calling

```
<System>:
N/A, using 0613 "Function calling"

<User>:
Bench nikkei against tsx in the energy sector for >4 degrees scenario in 2070

Function calling params:
[{'name': 'get_nav_params', 'description': 'Get navigational parameters based on user message and the scoped parameters', 'parameters': {'type': 'object', 'properties': {'Horizon': {'type': 'integer', 'description': 'parsed from user input to best match the closest next decade year', 'min': 2030, 'max': 2090, 'default': 2050}, 'Scenario': {'type': 'object', 'description': 'Climate crisis temperature rise scenario', 'enums': [{'id': 'RT3', 'name': 'ALL'}, {'id': 'Paris', 'name': '<2°'}, {'id': 'NDC', 'name': '2°- 4°'}, {'id': 'Hot House', 'name': '>4°'}], 'default': {'id': 'RT3', 'name': 'ALL'}}, 'Benchmark': {'type': 'object', 'description': 'The base benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}}, 'BenchmarkSector': {'type': 'string', 'description': 'The base benchmark portfolio/index sector', 'enums': ['Financials', 'Materials', 'Energy', 'Consumer Staples', 'Consumer Discretionary', 'Industrials', 'Real Estate', 'Information Technology', 'Utilities'], 'default': 'Index'}, 'Portfolio': {'type': 'object', 'description': 'The target benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}}, 'PortfolioSector': {'type': 'string', 'description': 'The target benchmark portfolio/index sector', 'enums': ['Financials', 'Industrials', 'Consumer Discretionary', 'Health Care', 'Utilities', 'Information Technology', 'Real Estate', 'Energy', 'Materials', 'Communications', 'Consumer Staples'], 'default': 'Index'}}}}]

Model: gpt-4-0613

<Assistant (Function calling)>:
{
  "name": "get_nav_params",
  "arguments": "{\n  \"Horizon\": 2070,\n  \"BenchmarkSector\": \"Nikkei\",\n  \"PortfolioSector\": \"TSX\"\n}"
}
28.63s over 10 runs

Model: gpt-3.5-turbo-0613

<Assistant (Function calling)>:
{
  "name": "get_nav_params",
  "arguments": "{\n  \"Horizon\": 2070,\n  \"BenchmarkSector\": \"Energy\",\n  \"PortfolioSector\": \"Energy\"\n}"
}
13.07s over 10 runs
```

Token `usage` (`gpt-4-0613`): `(132 * 0.03 / 1000) + (38 * 0.06 / 1000) = $0.00624` per call.
```json
{
  "prompt_tokens": 132,
  "completion_tokens": 38,
  "total_tokens": 170
}
```

Token `usage` (`gpt-3.5-turbo-0613`): `166 * 0.0015 / 1000 = $0.000249` per call
```json
// gpt-3.5-turbo-0613
{
  "prompt_tokens": 132,
  "completion_tokens": 34,
  "total_tokens": 166
}
```

The `arguments` field is what we need to use. It's dysfunctional (to say the least) for our use case, even though they are both really fast and cheap. From the tokens count, it looks like its proprietary _Function Calling_ `system` message may have been compressed too much which resulted in its less desirable outcome for our use case.

## 0613 Custom prompt-engineering

We rebase the same prompt engineering we used for the pre-0613 `gpt-4` model and apply it to both `gpt-4-0613` and `gpt-3.5-turbo-0613` (which has system steerability over the previous version).

```
<System>:
You are an app navigation copilot which converts user asks based on possible parameters to a JSON form. For errors, respond with {}

<User>:
Bench nikkei against tsx in the energy sector for >4 degrees scenario in 2070

Output format:
{"Horizon": 2050, "Scenario": {"id": "RT3", "name": "ALL"}, "Benchmark": {"name": "TSX 60", "index": 20, "portfolio_name": "TSX 60", "portfolio_id": 20}, "BenchmarkSector": "Index", "Portfolio": {"name": "S&P 500", "index": 22, "portfolio_name": "S&P 500", "portfolio_id": 22}, "PortfolioSector": "Index"}

Possible values for each parameter:
- Horizon: {'type': 'integer', 'description': 'parsed from user input to best match the closest next decade year', 'min': 2030, 'max': 2090, 'default': 2050}
- Scenario: {'type': 'object', 'description': 'Climate crisis temperature rise scenario', 'enums': [{'id': 'RT3', 'name': 'ALL'}, {'id': 'Paris', 'name': '<2°'}, {'id': 'NDC', 'name': '2°- 4°'}, {'id': 'Hot House', 'name': '>4°'}], 'default': {'id': 'RT3', 'name': 'ALL'}}
- Benchmark: {'type': 'object', 'description': 'The base benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}}
- BenchmarkSector: {'type': 'string', 'description': 'The base benchmark portfolio/index sector', 'enums': ['Financials', 'Materials', 'Energy', 'Consumer Staples', 'Consumer Discretionary', 'Industrials', 'Real Estate', 'Information Technology', 'Utilities'], 'default': 'Index'}
- Portfolio: {'type': 'object', 'description': 'The target benchmark portfolio/index', 'enums': [{'name': 'TSX 60', 'index': 20, 'portfolio_name': 'TSX 60', 'portfolio_id': 20}, {'name': 'NIKKEI 225', 'index': 23, 'portfolio_name': 'NIKKEI 225', 'portfolio_id': 23}, {'name': 'FTSE 100', 'index': 24, 'portfolio_name': 'FTSE 100', 'portfolio_id': 24}, {'name': 'DAX 40', 'index': 21, 'portfolio_name': 'DAX 40', 'portfolio_id': 21}, {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}, {'name': 'FTSE/JSE 40', 'index': 25, 'portfolio_name': 'FTSE/JSE 40', 'portfolio_id': 25}, {'name': 'BIST 30', 'index': 26, 'portfolio_name': 'BIST 30', 'portfolio_id': 26}, {'name': 'MOEX RUSSIA', 'index': 27, 'portfolio_name': 'MOEX RUSSIA', 'portfolio_id': 27}, {'name': 'IPC MEXICO', 'index': 28, 'portfolio_name': 'IPC MEXICO', 'portfolio_id': 28}, {'name': 'IDX80', 'index': 29, 'portfolio_name': 'IDX80', 'portfolio_id': 29}, {'name': 'FTSE MIB', 'index': 30, 'portfolio_name': 'FTSE MIB', 'portfolio_id': 30}, {'name': 'CAC 40', 'index': 31, 'portfolio_name': 'CAC 40', 'portfolio_id': 31}, {'name': 'ASX200', 'index': 32, 'portfolio_name': 'ASX200', 'portfolio_id': 32}, {'name': 'CSI300', 'index': 33, 'portfolio_name': 'CSI300', 'portfolio_id': 33}, {'name': 'KOSPI', 'index': 34, 'portfolio_name': 'KOSPI', 'portfolio_id': 34}, {'name': 'NIFTY50', 'index': 35, 'portfolio_name': 'NIFTY50', 'portfolio_id': 35}, {'name': 'S&P Merval', 'index': 36, 'portfolio_name': 'S&P Merval', 'portfolio_id': 36}, {'name': 'Tadawul All Share', 'index': 37, 'portfolio_name': 'Tadawul All Share', 'portfolio_id': 37}, {'name': 'Bovespa', 'index': 38, 'portfolio_name': 'Bovespa', 'portfolio_id': 38}, {'name': 'MSCI World', 'index': 39, 'portfolio_name': 'MSCI World', 'portfolio_id': 39}], 'default': {'name': 'S&P 500', 'index': 22, 'portfolio_name': 'S&P 500', 'portfolio_id': 22}}
- PortfolioSector: {'type': 'string', 'description': 'The target benchmark portfolio/index sector', 'enums': ['Financials', 'Industrials', 'Consumer Discretionary', 'Health Care', 'Utilities', 'Information Technology', 'Real Estate', 'Energy', 'Materials', 'Communications', 'Consumer Staples'], 'default': 'Index'}

Model: gpt-4-0613

<Assistant>:
{
  "Horizon": 2070,
  "Scenario": {
    "id": "Hot House",
    "name": ">4°"
  },
  "Benchmark": {
    "name": "NIKKEI 225",
    "index": 23,
    "portfolio_name": "NIKKEI 225",
    "portfolio_id": 23
  },
  "BenchmarkSector": "Energy",
  "Portfolio": {
    "name": "TSX 60",
    "index": 20,
    "portfolio_name": "TSX 60",
    "portfolio_id": 20
  },
  "PortfolioSector": "Energy"
}
72.48s over 10 runs

Model: gpt-3.5-turbo-0613

<Assistant>:
{
  "Horizon": 2070,
  "Scenario": {
    "id": "Hot House",
    "name": ">4°"
  },
  "Benchmark": {
    "name": "NIKKEI 225",
    "index": 23,
    "portfolio_name": "NIKKEI 225",
    "portfolio_id": 23
  },
  "BenchmarkSector": "Energy",
  "Portfolio": {
    "name": "TSX 60",
    "index": 20,
    "portfolio_name": "TSX 60",
    "portfolio_id": 20
  },
  "PortfolioSector": "Energy"
}
28.16s over 10 runs
```

Token `usage` (`gpt-4-0613`): `(1848 * 0.03 / 1000) + (140 * 0.06 / 1000) = $0.06384` per call, same as `gpt-4-0314`.
```json
{
  "prompt_tokens": 1848,
  "completion_tokens": 113,
  "total_tokens": 1961
}
```

Token `usage` (`gpt-3.5-turbo-0613`): `1961 * 0.0015 / 1000 = $0.0029415` per call.
```json
{
  "prompt_tokens": 1848,
  "completion_tokens": 113,
  "total_tokens": 1961
}
```

The result is excellent, where even though the speed is not as fast, and cost not as cheap, as OpenAI's in-house _Function Calling_ implementation, the same staple system and user prompt-engineering implementation is **5 times faster and 20 times cheaper** over `gpt-4-0314`, so all we had to do was a line of code change:

![code change](https://github.com/RiskThinking/work-samples/assets/2837532/f9e7e764-2a1e-4338-b778-427a7e2efcc1)

And take a look at the response time in the actual web app usage after the navigational assistant's update:

<video controls>
  <source src="https://github.com/RiskThinking/work-samples/assets/2837532/396120a6-fcaf-4a6c-aba2-a1d859854b4d" type="video/mp4">
  Your browser does not support HTML video.
</video>

## Recap

At [RiskThinking.AI](https://riskthinking.ai/), we embarked on a project to develop a navigational assistant using OpenAI's GPT model. Initially, we had to use `gpt-4-0314` as the model of choice due to system steerability not being enabled in `gpt-3.5-turbo-0301`. However, since OpenAI's June 13 update, system steerability became available for `gpt-3.5-turbo`, significantly improving our navigational assistant's speed and cost-effectiveness, now five times faster and twenty times cheaper.

While the June 13 update's _Function Calling_ feature shows promise and provides the fastest response time, we proved there's still room for further improvement.

![table](https://github.com/RiskThinking/work-samples/assets/2837532/0d7a2764-d81b-42f4-aa0d-db3706352472)

> FC = Function Calling
