# WAF 封鎖分析報告：giver-acl-waf

---

## 1. 執行摘要

| 項目 | 值 |
|------|-----|
| WAF ACL 名稱 | `giver-acl-waf` |
| 分析日期（Taipei UTC+8） | 2026-06-02 |
| 總封鎖次數 | **671** |
| 總允許次數 | 2,545 |
| 封鎖率 | **20.86%** |
| 尖峰封鎖時段 | 03:00–04:00（160 次）、02:00–03:00（148 次） |
| 主要封鎖原因（Top 1 規則） | HostingProviderIPList（AWSManagedRulesAnonymousIpList）|
| 主要來源國家 | 🇧🇷 巴西（82.1%）、🇺🇸 美國（16.8%） |

**整體評估：** 本日 WAF 共攔截 671 筆請求，封鎖率 20.86%，主要由來自巴西雲端/主機業者 IP 的自動化流量所驅動，佔比高達 82.1%，呈現典型 Bot/Automated 掃描特徵。凌晨 02:00–04:00 出現明顯的夜間攻擊波，應優先確認是否為持續性威脅行為者並考慮強化阻擋策略。

---

## 2. 封鎖前 5 名規則

> ⚠️ **注意事項**：`managed-anonymous-ip`（Priority 110）設定為 `OverrideAction: Count`，其下子規則 `HostingProviderIPList` 的 BlockedRequests 指標（651 次）代表「匹配此子規則且最終被封鎖（由後續 `anonymous-ip-captcha` CAPTCHA 失敗或 Default Block Action 執行）」的請求，而非由該規則直接封鎖。實際封鎖行為由 Priority 130 的 `anonymous-ip-captcha` 規則（CAPTCHA 挑戰失敗）或 WAF Default Block Action 完成。

| 排名 | 規則名稱（子規則） | 所屬規則群組 | 父規則 Priority | 封鎖次數 | 佔比 | 說明 |
|------|-----------------|------------|----------------|---------|------|------|
| 🥇 | `HostingProviderIPList` | AWSManagedRulesAnonymousIpList | P110 `managed-anonymous-ip` | 651 | 97.0% | 偵測來自已知主機/雲端服務商 IP 的匿名請求。父規則 Count OverrideAction，最終由 CAPTCHA 失敗或 Default Block 封鎖。 |
| 🥈 | `AWSManagedIPReputationList` | AWSManagedRulesAmazonIpReputationList | P30 `managed-reputation-ip` | 1 | 0.15% | Amazon 維護的惡意 IP 信譽清單，直接 Block。 |
| 🥉 | `AWSManagedReconnaissanceList` | AWSManagedRulesAmazonIpReputationList | P30 `managed-reputation-ip` | 1 | 0.15% | 偵測進行偵察掃描的惡意 IP（如 port scanner、fingerprinting 工具），直接 Block。 |
| 4 | Default Block Action | — | — | 18（推估）| 2.7% | 未被任何 Allow 規則明確放行的請求，由 WAF Default Action（Block）攔截。 |
| 5 | — | — | — | 0 | 0% | 無其他規則觸發封鎖。 |

---

## 3. 每小時封鎖時序（Taipei UTC+8）

