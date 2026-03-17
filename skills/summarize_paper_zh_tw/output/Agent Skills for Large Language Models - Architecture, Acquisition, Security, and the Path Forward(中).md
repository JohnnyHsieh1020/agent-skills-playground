# Agent Skills for Large Language Models: Architecture, Acquisition, Security, and the Path Forward

- 研究領域：LLM Agent、工具使用（Tool Use）、Computer-Use Agent（CUA）、AI 安全治理
- 主要問題：如何將「技能（Skill）」作為可攜、可組合、可治理的模組，補足大型語言模型在真實任務中的程序性知識缺口
- 一句話總結：本文提出並系統化「Agent Skills」生態，從架構、獲取、部署到安全風險進行整合，並給出分級信任治理框架，主張技能層將成為下一代 Agent 系統的核心抽象

## 論文原始章節地圖

- 1 Introduction
- 2 Background and Scope
  - 2.1 From Prompt Engineering to Skill Engineering
  - 2.2 Relationship to Prior Work
  - 2.3 Scope and Methodology
- 3 Architectural Foundations
  - 3.1 The SKILL.md Specification
  - 3.2 Skill Execution Lifecycle
  - 3.3 The Agentic Stack: Skills and MCP
  - 3.4 Advanced Tool Use Integration
- 4 Skill Acquisition and Learning
  - 4.1 Human-Authored Skills
  - 4.2 Reinforcement Learning with Skill Libraries
  - 4.3 Autonomous Skill Discovery
  - 4.4 Structured Skill Bases
  - 4.5 Compositional Skill Synthesis
  - 4.6 Skill Compilation: Multi-Agent to Single-Agent
- 5 Deployment: The Computer-Use Agent Stack
  - 5.1 GUI Agent Architectures
  - 5.2 GUI Grounding Advances
  - 5.3 Benchmark Landscape
- 6 Security of Agent Skills
  - 6.1 Prompt Injection via Skills
  - 6.2 Vulnerabilities at Scale
  - 6.3 Confirmed Malicious Skills
  - 6.4 Toward a Governance Framework
- 7 Open Challenges and Research Agenda
- 8 Discussion
- 9 Conclusion

## 1. Introduction

### 核心重點

- 技能（Agent Skills）是將程序性知識外部化的模組化封裝，可在不微調模型的情況下動態擴充能力
- 與 RAG 相比，技能不只提供被動資訊，而是提供可執行流程、工具使用策略與權限脈絡
- 本文主張 Skills + MCP 形成新興 agentic stack：Skills 定義「做什麼」，MCP 定義「怎麼連接」

### 詳細說明

- **直觀解釋**：若把 LLM 視為「通才員工」，skill 就是可即插即用的 SOP 與工具包，讓代理在任務當下立即具備領域作業能力
- **技術解釋**：skill 是檔案系統中的目錄結構，核心是 `SKILL.md`，可搭配腳本、參考文件、資產，透過觸發機制注入上下文
- **為何重要**：避免每次都靠超長提示詞重建流程，也避免每個新任務都做昂貴微調；更有利於版本控管與團隊共享
- **常見誤解**：skill 不是單一工具函式；它是「程序知識層」，會改變 agent 的推理與行動準備狀態

### 關鍵術語

- **Agent Skills**：可組合、可攜、按需載入的程序知識封裝
- **Progressive Disclosure**：分層載入資訊以節省上下文
- **Agentic Stack**：由技能層與連接層（MCP）組成的代理系統堆疊

### 本章小結

- 導論奠定本文主軸：skill 是把「能力」由模型權重轉移到可治理的外部模組，並且正在快速成為產業共識

## 2. Background and Scope

### 核心重點

- 能力擴充典範由 Prompt Engineering → Tool Use → Skill Engineering 演進
- 技能工程強調可重用、可治理與可攜，超越早期工具調用的原子操作
- 本文專注技能抽象層，而非泛化的 agent survey

### 詳細說明

