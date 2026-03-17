# ARES: Adaptive Reasoning Effort Selection for Efficient LLM Agents

- 研究領域：LLM Agent、推理效率最佳化（Reasoning Efficiency）、動態路由（Routing）、強化學習（RL）
- 主要問題：在多步驟 agent 任務中，如何動態選擇每一步推理強度（low/medium/high），在維持成功率前提下顯著降低推理 token 成本
- 一句話總結：本文提出 ARES，以輕量 router 逐步預測最小足夠推理強度，並結合 SFT 與 GRPO 訓練，在 TAU-Bench、BrowseComp-Plus、WebArena 上達到接近高強度推理的效能，同時最高節省 52.7% 推理 token

## 論文原始章節地圖

- 1. Introduction
- 2. Related Work
- 3. Method
  - 3.1. Problem Formulation
  - 3.2. Supervised Fine-tuning Pipeline
  - 3.3. Reinforcement Learning
- 4. Experiments
  - 4.1. Experimental Setup
  - 4.2. SFT Results
  - 4.3. RL Results
  - 4.4. Analysis of Reasoning Effort Selection
  - 4.5. Ablation Studies
  - 4.6. Generalization Evaluation
- 5. Conclusion
- Appendix
  - A. Prompts
  - B. Training Example
  - C. Dataset Statistics

## 1. Introduction

### 核心重點

- 長鏈式推理雖提升 agent 表現，但在多步驟任務會累積高昂 token 成本
- 固定使用單一推理強度（永遠 low 或永遠 high）常無法兼顧效能與成本
- ARES 主張逐步（per-step）動態選擇推理強度，而非整體固定策略

### 詳細說明

- **直觀解釋**：任務中的步驟難度不一致，簡單步驟不需高成本深度思考，困難步驟才需要高推理強度
- **技術解釋**：ARES 在每一步讀取歷史互動與當前觀測，預測下一步最小足夠推理等級，讓下游 agent 以該等級產生行動
- **與傳統 model routing 差異**：不是在不同模型間切換，而是在同一模型內切換 reasoning level，避免跨模型重編碼與 KV cache 無法重用
- **為何重要**：多步驟 agent 容易出現前步失誤連鎖效應，細粒度推理預算分配比粗粒度全域設定更有效

### 關鍵術語

- **Reasoning Effort**：推理強度設定（`low` / `medium` / `high`）
- **ARES Router**：預測每一步推理強度的輕量模型
- **KV Cache Reuse**：在同模型不同推理模式間保留上下文快取，降低額外延遲

### 本章小結

- 導論建立核心動機：在 agent 任務中，推理成本管理問題本質上是「逐步資源配置」問題，而非單次推理問題

## 2. Related Work

### 核心重點

- 既有 LLM routing 多聚焦單輪任務，難處理多輪決策相依性
- 既有 adaptive reasoning 多在數學/程式等單步設定，較少落地於多步驟 agent
- ARES 將動態推理選擇建模為序列決策，貼近實際 agent 工作流

### 詳細說明

- **LLM Routing 路線**：早期方法根據任務路由到不同模型；問題在多模型成本-效能邊界不穩定，且切換代價高
- **Adaptive Thinking 路線**：常見方法調整思考長度或預算，但多假設單步輸入輸出，較少考慮步驟間錯誤傳播
- **ARES 的定位**：把推理強度選擇嵌入多輪 agent 互動，每一步都依狀態重新決策
- **關鍵差異點**：明確處理「步驟間耦合」與「長鏈任務成本累積」

### 關鍵術語

- **Model Routing**：在多模型中選擇一個模型處理當前輸入
- **Single-turn vs Multi-turn**：單輪獨立決策與多輪序列決策
- **Error Propagation**：早期錯誤在後續步驟被放大

### 本章小結

- 本章指出研究缺口：多步驟 agent 的推理資源分配尚缺通用、可插拔且高效率方案，ARES 即填補此缺口

## 3. Method

### 核心重點

- ARES 將路由器目標定義為「成功率最大化 + 推理成本最小化」
- SFT 透過三階段資料管線建立高品質逐步標籤（最小足夠 effort）
- RL（GRPO）補足 SFT 的單步貪婪限制，優化全軌跡效率-效能平衡

### 詳細說明

- **3.1 問題定式**：
  - agent 在第 `t` 步依 `h_t, o_t, e_t` 產生行動 `a_t`
  - router 產生 `e_t ∈ {e_low, e_mid, e_high}`
  - 目標是在維持任務成功下，最小化整體 token 成本
- **3.2 SFT 管線（核心工程貢獻）**：
  1) 收集成功且步數最短的軌跡作為參考路徑
  2) 對每一步在三種 effort 下多次採樣，找出可穩定重現正確行動的最低 effort
  3) 由 teacher 生成 3–5 句 rationale，讓 router 學會「先判斷難度、再輸出標籤」
