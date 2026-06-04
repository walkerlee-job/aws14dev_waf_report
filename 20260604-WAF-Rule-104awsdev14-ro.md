# AWS WAF 規則說明與上週封鎖統計報告

**AWS Profile**：`104awsdev14-ro`  
**Scope**：CLOUDFRONT（us-east-1）  
**分析週期（Taipei UTC+8）**：2026-05-25（週一）00:00 ～ 2026-05-31（週日）23:59  
**UTC 查詢區間**：2026-05-24T16:00:00Z ～ 2026-06-01T15:59:59Z  
**報告產生時間**：2026-06-04 Taipei Time  

---

## 一、上週封鎖次數總覽（所有 ACL，週一至週日）

| Web ACL | 05/25(一) | 05/26(二) | 05/27(三) | 05/28(四) | 05/29(五) | 05/30(六) | 05/31(日) | **週合計** |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| beagiver-acl-waf | 179 | 509 | 804 | 913 | 338 | 144 | 139 | **3,026** |
| beagiver-event-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** |
| blog-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** |
| clinic-acl-waf | 695 | 407 | 769 | 213 | 17 | 163 | 272 | **2,536** |
| giver-acl-waf | 361 | 201 | 472 | 397 | 36 | 249 | 143 | **1,859** |
| go-acl-waf | 195 | 139 | 89 | 119 | 124 | 33 | 50 | **749** |
| internal-acl-waf | 251 | 195 | 171 | 27 | 83 | 28 | 294 | **1,049** |
| intro-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** |
| meet-acl-waf | 111 | 176 | 193 | 19 | 20 | 63 | 150 | **732** |
| nabi-acl-waf | 270 | 338 | 668 | 192 | 367 | 452 | 386 | **2,673** |
| **每日合計** | **2,062** | **1,965** | **3,166** | **1,880** | **985** | **1,132** | **1,434** | **12,624** |

> **週總封鎖次數：12,624 次**  
> 週三（05/27）為全週封鎖高峰，單日 3,166 次。週五（05/29）封鎖量最低，為 985 次。
> beagiver-acl-waf、nabi-acl-waf、clinic-acl-waf 為封鎖最多的前三大 ACL。

---

## 二、各 ACL 統計、規則說明與封鎖明細

---

### 2-1. beagiver-acl-waf

**描述**：beagiver 服務的 CloudFront WAF（預設動作：Block）  
**上週封鎖總量**：3,026 次（占全帳號 24.0%）  
**尖峰日**：週四 05/28（913 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 1,178 | 38.9% |
| Default Action（Block） | 預設封鎖 | 1,451 | 47.9% |
| managed-known-bad-input | AWS 受管 – 已知惡意輸入 | 258 | 8.5% |
| managed-core | AWS 受管 – 核心規則集 | 139 | 4.6% |
| 其他規則（SQL注入、惡意UA等） | - | 0 | 0% |

> Default Action 佔最高比例（47.9%），代表大量請求未符合任何 Allow 規則，直接被預設 Block 擋下。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | vulnerability-scan-whitelist | 自訂（IPSet） | **Allow** | beagiver-waf-vulnerability-scan-whitelist | 漏洞掃描白名單 IP，允許通過供安全測試 |
| 1 | bypass_static_files | 自訂（RegexMatch） | **Allow** | beagiver-waf-bypass-static-files | 靜態檔案路徑（/errorpages、/static、2023/2024giverreview）直接放行 |
| 2 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | beagiver-waf-managed-core | AWS 核心規則集，偵測 OWASP Top 10 攻擊（XSS、LFI、RCE 等）。部分規則調整為 Count：NoUserAgent_HEADER、GenericRFI_QUERYARGUMENTS、SizeRestrictions_BODY、SizeRestrictions_Cookie_HEADER |
| 3 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | beagiver-waf-managed-reputation-ip | Amazon 威脅情報 IP 聲譽清單，封鎖已知 Bot、掃描器、惡意來源 IP |
| 4 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | beagiver-waf-managed-sql-injection | SQL 注入攻擊防護，檢查 URI、查詢字串、Headers、Body 中的 SQL 注入特徵 |
| 5 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | beagiver-waf-managed-known-bad-input | 已知惡意輸入防護，包含 Log4j/JNDI 注入、SSRF、Spring Cloud RCE 等攻擊模式 |
| 6 | internal-ip-limitation | 自訂（IPSet） | **Allow** | beagiver-waf-internal-ip-limitation | 內部 IP 白名單，允許內部系統直接存取 |
| 7 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | beagiver-waf-managed-anonymous-ip | 匿名 IP 偵測（VPN、TOR、代理），設定為 Count 模式（僅標記不封鎖） |
| 8 | bot-white-list | 自訂（LabelMatch） | **Allow** | beagiver-waf-bot-whitelist | 放行 Bot-Control 標記為 email_client 類別的 Bot（如郵件爬蟲） |
| 9 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | beagiver-waf-anonymous-ip-captcha | 對匿名 IP 清單標記的流量發送 CAPTCHA 驗證 |
| 10 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | beagiver-waf-request-rate-breaker | 頻率限制（每 IP 5 分鐘 700 次），超出時計數（供監控用） |
| 11 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | beagiver-waf-request-rate-captcha | 頻率超限的 IP 發送 CAPTCHA 驗證 |
| 12 | malicious-user-agent | 自訂（And: RegexUA + GeoMatch CN） | **Block** | beagiver-waf-malicious-user-agent | 封鎖來自中國（CN）且 User-Agent 符合惡意特徵的請求 |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-2. beagiver-event-acl-waf