| 時段（+8） | 封鎖次數 | 允許次數 | 總請求數 | 封鎖率 | 備註 |
|-----------|---------|---------|---------|-------|------|
| 00:00–01:00 | 1 | 50 | 51 | 2.0% | |
| 01:00–02:00 | 2 | 0 | 2 | 100.0% | |
| 02:00–03:00 | 148 | 3 | 151 | 98.0% | ⚠️ 夜間攻擊波（巴西 IP） |
| 03:00–04:00 | 160 | 2 | 162 | 98.8% | ⚠️ **日最高峰**（巴西 IP） |
| 04:00–05:00 | 1 | 1 | 2 | 50.0% | |
| 05:00–06:00 | 3 | 0 | 3 | 100.0% | |
| 06:00–07:00 | 0 | 0 | 0 | — | 靜默期 |
| 07:00–08:00 | 1 | 1 | 2 | 50.0% | |
| 08:00–09:00 | 0 | 4 | 4 | 0.0% | |
| 09:00–10:00 | 0 | 369 | 369 | 0.0% | 早上上班流量湧入 |
| 10:00–11:00 | 111 | 432 | 543 | 20.4% | ⚠️ 第二波巴西攻擊 |
| 11:00–12:00 | 2 | 1240 | 1242 | 0.2% | 日最高允許流量（正常業務高峰） |
| 12:00–13:00 | 0 | 4 | 4 | 0.0% | |
| 13:00–14:00 | 42 | 105 | 147 | 28.6% | 美國 IP 封鎖突增 |
| 14:00–15:00 | 5 | 127 | 132 | 3.8% | |
| 15:00–16:00 | 0 | 9 | 9 | 0.0% | |
| 16:00–17:00 | 55 | 121 | 176 | 31.3% | 美國 IP 再次封鎖 |
| 17:00–18:00 | 1 | 8 | 9 | 11.1% | |
| 18:00–19:00 | 0 | 62 | 62 | 0.0% | |
| 19:00–20:00 | 0 | 2 | 2 | 0.0% | |
| 20:00–21:00 | 0 | 1 | 1 | 0.0% | |
| 21:00–22:00 | 1 | 1 | 2 | 50.0% | |
| 22:00–23:00 | 137 | 3 | 140 | 97.9% | ⚠️ 第三波巴西夜間攻擊 |
| 23:00–24:00 | 1 | 0 | 1 | 100.0% | |
| **合計** | **671** | **2,545** | **3,216** | **20.86%** | |

### ASCII 長條圖（封鎖次數，每 `█` = 10 次）

```
03:00 ████████████████ 160
02:00 ███████████████  148
22:00 █████████████▋   137
10:00 ███████████      111
16:00 █████▌           55
13:00 ████▏            42
14:00 ▌                5
05:00 ▎                3
11:00 ▏                2
01:00 ▏                2
00:00 ▏                1
04:00 ▏                1
07:00 ▏                1
17:00 ▏                1
21:00 ▏                1
23:00 ▏                1
```

**時序觀察：**
- **低水位時段**（06:00–09:00）：服務流量低，幾乎無封鎖，屬正常期。
- **夜間攻擊波**（01:00–05:00）：巴西 HostingProvider IP 集中在凌晨發動，02:00 與 03:00 各達 148/160 次，封鎖率近 100%，表明這段時間幾乎純為惡意自動化請求。
- **日間第二/三波**（10:00、13:00–16:00）：混入正常業務流量，以美國主機商 IP 為主，封鎖率 20–31%。
- **22:00 再現**：第三波巴西 IP 攻擊（137 次），夜間週期性行為明顯。

---

## 4. 來源國家分布

| 排名 | 國家 | 代碼 | 封鎖次數 | 佔比 | 主要觸發規則 |
|------|------|------|---------|------|------------|
| 1 | 🇧🇷 巴西 | BR | 551 | 82.1% | HostingProviderIPList（Count+DefaultBlock） |
| 2 | 🇺🇸 美國 | US | 113 | 16.8% | HostingProviderIPList（Count+DefaultBlock） |
| 3 | 🇯🇵 日本 | JP | 2 | 0.3% | Default Block / 其他 |
| 4 | 🇳🇱 荷蘭 | NL | 2 | 0.3% | Default Block / 其他 |
| 5 | 🇦🇺 澳洲 | AU | 1 | 0.1% | Default Block / 其他 |
| 6 | 🇬🇧 英國 | GB | 1 | 0.1% | Default Block / 其他 |
| 7 | 🇱🇺 盧森堡 | LU | 1 | 0.1% | Default Block / 其他 |