- **3.3 RL 優化**：
  - 使用 GRPO，獎勵由 outcome、cost、format 三部分組成
  - cost 僅在成功軌跡計入，避免 router「故意快速失敗」以規避成本懲罰
  - 加入資料過濾（如去除零成功樣本）強化訓練訊號

### 複雜概念拆解

- **概念：最小足夠推理強度（minimum sufficient effort）**
  - 直觀解釋：不是找「最強」推理，而是找「剛好能正確完成當步」的最便宜推理
  - 技術解釋：在每步上用多次採樣與等價驗證函式，計算可穩定重現 ground truth action 的候選 effort 集，再取最低成本者
  - 為何重要：將原本指數級組合搜尋問題分解為可監督的逐步分類問題
  - 常見誤解：以為 low effort 永遠最省；實際若導致後續失敗，總成本反而更高

### 關鍵方程式（如有）

- agent 行動分佈：
  $$
  a_t \sim P_{\phi}(a_t\mid h_t, o_t, e_t)
  $$
- 整體最佳化目標：
  $$
  \max_{\theta} \ \mathbb{E}_{x\sim\mathcal{X},\tau\sim\mathcal{T}(\theta,\phi)}\left[V(\tau, x)-\lambda\sum_{t=1}^{T}\mathrm{cost}(e_t)\right]
  $$
- 充足 effort 候選集合：
  $$
  C_t = \left\{e\in\mathcal{E}\ \middle|\ \sum_{k=1}^{K} V\big(\hat a^{(e)}_{t,k}, a_t^{*}\big) \ge M\right\}
  $$
- RL 總獎勵：
  $$
  R(\tau)=
  \begin{cases}
  R_{out}+R_{cost}+R_{form}, & \text{成功}\\
  R_{out}+R_{form}, & \text{失敗}
  \end{cases}
  $$
- 成本懲罰（文中設定）：
  $$
  c(e_t)=
  \begin{cases}
  -0.2,& e_t=e_{low}\\
  -0.5,& e_t=e_{mid}\\
  -1.0,& e_t=e_{high}
  \end{cases},\quad
  R_{cost}=\frac{1}{T}\sum_{t=1}^{T} c(e_t)
  $$

### 表格數據解讀（如有）

- 本章以 Figure 2 流程圖為主，重點不在數值表格
- **趨勢解讀**：方法設計強調「標籤品質」與「訓練訊號品質」先行，再做策略優化
- **深層洞察**：ARES 的成功關鍵不僅是路由模型本身，而是資料構建（成功軌跡、逐步驗證、rationale）

### 關鍵術語

- **SFT（Supervised Fine-tuning）**：監督式微調，學習逐步 effort 預測
- **GRPO**：群組相對策略最佳化，用於 router 強化學習
- **Rationale Generation**：先生成短推理說明再輸出 effort 標籤

### 本章小結

- 方法章核心貢獻是把多步驟推理預算分配轉化為可訓練、可驗證、可擴展的路由問題，並以 SFT+RL 形成完整閉環

## 4. Experiments

### 核心重點

- ARES 在多種 agent 基準（tool-use、deep-research、web）均展現穩定成本下降
- SFT 版本已能大幅接近 high-effort 上限；RL 進一步提升成功率與效率
- 消融與泛化實驗顯示：rationale、reward normalization、跨尺度遷移皆關鍵

### 詳細說明

- **4.1 Setup**：主幹為 `gpt-oss-20b`，router 為 `Qwen3-1.7B`；評測含 TAU-Bench、BrowseComp-Plus、WebArena
- **4.2 SFT 結果（Table 1）**：
  - Retail：ARES 準確率 54.8%，與 High 相同，但 `Ttotal` 652k 相較 High 1007k，約降 35.2%
  - BrowseComp-Plus：41.3% vs High 42.7%，`Ttotal` 1071k vs 1841k，約降 41.8%
  - WebArena：46.5% 高於 High 45.0%，`Ttotal` 1512k vs 2763k，約降 45.3%
- **4.3 RL 結果（Table 2）**：
  - Retail：ARES-RL 58.5%，高於 SFT 的 54.8%，且 `Ttotal` 476k 低於 SFT 的 652k
  - Airline：ARES-RL 42.0%，達到最佳準確率並將 `Ttotal` 壓至 133k（相較 SFT 678k 顯著下降）
- **4.4 Effort 選擇分析（Figure 3）**：
  - 任務早期多選 low
  - 任務後段高 effort 比例上升
  - `goback`/`branch` 等高風險動作更常觸發 high effort
- **4.5 消融（Table 3、4）**：
  - 移除 SFT：54.8% → 41.7%，顯示任務特化不可缺
  - 移除 rationale：54.8% → 51.3%，顯示顯式難度分析有助路由
  - 未正規化 cost reward：Airline 同精度但 token 較高（157k vs 133k）
