# Personalized Weather Assistant OpenSpec 專案架構

本文件定義「個人化天氣助理」的 OpenSpec 規格專案結構、需求撰寫方式、變更提案流程及主要規格內容。

## 1. 建議目錄結構

```text
personalized-weather-assistant/
├── openspec/
│   ├── project.md
│   ├── specs/
│   │   ├── itinerary/
│   │   │   └── spec.md
│   │   ├── environment-data/
│   │   │   └── spec.md
│   │   ├── data-quality/
│   │   │   └── spec.md
│   │   ├── decision-engine/
│   │   │   └── spec.md
│   │   ├── user-preferences/
│   │   │   └── spec.md
│   │   ├── llm-explanation/
│   │   │   └── spec.md
│   │   ├── feedback/
│   │   │   └── spec.md
│   │   └── evaluation/
│   │       └── spec.md
│   └── changes/
│       ├── add-personalized-weather-assistant/
│       │   ├── proposal.md
│       │   ├── design.md
│       │   ├── tasks.md
│       │   └── specs/
│       │       ├── itinerary/spec.md
│       │       ├── environment-data/spec.md
│       │       ├── data-quality/spec.md
│       │       ├── decision-engine/spec.md
│       │       ├── user-preferences/spec.md
│       │       ├── llm-explanation/spec.md
│       │       ├── feedback/spec.md
│       │       └── evaluation/spec.md
│       └── archive/
├── src/
│   ├── api/
│   ├── domain/
│   ├── infrastructure/
│   ├── application/
│   └── presentation/
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── fixtures/
│   └── e2e/
└── README.md
```

## 2. project.md

```markdown
# Personalized Weather Assistant

## Purpose

本專案建立一套 AI Agent，整合中央氣象署氣象資料與環境部空氣品質資料，
針對使用者的多站行程與個人偏好產生可追溯的行前建議。

## In-Scope

- 臺灣境內多站行程標準化
- 氣象預報與空氣品質資料取得
- 資料時空對齊及品質檢查
- 依規則產生降雨、紫外線、空品與冷熱建議
- 個人偏好設定及偏好候選確認
- 行後回饋及建議可追溯呈現
- 固定規則、固定模板與建議型 LLM 的比較評估

## Out-of-Scope

- 移動路線規劃
- 依移動路線逐秒更新天氣
- 健康診斷或處方
- 以使用者偏好取代官方健康防護指引
- 未經確認直接更新個人耐受度
- 取得官方資料範圍外的精確預報

## Core Principles

1. 官方資料優先；LLM 不可補造資料或改寫判定。
2. 每個行程站獨立取數與判斷。
3. 健康防護與主觀舒適分離。
4. 資料品質限制必須可見。
5. 個人化內容必須可檢視、修改及刪除。
6. 結構化決策與自然語言解釋分層處理。

## Technology Boundary

- 規則引擎負責門檻、優先序、數值判斷與可重現性。
- LLM 負責需求解析、內容解釋及追問，不負責建立環境判斷。
- 所有結論必須能連回資料來源、時間、地點、規則與使用者偏好。
```

## 3. 範例：itinerary/spec.md

```markdown
# Itinerary Capability

## Purpose

將使用者輸入的行程正規化為可驗證及可處批次處理的行程段。

## Requirements

### Requirement: 行程段資料驗證

系統 SHALL 要求每個行程段至少包含臺灣地點、有效時間區間、
活動型態及戶外暴露程度。

#### Scenario: 收到完整行程段

- WHEN 使用者提供有效地點、開始時間、結束時間、活動型態及暴露程度
- THEN 系統 SHALL 建立一個具唯一識別碼的行程段
- AND 系統 SHALL 執行地點與時段標準化

#### Scenario: 缺少必要欄位

- WHEN 行程段缺少地點、時間、活動或暴露程度
- THEN 系統 SHALL 回傳缺少欄位清單
- AND 系統 SHALL NOT 產生環境判定

### Requirement: 行程站獨立處理

系統 SHALL 對每個行程站建立獨立的資料快照與決策。

#### Scenario: 多站跨鄉鎮行程

- WHEN 使用者建立早上、下午及晚上位於不同鄉鎮的行程
- THEN 系統 SHALL 分別取得及對齊各站資料
- AND 系統 SHALL 最後彙整成一份全行程建議
```

