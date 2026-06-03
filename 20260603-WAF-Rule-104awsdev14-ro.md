# AWS WAF Web ACL 規則完整報告

> **產生日期**：2026-06-03  
> **帳號 ID**：518375879370  
> **AWS Profile**：104awsdev14-ro  
> **範圍**：CLOUDFRONT (us-east-1 / global)  
> **Web ACL 總數**：10

---

## AWS 管理規則集說明

| 規則集名稱 | 正體中文說明 |
|-----------|------------|
| **AWSManagedRulesCommonRuleSet** | **通用核心規則集（CRS）**：防禦多種常見 Web 攻擊，涵蓋跨站腳本（XSS）、路徑穿越（Path Traversal）、遠端檔案包含（RFI）、無 User-Agent 請求、請求體 / Cookie / 查詢字串大小限制等。適用於各類 Web 應用程式基礎防護。 |
| **AWSManagedRulesAmazonIpReputationList** | **Amazon IP 聲譽清單**：根據 Amazon 內部威脅情報，自動封鎖已知與惡意活動關聯的 IP 位址，包含殭屍網路控制節點、掃描器、DDoS 攻擊來源等。 |
| **AWSManagedRulesSQLiRuleSet** | **SQL 注入防護規則集**：偵測並封鎖包含 SQL 注入語法（如 `' OR 1=1`、`UNION SELECT`）的請求，防止攻擊者透過 SQL 注入存取或竄改後端資料庫。 |
| **AWSManagedRulesKnownBadInputsRuleSet** | **已知惡意輸入規則集**：封鎖含有已知惡意輸入特徵的請求，包含 Log4Shell（CVE-2021-44228）漏洞利用嘗試、SSRF（伺服器端請求偽造）攻擊、JavaDeserializationRCE 等高危漏洞探測。 |
| **AWSManagedRulesAnonymousIpList** | **匿名 IP 清單規則集**：偵測並標記使用匿名服務（VPN、Tor 出口節點、公開代理伺服器、主機代管 IP 段）隱藏真實來源 IP 的請求，常搭配 CAPTCHA 挑戰使用。 |
| **AWSManagedRulesBotControlRuleSet** | **Bot 控制規則集**：使用機器學習與傳統規則偵測自動化流量（爬蟲、掃描器、惡意 Bot），可區分已驗證的友好 Bot（如 Googlebot 等搜尋引擎）並給予豁免標籤。 |
| **AWSManagedRulesWordPressRuleSet** | **WordPress 防護規則集**：針對 WordPress 常見攻擊面（`wp-admin`、`wp-login.php`、`xmlrpc.php`、`wp-json` API）提供額外防護，封鎖針對 WordPress 特有路徑的注入和探測行為。 |
| **AWSManagedRulesPHPRuleSet** | **PHP 應用程式防護規則集**：防禦針對 PHP 語言特性的攻擊，包含 PHP 函數注入（`eval()`、`system()` 等）、PHP 命令注入、PHP 物件注入等。 |

---

## 目錄