**在 list-metrics 中出現但封鎖次數為 0 的國家**（代表有允許流量但未封鎖）：
BG（保加利亞）、CA（加拿大）、CN（中國）、DE（德國）、FR（法國）、ID（印尼）、IN（印度）、KR（韓國）、MY（馬來西亞）、RO（羅馬尼亞）、RU（俄羅斯）、SG（新加坡）、TW（台灣）

### ASCII 橫向長條圖

```
🇧🇷 BR ████████████████████████████████████████████████████████████████  551
🇺🇸 US █████████████                                                        113
🇯🇵 JP                                                                        2
🇳🇱 NL                                                                        2
🇦🇺 AU                                                                        1
🇬🇧 GB                                                                        1
🇱🇺 LU                                                                        1
```

> 巴西佔比 82.1% 明顯異常，且集中於主機商 IP 段（Hosting Provider），這是 Bot Farm 或 Proxy 網路的典型特徵。

---

## 5. 異常事件分析

### 事件 A：巴西主機商 IP 大規模自動化流量

- **觸發規則**：HostingProviderIPList（AWSManagedRulesAnonymousIpList）→ anonymous-ip-captcha / Default Block
- **發生時段**：02:00–04:00、10:00–11:00、22:00–23:00（Taipei +8）
- **次數**：共 551 次（巴西 IP）
- **評估**：威脅等級 **中高**。巴西 IP 集中來自主機/雲端提供商 IP 段，非終端用戶 IP，強烈暗示為租用 VPS 或 Bot 網路發起的自動化掃描或爬蟲行為。夜間三波週期性攻擊（02:00、10:00 為當地深夜/早晨）符合攻擊者調度模式。由於 `managed-anonymous-ip` 目前為 Count 模式，CAPTCHA 挑戰依賴用戶互動，對純自動化 Bot 效果有限。

### 事件 B：美國主機商 IP 日間滲透嘗試

- **觸發規則**：HostingProviderIPList → anonymous-ip-captcha / Default Block
- **發生時段**：13:00–14:00（42 次）、16:00–17:00（55 次）
- **次數**：共 113 次（美國 IP）
- **評估**：威脅等級 **中**。時段集中於台灣下午上班時間，與正常業務流量混合，封鎖率 28–31%。來源為美國主機商 IP（如 AWS/GCP/Azure 出口 IP），可能為使用雲端服務發起的 scraping 或 API 探測行為。

---

## 6. 共通性分析

### 攻擊類型分析
- **主要威脅類型**：匿名主機商 IP 流量（Bot/Automated 爬蟲、API 探測），佔總封鎖 97%。HostingProviderIPList 命中代表請求來源於已知 VPS/雲端主機商的 IP 段，而非真實終端用戶。
- **Attack 維度交叉驗證**：Attack 維度（EC2MetaDataSSRF、GenericLFI、KnownBadInputs、RestrictedExtensions）**全數為 0**，表明攻擊行為未觸發特定漏洞攻擊規則。VulnerabilityCategory=HostingProviderIPList 記錄到 372 次，與 ManagedRuleGroupRule 維度的 651 次有差異，係 CW 指標維度採樣機制差異。
- **未偵測到的攻擊類型**：SQL Injection（AWSManagedRulesSQLiRuleSet 無封鎖）、XSS/CommonRuleSet 子規則（EC2MetaDataSSRF_BODY/COOKIE、GenericLFI_URIPATH、RestrictedExtensions_URIPATH 全數為 0）、Known Bad Inputs（ReactJSRCE_BODY、ExploitablePaths_URIPATH 全數為 0）。

### 時間模式
- **夜間週期性攻擊**：02:00–04:00 每日凌晨為攻擊高峰，符合 Bot 排程執行特徵（Cron Job）。
- **業務流量與攻擊混雜**：10:00–16:00 出現攻擊流量與正常業務流量並行，封鎖率 20–31%，需持續監控。
- **持續性 vs 集中性**：攻擊呈間歇性集中（非持續低頻），每波 1 小時內完成，後恢復靜默。