- **直觀解釋**：Prompt 像臨時口頭交辦；Tool 像叫單一 API；Skill 像完整作業手冊 + 執行工具箱
- **技術解釋**：skill bundle 可包含指令、流程、程式碼與資源，能描述多步驟決策路徑，而非一次性函式調用
- **與先前工作差異**：Voyager、Toolformer 等偏向模型內生或受限環境的工具/技能生成；本文焦點是人類可審計、可部署的 `SKILL.md` 生態
- **方法學**：透過 arXiv、ACL、NeurIPS/ICML/ICLR 與 Anthropic 公開資料系統檢索，形成聚焦技能層的整理

### 關鍵術語

- **Prompt Engineering**：以提示詞誘導行為
- **Function Calling / Tool Use**：以明確介面呼叫外部工具
- **Skill Engineering**：以技能包管理程序知識

### 本章小結

- 本章釐清本文邊界：關注可移植程序知識模組，而非一般 agent 架構大全

## 3. Architectural Foundations

### 核心重點

- `SKILL.md` 與三層漸進揭露（metadata/instructions/resources）是核心架構創新
- skill 觸發會先改變 agent 上下文與可用能力，再進入任務解決
- Skills 與 MCP 互補：前者提供程序知識，後者提供標準化連接能力

### 詳細說明

- **直觀解釋**：像先看目錄（L1），需要時讀章節（L2），最後才打開附錄與工具（L3）
- **技術解釋**：
  - L1：啟動時只預載 YAML metadata（極低 token 成本）
  - L2：匹配任務後載入完整 `SKILL.md` 指令
  - L3：執行時按需載入腳本與資源，理論上可無上限擴展深度
- **Skills vs MCP（Table 1）**：
  - Skills 的單位是目錄/文件，目標是程序知識注入與權限脈絡重塑
  - MCP 的單位是 server/endpoint，目標是資料與工具可連接性
- **Advanced Tool Use 整合**：工具搜尋、程式化調用、工具學習，降低大規模工具場景下的 token 與調用錯誤成本

### 複雜概念拆解

- **概念：技能執行是「準備代理」而非「直接出答案」**
  - 直觀解釋：skill 像任務前 briefing，不是直接替你完成任務
  - 技術解釋：注入隱式上下文 + 開啟授權工具，改變的是策略空間與行動可達域
  - 為何重要：可讓同一模型在不同任務上下文下表現出不同專長
  - 常見誤解：把 skill 當成 prompt 模板或 function wrapper

### 關鍵方程式（如有）

- 本章無論文明示數學方程式；僅有架構分層與流程描述

### 表格數據解讀（如有）

- **Table 1 指標定義**：比較維度包含角色、單位、載入方式、上下文影響、持久性、標準狀態
- **基本趨勢**：Skills 偏向知識與流程，MCP 偏向連接與介面；兩者非替代關係而是堆疊關係
- **深層洞察**：業界標準化將能力拆成「知識模組化」與「連接模組化」兩條軸線，有助降低平台鎖定

### 關鍵術語

- **SKILL.md**：技能封裝的核心規格檔，描述觸發、任務與流程指令
- **Skill Router**：根據語意匹配決定是否載入某項技能的路由機制
- **MCP (Model Context Protocol)**：標準化代理與外部工具/資料來源連接的協定層

### 本章小結

- 本章建立技能層的工程骨架，指出其與 MCP 的邊界與耦合方式，是後續學習、部署與治理的共同前提

## 4. Skill Acquisition and Learning

### 核心重點

- 技能獲取可分為四大來源：人類編寫、RL 技能庫、自主探索、組合/編譯
- 研究趨勢由「手工技能」快速走向「可自我進化技能生態」
- 指標顯示技能庫可同時提升任務成功率與效率（步數、token）

### 詳細說明