**描述**：beagiver 活動頁面專用 WAF（預設動作：Block）  
**上週封鎖總量**：0 次（活動頁面 WAF 無流量觸發，可能尚未對外開放或流量極低）

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | bypass_static_files | 自訂（OrStatement Regex） | **Allow** | beagiver-event-waf-bypass-static-files | 靜態資源路徑放行：svg/jpg/png/js/css/ico/gif/json/txt/html、/css/、/img/、/index.html |
| 1 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | beagiver-event-waf-managed-core | AWS 核心規則集（XSS、LFI、RCE 等）。部分調整為 Count：NoUserAgent、GenericRFI、SizeRestrictions BODY/Cookie |
| 2 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | beagiver-event-waf-managed-reputation-ip | Amazon IP 聲譽清單，封鎖惡意來源 IP |
| 3 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | beagiver-event-waf-managed-sql-injection | SQL 注入攻擊防護 |
| 4 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | beagiver-event-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等已知惡意輸入防護 |
| 5 | internal-ip-limitation | 自訂（IPSet） | **Allow** | beagiver-event-waf-internal-ip-limitation | 內部 IP 白名單放行 |
| 6 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | beagiver-event-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式，僅標記） |
| 7 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | beagiver-event-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA 驗證 |
| 8 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | beagiver-event-waf-request-rate-breaker | 頻率限制計數 |
| 9 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | beagiver-event-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-3. blog-acl-waf

**描述**：WordPress 部落格網站專用 WAF（預設動作：Block）  
**上週封鎖總量**：0 次（無封鎖事件，但規則仍在運作）

> 注意：blog-acl-waf 含有 managed-bot-control 和 managed-anonymous-ip 均設定為 Count/OverrideAction=Count，多數攔截以標記為主，實際 Block 事件為零。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | request-rate-count | 自訂（And: Not+RegexPatternSet） | **Count** | blog-waf-request-rate-count | 符合特定 URI 模式的請求計數（計量用，不封鎖） |
| 1 | bypass_static_files | 自訂（OrStatement Regex） | **Allow** | blog-waf-bypass-static-files | 靜態資源放行：svg/jpg/png/js/css 等格式，及 /errorpages/、/wp-content/uploads/、/wp-includes/js/ 等路徑 |
| 2 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | blog-waf-managed-core | 核心規則集防護。Count 模式：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie、CrossSiteScripting_BODY |
| 3 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | blog-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 4 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | blog-waf-managed-sql-injection | SQL 注入防護 |
| 5 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | blog-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 6 | managed-wordpress-rules | AWS ManagedRuleGroup：AWSManagedRulesWordPressRuleSet | OverrideAction=**None** | blog-waf-managed-wordpress-rules | WordPress 專用防護：防禦 wp-admin 未授權存取、惡意插件利用等 WordPress 特有漏洞 |
| 7 | managed-php-rules | AWS ManagedRuleGroup：AWSManagedRulesPHPRuleSet | OverrideAction=**None** | blog-waf-managed-php-rules | PHP 應用防護：防禦 RFI（遠端檔案引入）、本地檔案引入、PHP 特殊字元注入等 |
| 8 | managed-bot-control | AWS ManagedRuleGroup：AWSManagedRulesBotControlRuleSet | OverrideAction=**Count** | blog-waf-managed-bot-control | Bot 流量偵測（Count 模式，僅標記 Bot 類別，不封鎖） |
| 9 | add-is-bot-header | 自訂（LabelMatch namespace） | **Count**（插入 X-WAF-IS-BOT: true） | blog-waf-add-is-bot-header | 對 Bot 流量插入 X-WAF-IS-BOT 請求頭，供後端識別 |
| 10 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | blog-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 11 | add-is-anonymous-header | 自訂（LabelMatch namespace） | **Count**（插入 X-WAF-IS-ANONYMOUS: true） | blog-waf-add-is-anonymous-header | 對匿名 IP 流量插入識別請求頭 |
| 12 | block-admin-from-external-ip | 自訂（And: Or(wp-admin/wp-login/wp-json...) + Not(internalIPSet) + Not(admin-ajax)） | **Block** | blog-waf-block-admin-from-external-ip | 封鎖來自非內部 IP 的 WordPress 管理後台存取（wp-admin、wp-login.php、wp-json/wp/v2/users 等） |
| 13 | internal-ip-limitation | 自訂（IPSet） | **Allow** | blog-waf-internal-ip-limitation | 內部 IP 白名單放行 |
| 14 | bot-white-list | 自訂（LabelMatch） | **Allow** | blog-waf-bot-whitelist | 放行 email_client 類別 Bot |
| 15 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | blog-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA 驗證 |
| 16 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | blog-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-4. clinic-acl-waf