### 地理模式
- **巴西 82.1% 異常**：巴西並非 giver 服務的主要目標用戶市場，此比例顯著偏高，強烈指向攻擊者利用巴西 VPS/Bot 節點。
- **美國主機商 IP**：美國佔 16.8%，主要集中在主機商段，可能為多跳 Proxy 或 Bot 節點。
- **與 Bot 活躍時段關聯**：凌晨攻擊集中時段（UTC 18:00–20:00）與美洲時區下班/深夜相符，可能為人工控制的 Bot 排程。

### 規則效果評估
- **HostingProviderIPList**：Count OverrideAction 限制了直接封鎖效能。651 次匹配僅能透過 CAPTCHA 或 Default Block 阻擋。建議評估改為 Block OverrideAction。
- **anonymous-ip-captcha**：CAPTCHA 對純 Bot 有效性有限，未在 BlockedRequests 維度中出現，代表 CAPTCHA 失敗次數不顯著或未獨立計量。
- **managed-reputation-ip**：僅封鎖 2 次，說明此次攻擊 IP 主要不在 Amazon IP 信譽清單，而是一般主機商 IP。
- **自訂規則（detected-malicious-ip、malicious-ip-block、malicious-user-agent）**：本日均無觸發，惡意 IP 清單未能覆蓋此波攻擊 IP。

---

## 7. 建議事項

### 短期建議（立即～2 週）

1. **調閱 Sampled Requests**：立即從 AWS Console → WAF → giver-acl-waf → managed-anonymous-ip，查看 HostingProviderIPList 的 Sampled Requests，確認實際請求 URI、User-Agent、來源 IP 段，判斷是否為特定 scraper 工具。
2. **評估 anonymous-ip OverrideAction 調整**：
   - 若確認巴西 HostingProvider IP 無正常業務需求，將 `managed-anonymous-ip` OverrideAction 從 `Count` 改為 `None`（啟用 Block），可直接封鎖 651 次匹配。
   - 或新增自訂 IP Set 規則，將 Sampled Requests 中的 CIDR 範圍加入 `detected_malicious_ip_set` 以 Block。
3. **確認 anonymous-ip-captcha 效果**：測試 CAPTCHA 在此 Bot 流量上的實際阻擋率。若 Bot 能自動解 CAPTCHA，考慮升級為 Block 動作。
4. **更新 malicious IP Set**：將 Sampled Requests 中出現的巴西/美國 Bot IP CIDR 段，手動加入 `detected_malicious_ip_set`（P0）或 `Malicious-ipset`（P1）IP Set。

### 中期建議（2 週～2 個月）

1. **CloudWatch Alarm 設定**：
   - 為 `Rule=ALL BlockedRequests` 設定 Alarm，閾值建議 > 300/小時（目前峰值 160/小時的 2 倍），通知 Security 團隊。
   - 為 `HostingProviderIPList BlockedRequests` 設定單獨 Alarm > 100/小時。
2. **啟用 WAF 完整 Logging**：目前可查詢的資訊僅為 CloudWatch 指標（無 IP/URI 詳情），強烈建議啟用 WAF 日誌並傳送至 S3 或 CloudWatch Logs，以支援 IP 黑名單自動更新和 Athena 查詢分析。
3. **Bot Control 考量**：評估是否導入 AWS WAF Bot Control（Managed Rule Group），可精確識別 known bots（搜尋引擎、監控工具）並封鎖 scraper，降低誤封正常 Bot（如 SendGrid、Googlebot 等）的風險。
4. **地理封鎖評估**：若 giver 服務確認無巴西用戶業務，可新增 GeoMatch 規則封鎖 BR，但需先確認不影響巴西員工/合作夥伴存取。

### 長期建議（2 個月以上）

