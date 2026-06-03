# nabi-acl-waf WAF 封鎖報告

| 項目 | 內容 |
|------|------|
| **報告日期** | 2026-06-03（報告產生） |
| **查詢時段** | 2026-06-02 00:00:00 ～ 23:59:59（Taipei UTC+8） |
| **等效 UTC** | 2026-06-01T16:00:00Z ～ 2026-06-02T16:00:00Z |
| **WAF ACL** | nabi-acl-waf |
| **Scope** | CLOUDFRONT（全球邊緣節點） |
| **Region** | us-east-1 |
| **WebACL ARN** | arn:aws:wafv2:us-east-1:518375879370:global/webacl/nabi-acl-waf/3bf577f1-e6b2-4e61-8c6c-f2e822ff7016 |

---

## 執行摘要

**本日封鎖請求總數：275 次**

整體活動明顯高於 meet-acl-waf，尤其 **22:00 UTC+8（14:00 UTC）出現 133 次集中封鎖**，主要來源為加拿大（CA）雲端 IP，由 HostingProviderIPList 規則觸發並透過 CAPTCHA 挑戰封鎖。排除此單一時段後，全日其餘 23 小時約 142 次封鎖，分布均勻，為正常噪音水平。

---

## 封鎖前 5 名規則

| # | 規則名稱 | 所屬規則群組 | Priority | 封鎖次數 | 說明 |
|---|---------|------------|:-------:|:-------:|------|
| 1 | **HostingProviderIPList** | AWSManagedRulesAnonymousIpList<br>→ anonymous-ip-captcha（P11→P17） | 11 | **177** | 來自雲端/VPS 託管服務商 IP 段。22:00 UTC+8 出現加拿大 IP 突波（155 次規則命中，131 次為 CA 來源），CAPTCHA 無法通過後封鎖。此規則為本日封鎖主因（64%） |
| 2 | **ip_reputation_lists**（自訂） | 自定義信譽 IP 清單（IPv4+IPv6 IPSet） | 10 | **6** | 內部維護的已知惡意 IP 清單直接封鎖。全部發生於凌晨（01:00×2, 05:00×2, 06:00×1, 07:00×1 UTC+8），符合低流量時段探測行為 |
| 3 | **AWSManagedReconnaissanceList**<br>**AWSManagedIPReputationList** | AWSManagedRulesAmazonIpReputationList<br>（managed-reputation-ip） | 6 | **4**<br>（3+1） | AWS 信譽清單：偵察工具 IP（AWSManagedReconnaissanceList）3 次，發生於 17:00；已知惡意 IP（AWSManagedIPReputationList）1 次，發生於 15:00 |
| 4 | **EC2MetaDataSSRF_COOKIE** | AWSManagedRulesCommonRuleSet<br>（managed-core） | 4 | **1** | 透過 Cookie 嘗試進行 EC2 元數據服務（Instance Metadata Service）SSRF 攻擊，發生於 11:00 UTC+8 |
| 5 | **ExploitablePaths_URIPATH** | AWSManagedRulesKnownBadInputsRuleSet<br>（managed-known-bad-input） | 8 | **1** | 嘗試存取已知漏洞路徑（如 `/wp-admin`、`/.env`、`/.git` 等），發生於 13:00 UTC+8 |

> **注意：** HostingProviderIPList 的 177 次為 ManagedRuleGroupRule 維度計數（規則命中次數），Rule=ALL 總計 275 次（終結動作計數）。兩者差異因 CAPTCHA 挑戰計量時機不同所致，屬正常。
>
> **其餘 ~86 次封鎖**（275 - 177 - 6 - 4 - 1 - 1 = 86，扣除各規則獨立封鎖後 NabiWaf 層級計）來自：anonymous-ip-captcha CAPTCHA 封鎖未被 HostingProviderIPList 覆蓋的部分、scanners_and_probes / http_flood / bad_bot 自定義 IP 清單（本期間未獨立計量，但規則存在）及 geo-rate-limit 等規則。