**描述**：clinic 服務 WAF（預設動作：Block）  
**上週封鎖總量**：2,536 次（占全帳號 20.1%）  
**尖峰日**：週三 05/27（769 次），週一 05/25（695 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 2,141 | 84.4% |
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 365 | 14.4% |
| managed-core | AWS 受管 – 核心規則集 | 17 | 0.7% |
| managed-known-bad-input | AWS 受管 – 已知惡意輸入 | 13 | 0.5% |
| 其他（malicious-user-agent 等） | - | 0 | 0% |

> Default Action 高達 84.4%，表示大量外部流量因不符合允許規則而被擋下，此為預設 Block 架構的正常現象。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | vulnerability_scan_whitelist | 自訂（IPSet） | **Allow** | vulnerability_scan_whitelist | 漏洞掃描白名單 IP |
| 1 | bypass_static_files | 自訂（RegexPatternSet） | **Allow** | clinic-waf-bypass-static-files | 靜態檔案路徑白名單（依正則表達式 Pattern Set 定義） |
| 2 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | clinic-waf-managed-core | 核心規則集。Count 模式：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie |
| 3 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | clinic-waf-managed-reputation-ip | Amazon IP 聲譽清單，封鎖惡意來源 |
| 4 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | clinic-waf-managed-sql-injection | SQL 注入防護 |
| 5 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | clinic-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 6 | internal-ip-limitation | 自訂（IPSet） | **Allow** | clinic-waf-internal-ip-limitation | 內部 IP 白名單 |
| 7 | surveycake-ip-allowlist | 自訂（IPSet） | **Allow** | clinic-waf-surveycake-ip-allowlist | SurveyCake 問卷服務 IP 白名單，允許其 Webhook 回調 |
| 8 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | clinic-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 9 | bot-white-list | 自訂（LabelMatch） | **Allow** | clinic-waf-bot-whitelist | 放行 email_client Bot |
| 10 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | clinic-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 11 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | clinic-waf-request-rate-breaker | 頻率超限計數 |
| 12 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | clinic-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| 13 | malicious-user-agent | 自訂（And: RegexUA + GeoMatch CN） | **Block** | clinic-waf-malicious-user-agent | 封鎖來自中國且 User-Agent 符合惡意特徵的請求 |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-5. giver-acl-waf

**描述**：giver 服務 WAF（預設動作：Block）  
**上週封鎖總量**：1,859 次（占全帳號 14.7%）  
**尖峰日**：週三 05/27（472 次），週四 05/28（397 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 1,669 | 89.8% |
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 155 | 8.3% |
| managed-core | AWS 受管 – 核心規則集 | 21 | 1.1% |
| managed-known-bad-input | AWS 受管 – 已知惡意輸入 | 14 | 0.8% |
| 其他（detected-malicious-ip、malicious-ip-block 等） | - | 0 | 0% |

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | detected-malicious-ip | 自訂（IPSet: detected_malicious_ip_set） | **Block** | giver-waf-detected-malicious-ip | 封鎖已被偵測為惡意的 IP（動態更新的威脅情報 IPSet） |
| 1 | malicious-ip-block | 自訂（IPSet: Malicious-ipset） | **Block** | giver-waf-malicious-ip-block | 封鎖已知惡意 IP（靜態黑名單 IPSet） |
| 10 | StaticFilePatterns | 自訂（RegexPatternSet） | **Allow** | giver-waf-static-file-patterns | 靜態檔案路徑放行 |
| 20 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | giver-waf-managed-core | 核心規則集。ExcludedRules（排除不封鎖）：NoUserAgent_HEADER、GenericRFI_QUERYARGUMENTS、SizeRestrictions_BODY/Cookie |
| 30 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | giver-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 40 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | giver-waf-managed-sql-injection | SQL 注入防護 |
| 50 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | giver-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 100 | internal-ip-limitation | 自訂（IPSet） | **Allow** | giver-waf-internal-ip-limitation | 內部 IP 白名單 |
| 110 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | giver-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 120 | bot-white-list | 自訂（LabelMatch） | **Allow** | giver-waf-bot-whitelist | 放行 email_client Bot |
| 130 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | giver-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 140 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | giver-waf-request-rate-breaker | 頻率超限計數 |
| 150 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | giver-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| 160 | malicious-user-agent | 自訂（And: RegexUA + GeoMatch CN） | **Block** | giver-waf-malicious-user-agent | 封鎖來自中國且 User-Agent 符合惡意特徵的請求 |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-6. go-acl-waf

