# WAF 封鎖分析報告 — beagiver-acl-waf

> 分析日期：2026-06-02（Taipei UTC+8）
> 查詢視窗（UTC）：2026-06-01T16:00:00Z → 2026-06-02T16:00:00Z

---

## 1. 執行摘要

| 項目 | 值 |
|------|-----|
| WAF ACL 名稱 | beagiver-acl-waf |
| 分析日期（Taipei） | 2026-06-02 |
| Scope / Region | CLOUDFRONT / us-east-1 |
| ACL ID | 3f7edf3e-412d-4225-b24c-68e5f17eb507 |
| Default Action | **Block**（白名單模式） |
| 總封鎖次數 | **425** |
| 總允許次數 | **47** |
| 封鎖率 | **90.0%** |
| 尖峰封鎖時段 | 14:00 Taipei（380 次）、21:00 Taipei（16 次） |
| 主要封鎖原因（Top 1） | AWSManagedReconnaissanceList（偵察 IP 清單） |
| 主要來源國家 | 伊朗 IR（89.4%）、美國 US（8.7%） |

**整體評估**：本日封鎖呈現**強烈集中性**，14:00 Taipei 出現 380 次突發封鎖事件，來源集中於伊朗 IP 區段，同時觸發 IP 信譽規則（AWSManagedReconnaissanceList）及受限副檔名探測（RestrictedExtensions_URIPATH），具備自動化漏洞掃描特徵。非 14:00 時段封鎖量低且分散，屬 WAF 正常防護範圍。建議優先調閱 14:00 Sampled Requests 確認伊朗 IP 詳情。

---

## 2. 封鎖前 5 名規則

> **注意**：`managed-anonymous-ip`（Priority 7）設定為 `OverrideAction: Count`，其子規則 HostingProviderIPList 觸發後不直接封鎖，而是附加 Label 繼續評估；若後續無 Allow 規則命中，則由 **Default Action（Block）** 完成封鎖。因此 HostingProviderIPList 封鎖數記錄在 Default Action 維度下。

| 排名 | 規則名稱（子規則） | 所屬規則群組 | 父規則 Priority | 封鎖次數 | 佔比（%） | 說明 |
|------|-------------------|-------------|----------------|---------|----------|------|
| 🥇 | AWSManagedReconnaissanceList | AWSManagedRulesAmazonIpReputationList | 3 | 327 | 77.0% | AWS 偵察活動 IP 清單，收錄已知用於掃描、探測之 IP，直接封鎖 |
| 🥈 | RestrictedExtensions_URIPATH | AWSManagedRulesCommonRuleSet | 2 | 52 | 12.2% | URI 路徑含受限副檔名（.php/.asp/.bak 等），封鎖可能的 Web Shell 存取或後端檔案探測 |
| 🥉 | Default Action（BeagiverWaf） | —（ACL 預設動作） | N/A | 35 | 8.2% | 未命中任何明確 Allow/Block 規則的請求；主要為 HostingProviderIPList（Count 模式，29 次）後續無 Allow 規則命中，及其他未識別流量（6 次） |
| 4 | AWSManagedIPReputationList | AWSManagedRulesAmazonIpReputationList | 3 | 9 | 2.1% | AWS IP 信譽黑名單，封鎖已知惡意或低信譽 IP |
| 5 | ReactJSRCE_BODY | AWSManagedRulesKnownBadInputsRuleSet | 5 | 1 | 0.24% | 偵測 React.js 遠端程式碼執行（RCE）攻擊特徵於 Request Body |

**其他已觸發規則（封鎖 1 次）**：GenericLFI_URIPATH（AWSManagedRulesCommonRuleSet, Priority 2）：1 次（LFI 路徑遍歷）