---

## 每小時封鎖時序（Taipei UTC+8）

| 時間 (UTC+8) | 封鎖次數 | 主要來源 | 說明 |
|:-----------:|:-------:|---------|------|
| 2026-06-02 00:00 | 2 | TW(2) | 台灣 IP，NabiWaf Default |
| 2026-06-02 01:00 | 15 | TW(13), NL(2) | 台灣深夜高量；ip_reputation_lists 2 次 |
| 2026-06-02 02:00 | 3 | TW(2), US(1) | 正常噪音 |
| 2026-06-02 03:00 | 3 | US(3) | 正常噪音 |
| 2026-06-02 04:00 | 1 | US(1) | 正常噪音 |
| 2026-06-02 05:00 | 4 | TW(2), ip_rep(2) | ip_reputation_lists 2 次（TW IP 含） |
| 2026-06-02 06:00 | 3 | TW(2), ip_rep(1) | ip_reputation_lists 1 次 |
| 2026-06-02 07:00 | 6 | UA(1), ip_rep(1)... | ip_reputation_lists 1 次 |
| 2026-06-02 08:00 | 7 | FR(2), ID(2), BR(1)... | 多國分散流量 |
| 2026-06-02 09:00 | 9 | US(6), UA(2)... | HostingProviderIPList 5 次 |
| 2026-06-02 10:00 | 2 | US(2) | 正常 |
| 2026-06-02 11:00 | 4 | TW(1), US(3) | **EC2MetaDataSSRF_COOKIE** 1 次 |
| 2026-06-02 12:00 | 4 | US(4) | 正常 |
| 2026-06-02 13:00 | 9 | US(9) | **ExploitablePaths_URIPATH** 1 次；HostingProviderIPList 1 次 |
| 2026-06-02 14:00 | 11 | US(11) | 美國 IP 高峰 |
| 2026-06-02 15:00 | 7 | US(6), EE(1) | **AWSManagedIPReputationList** 1 次 |
| 2026-06-02 16:00 | 8 | US(6), DE(2) | HostingProviderIPList 4 次 |
| 2026-06-02 17:00 | 8 | SG(2), JP(3), DE(1)... | **AWSManagedReconnaissanceList** 3 次；HostingProviderIPList 5 次 |
| 2026-06-02 18:00 | 7 | JP(3), US(3), NL(1) | 正常 |
| 2026-06-02 19:00 | 0 | — | |
| 2026-06-02 20:00 | 3 | US(2), TW(1) | 正常 |
| 2026-06-02 21:00 | 5 | US(4), TW(1) | HostingProviderIPList 2 次 |
| **2026-06-02 22:00** | **133** ⚠️ | **CA(131), 其餘(2)** | **重大突波：加拿大 IP 大量觸發 HostingProviderIPList（155 次命中），CAPTCHA 封鎖** |
| 2026-06-02 23:00 | 21 | TW(10), US(11) | 台灣及美國高峰 |

---

## 來源國家分布

| 排名 | 國家 | 封鎖次數 | 比例 | 關聯規則 |
|:---:|------|:-------:|:---:|---------|
| 1 | 加拿大 (CA) | **131** | **47.6%** | HostingProviderIPList（22:00 突波，雲端 IP） |
| 2 | 美國 (US) | **79** | **28.7%** | 分散於全日，含 HostingProviderIPList、Default Block |
| 3 | 台灣 (TW) | **36** | **13.1%** | 主要在凌晨（00-07:00），Default Block |
| 4 | 日本 (JP) | **6** | **2.2%** | 17:00-18:00，HostingProviderIPList 相關 |
| 5 | 烏克蘭 (UA) | **3** | **1.1%** | 07-09:00，HostingProviderIPList |
| 6 | 德國 (DE) | 3 | 1.1% | 16-17:00 |
| 7 | 荷蘭 (NL) | 3 | 1.1% | 含 ip_reputation_lists 命中 |
| 8 | 印尼 (ID) | 2 | 0.7% | 08:00 |
| 9 | 法國 (FR) | 2 | 0.7% | 08:00 |
| 10 | 新加坡 (SG) | 2 | 0.7% | 17:00，AWSManagedReconnaissanceList 相關 |
| — | 其餘（KR/BR/EE/GB 各 1）| 4 | 1.5% | 零星個別 |