**描述**：go（WordPress 版）服務 WAF（預設動作：Block）  
**上週封鎖總量**：749 次（占全帳號 5.9%）  
**尖峰日**：週一 05/25（195 次），週二 05/26（139 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 728 | 97.2% |
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 15 | 2.0% |
| managed-known-bad-input | AWS 受管 – 已知惡意輸入 | 6 | 0.8% |
| 其他（block-admin、managed-core 等） | - | 0 | 0% |

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | bypass_static_files | 自訂（OrStatement Regex） | **Allow** | go-waf-bypass-static-files | 靜態資源放行：svg/jpg/png/js/css 等格式，及 /errorpages/、/wp-content/uploads/、/wp-includes/js/ 等路徑 |
| 1 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | go-waf-managed-core | 核心規則集。Count：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie、CrossSiteScripting_BODY |
| 2 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | go-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 3 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | go-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | go-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 5 | managed-wordpress-rules | AWS ManagedRuleGroup：AWSManagedRulesWordPressRuleSet | OverrideAction=**None** | go-waf-managed-wordpress-rules | WordPress 專用防護：防禦 wp-admin 未授權存取、惡意插件利用等 |
| 6 | managed-php-rules | AWS ManagedRuleGroup：AWSManagedRulesPHPRuleSet | OverrideAction=**None** | go-waf-managed-php-rules | PHP 應用防護：防禦 RFI、本地檔案引入、PHP 注入等 |
| 7 | managed-bot-control | AWS ManagedRuleGroup：AWSManagedRulesBotControlRuleSet | OverrideAction=**Count** | go-waf-managed-bot-control | Bot 流量偵測（Count 模式，僅標記） |
| 8 | add-is-bot-header | 自訂（LabelMatch namespace） | **Count**（插入 X-WAF-IS-BOT: true） | go-waf-add-is-bot-header | 對 Bot 流量插入識別 Header |
| 9 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | go-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 10 | add-is-anonymous-header | 自訂（LabelMatch namespace） | **Count**（插入 X-WAF-IS-ANONYMOUS: true） | go-waf-add-is-anonymous-header | 對匿名 IP 流量插入識別 Header |
| 11 | block-admin-from-external-ip | 自訂（And: Or(wp-admin/wp-login/wp-json) + Not(internalIPSet) + Not(admin-ajax)） | **Block** | go-waf-block-admin-from-external-ip | 封鎖非內部 IP 存取 WordPress 管理後台 |
| 12 | internal-ip-limitation | 自訂（IPSet） | **Allow** | go-waf-internal-ip-limitation | 內部 IP 白名單 |
| 13 | bot-white-list | 自訂（LabelMatch） | **Allow** | go-waf-bot-whitelist | 放行 email_client Bot |
| 14 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | go-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 15 | request-rate-breaker | 自訂（RateBased 300/5min） | **Count** | go-waf-request-rate-breaker | 頻率計數（較嚴格，每 IP 5 分鐘 300 次閾值） |
| 16 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | go-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-7. internal-acl-waf

**描述**：內部服務專用 WAF，設計為只允許內部 IP 存取（預設動作：Block）  
**上週封鎖總量**：1,049 次（占全帳號 8.3%）  
**尖峰日**：週日 05/31（294 次），週一 05/25（251 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 1,049 | **100%** |