- **4.1 Human-Authored**：門檻低、落地快，企業可直接封裝內部流程，並透過審核機制上架
- **4.2 SAGE（RL + Skill Library）**：以 sequential rollout 讓先前任務產生的技能可重用，並在 reward 中鼓勵可重用技能
- **4.3 SEAgent（Autonomous Discovery）**：世界狀態模型 + 課程產生器，讓代理在未知軟體環境中遞進式學習
- **4.4 CUA-Skill（Structured Skill Base）**：以參數化執行圖與組合圖，顯式表示前置條件與可組合規則
- **4.5 Agentic Proposing（Compositional Synthesis）**：任務中動態選擇與組合推理技能
- **4.6 Skill Compilation**：多代理系統可壓縮為單代理技能庫，但存在選擇準確率相變（phase transition）

### 複雜概念拆解

- **概念：技能庫規模與路由品質的相變**
  - 直觀解釋：技能越多不一定越好，超過臨界點會「選錯技能」
  - 技術解釋：路由器在高維技能空間的檢索/排序誤差放大，導致有效策略命中率下降
  - 為何重要：企業技能治理需做分層索引、版本裁剪與領域隔離
  - 常見誤解：只要擴增技能數量就能線性增強能力

### 關鍵方程式（如有）

- 成功率提升可形式化為：
  $$
  \Delta \mathrm{SR} = \mathrm{SR}_{\text{with-skill}} - \mathrm{SR}_{\text{baseline}}
  $$
- 例：SEAgent 在 OSWorld（五個新軟體）
  $$
  \Delta \mathrm{SR} = 34.5\% - 11.3\% = 23.2\%\text{ points}
  $$

### 表格數據解讀（如有）

- **Table 2 指標定義**：以各方法在原論文 benchmark 的核心指標（成功率、分數、token/步數效率）比較
- **基本趨勢**：
  - 人工技能顯示生態採納速度快（GitHub star 高）
  - RL/自主探索路線顯著提高成功率
  - 結構化知識表示與組合式推理在複雜任務具優勢
- **可能原因**：技能作為中介記憶，使跨任務知識遷移更穩定，降低每次從零規劃的成本
- **更深層洞察**：未來競爭焦點可能從「模型參數規模」轉向「技能生態品質（可重用/可組合/可治理）」

### 關鍵術語

- **SAGE**：結合 GRPO 與技能庫的自我演化強化學習方法
- **SEAgent**：以世界模型與課程生成器驅動的自主技能探索框架
- **Skill Compilation**：將多代理協作策略壓縮為單代理技能庫的過程

### 本章小結

- 本章展示技能獲取已形成完整技術光譜，且「會學習的技能庫」正成為 agent 持續進化的核心機制

## 5. Deployment: The Computer-Use Agent Stack

### 核心重點

- CUA 是技能最重要的實戰場景，因 GUI 任務天然需要多步驟感知-推理-行動鏈
- GUI grounding（定位可互動元素）是效能瓶頸與突破口
- 基準結果顯示 CUA 效能已快速逼近甚至超過部分人類基準

### 詳細說明

- **5.1 架構層面**：UI-TARS、UI-TARS-2、Agent S2、OpenCUA 等模型從感知、動作建模、RL 穩定性持續升級
- **5.2 Grounding 進展**：大規模 GUI 視覺資料集與自演化訓練使小模型在特定任務上可超越大模型
- **5.3 基準觀察**：OSWorld 指標從早期低成功率躍升至接近/超越人類；但長時程與專業工作流仍存在明顯落差

### 複雜概念拆解

- **概念：為何 GUI 任務適合技能化**
  - 直觀解釋：點擊、輸入、檔案操作、錯誤恢復都可被 SOP 化
  - 技術解釋：GUI 任務狀態轉移可由技能圖（skill graph）分解，並透過記憶/回饋形成可重用策略
  - 為何重要：這是技能在企業流程自動化中的最大落地入口
  - 常見誤解：把 GUI agent 看成純視覺模型問題，忽略程序知識的重要性

### 關鍵方程式（如有）