---

## 異常事件分析

### ⚠️ 22:00 UTC+8 加拿大 IP 突波（最高優先）

| 項目 | 數值 |
|------|------|
| 時間 | 2026-06-02 22:00 UTC+8（14:00 UTC） |
| 總封鎖數 | 133 次（全日 48.4%） |
| 主要來源國 | 加拿大（131 次，99%） |
| HostingProviderIPList 命中 | 155 次（含重複計量） |
| VulnerabilityCategory 記錄 | HostingProviderIPList: 81 次 |
| 性質判斷 | 來自加拿大 AWS/雲端 IP 段的高密度掃描或測試行為 |

**事件解讀：**
- 加拿大 IP 在短時間內密集觸發 HostingProviderIPList（AWS 匿名 IP 清單中的雲端服務商 IP 類別）
- anonymous-ip-captcha 規則（P17）對這些 IP 發出 CAPTCHA 挑戰
- 大量請求無法通過 CAPTCHA，最終被封鎖計入 BlockedRequests
- 加拿大多個 AWS/雲端節點（ca-central-1 或 ca-west-1 區域 IP 範圍）可能是協調性掃描工具的出口節點
- **注意：** 133 次封鎖在一個小時內屬於中等規模突波，不構成 DDoS 威脅，但需追蹤是否持續

---

## 共通性分析

### 1. 匿名/代理 IP 為主要封鎖來源（64%+）
全日最主要封鎖原因為 HostingProviderIPList，即來自雲端服務商、VPS 及 Hosting 業者的 IP 段：
- 此類 IP 常被自動化工具（掃描器、爬蟲、滲透測試框架）使用
- 現行防護：managed-anonymous-ip（Count + label）→ anonymous-ip-captcha（Captcha）→ 未解題則封鎖
- **CAPTCHA 機制有效運作**，成功阻擋無法解題的自動化程序

### 2. 台灣 IP 封鎖值得關注（13.1%，36 次）
台灣 IP 是封鎖第三名（36 次，凌晨時段），有以下可能解讀：
- **合法用戶誤封**：若 nabi 服務主要面向台灣用戶，Default Action Block 可能誤攔正常用戶，需確認
- **本地 VPS 掃描**：凌晨 01:00 出現 13 次高峰，可能為自動化掃描或爬蟲
- `geo-rate-limit-minor-countries` 規則（P21）排除 TW 在速率限制外（TW 在白名單中），這些封鎖可能來自其他規則
- **建議：** 確認 01:00 UTC+8 的 TW 封鎖內容（Sampled Requests）

### 3. 自定義 IP 信譽清單有效運作（ip_reputation_lists，6 次）
內部維護的信譽 IP 清單（ip_reputation_lists，P10）封鎖 6 次，全部在凌晨低流量時段，顯示：
- 清單維護有效，能識別並封鎖已知惡意 IP
- 凌晨時段是常見的自動化攻擊時間

### 4. AWS 偵察工具 IP 成功識別（AWSManagedReconnaissanceList，3 次）
17:00 UTC+8 出現 3 次 AWSManagedReconnaissanceList 封鎖（新加坡/德國 IP），此子規則封鎖已知大規模掃描工具（如 Masscan、Shodan Scanner），顯示有組織的外部偵察行為。

### 5. 進階攻擊嘗試（SSRF、路徑掃描）
- **EC2MetaDataSSRF_COOKIE**（11:00）：攻擊者嘗試透過 Cookie Header 傳遞 SSRF Payload，目標為 EC2 Instance Metadata Service（169.254.169.254），此手法用於竊取 IAM 憑證
- **ExploitablePaths_URIPATH**（13:00）：自動化已知漏洞路徑掃描，屬全球常見噪音