**子規則層級封鎖驗算**：
- managed-reputation-ip（Priority 3）：AWSManagedReconnaissanceList(327) + AWSManagedIPReputationList(9) = **336 次**
- managed-core（Priority 2）：RestrictedExtensions_URIPATH(52) + GenericLFI_URIPATH(1) = **53 次**
- managed-known-bad-input（Priority 5）：ReactJSRCE_BODY(1) = **1 次**
- Default Action（BeagiverWaf）：**35 次**
- 合計：336 + 53 + 1 + 35 = **425 次** ✓

---

## 3. 每小時封鎖時序（Taipei UTC+8）

| 時段（+8） | 封鎖次數 | 允許次數 | 總請求數 | 封鎖率（%） | 備註 |
|-----------|---------|---------|---------|-----------|------|
| 00:00–01:00 | 10 | 40 | 50 | 20.0% | US/NL HostingProvider IPs；大量正常允許請求 |
| 01:00–02:00 | 2 | 0 | 2 | 100.0% | NL(1)、LU(1) HostingProvider |
| 02:00–03:00 | 0 | 0 | 0 | — | 無流量 |
| 03:00–04:00 | 0 | 0 | 0 | — | 無流量 |
| 04:00–05:00 | 0 | 0 | 0 | — | 無流量 |
| 05:00–06:00 | 0 | 0 | 0 | — | 無流量 |
| 06:00–07:00 | 1 | 0 | 1 | 100.0% | NL HostingProvider |
| 07:00–08:00 | 1 | 0 | 1 | 100.0% | US HostingProvider |
| 08:00–09:00 | 0 | 0 | 0 | — | 無流量 |
| 09:00–10:00 | 0 | 0 | 0 | — | 無流量 |
| 10:00–11:00 | 0 | 0 | 0 | — | 無流量 |
| 11:00–12:00 | 1 | 0 | 1 | 100.0% | US HostingProvider |
| 12:00–13:00 | 0 | 0 | 0 | — | 無流量 |
| 13:00–14:00 | 1 | 0 | 1 | 100.0% | ReactJSRCE_BODY（KnownBadInputs） |
| **14:00–15:00** | **380** | 6 | 386 | **98.4%** | ⚠️ 伊朗 IP 集中掃描事件（AWSManagedReconnaissanceList 327 + RestrictedExtensions 52 + LFI 1） |
| 15:00–16:00 | 1 | 0 | 1 | 100.0% | US AWSManagedIPReputationList |
| 16:00–17:00 | 1 | 0 | 1 | 100.0% | US AWSManagedIPReputationList |
| 17:00–18:00 | 1 | 0 | 1 | 100.0% | US AWSManagedIPReputationList |
| 18:00–19:00 | 1 | 0 | 1 | 100.0% | US AWSManagedIPReputationList |
| 19:00–20:00 | 1 | 0 | 1 | 100.0% | US AWSManagedIPReputationList |
| 20:00–21:00 | 2 | 0 | 2 | 100.0% | US AWSManagedIPReputationList |
| **21:00–22:00** | **16** | 0 | 16 | **100.0%** | ⚠️ US HostingProvider 突增（Default Block） |
| 22:00–23:00 | 2 | 0 | 2 | 100.0% | US Default Block |
| 23:00–24:00 | 4 | 1 | 5 | 80.0% | TW Default Block（4）＋1 允許 |
| **合計** | **425** | **47** | **472** | **90.0%** | |

### ASCII 每小時封鎖長條圖（24 小時，每 █ = 10 次）

```
時段（Taipei）  封鎖次數
00:00  ██ (10)
01:00  ▏ (2)
02:00  · (0)
03:00  · (0)
04:00  · (0)
05:00  · (0)
06:00  ▏ (1)
07:00  ▏ (1)
08:00  · (0)
09:00  · (0)
10:00  · (0)
11:00  ▏ (1)
12:00  · (0)
13:00  ▏ (1)
14:00  ████████████████████████████████████████ (380) ⚠️
15:00  ▏ (1)
16:00  ▏ (1)
17:00  ▏ (1)
18:00  ▏ (1)
19:00  ▏ (1)
20:00  ▏ (2)
21:00  ██ (16) ⚠️
22:00  ▏ (2)
23:00  ▏ (4)
```