- 人類對比差值可表示為：
  $$
  \Delta_{\text{human}} = \mathrm{SR}_{\text{agent}} - \mathrm{SR}_{\text{human}}
  $$
- 例：OSWorld-V（文中）
  $$
  \Delta_{\text{human}} = 72.6\% - 72.4\% = 0.2\%\text{ points}
  $$

### 表格數據解讀（如有）

- **Table 3 指標定義**：`SR`（Success Rate）衡量任務成功比例
- **基本趨勢**：
  - OSWorld：最佳代理 59.9，OSWorld-V 已達 72.6（高於 72.4 人類基準）
  - AndroidWorld：73.3，顯示行動端 GUI 任務進步快速
  - SWE-bench Verified：79.2，說明技能化思維可外溢到程式工程任務
- **可能原因**：技能模組把高頻操作流程沉澱為可重用策略，降低每回合探索負擔
- **更深層洞察**：雖有「平均指標亮眼」，但極端情境（長鏈任務、跨工具依賴、專業軟體）仍是下一波主戰場

### 關鍵術語

- **CUA (Computer-Use Agent)**：可在作業系統/GUI 環境中自主執行操作的代理
- **GUI Grounding**：將語意目標精準對應到畫面可互動元素的能力
- **OSWorld / AndroidWorld / SWE-bench**：衡量跨平台任務完成度與工程實作能力的重要基準

### 本章小結

- 部署層面證明技能不是概念性包裝，而是可在高複雜度 GUI 工作流中帶來實質性能與效率收益

## 6. Security of Agent Skills

### 核心重點

- 技能生態快速成長同時暴露新攻擊面：提示注入、資料外洩、權限提升、供應鏈風險
- 實證研究顯示社群技能漏洞率高（26.1%），且含腳本技能風險顯著更高
- 論文提出分級信任 + 驗證閘門 + 執行期監控的治理框架

### 詳細說明

- **6.1 Prompt Injection**：攻擊者可在長 `SKILL.md` 或引用腳本隱藏惡意指令，繞過使用者直觀審核
- **6.2 大規模漏洞分析**：在 42,447 個技能中，31,132 個可分析樣本顯示 26.1% 含至少一個漏洞
- **6.3 惡意技能驗證**：確認兩類攻擊原型——Data Thieves（偷憑證）與 Agent Hijackers（劫持決策）
- **6.4 治理框架**：
  - G1 靜態分析
  - G2 語意意圖分類
  - G3 沙盒行為審計
  - G4 權限宣告對照
  並映射到 T1–T4 信任等級（最小權限原則）

### 複雜概念拆解

- **概念：為何「隱式信任」在技能層失效**
  - 直觀解釋：只要技能被載入，代理就可能把其中指令當「可信任內部規則」
  - 技術解釋：progressive disclosure 使 L2/L3 內容在執行期才暴露，增加審計難度與延遲發現風險
  - 為何重要：若無分級權限，單一惡意技能可跨工具鏈擴散
  - 常見誤解：以為「有人工審核」即可覆蓋動態行為風險

### 關鍵方程式（如有）

- 漏洞機率比（論文報告）可寫為：
  $$
  \mathrm{OR} = \frac{P(V\mid \text{script-bundled})/(1-P(V\mid \text{script-bundled}))}{P(V\mid \text{instruction-only})/(1-P(V\mid \text{instruction-only}))} = 2.12
  $$
- 意義：含可執行腳本的技能其漏洞風險顯著高於僅指令技能

### 表格數據解讀（如有）

- 本章主要以 Figure 3（治理框架）整合資料，非傳統數字表格
- **趨勢解讀**：
  - L1 metadata 風險最低
  - L2 instructions 為提示注入高風險區
  - L3 scripts 風險最高（可執行行為）
- **深層洞察**：治理策略必須與架構層級對齊，不能只做「一次性上架審核」

### 關鍵術語