### 6. SQLi/LFI/XSS 攻擊量為零
本日 Attack 維度 GenericLFI、SQLi、RestrictedExtensions 均無封鎖記錄（0 次），顯示：
- 攻擊者未針對 nabi 服務發動 Web 應用層攻擊
- 或相關攻擊嘗試量不足以觸發 CloudWatch 記錄閾值

---

## 建議事項

### 短期（High Priority）

**1. 調查 22:00 UTC+8 加拿大 IP 突波**
- 開啟 WAF Sampled Requests 或 WAF Full Logging，確認這 131 次封鎖的：
  - URI 路徑（是否針對特定 API 端點？）
  - User-Agent（是否為已知掃描工具？）
  - IP 位址範圍（是否可加入自定義封鎖清單？）
- 若為已知掃描工具，可將相關 IP 加入 `ip_reputation_lists` 或 `scanners_and_probes` IPSet

**2. 確認 TW 台灣 IP 封鎖是否誤攔合法用戶**
- 台灣 IP 36 次封鎖（13%），若 nabi 服務主要服務台灣用戶，需確認凌晨 01:00 的 13 次高峰是否含合法請求
- nabi-acl-waf Default Action = Block，所有未通過 Allow 規則的請求都被封鎖
- **行動：** 確認 nabi 服務的正常 API 路徑是否有被 bypass_static_files（P3）或其他 Allow 規則放行

**3. 開啟 WAF 完整 Logging**
```bash
aws wafv2 put-logging-configuration \
  --resource-arn arn:aws:wafv2:us-east-1:518375879370:global/webacl/nabi-acl-waf/3bf577f1-e6b2-4e61-8c6c-f2e822ff7016 \
  --logging-configuration LogDestinationConfigs=<s3-or-cloudwatch-logs-arn>
```

### 中期（Medium Priority）