### 時序觀察

- **低水位時段**（02:00–05:00、08:00–12:00）：幾乎無流量，正常靜默時段
- **尖峰 1 — 14:00**：380 次集中爆發，佔全日封鎖量的 89.4%，全數來自伊朗 IP，且同時觸發 IP 信譽規則與檔案探測規則，高度疑似自動化掃描工具
- **尖峰 2 — 21:00**：16 次 US 雲端 IP（HostingProvider）觸發 Default Block，與 14:00 性質不同，可能為定期爬蟲或誤設定 Bot
- **允許流量集中在 00:00**（40 次），性質待確認，可能為定期維護作業或 API 輪詢

---

## 4. 來源國家分布

| 排名 | 國家 | 代碼 | 封鎖次數 | 佔比（%） | 主要觸發規則 |
|------|------|------|---------|----------|-------------|
| 🥇 | 🇮🇷 伊朗 | IR | 380 | 89.4% | AWSManagedReconnaissanceList(327) + RestrictedExtensions_URIPATH(52) + GenericLFI_URIPATH(1) |
| 🥈 | 🇺🇸 美國 | US | 37 | 8.7% | AWSManagedIPReputationList(9) + HostingProviderIPList → Default Block(28) |
| 🥉 | 🇹🇼 台灣 | TW | 4 | 0.9% | Default Block（無規則命中） |
| 4 | 🇳🇱 荷蘭 | NL | 3 | 0.7% | HostingProviderIPList → Default Block |
| 5 | 🇱🇺 盧森堡 | LU | 1 | 0.2% | HostingProviderIPList → Default Block |
| — | 其他（22 國） | — | 0 | 0% | 僅有允許或 Captcha 流量，或歷史記錄殘留於 list-metrics |

**在 list-metrics 中出現但本日封鎖為 0 的國家**（代表這些國家在其他時期有封鎖記錄，或本日僅有允許 / Captcha 流量）：
CN、FR、AR、GB、DE、SG、JP、MY、BY、UA、CA、KR、SE、IN、ZA、NZ、RU、BG、BR、HK、RO、FI

### ASCII 橫向封鎖長條圖（國家）

```
🇮🇷 IR │████████████████████████████████████████ 380 (89.4%)
🇺🇸 US │████ 37 (8.7%)
🇹🇼 TW │▏ 4 (0.9%)
🇳🇱 NL │▏ 3 (0.7%)
🇱🇺 LU │▏ 1 (0.2%)
```
（每 █ ≈ 9.5 次）

---

## 5. 異常事件分析

### 事件 A：伊朗 IP 集中自動化掃描（14:00 Taipei，UTC 06:00）

| 項目 | 內容 |
|------|------|
| 發生時段 | 2026-06-02 14:00–15:00 Taipei（UTC 06:00–07:00） |
| 封鎖次數 | 380 次（佔全日 89.4%） |
| 來源國家 | 伊朗（IR）100% |
| 觸發規則 | AWSManagedReconnaissanceList（327）、RestrictedExtensions_URIPATH（52）、GenericLFI_URIPATH（1） |
| 威脅等級 | **高** |

**評估**：來自伊朗 IP 區段的集中式請求，在單一小時內觸發 380 次封鎖，且同時命中多個規則類型：

1. **AWSManagedReconnaissanceList**（327 次）：這些 IP 已被 AWS 識別為已知偵察 / 掃描來源，意味流量本身來自自動化掃描基礎設施
2. **RestrictedExtensions_URIPATH**（52 次）：嘗試存取 `.php`, `.asp`, `.aspx`, `.bak`, `.env` 等受限副檔名，典型 Web Shell 探測或後端文件查找
3. **GenericLFI_URIPATH**（1 次）：路徑遍歷嘗試（`../` 序列）