1. **自動化 WAF 報告機制**：建立定期（每日/每週）自動化 WAF 分析報告，整合 CloudWatch Metrics + WAF Logs + Athena 查詢，提供趨勢分析和攻擊者 IOC（Indicator of Compromise）。
2. **WAF Logging + Athena 分析架構**：WAF Logs → S3 → Athena，支援按 IP、URI、User-Agent、Country 的自助式查詢，實現 IP 信譽自動黑名單更新。
3. **安全架構改進**：考慮在 CloudFront Distribution 前端整合 AWS Shield Advanced，提升 DDoS 防護，並與 WAF 聯動自動封鎖高流量攻擊源。

---

## 8. WAF ACL 規則完整清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---------|---------|------|------------------------|-------------|------|
| 0 | `detected-malicious-ip` | 自訂（IP Set） | **Block** | `giver-waf-detected-malicious-ip` | 動態偵測惡意 IP 清單（`detected_malicious_ip_set`）直接封鎖 |
| 1 | `malicious-ip-block` | 自訂（IP Set） | **Block** | `giver-waf-malicious-ip-block` | 靜態惡意 IP 清單（`Malicious-ipset`）直接封鎖 |
| 10 | `StaticFilePatterns` | 自訂（Regex） | **Allow** | `giver-waf-static-file-patterns` | 靜態資源路徑（cf-static-file-path-patterns）優先放行 |
| 20 | `managed-core` | ManagedRuleGroup（AWSManagedRulesCommonRuleSet） | OverrideAction: **None** | `giver-waf-managed-core` | AWS 核心規則集；排除 NoUserAgent、GenericRFI_QUERYARGUMENTS、SizeRestrictions_BODY/Cookie |
| 30 | `managed-reputation-ip` | ManagedRuleGroup（AWSManagedRulesAmazonIpReputationList） | OverrideAction: **None** | `giver-waf-managed-reputation-ip` | Amazon IP 信譽清單；封鎖已知惡意/偵察 IP |
| 40 | `managed-sql-injection` | ManagedRuleGroup（AWSManagedRulesSQLiRuleSet） | OverrideAction: **None** | `giver-waf-managed-sql-injection` | SQL Injection 防護 |
| 50 | `managed-known-bad-input` | ManagedRuleGroup（AWSManagedRulesKnownBadInputsRuleSet） | OverrideAction: **None** | `giver-waf-managed-known-bad-input` | 已知惡意輸入模式防護（Log4Shell、SSRF 等） |
| 100 | `internal-ip-limitation` | 自訂（IP Set） | **Allow** | `giver-waf-internal-ip-limitation` | 內部 IP（`internal-ipset`）無條件放行 |
| 110 | `managed-anonymous-ip` | ManagedRuleGroup（AWSManagedRulesAnonymousIpList） | OverrideAction: **Count** | `giver-waf-managed-anonymous-ip` | 匿名 IP（Tor、VPN、主機商）標記計數；加 Label 供後續規則使用 |
| 120 | `bot-white-list` | 自訂（Label Match） | **Allow** | `giver-waf-bot-whitelist` | 白名單 Email Client Bot（`awswaf:managed:aws:bot-control:bot:category:email_client`）放行 |
| 130 | `anonymous-ip-captcha` | 自訂（Label Match） | **Captcha** | `giver-waf-anonymous-ip-captcha` | 匿名 IP 請求觸發 CAPTCHA 挑戰（Label: `awswaf:managed:aws:anonymous-ip-list:AnonymousIPList`） |
| 140 | `request-rate-breaker` | 自訂（Rate Based） | **Count** | `giver-waf-request-rate-breaker` | 速率限制計數（700 次 / 300 秒 / IP），僅計數不封鎖 |
| 150 | `request-rate-captcha` | 自訂（Rate Based） | **Captcha** | `giver-waf-request-rate-captcha` | 超速 IP 觸發 CAPTCHA（700 次 / 300 秒 / IP） |
| 160 | `malicious-user-agent` | 自訂（AND: Regex + Geo） | **Block** | `giver-waf-malicious-user-agent` | 特定惡意 User-Agent 且來源為 CN（中國）的請求直接封鎖 |
| — | **Default Action** | — | **Block** | — | 未被任何規則明確 Allow 的請求一律封鎖 |