**4. 針對 EC2MetaDataSSRF 加強防護**
- 本日出現 1 次 `EC2MetaDataSSRF_COOKIE` 攻擊嘗試，此攻擊類型高危：若成功可竊取 IAM Role 憑證
- 確認 `managed-core` 規則已將 EC2MetaDataSSRF_BODY、EC2MetaDataSSRF_COOKIE 保持 Block（非 Count 模式）
- 目前設定：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_*` 被覆寫為 Count，EC2MetaDataSSRF 系列保持 Block ✓

**5. 評估 IP 速率限制規則的有效性**
- `page_view_rate_breaker`（P13）：100req/5min 的 page_view（非 API）請求 → Captcha
- `geo-rate-limit-minor-countries`（P21）：非 TW/CN/SG/US/HK/JP 來源，100req/5min → Challenge
- `request-rate-captcha`（P19）：700req/5min → Captcha
- 建議監控這些規則的 CaptchaRequests 指標，確認觸發頻率是否合理

**6. 定期更新自定義 IP 清單**
- `ip_reputation_lists`（P10）、`scanners_and_probes`（P14）、`http_flood`（P15）、`bad_bot`（P16）均為自定義 IPSet
- 建議建立自動化更新機制，定期從威脅情報來源（如 AbuseIPDB、Emerging Threats 等）更新清單

### 長期（Low Priority）

**7. 惡意 User-Agent 規則擴充**
- `malicious-user-agent`（P22）：僅限 CN 來源 + 惡意 UA 的組合
- `malicious-user-agent-no-area`（P23）：本日 0 封鎖
- 考慮擴充惡意 UA Regex 清單，加入常見掃描工具 UA（如 zgrab、masscan、nuclei 等）

**8. geo-rate-limit 規則效果監控**
- `geo-rate-limit-minor-countries`（P21）：對非主要國家（TW/CN/SG/US/HK/JP 以外）實施 Challenge
- 本日 UA/DE/FR 等國家有封鎖記錄，確認這些封鎖是否有部分應歸因於此規則

---

## WAF ACL 規則完整清單

| Priority | 規則名稱 | 動作 | 說明 | 24h 封鎖 |
|:--------:|---------|:----:|------|:-------:|
| 0 | detect_page_view | Count + Label `page_view` | 偵測 page view 請求（非 API/靜態） | — |
| 1 | label_static_file_route | Count + Label `static_file_route` | 標記靜態資源路徑 | — |
| 2 | label_ajax_route | Count + Label `ajax_route` | 標記 /api、/v2 路徑 | — |
| 3 | bypass_static_files | **Allow** | 放行靜態資源（errorpages/static/js/css 等） | — |
| 4 | managed-core | None（managed） | AWSManagedRulesCommonRuleSet（部分 Count 覆寫） | **1** |
| 5 | query-string-size-limit | **Block** | Query String > 4096 bytes 封鎖 | — |
| 6 | managed-reputation-ip | None（managed） | AWSManagedRulesAmazonIpReputationList | **4** |
| 7 | managed-sql-injection | None（managed） | AWSManagedRulesSQLiRuleSet | 0 |
| 8 | managed-known-bad-input | None（managed） | AWSManagedRulesKnownBadInputsRuleSet | **1** |
| 9 | internal-ip-limitation | **Allow** | 內部 IP 白名單 | — |
| 10 | ip_reputation_lists | **Block** | 自定義信譽 IP 清單（IPv4+IPv6） | **6** |
| 11 | managed-anonymous-ip | **Count**（override） | AWSManagedRulesAnonymousIpList | — |
| 12 | bot-white-list | **Allow** | Email Client Bot 白名單 | — |
| 13 | page_view_rate_breaker | **Captcha** | Page View 速率 100req/5min | — |
| 14 | scanners_and_probes | **Block** | 自定義掃描器/探測 IP 清單 | — |
| 15 | http_flood | **Block** | 自定義 HTTP Flood IP 清單 | — |
| 16 | bad_bot | **Block** | 自定義惡意 Bot IP 清單 | — |
| 17 | anonymous-ip-captcha | **Captcha** | AnonymousIPList 標籤觸發 CAPTCHA | **177**（HostingProviderIPList 歸因） |
| 18 | request-rate-breaker | Count | 速率限制 700req/5min（全域） | — |
| 19 | request-rate-captcha | **Captcha** | 速率限制 700req/5min（全域） | — |
| 20 | geo-rate-limit-captcha | Count | 非 TW 來源速率限制（700req/5min） | — |
| 21 | geo-rate-limit-minor-countries | **Challenge** | 非主要國家 100req/5min（Challenge） | — |
| 22 | malicious-user-agent | **Block** | 惡意 UA + 來自 CN 組合封鎖 | 0 |
| 23 | malicious-user-agent-no-area | **Block** | 惡意 UA（不限國家）封鎖 | 0 |
| 24 | manufacturer-ip-limitation | **Allow** | 廠商 IP 白名單 | — |
| — | Default Action | **Block** | 所有未匹配規則請求 | ~86（估計） |

---

## 資料來源與查詢說明

- **Namespace:** `AWS/WAFV2`
- **Metric:** `BlockedRequests`
- **Period:** 3600 秒（每小時一個資料點）
- **Statistics:** Sum
- **查詢維度：**
  - `Rule` 維度：ALL、NabiWaf（WebACL 層級）、各命名規則
  - `ManagedRuleGroup` + `ManagedRuleGroupRule` 維度：各受管規則群組子規則
  - `Attack` 維度：GenericLFI、SQLi、KnownBadInputs、RestrictedExtensions、EC2MetaDataSSRF
  - `VulnerabilityCategory` 維度：HostingProviderIPList、AWSManagedIPReputationList、AWSManagedReconnaissanceList
  - `Country` 維度：20 個主要國家
- **注意：** CloudFront WAF CloudWatch metrics 不含 `Region=CloudFront` dimension

---

*報告由 Claude Code + AWS API MCP（profile: 104awsdev14-ro）自動生成*  
*生成時間：2026-06-03 10:30 UTC+8*