VulnerabilityCategory 交叉驗證：`VulnerabilityCategory=AWSManagedReconnaissanceList` 顯示 344 次（高於子規則 327 次），代表部分 RestrictedExtensions 觸發的 IP 同時也在 Recon 清單中，進一步確認為統一掃描基礎設施。

**可能原因**：自動化漏洞掃描工具（如 Nuclei、Nikto）部署於伊朗雲端 / VPS 節點，定時對目標域名執行廣度掃描。

---

### 事件 B：美國 HostingProvider IP 21:00 突增

| 項目 | 內容 |
|------|------|
| 發生時段 | 2026-06-02 21:00–22:00 Taipei |
| 封鎖次數 | 16 次（日內次高） |
| 來源國家 | 美國（US） |
| 觸發規則 | HostingProviderIPList（Count）→ Default Block |
| 威脅等級 | **低至中** |

**評估**：美國雲端 / 機房 IP（HostingProviderIPList）因 `managed-anonymous-ip` 設定 OverrideAction=Count，不被直接規則封鎖，流量穿透至 Default Action Block。可能為定期執行的自動化 Bot（SEO 爬蟲、監控工具）、誤設定的 API 呼叫客戶端，或合法服務的機器人流量未取得 Allow 白名單。

---

## 6. 共通性分析

### 攻擊類型分析

**主要威脅類型（按封鎖量）**：
1. **IP 信譽封鎖**（Priority 3, AWSManagedRulesAmazonIpReputationList）：**336 次（79.1%）**，分為偵察清單（327）與一般惡意 IP（9）
2. **受限副檔名探測**（RestrictedExtensions_URIPATH）：**52 次（12.2%）**，嘗試存取 Web Shell 相關檔案類型
3. **雲端 / 機房 IP Default Block**（HostingProviderIPList → Default Action）：**29 次（6.8%）**
4. **已知惡意輸入**（ReactJSRCE_BODY、GenericLFI_URIPATH）：**2 次（0.5%）**

**Attack / VulnerabilityCategory 交叉驗證**：
- Attack=GenericLFI：1 次（確認 LFI 存在）
- Attack=KnownBadInputs：1 次（確認 ReactJSRCE）
- Attack=RestrictedExtensions：52 次（確認副檔名探測規模）
- VulnerabilityCategory=AWSManagedIPReputationList：9 次
- VulnerabilityCategory=AWSManagedReconnaissanceList：344 次（跨規則重疊計算，見事件 A 說明）
- VulnerabilityCategory=HostingProviderIPList：260 次（VulnerabilityCategory 維度涵蓋多維度交集，遠高於 ManagedRuleGroupRule 29 次）
- VulnerabilityCategory=AWSManagedIPDDoSList：0 次（無 DDoS 規模）

**未偵測到的攻擊類型**：本日 SQL Injection（managed-sql-injection, Priority 4）、XSS 等注入式攻擊均未出現，掃描者主要進行探測性攻擊而非注入式攻擊。

---

### 時間模式

- 封鎖呈現**極度集中性**：14:00 單一小時佔 89.4%，明顯異於隨機分布
- 非尖峰時段（15:00–24:00）封鎖呈**持續低水位**（每小時 1–16 次），以 US IP Reputation 為主
- 允許流量集中在 00:00（40 次），可能為定期批次作業

---

### 地理模式

- 伊朗 IR 89.4% 的佔比**高度異常**，一般 WAF 封鎖來源應更分散
- IR IP 同時匹配 AWSManagedReconnaissanceList 及 RestrictedExtensions，代表這些 IP 是**掃描專用基礎設施**，而非一般用戶
- 台灣（TW）觸發 Default Block（無任何規則命中），可能是未列入白名單的內部或合作夥伴 IP，建議確認後加入 Allow IP Set

---

### 規則效果評估

