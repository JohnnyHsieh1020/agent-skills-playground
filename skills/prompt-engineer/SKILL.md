---
name: prompt-engineer
description: 幫助使用者將模糊的任務描述轉化為結構化、高品質的 prompt。當使用者說「幫我寫一個 prompt」、「我想讓 AI 幫我做 X」、「設計一個給 LLM 用的指令」、「我需要一個 agent 的 prompt」，或任何涉及產出 prompt、system prompt、AI 指令的需求時，務必使用此 skill。即使使用者只是描述一個任務而沒有明確說「寫 prompt」，只要情境暗示他們需要一個可重複使用的 AI 指令，也應觸發此 skill。
---

# Prompt Engineer Skill

幫助使用者將任務需求轉化為結構化的高品質 prompt，適用於 LLM（Claude、ChatGPT 等）或 AI agent / workflow 工具。

---

## 工作流程

### 第一步：理解初始需求

使用者描述任務後，立刻產出一份 **prompt 草稿**，同時附上 **2–4 個釐清問題**，幫助精煉需求。

在草稿之前，用一句話說明你對需求的理解，例如：
> 「我理解你想要一個能幫你 X 的 prompt，以下是初版草稿：」

在草稿之後，附上釐清問題，格式如下：
> 💬 **需求確認**
> 1. ...
> 2. ...
>
> 當你覺得需求已對齊，請告訴我「可以了」，我會產出最終版本。

### 第二步：來回精煉

根據使用者的回答，更新 prompt 草稿。每次更新都要：
- 標示本次修改的重點（一行說明）
- 繼續附上剩餘的釐清問題（若還有疑點）
- 重複提醒使用者：「滿意後請說『可以了』」

持續迭代，直到使用者確認對齊。

### 第三步：產出最終 prompt

使用者說「可以了」或表示滿意後，產出**乾淨的最終版 prompt**（不含任何說明文字，直接可複製使用）。

---

## Prompt 結構規範

使用 Markdown 區塊，根據任務複雜度選擇適合的區塊組合。

### 標準區塊（依需求選用）

```
## Role
[AI 扮演的角色與專業背景]

## Context
[任務的背景資訊、使用場景、目標受眾]

## Task
[具體要完成的任務，動詞開頭，清楚描述]

## Steps
[執行步驟，適用於有明確流程的任務]
1. ...
2. ...

## Output Format
[輸出的格式、長度、語言、結構要求]

## Constraints
[限制條件、禁止事項、邊界條件]

## Examples
[few-shot 範例，適用於需要示範風格或格式的任務]
Input: ...
Output: ...
```

### 區塊選用原則

| 任務類型 | 建議包含的區塊 |
|---------|-------------|
| 簡單單一任務 | Role, Task, Output Format |
| 複雜多步驟任務 | Role, Context, Task, Steps, Output Format, Constraints |
| AI Agent / Workflow | Role, Context, Task, Steps, Output Format, Constraints |
| 需要風格模仿 | Role, Task, Examples, Constraints |
| Few-shot 學習 | Task, Examples, Output Format |

---

## 高品質 Prompt 的核心原則

### 1. 角色設定要具體
❌ `You are a helpful assistant.`
✅ `You are a senior UX researcher with 10 years of experience conducting user interviews for B2B SaaS products.`

### 2. 任務描述要可執行
❌ `Help me with my marketing.`
✅ `Write a LinkedIn post announcing a new product feature. The post should drive traffic to the product page and encourage comments.`

### 3. 輸出格式要明確
❌ `Give me a summary.`
✅ `Provide a summary in bullet points (max 5 bullets), each under 20 words, in Traditional Chinese.`

### 4. 約束條件要清楚邊界
- 語言限制（繁體中文 / 英文 / 雙語）
- 長度限制（字數、段落數、bullet 數）
- 禁止行為（不要加入個人意見、不要使用術語等）
- 目標受眾（技術背景 / 非技術背景）

### 5. Agent Prompt 額外注意
- 明確定義 agent 的「能力邊界」（它能做什麼、不能做什麼）
- 定義錯誤處理行為（遇到不確定時應該怎麼做）
- 若有 tool use，描述何時應呼叫哪個工具

---

## 來回對話中常用的釐清問題

視情況從以下問題中挑選最相關的 2–4 個：

**關於角色與背景**
- 這個 prompt 會給什麼樣的 AI 使用？（Claude / GPT / 開源模型）
- 使用者是誰？有什麼背景知識？
- 這個 AI 在什麼情境下被使用？（standalone chat / embedded in app / agent pipeline）

**關於任務**
- 任務是一次性的還是重複性的？
- 輸入資料是什麼形式？（文字、表格、程式碼、圖片描述）
- 最重要的成功標準是什麼？

**關於輸出**
- 輸出語言？（中文 / 英文 / 依輸入而定）
- 輸出長度或格式有限制嗎？
- 輸出會被人讀還是被程式解析？

**關於限制**
- 有什麼絕對不能出現在輸出中的內容？
- 有品牌語調或風格要求嗎？