> 此 ACL 架構極為簡潔：唯一規則為內部 IP 白名單（Allow），所有非內部 IP 來源的請求全部由 Default Action Block 擋下。1,049 次封鎖均為非內部 IP 嘗試存取內部服務。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | internal-ip-limitation | 自訂（IPSet） | **Allow** | internal-ip-limitation | 僅允許內部 IP 白名單通過，其餘全部封鎖 |
| - | **Default Action** | - | **Block** | - | 所有非內部 IP 均被封鎖 |

---

### 2-8. intro-acl-waf

**描述**：intro（介紹頁）服務 WAF（預設動作：Block）  
**上週封鎖總量**：0 次

> intro 為靜態介紹頁面，包含多個子服務（beagiver、blog、meet、talentmarket、lms、salary、personalbrand、senior 等），上週無封鎖事件。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | bypass_static_files | 自訂（OrStatement Regex） | **Allow** | intro-waf-bypass-static-files | 靜態資源及各子服務路徑放行（.js/.css、/errorpages/、/intro_beagiver__/、/intro_blog__/ 等） |
| 1 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | intro-waf-managed-core | 核心規則集。Count：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie、CrossSiteScripting_BODY |
| 2 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | intro-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 3 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | intro-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | intro-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 5 | managed-bot-control | AWS ManagedRuleGroup：AWSManagedRulesBotControlRuleSet | OverrideAction=**Count** | intro-waf-managed-bot-control | Bot 偵測（Count 模式） |
| 6 | add-is-bot-header | 自訂（LabelMatch namespace） | **Count**（插入 X-WAF-IS-BOT: true） | intro-waf-add-is-bot-header | 對 Bot 流量插入識別 Header |
| 7 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | intro-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 8 | internal-ip-limitation | 自訂（IPSet） | **Allow** | intro-waf-internal-ip-limitation | 內部 IP 白名單 |
| 9 | bot-white-list | 自訂（LabelMatch） | **Allow** | intro-waf-bot-whitelist | 放行 email_client Bot |
| 10 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | intro-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 11 | request-rate-breaker | 自訂（RateBased 300/5min） | **Count** | intro-waf-request-rate-breaker | 頻率計數（300 次/5min 閾值） |
| 12 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | intro-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-9. meet-acl-waf

**描述**：meet 服務 WAF（預設動作：Block）  
**上週封鎖總量**：732 次（占全帳號 5.8%）  
**尖峰日**：週三 05/27（193 次），週二 05/26（176 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 520 | 71.1% |
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 196 | 26.8% |
| managed-known-bad-input | AWS 受管 – 已知惡意輸入 | 12 | 1.6% |
| managed-core | AWS 受管 – 核心規則集 | 4 | 0.5% |

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | bypass_static_files | 自訂（OrStatement Regex） | **Allow** | meet-waf-bypass-static-files | 靜態資源放行：svg/jpg/png/js/css 等格式，及 /errorpages\|static/ 路徑 |
| 1 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | meet-waf-managed-core | 核心規則集。Count：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie |
| 2 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | meet-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 3 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | meet-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | meet-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 5 | internal-ip-limitation | 自訂（IPSet） | **Allow** | meet-waf-internal-ip-limitation | 內部 IP 白名單 |
| 6 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | meet-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 7 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | meet-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 8 | bot-white-list | 自訂（LabelMatch） | **Allow** | meet-waf-bot-whitelist | 放行 email_client Bot |
| 9 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | meet-waf-request-rate-breaker | 頻率超限計數 |
| 10 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | meet-waf-request-rate-captcha | 頻率超限發送 CAPTCHA |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

### 2-10. nabi-acl-waf

**描述**：nabi 服務 WAF，規則最複雜，含多種自訂防護（預設動作：Block）  
**上週封鎖總量**：2,673 次（占全帳號 21.2%）  
**尖峰日**：週三 05/27（668 次），週六 05/30（452 次）

#### 規則封鎖統計

| 規則名稱 | 規則類型 | 上週封鎖次數 | 佔比 |
|---|---|---:|---:|
| Default Action（Block） | 預設封鎖 | 2,067 | 77.3% |
| managed-reputation-ip | AWS 受管 – IP 聲譽清單 | 576 | 21.5% |
| ip_reputation_lists | 自訂（IPSet 組合） | 20 | 0.7% |
| managed-core | AWS 受管 – 核心規則集 | 8 | 0.3% |
| malicious-user-agent-no-area | 自訂（RegexUA） | 2 | 0.1% |
| 其他（scanners、http_flood、bad_bot 等） | - | 0 | 0% |

> nabi-acl-waf 擁有最豐富的自訂防護層，包含威脅情報 IPSet（ip_reputation_lists、scanners_and_probes、http_flood、bad_bot）及地理速率限制。managed-reputation-ip 單周攔截 576 次，是最有效的防護規則。