---

## 9. 資料來源與查詢說明

### 查詢環境

| 項目 | 值 |
|------|-----|
| AWS Profile | `104awsdev14-ro` |
| WAF Scope | `CLOUDFRONT` |
| CW Namespace | `AWS/WAFV2` |
| Period | 3600 秒（每小時） |
| Region | `us-east-1`（CloudFront WAF 固定） |
| UTC 查詢區間 | `2026-06-01T16:00:00Z` ～ `2026-06-02T16:00:00Z` |
| 台北時間 | `2026-06-02 00:00` ～ `2026-06-02 23:59` |

### 查詢 Dimension 說明

| Dimension 組合 | 用途 |
|---------------|------|
| `WebACL=giver-acl-waf, Rule=ALL` | 總封鎖量基準 |
| `WebACL=giver-acl-waf, Rule={MetricName}` | 各父規則封鎖量 |
| `WebACL=giver-acl-waf, ManagedRuleGroup={}, ManagedRuleGroupRule={}` | 各 Managed 子規則封鎖量 |
| `WebACL=giver-acl-waf, Country={code}` | 各來源國家封鎖量 |
| `WebACL=giver-acl-waf, Attack={}` | 攻擊類型交叉驗證 |
| `WebACL=giver-acl-waf, VulnerabilityCategory={}` | 弱點類別交叉驗證 |
| `AllowedRequests, WebACL=giver-acl-waf, Rule=ALL` | 總允許量（封鎖率計算基準） |

### 資料限制

1. **CloudWatch 為彙總統計**：僅提供每小時計數，無法得知來源 IP、請求 URI、User-Agent 等詳細資訊。需啟用 WAF Logging 至 S3 / CloudWatch Logs 才能進行細粒度分析。
2. **ManagedRuleGroup Count OverrideAction 行為**：`managed-anonymous-ip`（P110）設定 Count OverrideAction，其子規則（HostingProviderIPList）的 `BlockedRequests` 指標記錄「匹配此規則的請求被最終封鎖的次數」，反映的是規則配對結果，而非該規則直接封鎖。
3. **Country 維度代表性**：Country 維度僅反映封鎖事件的 IP 地理位置，使用 VPN/Proxy 的攻擊者可偽裝來源國家。
4. **Attack / VulnerabilityCategory 維度採樣差異**：此維度的 BlockedRequests 計數可能低於 ManagedRuleGroupRule 維度，兩者用途不同，僅供交叉驗證參考。

### 查詢步驟摘要

1. `aws wafv2 list-web-acls`：列出所有 CloudFront WAF ACL，取得 `giver-acl-waf` 的 ID。
2. `aws wafv2 get-web-acl`：取得完整規則清單（14 條規則 + Default Action）。
3. `aws cloudwatch list-metrics`：列出 `AWS/WAFV2` Namespace 下所有含 `WebACL=giver-acl-waf` 的 `BlockedRequests` Dimensions（共 57 組）。
4. `aws cloudwatch get-metric-statistics`（3 批、共 38 次查詢）：分批查詢所有 Dimensions，涵蓋 Rule、ManagedRuleGroupRule、Country（20 國）、Attack（4 類）、VulnerabilityCategory（3 類）、AllowedRequests。

---

*報告產生時間：2026-06-03 Taipei Time*
*資料來源：AWS CloudWatch Metrics（AWS/WAFV2）、AWS WAFv2 API（us-east-1）*
*分析工具：AWS CLI（profile: 104awsdev14-ro）、Python 3*