| 規則 | 狀態 | 效果評估 |
|------|------|---------|
| managed-core（Priority 2）| OverrideAction: None（部分 Count 覆蓋）| 有效封鎖 RestrictedExtensions 及 GenericLFI |
| managed-reputation-ip（Priority 3）| OverrideAction: None | 本日最高效規則，封鎖 336 次（79.1%） |
| managed-sql-injection（Priority 4）| OverrideAction: None | 未觸發，本日無 SQL 注入 |
| managed-known-bad-input（Priority 5）| OverrideAction: None | 封鎖 ReactJSRCE_BODY 1 次 |
| managed-anonymous-ip（Priority 7）| OverrideAction: **Count** | 僅標記 Label，本身不封鎖；HostingProviderIPList 需依賴 Default Action 處理，建議評估補強 Captcha 涵蓋 |
| anonymous-ip-captcha（Priority 9）| Captcha | 針對 AnonymousIPList 標籤；HostingProviderIPList 無對應 Captcha 規則 |
| request-rate-breaker/captcha（10/11）| Count/Captcha | 速率限制 700 req/5min；14:00 事件（380/hr）未觸發閾值 |
| malicious-user-agent（Priority 12）| Block（CN + 惡意 UA）| 本日未觸發（CN 封鎖量為 0） |

---

## 7. 建議事項

### 短期建議（立即～2 週）

1. **調閱 14:00 Sampled Requests**
   前往 AWS Console → WAF → beagiver-acl-waf → Sampled Requests，過濾 14:00 時段 `AWSManagedReconnaissanceList` 規則命中紀錄，確認具體 URI、User-Agent 及 IP 清單，評估是否為已知掃描工具（Nuclei、Shodan、Censys 等）。

2. **確認 14:00 的 6 次允許請求**
   14:00 有 6 次允許請求搭配 380 次封鎖，需確認這 6 次是否為合法流量或誤判，可調閱 Sampled Requests Allow 記錄。

3. **評估台灣 TW Default Block 原因**
   TW 在 23:00 觸發 4 次 Default Block（BeagiverWaf），請確認這些 IP 是否為合作夥伴或內部服務，若為合法流量需加入 vulnerability-scan-whitelist 或 internal-ip-limitation IP Set。

4. **評估 US HostingProvider 21:00 突增**
   調閱 Sampled Requests，確認觸發 HostingProviderIPList 的 US IP 是否為合法 API 呼叫端，若是則加入 Allow IP Set，避免持續觸發 Default Block。

---

### 中期建議（2 週～2 個月）

1. **設定 CloudWatch Alarm**
   為 `Rule=ALL BlockedRequests` 設定每小時閾值告警（建議 > 100 次/小時），通知 SNS → 資安團隊，確保異常事件即時通報。

2. **HostingProviderIPList 處理策略調整**
   目前 `managed-anonymous-ip` 設定 OverrideAction=Count，HostingProviderIPList 流量透過 Default Block 處理。建議評估是否為 HostingProviderIPList 標籤新增 Captcha 規則（類似 anonymous-ip-captcha），或設定更明確的 Block 行為，以取代依賴 Default Action。

3. **啟用 WAF 全量 Logging**
   啟用 WAF Logging（S3 + Kinesis Firehose），取得每次請求的 IP、URI、規則命中詳情，為後續 Athena 分析奠定基礎。

4. **評估 Rate-based Rule 閾值**
   本次 14:00 事件 380 次請求集中在 1 小時，但 rate-based rule 的 700 req/5min（= 8,400/hr）遠高於事件規模，建議評估是否調低，或增加 IP-based Rate Limit 規則加速攔截。

---

### 長期建議（2 個月以上）

1. **建立自動化 WAF 分析報告**
   整合 CloudWatch Metrics API + Lambda + S3，每日自動產生封鎖摘要，排程發送至資安負責人。