#### 完整規則清單

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|---|---|---|---|---|---|
| 0 | detect_page_view | 自訂（NotStatement OrStatement Regex） | **Count**（插入 X-ROUTE-TYPE: page_view） | detect_page_view | 偵測頁面瀏覽型請求（非 API、非靜態資源），插入 page_view 標籤供後續規則使用 |
| 1 | label_static_file_route | 自訂（RegexMatch） | **Count**（插入 X-ROUTE-TYPE: static_file_route） | label_static_file_route | 標記靜態資源路徑（_nuxt、asset、css、img、js 等） |
| 2 | label_ajax_route | 自訂（RegexMatch） | **Count**（插入 X-ROUTE-TYPE: ajax_route） | label_ajax_route | 標記 API 路徑（/api/、/v2/） |
| 3 | bypass_static_files | 自訂（RegexMatch） | **Allow** | bypass_static_files | 靜態資源路徑直接放行 |
| 4 | managed-core | AWS ManagedRuleGroup：AWSManagedRulesCommonRuleSet | OverrideAction=**None** | nabi-waf-managed-core | 核心規則集。Count：NoUserAgent、GenericRFI、SizeRestrictions_BODY/Cookie/QUERYSTRING |
| 5 | query-string-size-limit | 自訂（SizeConstraint QueryString > 4096 bytes） | **Block** | nabi-waf-query-string-size-limit | 封鎖查詢字串超過 4,096 bytes 的請求（防禦異常長 URL 攻擊） |
| 6 | managed-reputation-ip | AWS ManagedRuleGroup：AWSManagedRulesAmazonIpReputationList | OverrideAction=**None** | nabi-waf-managed-reputation-ip | Amazon IP 聲譽清單 |
| 7 | managed-sql-injection | AWS ManagedRuleGroup：AWSManagedRulesSQLiRuleSet | OverrideAction=**None** | nabi-waf-managed-sql-injection | SQL 注入防護 |
| 8 | managed-known-bad-input | AWS ManagedRuleGroup：AWSManagedRulesKnownBadInputsRuleSet | OverrideAction=**None** | nabi-waf-managed-known-bad-input | Log4j、SSRF、Spring RCE 等惡意輸入防護 |
| 9 | internal-ip-limitation | 自訂（IPSet） | **Allow** | nabi-waf-internal-ip-limitation | 內部 IP 白名單 |
| 10 | ip_reputation_lists | 自訂（Or: IPSet v4 + v6） | **Block** | ip_reputation_lists | 自訂威脅情報 IP 封鎖清單（IPv4 + IPv6），補充 AWS 受管清單不含的惡意 IP |
| 11 | managed-anonymous-ip | AWS ManagedRuleGroup：AWSManagedRulesAnonymousIpList | OverrideAction=**Count** | nabi-waf-managed-anonymous-ip | 匿名 IP 偵測（Count 模式） |
| 12 | bot-white-list | 自訂（LabelMatch） | **Allow** | nabi-waf-bot-whitelist | 放行 email_client Bot |
| 13 | page_view_rate_breaker | 自訂（RateBased 100/5min，ScopeDown: label=page_view） | **Captcha** | page_view_rate_breaker | 針對頁面瀏覽型請求，每 IP 5 分鐘 100 次閾值，超限發送 CAPTCHA（精準防禦人工頁面爬取） |
| 14 | scanners_and_probes | 自訂（Or: IPSet v4 + v6） | **Block** | scanners_and_probes | 封鎖已知掃描器和探測工具 IP |
| 15 | http_flood | 自訂（Or: IPSet v4 + v6） | **Block** | http_flood | 封鎖已偵測到進行 HTTP 洪水攻擊的 IP |
| 16 | bad_bot | 自訂（Or: IPSet v4 + v6） | **Block** | bad_bot | 封鎖已知惡意 Bot IP |
| 17 | anonymous-ip-captcha | 自訂（LabelMatch） | **Captcha** | nabi-waf-anonymous-ip-captcha | 對匿名 IP 發送 CAPTCHA |
| 18 | request-rate-breaker | 自訂（RateBased 700/5min） | **Count** | nabi-waf-request-rate-breaker | 全域頻率計數 |
| 19 | request-rate-captcha | 自訂（RateBased 700/5min） | **Captcha** | nabi-waf-request-rate-captcha | 全域頻率超限發送 CAPTCHA |
| 20 | geo-rate-limit-captcha | 自訂（RateBased 700/5min，非 TW 地區，按 Country 聚合） | **Count** | nabi-waf-geo-rate-limit-captcha | 針對非台灣來源，依國家代碼聚合頻率計數（供監控海外異常流量） |
| 21 | geo-rate-limit-minor-countries | 自訂（RateBased 100/5min，非 TW/CN/SG/US/HK/JP） | **Challenge** | nabi-waf-geo-rate-limit-minor-countries | 對低流量國家（排除主要市場）超過 100 次/5min 的 IP 發送 JavaScript Challenge |
| 22 | malicious-user-agent | 自訂（And: RegexUA + GeoMatch CN） | **Block** | nabi-waf-malicious-user-agent | 封鎖來自中國且 User-Agent 符合惡意特徵的請求 |
| 23 | malicious-user-agent-no-area | 自訂（RegexPatternSet on User-Agent） | **Block** | nabi-waf-malicious-user-agent-no-area | 封鎖所有地區中 User-Agent 符合惡意特徵的請求（不限地理位置） |
| 24 | manufacturer-ip-limitation | 自訂（IPSet） | **Allow** | nabi-waf-manufacturer-ip-limitation | 廠商/合作夥伴 IP 白名單（允許特定 B2B 合作方存取） |
| - | **Default Action** | - | **Block** | - | 未符合任何規則的請求一律封鎖 |