## 4. environment-data/spec.md

主要能力：

- 中央氣象署天氣預報資料取得
- 環境部即時空氣品質資料取得
- 環境部隔日空氣品質預報取得
- 鄉鎮、測站及空品區之地點對齊
- 預報起訖、發布時間及取得時間保存

需求樣例：

```markdown
### Requirement: 官方資料來源標記

系統 SHALL 保存資料來源名稱、空間參考、觀測或預報時間、
發布時間及取得時間。

#### Scenario: 顯示建議依據

- WHEN 使用者展開建議的資料依據
- THEN 系統 SHALL 顯示來源、地點粒度及有效時間
- AND 系統 SHALL 區分觀測值與預報值

### Requirement: 空品預報範圍限制

系統 SHALL NOT 將隔日空品區預報呈現為單一地點或小時觀測值。

#### Scenario: 行程超出預報範圍

- WHEN 行程時段超出環境部官方空品預報範圍
- THEN 系統 SHALL 仍提供天氣建議
- AND 系統 SHALL 標示空品尚無官方預報
- AND 系統 SHALL NOT 產生空品風險判定
```

## 5. data-quality/spec.md

主要能力：

- 缺漏偵測
- 資料過期偵測
- 空間粒度檢查
- 測站距離檢查
- 預報有效時段檢查
- 降低或停止相關判定

需求樣例：

```markdown
### Requirement: 關鍵資料品質驗證

系統 SHALL 驗證資料是否完整、最新、具有效時段且符合地點粒度。

#### Scenario: 天氣更新而空品資料過期

- WHEN 天氣預報為最新資料而空品資料已超過允許新鮮度
- THEN 系統 SHALL 保留天氣建議
- AND 系統 SHALL 標示空品資料過期
- AND 系統 SHALL 限制或停止空品風險判定

### Requirement: 禁止資料補值

系統 SHALL NOT 自動補出缺失數值或將觀測資料標示為預報資料。

#### Scenario: 資料來源無可用數值

- WHEN 關鍵資料欄位不存在
- THEN 系統 SHALL 回傳資料限制狀態
- AND 系統 SHALL NOT 由 LLM 推測數值
```

## 6. decision-engine/spec.md

規則引擎涵蓋：

- 降雨準備
- 紫外線防護
- 空氣品質防護
- 冷熱舒適
- 資料品質限制
- 建議優先序

每項建議至少包含：

- `ruleId`
- 觸發條件
- 建議動作
- 優先度
- 輸入證據
- 資料限制

需求樣例：

```markdown
### Requirement: 可重現規則判定

系統 SHALL 為每項結構化建議保存規則版本、規則編號及觸發證據。

#### Scenario: 重現既有建議

- WHEN 使用相同的資料快照、行程段、偏好與規則版本重新執行
- THEN 系統 SHALL 產生相同的結構化建議及優先序

### Requirement: 健康與舒適分離

系統 SHALL NOT 使用主觀舒適偏好刪除官方健康防護提醒。

#### Scenario: 使用者偏好低空品敏感度

- WHEN 官方空品資料觸發健康防護規則
- AND 使用者設定低空品提醒敏感度
- THEN 系統 SHALL 可降低舒適或次要提醒的呈現優先度
- AND 系統 SHALL NOT 刪除必要防護提醒
```

## 7. user-preferences/spec.md

偏好項目：

- 降雨接受度
- 防曬重視度
- 空品提醒敏感度
- 偏熱或偏冷傾向
- 偏好確認版本

需求樣例：

```markdown
### Requirement: 使用者確認後更新偏好

系統 SHALL 不直接將回饋或推估結果覆寫既有 UserProfile。

#### Scenario: 建立偏好更新候選

- WHEN 回饋資料足以產生偏好候選
- THEN 系統 SHALL 建立 pending-confirmation 狀態的候選
- AND 系統 SHALL 等待使用者接受或拒絕

#### Scenario: 使用者接受候選

- WHEN 使用者確認偏好候選
- THEN 系統 SHALL 建立新偏好版本
- AND 系統 SHALL 保留舉證摘要及更新時間
```