2. **WAF Logging + Amazon Athena 精細分析**
   啟用全量日誌後，透過 Athena 分析 Top 10 封鎖 IP（供 IP Set 黑名單管理）、URI Path 分布（識別攻擊目標頁面）、User-Agent 分布（識別掃描工具指紋）。

3. **考慮啟用 Bot Control Managed Rule Group**
   若流量規模增大，評估加入 `AWSManagedRulesBotControlRuleSet`，對 Bot 流量進行更細緻分類與管控。

4. **IP Set 黑名單自動化更新**
   結合 WAF Logging + Lambda，對累積超過閾值（如 100 次 / 日）的 IP 自動加入 Block IP Set，實現動態防禦。

---

## 8. WAF ACL 規則完整清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---------|---------|------|------------------------|-------------|------|
| 0 | vulnerability-scan-whitelist | IP Set | **Allow** | beagiver-waf-vulnerability-scan-whitelist | 允許已知弱點掃描工具 IP（白名單） |
| 1 | bypass_static_files | Regex Match（URI Path） | **Allow** | beagiver-waf-bypass-static-files | 允許靜態資源路徑（/errorpages、/static、2023/2024giverreview） |
| 2 | managed-core | AWSManagedRulesCommonRuleSet | **OverrideAction: None**（部分子規則 Count） | beagiver-waf-managed-core | 核心規則集；NoUserAgent / GenericRFI_QUERYARGUMENTS / SizeRestrictions（Body/Cookie）設為 Count |
| 3 | managed-reputation-ip | AWSManagedRulesAmazonIpReputationList | **OverrideAction: None** | beagiver-waf-managed-reputation-ip | AWS IP 信譽清單（偵察 IP、惡意 IP） |
| 4 | managed-sql-injection | AWSManagedRulesSQLiRuleSet | **OverrideAction: None** | beagiver-waf-managed-sql-injection | SQL Injection 防護 |
| 5 | managed-known-bad-input | AWSManagedRulesKnownBadInputsRuleSet | **OverrideAction: None** | beagiver-waf-managed-known-bad-input | 已知惡意輸入（ReactJSRCE、ExploitablePaths 等） |
| 6 | internal-ip-limitation | IP Set | **Allow** | beagiver-waf-internal-ip-limitation | 允許內部服務 IP |
| 7 | managed-anonymous-ip | AWSManagedRulesAnonymousIpList | **OverrideAction: Count** | beagiver-waf-managed-anonymous-ip | 匿名 IP 清單（含 HostingProvider、Tor 等）；Count 模式不直接封鎖，僅附加 Label |
| 8 | bot-white-list | Label Match（email_client Bot） | **Allow** | beagiver-waf-bot-whitelist | 允許 email_client 類型 Bot（郵件預覽爬蟲） |
| 9 | anonymous-ip-captcha | Label Match（AnonymousIPList） | **Captcha** | beagiver-waf-anonymous-ip-captcha | 對 AnonymousIPList 標籤的 IP 發出 Captcha 挑戰 |
| 10 | request-rate-breaker | Rate-based（IP, 700/5min） | **Count** | beagiver-waf-request-rate-breaker | 速率限制計數（超速不封鎖，僅計數） |
| 11 | request-rate-captcha | Rate-based（IP, 700/5min） | **Captcha** | beagiver-waf-request-rate-captcha | 超速後發出 Captcha 挑戰 |
| 12 | malicious-user-agent | AND（惡意 UA Regex + GeoMatch CN）| **Block** | beagiver-waf-malicious-user-agent | 封鎖來自中國且使用惡意 User-Agent 的請求 |
| — | Default Action | — | **Block** | BeagiverWaf | 所有未被規則明確 Allow/Block 的請求均封鎖（白名單模式） |

---

## 9. 資料來源與查詢說明

### 查詢環境