---

## 三、AWS 受管規則集說明（正體中文）

| 規則集名稱 | 主要功能 | 使用此規則集的 ACL |
|---|---|---|
| **AWSManagedRulesCommonRuleSet** | AWS 核心防護規則集，涵蓋 OWASP Top 10 常見漏洞：XSS（跨站腳本）、LFI（本地檔案引入）、RFI（遠端檔案引入）、RCE（遠端程式碼執行）、HTTP 協定濫用等。各 ACL 針對業務需求將部分規則調整為 Count 模式以避免誤擋。 | 全部 ACL |
| **AWSManagedRulesAmazonIpReputationList** | Amazon 威脅情報 IP 聲譽清單，根據 Amazon 自身龐大的雲端基礎設施所收集的威脅情報，持續更新並封鎖已知惡意 IP、Bot 操控 IP、DDoS 攻擊來源、掃描器、TOR 出口節點等。本週在多個 ACL 上表現為最有效的防護規則。 | 全部 ACL |
| **AWSManagedRulesSQLiRuleSet** | SQL 注入攻擊專用防護規則集，全面檢查 HTTP 請求的 URI Path、Query String、Headers、Body 等各欄位，偵測並封鎖各種 SQL 注入攻擊手法（含 Union、Blind、Time-based 等）。 | 全部 ACL |
| **AWSManagedRulesKnownBadInputsRuleSet** | 已知惡意輸入防護，針對近年重大漏洞的攻擊特徵：Log4Shell（CVE-2021-44228）JNDI 注入、SSRF（伺服器端請求偽造）、Spring Cloud RCE（CVE-2022-22963）、以及其他已知零日漏洞利用模式。 | 全部 ACL |
| **AWSManagedRulesAnonymousIpList** | 匿名 IP 清單，識別透過 VPN 服務、TOR 網路、匿名代理伺服器、託管 IP 提供商的請求來源。這些 IP 常用於隱藏真實攻擊來源。各 ACL 設定為 Count 模式（僅標記），再透過後續規則（anonymous-ip-captcha）對標記的流量進行 CAPTCHA 驗證。 | 全部 ACL |
| **AWSManagedRulesBotControlRuleSet** | Bot 流量管理規則集，能識別並分類各種自動化機器人：搜尋引擎爬蟲（Google、Bing）、監控 Bot、廣告 Bot、惡意 Bot 等。blog-acl-waf、go-acl-waf、intro-acl-waf 使用 Count 模式（僅標記 Bot 類別），配合後端請求頭（X-WAF-IS-BOT）進行精細化流量分析。 | blog、go、intro |
| **AWSManagedRulesWordPressRuleSet** | WordPress 應用程式專用保護規則集，防禦 WordPress 特有攻擊向量：未授權 wp-admin 存取、wp-login.php 暴力破解、XML-RPC 濫用、WordPress REST API 使用者列舉（/wp-json/wp/v2/users）、惡意插件/主題漏洞利用等。 | blog、go |
| **AWSManagedRulesPHPRuleSet** | PHP 應用程式專用防護，針對 PHP 語言特有的安全風險：遠端檔案引入（RFI）、本地檔案引入（LFI）、PHP 型別混淆、PHP 特殊字元注入（serialize/unserialize 攻擊）、PHP 配置注入等。 | blog、go |

---

## 四、費用預估

### 本次盤查產生的 AWS API 費用