1. [beagiver-acl-waf](#1-beagiver-acl-waf)
2. [beagiver-event-acl-waf](#2-beagiver-event-acl-waf)
3. [blog-acl-waf](#3-blog-acl-waf)
4. [clinic-acl-waf](#4-clinic-acl-waf)
5. [giver-acl-waf](#5-giver-acl-waf)
6. [go-acl-waf](#6-go-acl-waf)
7. [internal-acl-waf](#7-internal-acl-waf)
8. [intro-acl-waf](#8-intro-acl-waf)
9. [meet-acl-waf](#9-meet-acl-waf)
10. [nabi-acl-waf](#10-nabi-acl-waf)

---

## 1. beagiver-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `3f7edf3e-412d-4225-b24c-68e5f17eb507` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/beagiver-acl-waf/3f7edf3e-412d-4225-b24c-68e5f17eb507` |
| **描述** | beagiver waf acl |
| **預設動作** | **Block**（白名單模式，未命中任何 Allow 規則一律封鎖） |
| **規則數量** | 13 條 |
| **WAF 容量單位** | 1213 WCU |
| **CloudWatch 指標** | `BeagiverWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `vulnerability-scan-whitelist` | IP Set | **Allow** | 弱點掃描白名單 IP Set，允許授權的漏洞掃描工具來源 IP 通過，避免掃描工具被 WAF 攔截 |
| 1 | `bypass_static_files` | Regex Match（URI Path） | **Allow** | URI 路徑符合正則 `^\/errorpages\|static\|2023giverreview\|2024giverreview\/` 時直接放行靜態資源與活動頁面 |
| 2 | `managed-core` | AWS Managed Rule | **None**（保留原動作） | **AWSManagedRulesCommonRuleSet**（通用核心規則集）：防禦 XSS、路徑穿越、RFI 等常見攻擊。以下子規則改為 Count（僅計數不封鎖）：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` |
| 3 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP，包含殭屍網路節點、DDoS 攻擊來源等 |
| 4 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護）：偵測並封鎖 SQL 注入攻擊，防止後端資料庫遭到非法存取或竄改 |
| 5 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入）：封鎖 Log4Shell、SSRF 等高危漏洞利用嘗試 |
| 6 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行，供公司內部人員或服務使用 |
| 7 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：偵測 VPN / Tor / 代理 IP，改為僅計數記錄，不直接封鎖（搭配後續 CAPTCHA 規則處理） |
| 8 | `bot-white-list` | Label Match | **Allow** | 標籤 `awswaf:managed:aws:bot-control:bot:category:email_client` 的 Email 客戶端 Bot 直接放行，避免合法 Email 服務被攔截 |
| 9 | `anonymous-ip-captcha` | Label Match | **Captcha** | 帶有 `awswaf:managed:aws:anonymous-ip-list:AnonymousIPList` 標籤的匿名 IP 請求，要求通過 CAPTCHA 人機驗證 |
| 10 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄，作為速率監控基準 |
| 11 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時，要求通過 CAPTCHA 驗證，防禦自動化高頻攻擊 |
| 12 | `malicious-user-agent` | AND（RegexSet + Geo） | **Block** | User-Agent 標頭符合 `malicious_user_agents` 正則集合，**且**來源國為中國（CN）時，直接封鎖請求 |

---

## 2. beagiver-event-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `1603723b-a9bd-437c-93ac-1b0d86d630b7` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/beagiver-event-acl-waf/1603723b-a9bd-437c-93ac-1b0d86d630b7` |
| **描述** | beagiver event waf acl |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 10 條 |
| **WAF 容量單位** | 1203 WCU |
| **CloudWatch 指標** | `BeagiverEventWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `bypass_static_files` | OR（多條 Regex，URI Path） | **Allow** | URI 路徑符合下列任一條件時直接放行靜態資源：副檔名 `.svg/.jpg/.png/.js/.css/.ico/.gif/.json/.txt/.html`，或路徑前綴 `/css/`、`/img/`、`/index.html` |
| 1 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）：防禦 XSS、RFI 等常見攻擊。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` |
| 2 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 3 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護）：偵測並封鎖 SQL 注入攻擊 |
| 4 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入）：封鎖 Log4Shell、SSRF 等高危漏洞利用嘗試 |
| 5 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 6 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：偵測 VPN / Tor / 代理 IP，僅計數不封鎖 |
| 7 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求通過 CAPTCHA 驗證 |
| 8 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄 |
| 9 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |

---

## 3. blog-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `a2e8855e-360e-4828-b641-1fe7149d0a61` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/blog-acl-waf/a2e8855e-360e-4828-b641-1fe7149d0a61` |
| **描述** | blog waf acl（WordPress 部落格站） |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 17 條 |
| **WAF 容量單位** | 1538 WCU |
| **CloudWatch 指標** | `BlogWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `request-rate-count` | AND（RegexSet NOT + RegexSet） | **Count** | URI 路徑同時符合 `uri-regex-pattern-set` 且不符合 `uri-not-regex-pattern-set` 時計數，用於追蹤特定頁面流量速率 |
| 1 | `bypass_static_files` | OR（多條 Regex，URI Path） | **Allow** | 放行 WordPress 靜態資源：副檔名 `.svg/.jpg/.png/.js/.css/.ico/.gif/.json/.txt`，或路徑 `/errorpages/`、`/wp-content/uploads/`、`/wp-content/human-tools/`、`/wp-includes/js/`、WordPress 主題 / 外掛 assets 與 vendors 資料夾 |
| 2 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` |
| 3 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 4 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 5 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 6 | `managed-wordpress-rules` | AWS Managed Rule | **None** | **AWSManagedRulesWordPressRuleSet**（WordPress 防護規則集）：針對 `wp-admin`、`wp-login.php`、`xmlrpc.php`、`wp-json` 等 WordPress 特有端點提供額外防護 |
| 7 | `managed-php-rules` | AWS Managed Rule | **None** | **AWSManagedRulesPHPRuleSet**（PHP 應用程式防護規則集）：防禦 PHP 函數注入、命令注入、物件注入等 PHP 特有攻擊手法 |
| 8 | `managed-bot-control` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesBotControlRuleSet**（Bot 控制規則集）：偵測並分類 Bot 流量，改為僅計數並附加標籤，不直接封鎖 |
| 9 | `add-is-bot-header` | Label Match | **Count** | 帶有 `awswaf:managed:aws:bot-control:bot:` 前綴標籤的請求計數，用於 Bot 流量監控 |
| 10 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：偵測匿名 IP，僅計數不封鎖 |
| 11 | `add-is-anonymous-header` | Label Match | **Count** | 帶有 `awswaf:managed:aws:anonymous-ip-list:` 前綴標籤的請求計數，用於匿名 IP 流量監控 |
| 12 | `block-admin-from-external-ip` | AND（OR URI Match + NOT IP Set + NOT URI Match） | **Block** | 請求路徑符合 WordPress 管理路徑（`/wp-admin`、`/wp-login.php`、`/wp-json/wp/v2/users`、`/wp-json/oembed/1.0/embed`）且來源 **不在** `internal-ipset` 內，且路徑**不是** `/wp-admin/admin-ajax.php` 時，封鎖請求（防止外部 IP 存取 WordPress 後台） |
| 13 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 14 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 15 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 16 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |

---

## 4. clinic-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `be8118b1-8eb8-4319-be64-ea98669deec8` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/clinic-acl-waf/be8118b1-8eb8-4319-be64-ea98669deec8` |
| **描述** | clinic waf acl |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 14 條 |
| **WAF 容量單位** | 1236 WCU |
| **CloudWatch 指標** | `ClinicWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `vulnerability_scan_whitelist` | IP Set | **Allow** | 弱點掃描白名單 IP Set（`vulnerability_scan_whitelist`），授權掃描工具 IP 直接通過 |
| 1 | `bypass_static_files` | RegexPatternSet（URI Path） | **Allow** | URI 路徑符合靜態資源正則集合時直接放行 |
| 2 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` |
| 3 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 4 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 5 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 6 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 7 | `surveycake-ip-allowlist` | IP Set | **Allow** | SurveyCake 問卷服務 IP（`surveycake-iplist`）直接放行，允許第三方問卷服務回調 |
| 8 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：偵測匿名 IP，僅計數不封鎖 |
| 9 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 10 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 11 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄 |
| 12 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |
| 13 | `malicious-user-agent` | AND（RegexSet UA + Geo CN） | **Block** | User-Agent 符合 `malicious_user_agents` 正則集合，**且**來源國為中國（CN）時，直接封鎖 |

---

## 5. giver-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `ecb66ed9-7d5b-411f-883b-457fc0533d2d` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/giver-acl-waf/ecb66ed9-7d5b-411f-883b-457fc0533d2d` |
| **描述** | giver waf acl |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 14 條 |
| **WAF 容量單位** | 1236 WCU |
| **CloudWatch 指標** | `GiverWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `detected-malicious-ip` | IP Set | **Block** | 動態偵測到的惡意 IP（`detected_malicious_ip_set`）直接封鎖，此清單由自動化威脅偵測機制維護 |
| 1 | `malicious-ip-block` | IP Set | **Block** | 靜態惡意 IP 封鎖清單（`Malicious-ipset`），封鎖已知惡意 IP |
| 10 | `StaticFilePatterns` | RegexPatternSet（URI Path） | **Allow** | URI 路徑符合靜態資源正則集合時直接放行 |
| 20 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）：防禦 XSS、RFI 等常見攻擊（無 Count 覆蓋，完整執行所有子規則） |
| 30 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 40 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 50 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 100 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 110 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：僅計數不封鎖 |
| 120 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 130 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 140 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄 |
| 150 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |
| 160 | `malicious-user-agent` | AND（RegexSet UA + Geo CN） | **Block** | User-Agent 符合 `malicious_user_agents` 正則集合，**且**來源國為中國（CN）時，直接封鎖 |

---

## 6. go-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `26ecd99f-ce28-4aee-8727-d167b69e5995` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/go-acl-waf/26ecd99f-ce28-4aee-8727-d167b69e5995` |
| **描述** | go waf acl（WordPress 部落格站） |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 17 條 |
| **WAF 容量單位** | 1483 WCU |
| **CloudWatch 指標** | `GoWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `bypass_static_files` | OR（多條 Regex，URI Path） | **Allow** | 放行 WordPress 靜態資源：副檔名 `.svg/.jpg/.png/.js/.css/.ico/.gif/.json/.txt`，或路徑 `/errorpages/`、`/wp-content/uploads/`、`/wp-includes/js/`、WordPress 主題 / 外掛 assets 與 vendors 資料夾 |
| 1 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` |
| 2 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 3 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 4 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 5 | `managed-wordpress-rules` | AWS Managed Rule | **None** | **AWSManagedRulesWordPressRuleSet**（WordPress 防護規則集）：保護 `wp-admin`、`wp-login.php`、`xmlrpc.php` 等 WordPress 管理端點 |
| 6 | `managed-php-rules` | AWS Managed Rule | **None** | **AWSManagedRulesPHPRuleSet**（PHP 防護規則集）：防禦 PHP 函數注入、命令注入等攻擊 |
| 7 | `managed-bot-control` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesBotControlRuleSet**（Bot 控制規則集）：偵測並分類 Bot 流量，僅計數並附加標籤 |
| 8 | `add-is-bot-header` | Label Match | **Count** | Bot 標籤請求計數，用於流量監控 |
| 9 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：偵測匿名 IP，僅計數不封鎖 |
| 10 | `add-is-anonymous-header` | Label Match | **Count** | 匿名 IP 標籤請求計數，用於流量監控 |
| 11 | `block-admin-from-external-ip` | AND（OR URI Match + NOT IP Set + NOT URI Match） | **Block** | 請求路徑符合 WordPress 管理路徑（`/wp-admin`、`/wp-login.php`、`/wp-json/wp/v2/users`）且來源 **不在** `internal-ipset` 內，且路徑**不是** `/wp-admin/admin-ajax.php` 時，封鎖請求 |
| 12 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 13 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 14 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 15 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 **300** 次時計數記錄（閾值低於其他 ACL，對 blog 流量更嚴格） |
| 16 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |

---

## 7. internal-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `4833f500-eb14-4fd3-a877-1f45f135dc81` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/internal-acl-waf/4833f500-eb14-4fd3-a877-1f45f135dc81` |
| **描述** | internal service acl waf（內部服務專用） |
| **預設動作** | **Block**（白名單模式，僅允許內部 IP） |
| **規則數量** | 1 條 |
| **WAF 容量單位** | 1 WCU |
| **CloudWatch 指標** | `internal-waf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `internal-ip-limitation` | IP Set | **Allow** | 僅允許 `internal-ipset` 中的內部 IP 存取，所有外部 IP 均被預設動作 Block 封鎖。此 ACL 為最簡潔的內部服務存取控制。 |

---

## 8. intro-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `590fce66-f400-447d-b31f-e1b1a07361ad` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/intro-acl-waf/590fce66-f400-447d-b31f-e1b1a07361ad` |
| **描述** | intro waf acl |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 13 條 |
| **WAF 容量單位** | 1273 WCU |
| **CloudWatch 指標** | `IntroWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `bypass_static_files` | OR（多條 Regex，URI Path） | **Allow** | 放行靜態資源：副檔名 `.js/.css`、路徑 `/errorpages/`，以及各產品 intro 靜態頁面路徑（`intro_beagiver__`、`intro_blog__`、`intro_meet__`、`intro_talentmarket__`、`intro_lms__`、`intro_salary__`、`intro_personalbrand__`、`intro_senior__`） |
| 1 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` |
| 2 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 3 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 4 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 5 | `managed-bot-control` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesBotControlRuleSet**（Bot 控制規則集）：偵測並分類 Bot 流量，僅計數並附加標籤 |
| 6 | `add-is-bot-header` | Label Match | **Count** | Bot 標籤請求計數，用於流量監控 |
| 7 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：僅計數不封鎖 |
| 8 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 9 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 10 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 11 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 **300** 次時計數記錄 |
| 12 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |

---

## 9. meet-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `6ee5a35e-b00c-4cf0-92a1-8e93976972f4` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/meet-acl-waf/6ee5a35e-b00c-4cf0-92a1-8e93976972f4` |
| **描述** | meet waf acl |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 11 條 |
| **WAF 容量單位** | 1198 WCU |
| **CloudWatch 指標** | `MeetWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `bypass_static_files` | OR（多條 Regex，URI Path） | **Allow** | 放行靜態資源：副檔名 `.svg/.jpg/.png/.js/.css/.ico/.gif/.json/.txt`，或路徑 `/errorpages\|static/` |
| 1 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` |
| 2 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 3 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 4 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 5 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 6 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：僅計數不封鎖 |
| 7 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 8 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 9 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄 |
| 10 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |

---

## 10. nabi-acl-waf

| 項目 | 值 |
|------|-----|
| **ID** | `3bf577f1-e6b2-4e61-8c6c-f2e822ff7016` |
| **ARN** | `arn:aws:wafv2:us-east-1:518375879370:global/webacl/nabi-acl-waf/3bf577f1-e6b2-4e61-8c6c-f2e822ff7016` |
| **描述** | nabi waf acl（規則最複雜，具備路由分類標記與地理速率限制） |
| **預設動作** | **Block**（白名單模式） |
| **規則數量** | 25 條 |
| **WAF 容量單位** | 1339 WCU |
| **CloudWatch 指標** | `NabiWaf` |

### 規則清單

| 優先順序 | 規則名稱 | 類型 | 動作 | 說明 |
|----------|----------|------|------|------|
| 0 | `detect_page_view` | NOT（OR Regex） | **Count** | 請求路徑**不符合**靜態資源或 API 路徑正則時，計數並插入請求標頭 `X-ROUTE-TYPE: page_view`，同時附加標籤 `page_view`，供後續速率限制規則使用 |
| 1 | `label_static_file_route` | Regex Match（URI Path） | **Count** | URI 路徑符合靜態資源正則（`/errorpages\|static\|robots.txt\|_nuxt\|asset/img...`）時計數，插入 `X-ROUTE-TYPE: static_file_route` 並附加標籤 `static_file_route` |
| 2 | `label_ajax_route` | Regex Match（URI Path） | **Count** | URI 路徑符合 API 路徑正則（`/(api\|v2)/?`）時計數，插入 `X-ROUTE-TYPE: ajax_route` 並附加標籤 `ajax_route` |
| 3 | `bypass_static_files` | Regex Match（URI Path） | **Allow** | URI 路徑符合靜態資源正則時直接放行（與 label_static_file_route 相同正則，但此規則實際允許通過） |
| 4 | `managed-core` | AWS Managed Rule | **None** | **AWSManagedRulesCommonRuleSet**（通用核心規則集）。Count 子規則：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`SizeRestrictions_QUERYSTRING` |
| 5 | `query-string-size-limit` | Size Constraint（QueryString） | **Block** | 查詢字串（Query String）長度超過 **4096 bytes** 時直接封鎖，防禦查詢字串溢位攻擊 |
| 6 | `managed-reputation-ip` | AWS Managed Rule | **None** | **AWSManagedRulesAmazonIpReputationList**（Amazon IP 聲譽清單）：封鎖已知惡意聲譽 IP |
| 7 | `managed-sql-injection` | AWS Managed Rule | **None** | **AWSManagedRulesSQLiRuleSet**（SQL 注入防護） |
| 8 | `managed-known-bad-input` | AWS Managed Rule | **None** | **AWSManagedRulesKnownBadInputsRuleSet**（已知惡意輸入） |
| 9 | `internal-ip-limitation` | IP Set | **Allow** | 內部 IP（`internal-ipset`）直接放行 |
| 10 | `ip_reputation_lists` | OR（IP Set IPv4 + IPv6） | **Block** | 自訂 IP 聲譽黑名單（`reputation_list_set_ipv4` + `reputation_list_set_ipv6`），同時支援 IPv4 與 IPv6，封鎖已知惡意聲譽 IP |
| 11 | `managed-anonymous-ip` | AWS Managed Rule | **Count**（覆蓋） | **AWSManagedRulesAnonymousIpList**（匿名 IP 清單）：僅計數不封鎖 |
| 12 | `bot-white-list` | Label Match | **Allow** | Email 客戶端 Bot 標籤請求直接放行 |
| 13 | `page_view_rate_breaker` | Rate Based（IP + Label Scope Down） | **Captcha** | 帶有 `page_view` 標籤的請求，同一 IP 於 300 秒內超過 **100** 次時要求 CAPTCHA（專門限制頁面瀏覽速率，防止爬蟲抓取） |
| 14 | `scanners_and_probes` | OR（IP Set IPv4 + IPv6） | **Block** | 自訂掃描器與探測 IP 黑名單（`scanners_probes_set_ipv4/v6`），封鎖已知的掃描工具來源 IP |
| 15 | `http_flood` | OR（IP Set IPv4 + IPv6） | **Block** | 自訂 HTTP 洪水攻擊 IP 黑名單（`http_flood_set_ipv4/v6`），封鎖已知的 HTTP Flood 攻擊來源 IP |
| 16 | `bad_bot` | OR（IP Set IPv4 + IPv6） | **Block** | 自訂惡意 Bot IP 黑名單（`bad_bot_set_ipv4/v6`），封鎖已知惡意爬蟲與 Bot 來源 IP |
| 17 | `anonymous-ip-captcha` | Label Match | **Captcha** | 匿名 IP 標籤請求要求 CAPTCHA 驗證 |
| 18 | `request-rate-breaker` | Rate Based（IP） | **Count** | 同一 IP 於 300 秒內請求超過 700 次時計數記錄 |
| 19 | `request-rate-captcha` | Rate Based（IP） | **Captcha** | 同一 IP 於 300 秒內請求超過 700 次時要求 CAPTCHA 驗證 |
| 20 | `geo-rate-limit-captcha` | Rate Based（Custom Keys + Geo Scope Down） | **Count** | 來源國**非台灣（TW）**的請求，依地理標籤分組，每 300 秒超過 700 次時計數（監控非台灣地區流量速率） |
| 21 | `geo-rate-limit-minor-countries` | Rate Based（Custom Keys + Geo Scope Down） | **Challenge** | 來源國**不在白名單**（TW、CN、SG、US、HK、JP）的請求，依地理標籤分組，每 300 秒超過 **100** 次時觸發 JavaScript Challenge 挑戰（防禦小眾地區的高頻攻擊） |
| 22 | `malicious-user-agent` | AND（RegexSet UA + Geo CN） | **Block** | User-Agent 符合 `malicious_user_agents` 正則集合，**且**來源國為中國（CN）時，直接封鎖，並附加標籤 `malicious-user-agent` |
| 23 | `malicious-user-agent-no-area` | RegexPatternSet（User-Agent Header） | **Block** | User-Agent 符合 `malicious_user_agent` 正則集合時**不限地區**直接封鎖，附加標籤 `malicious-user-agent-no-area`（更嚴格的 UA 封鎖，不依賴地理位置限制） |
| 24 | `manufacturer-ip-limitation` | IP Set | **Allow** | 廠商 IP（`manufacturer-ipset`）直接放行，允許特定硬體廠商或合作夥伴 IP 存取 |

---

*報告產生時間：2026-06-03 | Profile: 104awsdev14-ro | Account: 518375879370*