| 項目 | 值 |
|------|-----|
| AWS Profile | 104awsdev14-ro（唯讀） |
| Scope | CLOUDFRONT |
| CloudWatch Namespace | AWS/WAFV2 |
| Period（粒度） | 3600 秒（每小時） |
| Region | us-east-1 |
| UTC 查詢區間 | 2026-06-01T16:00:00Z → 2026-06-02T16:00:00Z |
| Taipei 時間對應 | 2026-06-02 00:00 → 2026-06-02 24:00（UTC+8） |

### 查詢 Dimension 說明

| Dimension 組合 | 用途 |
|---------------|------|
| WebACL + Rule=ALL | 全量封鎖總計（基準值） |
| WebACL + Rule={MetricName} | 各規則層級封鎖量 |
| ManagedRuleGroup + WebACL + ManagedRuleGroupRule | Managed Rule 子規則層級封鎖量 |
| WebACL + Country | 依來源國家統計封鎖量 |
| WebACL + Attack | 依攻擊類型統計（交叉驗證） |
| WebACL + VulnerabilityCategory | 依漏洞類別統計（交叉驗證，含多維度重疊計算） |
| WebACL + Rule=ALL（AllowedRequests） | 全量允許總計 |

### 資料限制

1. **CloudWatch 為彙總統計**：無法取得個別 IP 位址、URI 路徑、User-Agent 等詳細資訊，需調閱 WAF Sampled Requests 或啟用 WAF Logging 取得。
2. **ManagedRuleGroup Count OverrideAction 行為**：`managed-anonymous-ip`（Priority 7）設定 OverrideAction=Count，其子規則（HostingProviderIPList 等）觸發的 BlockedRequests 統計反映的是「最終被 Default Action 封鎖，且此子規則曾觸發」的請求，非子規則直接封鎖。
3. **VulnerabilityCategory 維度重疊**：單一請求可被標記多個 VulnerabilityCategory，因此各類別總和可能超過 Rule=ALL 的 BlockedRequests 總計。
4. **Country 維度**：反映封鎖事件的 IP 來源地理位置，不代表攻擊者實際地理位置（攻擊者可能透過 VPN / 代理）。
5. **在 list-metrics 出現但封鎖為 0 的國家**：代表這些國家在本分析時間窗口內無封鎖事件，可能在其他日期或時段有記錄。

### 查詢步驟

1. `aws wafv2 list-web-acls` — 取得 ACL ID 與 ARN
2. `aws wafv2 get-web-acl` — 取得完整規則清單與設定
3. `aws cloudwatch list-metrics`（含 WebACL Dimension 過濾）— 確認存在的 BlockedRequests Dimension 組合
4. `aws cloudwatch get-metric-statistics` × 49 次（分 3 批，每批最多 20 個）— 依 Rule、ManagedRuleGroupRule、Country、Attack、VulnerabilityCategory 維度批量查詢 BlockedRequests；另 1 次查詢 AllowedRequests

---

## 附錄：AWS API 費用估算

| 服務 | API 操作 | 呼叫次數 | 單價（USD） | 費用（USD） |
|------|---------|---------|-----------|-----------|
| WAFv2 | ListWebACLs | 1 | $1.00 / 百萬次 | < $0.000001 |
| WAFv2 | GetWebACL | 1 | $1.00 / 百萬次 | < $0.000001 |
| CloudWatch | ListMetrics | 2 | $0.01 / 1,000 次 | $0.00002 |
| CloudWatch | GetMetricStatistics | 49 | $0.01 / 1,000 次 | $0.00049 |
| **合計** | | **53 次** | | **≈ $0.001** |

> 本次分析全程為唯讀操作，無寫入或修改任何 AWS 資源。費用依 AWS 定價頁面（2026-06）估算，實際費用可能因帳號方案或 Free Tier 而有所不同。

---

*報告產生時間：2026-06-03 Taipei Time*
*資料來源：AWS WAFv2 API（us-east-1）+ Amazon CloudWatch Metrics（AWS/WAFV2 Namespace）*
*查詢工具：AWS CLI via Claude Code + AWS API MCP (profile: 104awsdev14-ro)*