| 服務 | 操作 | 呼叫次數 | 單價 | 小計（估算） |
|---|---|---:|---|---|
| AWS WAFv2 | `ListWebACLs`（3 次） | 3 | 免費（WAF 標準 API） | $0.00 |
| AWS WAFv2 | `GetWebACL`（10 個 ACL） | 10 | 免費（WAF 標準 API） | $0.00 |
| Amazon CloudWatch | `GetMetricStatistics`（Rule=ALL 日維度，10 ACL） | 10 | $0.01 / 1,000 requests | $0.0001 |
| Amazon CloudWatch | `GetMetricStatistics`（規則層級，批次 1：20 次） | 20 | $0.01 / 1,000 requests | $0.0002 |
| Amazon CloudWatch | `GetMetricStatistics`（規則層級，批次 2：20 次） | 20 | $0.01 / 1,000 requests | $0.0002 |
| **合計** | | **63** | | **< $0.01** |

> **說明**：
> - AWS WAFv2 的 ListWebACLs / GetWebACL 等 API 操作費用計入 WAF 月費（Web ACL 費用 $5/月/ACL，規則費用 $1/月/Rule），讀取操作本身不另計費。
> - Amazon CloudWatch GetMetricStatistics API：前 1,000,000 次請求/月免費（Free Tier）。本次 63 次呼叫完全在免費額度內，實際費用為 $0。
> - **本次盤查產生的額外 AWS 費用：$0.00（完全在 Free Tier 範圍內）**

---

## 五、資料來源與查詢說明

### 查詢環境

| 項目 | 值 |
|---|---|
| AWS Profile | 104awsdev14-ro |
| WAF Scope | CLOUDFRONT |
| CloudWatch Namespace | AWS/WAFV2 |
| CloudWatch Region | us-east-1 |
| 分析週期（Taipei） | 2026-05-25 00:00 ～ 2026-05-31 23:59 |
| UTC 查詢區間 | 2026-05-24T16:00:00Z ～ 2026-06-01T15:59:59Z |
| 日維度 Period | 86,400 秒（1 天） |
| 週維度 Period | 604,800 秒（7 天） |

### CloudWatch 查詢 Dimension

| Dimension 組合 | 用途 |
|---|---|
| `WebACL={acl_name}` + `Rule=ALL` + Period=86400 | 取得每日封鎖次數，計算週總量及日趨勢 |
| `WebACL={acl_name}` + `Rule={MetricName}` + Period=604800 | 取得各規則週封鎖總量，進行規則層級分析 |

### 資料限制

1. **CloudWatch 為彙總統計**：無法取得個別 IP、URI、User-Agent 等詳細資訊，需透過 WAF Sampled Requests 或 WAF Logging（Kinesis/S3）取得詳細記錄。
2. **Default Action Block**：未命中任何規則（Allow/Block/Captcha）的請求由 Default Action Block 處理，記入 Rule=ALL 但不記入任何具名規則 Metric，造成各規則加總與 ALL 有差距。
3. **Captcha / Challenge Action**：Captcha 和 Challenge 動作計入 `CaptchaRequests` / `ChallengeRequests` 指標，不計入 `BlockedRequests`，因此本報告的封鎖次數不含 CAPTCHA/Challenge 事件。
4. **CloudFront WAF**：CloudFront 關聯的 WAF 指標發布至 us-east-1 的 CloudWatch，Dimension 不含 `Region=CloudFront`。
5. **giver-acl-waf 使用 ExcludedRules**：managed-core 使用舊版 `ExcludedRules` 語法排除規則，與其他 ACL 使用 `RuleActionOverrides` 效果相同，但舊版語法在未來版本可能被棄用。

### 查詢步驟摘要

1. `aws wafv2 list-web-acls --scope CLOUDFRONT`：取得帳號下所有 CloudFront Web ACL 清單（含分頁）
2. `aws wafv2 list-web-acls --scope REGIONAL`（us-east-1、ap-northeast-1）：確認無 Regional ACL
3. `aws wafv2 get-web-acl`（× 10）：取得各 ACL 完整規則設定
4. `aws cloudwatch get-metric-statistics`（BlockedRequests Rule=ALL，Period=86400，× 10）：取得每日封鎖次數
5. `aws cloudwatch get-metric-statistics`（BlockedRequests Rule={MetricName}，Period=604800，× 40）：取得各規則週封鎖次數

---

*報告產生時間：2026-06-04 Taipei Time*  
*資料來源：AWS CloudWatch Metrics（Namespace: AWS/WAFV2）via AWS CLI（Profile: 104awsdev14-ro）*