## 8. llm-explanation/spec.md

LLM 限制：

- 可解析自然語言補充欄位
- 可改寫既有結構化決策
- 可回答既有決策的追問
- 不可取得未記錄數值
- 不可修改優先序
- 不可補造資料來源或限制

需求樣例：

```markdown
### Requirement: 解釋限定於結構化決策

LLM SHALL ONLY 使用 TripRecommendation、SegmentDecision、
EnvironmentalSnapshot 的證據及限制產生文字。

#### Scenario: 使用者追問為何帶傘

- WHEN 使用者詢問建議原因
- THEN LLM SHALL 引用降雨規則、降雨機率、時段及暴露條件
- AND LLM SHALL NOT 改變規則判定結果

### Requirement: 不確定欄位需確認

LLM SHALL 在自然語言欄位解析結果不確定時要求使用者確認。

#### Scenario: 無法判斷戶外暴露程度

- WHEN 使用者未提供明確暴露程度且無法可靠推斷
- THEN 系統 SHALL 顯示確認問題
- AND 系統 SHALL NOT 自動假設為長時間暴露
```

## 9. feedback/spec.md

需求樣例：

```markdown
### Requirement: 行後回饋保存情境

系統 SHALL 保存行程、行程站、提交時間、實際天氣感受、
空品感受、建議符合度及自由文字。

#### Scenario: 提交行後問卷

- WHEN 使用者完成行程後提交問卷
- THEN 系統 SHALL 建立 FeedbackRecord
- AND 系統 SHALL 保留當時建議、資料快照及規則版本

### Requirement: 偏好候選需附證據

系統 SHALL 為每個 PreferenceUpdateCandidate 保存
偏好鍵、建議值、原因及證據摘要。
```

## 10. evaluation/spec.md

比較組：

- Baseline A：僅呈現官方天氣與 AQI 原始數值
- Baseline B：固定規則及固定文字模板
- Proposed：規則引擎、個人偏好及 LLM 解釋

必要測試情境：

- 多站、跨鄉鎮及不同停留時段
- 降雨機率高且使用者不願攜帶雨具
- 高 UV 及長時間戶外活動
- 相同 AQI、不同主要污染物
- 即時測站與隔日預報切換
- 資料過期、缺漏及超出預報範圍

評估指標：

- 建議可重現性
- 重要提醒遺漏率
- 資料限制揭露率
- 建議符合度
- 建議實用性
- 建議可理解性
- API 成功率
- 資料過期處理率
- 回應時間

## 11. proposal.md 範例

```markdown
# Proposal: Add Personalized Weather Assistant

## Summary

建立一套混合式個人化天氣助理，整合中央氣象署與環境部資料，
依行程、活動與個人偏好產生可追溯的行前建議。

## Motivation

目前使用者需要自行切換天氣與空氣品質來源並整合多項資訊。
通用天氣服務也未充分反映個人偏好、戶外暴露及行動成本。

## Proposed Solution

- 行程與需求結構化
- 官方資料時空對齊及品質檢查
- 規則引擎產生結構化決策
- LLM 僅解釋既有決策
- 偏好與回饋分離保存
- 建議附資料來源、規則、限制及時間

## Capabilities

- itinerary
- environment-data
- data-quality
- decision-engine
- user-preferences
- llm-explanation
- feedback
- evaluation

## Risks

- 預報不確定性可能被簡化為確定性結論
- 資料更新時間不一致可能導致錯誤組合
- 偏好回饋可能反映衣著、活動量或偶發狀況，而非長期耐受度
- LLM 可能擅自補充數值或改寫判定
- 相同 AQI 卻可能來自不同主要污染物

## Mitigations

- 保存預報與觀測資料的有效時間及發布時間
- 要求 LLM 僅引用結構化證據
- 健康防護使用官方指引，不以舒適偏好覆蓋
- 偏好更新需使用者確認
- 測試資料缺漏、過期及超出預報範圍情境
```

