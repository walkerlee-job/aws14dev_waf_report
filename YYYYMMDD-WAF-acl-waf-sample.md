# meet-acl-waf WAF 封鎖報告

| 項目 | 內容 |
|------|------|
| **報告日期** | 2026-06-03 |
| **查詢時段** | 2026-06-02T00:00:00Z ～ 2026-06-03T00:00:00Z（UTC，24 小時） |
| **WAF ACL** | meet-acl-waf |
| **Scope** | CLOUDFRONT（全球邊緣節點） |
| **Region** | us-east-1 |
| **WebACL ARN** | arn:aws:wafv2:us-east-1:518375879370:global/webacl/meet-acl-waf/6ee5a35e-b00c-4cf0-92a1-8e93976972f4 |

---

## 執行摘要

**近 24 小時封鎖請求總數：6 次**

整體封鎖量極低，無批量或持續性攻擊跡象。封鎖事件分布在 5 個時間點，均為零星個別請求，顯示當前威脅環境溫和。主要封鎖來源為匿名/代理 IP（33%）、WebACL 預設封鎖（33%）、IP 信譽清單（17%）、已知漏洞路徑掃描（17%）。

---

## 封鎖前 5 名規則

| # | 規則名稱 | 所屬規則群組 | Priority | 封鎖次數 | 說明 |
|---|---------|------------|:-------:|:-------:|------|
| 1 | Default Action (Block) | WebACL 預設封鎖 | — | **4** | 未命中任何 Allow 規則的請求。其中 2 次含 HostingProviderIPList CAPTCHA 封鎖、2 次為純 Default Block（美國 IP）。此值為 WebACL 層級 MeetWaf 指標合計 |
| 2 | HostingProviderIPList | AWSManagedRulesAnonymousIpList（managed-anonymous-ip, P6） | 6 | **2** | 來自雲端/VPS 託管服務商 IP 段（韓國、新加坡）。managed-anonymous-ip 以 Count 模式標記並加 label，後續 anonymous-ip-captcha（P7）觸發 CAPTCHA 挑戰並封鎖 |
| 3 | AWSManagedIPReputationList | AWSManagedRulesAmazonIpReputationList（managed-reputation-ip, P2） | 2 | **1** | IP 命中 AWS 信譽黑名單（已知惡意 IP、Tor 出口節點等）。來源：美國(US)，發生於 02:00 UTC+8 |
| 4 | ExploitablePaths_URIPATH | AWSManagedRulesKnownBadInputsRuleSet（managed-known-bad-input, P4） | 4 | **1** | 嘗試存取已知漏洞路徑（如 `/wp-admin`、`/.env`、`/.git`），典型自動化掃描行為。來源：盧森堡(LU)，發生於 01:00 UTC+8 |
| 5 | （所有其他規則） | AWSManagedRulesCommonRuleSet / SQLiRuleSet / 其餘子規則 | 1/3/… | **0** | AWSManagedRulesCommonRuleSet（XSS、LFI、SQLi 等）及 AWSManagedRulesSQLiRuleSet 在本查詢期間無封鎖事件 |

> **說明：** CloudWatch Rule 維度（MeetWaf = 4、known-bad-input = 1、reputation-ip = 1）合計 = Rule=ALL（6）。ManagedRuleGroupRule 子維度（HostingProviderIPList = 2、ExploitablePaths_URIPATH = 1、AWSManagedIPReputationList = 1）為更細粒度分類，與 Rule 維度為交叉記錄，非重複加總。

---

## 每小時封鎖時序

| 時間 (UTC+8) | 封鎖次數 | 觸發規則 / 子規則 | 來源國 |
|:-----------:|:-------:|-----------------|:------:|
| 2026-06-02 15:00 | 2 | HostingProviderIPList → anonymous-ip-captcha（CAPTCHA 封鎖） | KR / SG |
| 2026-06-02 23:00 | 1 | Default Action Block | US |
| 2026-06-03 01:00 | 1 | ExploitablePaths_URIPATH（managed-known-bad-input） | LU |
| 2026-06-03 02:00 | 1 | AWSManagedIPReputationList（managed-reputation-ip） | US |
| 2026-06-03 06:00 | 1 | Default Action Block | US |

---

## 來源國家分布

| 國家 | 封鎖次數 | 比例 | 關聯規則 |
|------|:-------:|:---:|---------|
| 美國 (US) | 3 | 50% | Default Block × 2、IP Reputation × 1 |
| 韓國 (KR) | 1 | 17% | HostingProviderIPList（雲端 IP，CAPTCHA 封鎖） |
| 新加坡 (SG) | 1 | 17% | HostingProviderIPList（雲端 IP，CAPTCHA 封鎖） |
| 盧森堡 (LU) | 1 | 17% | ExploitablePaths_URIPATH（已知漏洞路徑掃描） |

---

## 共通性分析

### 1. 整體低風險、低流量
24 小時僅 6 次封鎖，無任何高頻率或持續性攻擊。符合小型或內部限制存取服務的流量特徵；CloudFront WAF Default Action 為 Block，說明服務存取需通過明確 Allow 規則。

### 2. 匿名/代理 IP 威脅（33%）
兩次 HostingProviderIPList 封鎖來自韓國、新加坡的雲端 IP，時間集中在同一小時（15:00 UTC+8）：
- 自動化工具（爬蟲、掃描器）慣用 VPS/雲端 IP 規避地理封鎖
- 現行防護鏈：`managed-anonymous-ip`（Count 標記）→ `anonymous-ip-captcha`（CAPTCHA）→ 無法解題則封鎖
- 此兩次封鎖表示 CAPTCHA 機制有效發揮作用