- **4.6 泛化（Table 5）**：
  - router 雖訓練於 `gpt-oss-20b`，在 `gpt-oss-120b` 仍達 65.2%
  - 相較 high 的 67.8%，以約 23% token 降幅換取小幅精度差距

### 複雜概念拆解

- **概念：Overthinking 風險與中等推理優勢**
  - 直觀解釋：想太多不一定更好，反而可能讓決策偏離
  - 技術解釋：在 Airline 任務中，固定 High（38.0%）低於 Medium（42.0%）；RL 學到降低 high 使用率並提高有效 effort 配置
  - 為何重要：證明「推理強度」不是單調增益，必須由狀態驅動動態分配
  - 常見誤解：把 high effort 當成通用最優策略

### 關鍵方程式（如有）

- 成本節省率可表示為：
  $$
  \mathrm{Saving}(\%) = \frac{T_{\text{baseline}}-T_{\text{ARES}}}{T_{\text{baseline}}}\times 100\%
  $$
- 例（Retail, SFT vs High）：
  $$
  \frac{1007k-652k}{1007k}\approx 35.2\%
  $$
- 例（WebArena, SFT vs High）：
  $$
  \frac{2763k-1512k}{2763k}\approx 45.3\%
  $$

### 表格數據解讀（如有）

- **Table 1（SFT 主結果）**：ARES 在三類場景普遍保留高效能並顯著降 token，顯示方法具跨任務穩定性
- **Table 2（RL 結果）**：RL 不只是省 token，還可提升成功率（Retail +3.7 points, 相對 SFT）
- **Table 3（Retail 消融）**：SFT 與 rationale 皆有可量化貢獻，且 SFT 是主效因子
- **Table 4（Airline 消融）**：reward normalization 改善效率-效能協調（約再省 15% token）
- **Table 5（跨尺度泛化）**：路由策略可遷移至更大 backbone，顯示 effort cue 具尺度不變性
- **Table 6（附錄資料統計）**：
  - TAU-Bench：43,358 筆（Low 24,875 為主）
  - BrowseComp-Plus：12,366 筆（High/Medium/Low 分布較均衡）
  - WebArena：1,718 筆（High 比例偏高）
  - **洞察**：不同環境對推理強度需求分布差異大，支持「動態而非固定」策略必要性

### 關鍵術語

- **Ttotal / Ttask / Tstep**：總 token、每任務平均 token、每步平均 token
- **Overthinking**：高推理強度造成不必要推理漂移與效率損失
- **Cross-scale Generalization**：在不同模型規模間維持有效路由策略

### 本章小結

- 實驗結果清楚顯示 ARES 能在多種 agent 場景中維持接近高推理效能，並大幅降低成本；RL 與設計細節進一步放大收益

## 5. Conclusion

### 核心重點

- ARES 證明了逐步推理強度路由是高效 agent 的可行路徑
- 在成本與效能的 Pareto 折衷上，ARES 相較固定策略更優
- 方法具跨資料來源與跨模型尺度泛化能力

### 詳細說明

- 本文從方法、資料建構、訓練與評估形成完整閉環，特別強調「最小足夠 effort」標註與訓練訊號品質
- 實驗表明即使 backbone 能力改變，router 仍能遷移有效策略
- 未來方向包括多模態情境與更廣泛部署場景

### 關鍵術語

- **Pareto Trade-off**：在效能與成本間取得更佳前緣平衡
- **Plug-and-Play Router**：可插拔、可整合於既有 agent 架構的路由器

### 本章小結

- 結論可概括為：動態推理強度選擇是一條可落地且高收益的 agent 效率優化主軸

## 附錄重點（A/B/C）

### 核心重點

- 附錄提供可重現訓練的 prompt 模板、標註樣例與資料分布統計
- 顯示 ARES 不是僅概念方法，而是具完整資料工程規格

### 詳細說明

- **A. Prompts**：明確定義 router、annotator、rationale generator 的角色與輸出約束
- **B. Training Example**：示範單筆樣本如何從互動歷史推導 `High` 標籤與理由
- **C. Dataset Statistics**：揭露各 benchmark 的標籤分布，有助理解 router 偏好來源

### 關鍵術語

- **Reasoning Effort Annotator**：判斷不同 effort 是否能穩定重現正確行動
- **Rationale Template**：控制 router 說明長度與語意聚焦的模板

### 本章小結

- 附錄提高方法可復現性與可審計性，是 ARES 工程價值的重要支撐

## 全文整體洞察

- 研究層面：把「推理預算分配」從 heuristic 變成可監督、可強化學習的策略優化問題
- 系統層面：同模型內 effort 路由避開多模型切換負擔，兼顧成本與延遲
- 實證層面：在多個 agent benchmark 上，ARES 穩定逼近 high-effort 表現並大幅省 token
- 方法層面：資料標註品質（最小足夠 effort）與 reward 設計（含 normalization）是成效關鍵
- 未來方向：多模態代理、長時程任務與更強安全/可靠性約束下的動態 effort 控制