## 12. tasks.md 範例

```markdown
# Implementation Tasks

## Phase 1: Data and Domain Model

- [ ] 定義 UserProfile、ItinerarySegment、EnvironmentalSnapshot
- [ ] 定義 SegmentDecision、TripRecommendation 與 FeedbackRecord
- [ ] 實作中央氣象署資料介接
- [ ] 實作環境部即時及預報資料介接
- [ ] 實作資料新鮮度、時段及空間粒度檢查

## Phase 2: Decision Engine

- [ ] 實作降雨規則
- [ ] 實作紫外線規則
- [ ] 實作空品規則
- [ ] 實作冷熱舒適規則
- [ ] 為規則及建議保存 ruleId 與 evidence
- [ ] 實作重要防護優先於舒適提醒

## Phase 3: Personalization

- [ ] 實作偏好設定介面
- [ ] 實作偏好候選產生
- [ ] 實作使用者確認及偏好版本化
- [ ] 實作行後回饋蒐集

## Phase 4: LLM Boundary

- [ ] 實作自然語言欄位解析
- [ ] 實作不確定欄位確認流程
- [ ] 實作 TripRecommendation 說明產生
- [ ] 實作證據限定追問
- [ ] 建立禁止補造資料的回歸測試

## Phase 5: Evaluation

- [ ] 建立 Baseline A
- [ ] 建立 Baseline B
- [ ] 建立 Proposed 方法
- [ ] 建立多站及資料異常測試情境
- [ ] 測量正確性、遺漏率、理解度、成功率及回應時間
- [ ] 進行使用者回饋評估
```

## 13. design.md 範例

```markdown
# Design: Personalized Weather Assistant

## Architecture

UI -> Itinerary Parser -> Normalization -> Data Fetch
-> Quality & Alignment -> Rule Decision -> Trip Aggregation
-> LLM Explanation -> User Interface

並行保存：

- UserProfile
- ItineraryRecord
- EnvironmentalSnapshot
- SegmentDecision
- TripRecommendation
- FeedbackRecord
- PreferenceUpdateCandidate

## Component Responsibilities

### Itinerary Parser

解析表單或自然語言補充，產生待確認的行程段草稿。

### Normalization

驗證臺灣地點、轉成行政區、座標及空品區、切分有效時段。

### Data Fetch

取得中央氣象署及環境部資料並保存來源 metadata。

### Quality & Alignment

比對發布時間、有效時段、測站距離及空間粒度。

### Rule Decision Engine

依環境快照、行程段及偏好產生結構化建議。

### Trip Aggregator

彙整跨站決策、排序重要風險及合併重複提醒。

### LLM Explainer

僅將既有結構化結果改寫為自然語言。

## Failure Strategy

- 關鍵資料缺漏：顯示限制並停止相應判定。
- 資料過期：標示資料年齡並降低結論強度。
- 空間粒度不足：說明空間參考限制。
- 預報超出範圍：不產生未經資料支持的空品結論。
- 偏好候選：等待使用者確認，不直接更新。
- LLM 失敗：保留規則產生的結構化建議及可讀模板。

## Traceability

每項最終建議至少保存：

- 行程段
- 資料來源
- 資料發布時間及有效時段
- 空間參考
- 資料品質標記
- 規則編號及規則版本
- 偏好因素
- 限制
- 最終文字是否由規則模板或 LLM 解釋產生
```

## 14. OpenSpec 操作建議

1. 新功能先建立 `openspec/changes/<change-id>/`。
2. 未實作功能只修改變更目錄，不直接修改現行 `openspec/specs/`。
3. 需求使用 SHALL、SHOULD、MUST 及 MAY 清楚表達強制程度。
4. 每個需求至少包含一個具體 Scenario。
5. Scenario 應描述 WHEN、THEN 與必要的 AND 條件。
6. 變更完成後，將能力增量合併至主規格並移至 `changes/archive/`。
7. 每次規則門檻或偏好更新時，增加對應規格及回歸測試。