### 3. 自動化漏洞掃描（17%）
盧森堡 IP 觸發 ExploitablePaths_URIPATH（01:00 UTC+8），為全球普遍的自動化掃描行為，目標隨機不針對特定服務：
- 常見掃描目標：`/wp-admin`、`/.env`、`/.git/HEAD`、`/etc/passwd` 等
- 單次觸發，無持續性，屬正常環境噪音

### 4. IP 信譽封鎖有效（17%）
美國 IP 命中 `AWSManagedIPReputationList`（02:00 UTC+8），`managed-reputation-ip` 規則正常攔截已知惡意來源。

### 5. Default Action Block 值得關注（33%）
兩次美國 IP 請求（23:00、06:00 UTC+8）被 WebACL Default Action（Block）直接封鎖：
- Default Action Block 代表請求未通過任何 Allow 規則（非靜態資源、非內部 IP）
- **需確認**：這兩次是惡意請求，還是合法外部用戶存取動態路徑被誤攔？

---

## 建議事項

### 短期（High Priority）

**1. 確認 Default Action Block 不誤殺合法流量**

目前 Allow 規則僅涵蓋：
- Priority 0：靜態資源副檔名（svg/jpg/png/js/css/ico 等）
- Priority 5：內部 IP 白名單

若服務有外部用戶需存取動態頁面（非靜態資源），Default Block 可能誤攔合法請求。
- **行動**：至 WAF 主控台查看 Sampled Requests，確認 23:00 和 06:00 UTC+8 的美國封鎖請求內容

**2. 啟用 WAF 完整 Log 至 S3/CloudWatch Logs**

目前透過 CloudWatch Metrics 分析，無法得知具體 URI、IP、User-Agent。建議：
```
aws wafv2 put-logging-configuration \
  --resource-arn <webacl-arn> \
  --logging-configuration LogDestinationConfigs=<s3-or-cwl-arn>
```

### 中期（Medium Priority）

**3. 評估 anonymous-ip 規則改為 Block**

`managed-anonymous-ip`（Priority 6）現為 OverrideAction: Count，依靠 anonymous-ip-captcha（Priority 7）的 CAPTCHA 挑戰。
- 若 meet 服務不需允許 VPN/Proxy/雲端節點存取，可改為 OverrideAction: None（保留 managed rule 原始 Block 動作）
- 這樣可在 Priority 6 直接封鎖，跳過 CAPTCHA 流程，降低潛在旁路風險

**4. 確認 AWSManagedReconnaissanceList 啟用狀態**

`AWSManagedReconnaissanceList` 在本期間封鎖次數為 0（未觸發），但此子規則封鎖已知偵察工具（如 Masscan、Shodan scanner IP）。建議確認其動作為 Block（非 Count）。

### 長期（Low Priority）

**5. Rate-Based 規則調整**

`request-rate-breaker`（Priority 9）與 `request-rate-captcha`（Priority 10）均設定 700 req/5min，動作為 Count / Captcha（非 Block）：
- 若服務正常流量遠低於此閾值，建議降低限制（如 300–500 req/5min）
- 考慮將 request-rate-breaker 改為 Block 動作，增強速率限制防護

---

## WAF ACL 規則完整清單

| Priority | 規則名稱 | 動作 | 所用規則群組 / 條件 | 24h 封鎖 |
|:--------:|---------|:----:|------------------|:-------:|
| 0 | bypass_static_files | **Allow** | 靜態資源副檔名 + `/errorpages|static/` 路徑 | — |
| 1 | managed-core | None（managed） | AWSManagedRulesCommonRuleSet（NoUserAgent/GenericRFI/SizeRestrictions 設為 Count） | 0 |
| 2 | managed-reputation-ip | None（managed） | AWSManagedRulesAmazonIpReputationList | **1** |
| 3 | managed-sql-injection | None（managed） | AWSManagedRulesSQLiRuleSet | 0 |
| 4 | managed-known-bad-input | None（managed） | AWSManagedRulesKnownBadInputsRuleSet | **1** |
| 5 | internal-ip-limitation | **Allow** | 內部 IP 白名單（IPSet） | — |
| 6 | managed-anonymous-ip | **Count**（override） | AWSManagedRulesAnonymousIpList | — |
| 7 | anonymous-ip-captcha | **Captcha** | Label: `awswaf:managed:aws:anonymous-ip-list:AnonymousIPList` | **2** |
| 8 | bot-white-list | **Allow** | Label: `awswaf:managed:aws:bot-control:bot:category:email_client` | — |
| 9 | request-rate-breaker | **Count** | Rate-Based: 700 req / 300s（per IP） | — |
| 10 | request-rate-captcha | **Captcha** | Rate-Based: 700 req / 300s（per IP） | — |
| — | Default Action | **Block** | 所有未匹配規則的請求 | **2** |

---

## 資料來源與查詢說明

- **Namespace:** `AWS/WAFV2`
- **Metric:** `BlockedRequests`
- **Period:** 3600 秒（每小時一個資料點）
- **Statistics:** Sum
- **查詢維度：**
  - `Rule` 維度：ALL、各命名規則（meet-waf-*）
  - `ManagedRuleGroup` + `ManagedRuleGroupRule` 維度：各受管規則群組子規則
  - `Country` 維度：16 個國家逐一查詢
- **注意：** CloudFront WAF CloudWatch metrics 不含 `Region=CloudFront` dimension

---

*報告由 Claude Code + AWS API MCP（profile: 104awsdev14-ro）自動生成*  
*生成時間：2026-06-03 09:50 UTC+8*