- **Prompt Injection**：以自然語言指令污染代理決策流程的攻擊手法
- **Permission Manifest**：技能宣告所需能力邊界（工具、路徑、網路）的權限清單
- **Trust Tier (T1–T4)**：依驗證深度與來源可信度分配的分級部署權限

### 本章小結

- 安全議題已非邊緣問題，而是技能生態能否規模化的決定因素；本文框架提供具工程可行性的治理起點

## 7. Open Challenges and Research Agenda

### 核心重點

- 論文提出七大開放問題：可攜性、大規模選擇、組合協同、能力型權限、驗證測試、持續學習、評估方法
- 主要瓶頸從「能不能做」轉向「如何可靠、可驗證、可維運地做」

### 詳細說明

- **Challenge 1**：跨平台可攜仍受限於特定平台執行語義
- **Challenge 2**：技能庫規模化導致路由困難與組合爆炸
- **Challenge 3**：多技能編排缺乏成熟衝突解決與故障恢復機制
- **Challenge 4**：需由隱式信任改為 capability-based permissions
- **Challenge 5**：缺乏技能專屬測試框架與 CI/CD 準則
- **Challenge 6**：持續學習需避免災難性遺忘與能力覆寫
- **Challenge 7**：缺少對 skill 可重用、可組合、可維護性的直接評估指標

### 關鍵術語

- **Capability-Based Permission**：按能力宣告授權，而非全域放權
- **Catastrophic Forgetting**：新技能學習造成舊能力退化
- **Skill Quality Metrics**：重用性、組合性、維護性

### 本章小結

- 研究議程已明確轉向「技能工程學（Skill Engineering）基礎設施化」：標準、測試、治理三位一體

## 8. Discussion

### 核心重點

- 技能典範代表從「單體智慧」到「模組化專長」的系統轉向
- 開放生態可形成網路效應，但也同步放大供應鏈與治理風險
- 技能學習與技能部署之間仍存在可審計性落差

### 詳細說明

- 對組織層面，skill 可作為可持續保存的數位 SOP，降低人員流動造成的知識流失
- 對生態層面，開放標準提高互操作與共享速度，促進「技能市場」形成
- 對研究層面，RL 自學技能若停留模型內部，將難以審核、共享與治理；需外化為可攜 artifacts

### 關鍵術語

- **Modular Expertise**：可替換、可重組的專長模組
- **Pre-Governance Phase**：生態先成長、治理後補課的早期階段

### 本章小結

- 討論章強調：技能革命的真正門檻不在能力展示，而在治理能力是否跟上採納速度

## 9. Conclusion

### 核心重點

- Agent Skills 已成為新一代 LLM agent 的基礎抽象層
- 本文完成四維整合：架構、獲取、部署、安全
- 未來關鍵在可審計技能學習、可用且安全的權限模型、與技能品質評測體系

### 詳細說明

- 產業在短時間內快速收斂到 skill abstraction，顯示其工程價值明確
- 下一階段不只是提升任務成功率，而是建立可信、可持續演進的技能生態

### 關鍵術語

- **Trustworthy Skill Ecosystem**：可信任且可持續治理的技能生態
- **Inspectable Artifacts**：可檢查、可審計、可追溯的技能產物

### 本章小結

- 本文結論可概括為：技能化是 agent 系統從「模型能力」走向「系統能力」的必要演化

## 全文整體洞察

- 技術主線：以 `SKILL.md` 為核心的程序知識外部化，正把代理能力從參數空間轉移到可管理的技能空間
- 工程主線：progressive disclosure 解決上下文成本，MCP 解決連接標準，兩者形成可擴展 agentic stack
- 實證主線：CUA 與多基準數據證明技能化能同時帶來性能提升與執行效率收益
- 安全主線：26.1% 漏洞率與 2.12× 風險比顯示治理不可選配；必須採分級信任與持續監控
- 研究主線：下一步不是「更多技能」，而是「更可驗證、更可編排、更可攜、更可治理的技能系統」
