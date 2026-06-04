# WAF ACL 規則盤查報告

**Profile**: `104awsdev14-ro`  
**Scope**: CLOUDFRONT (us-east-1)  
**分析時間範圍**: 上週一 2026-05-26（台北 00:00）至 上週日 2026-06-01（台北 23:59）  
**UTC 區間**: `2026-05-25T16:00:00Z` ～ `2026-06-01T16:00:00Z`  
**報告產生日期**: 2026-06-04 Taipei Time  

---

## 各 ACL 封鎖統計總覽

| ACL 名稱 | 週一 5/26 | 週二 5/27 | 週三 5/28 | 週四 5/29 | 週五 5/30 | 週六 5/31 | 週日 6/1 | 週總封鎖 | 週總允許 | 封鎖率 |
|---------|---------|---------|---------|---------|---------|---------|---------|--------|--------|------|
| beagiver-acl-waf | 509 | 804 | 913 | 338 | 144 | 139 | 128 | **2,975** | 2,504 | 54.3% |
| beagiver-event-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** | 0 | — |
| blog-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** | 0 | — |
| clinic-acl-waf | 407 | 769 | 213 | 17 | 163 | 272 | 373 | **2,214** | 1,310 | 62.8% |
| giver-acl-waf | 201 | 472 | 397 | 36 | 249 | 143 | 302 | **1,800** | 18,549 | 8.8% |
| go-acl-waf | 139 | 89 | 119 | 124 | 33 | 50 | 99 | **653** | 12,154 | 5.1% |
| internal-acl-waf | 195 | 171 | 27 | 83 | 28 | 294 | 74 | **872** | 602 | 59.2% |
| intro-acl-waf | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **0** | 212 | 0% |
| meet-acl-waf | 176 | 193 | 19 | 20 | 63 | 150 | 21 | **642** | 37,682 | 1.7% |
| nabi-acl-waf | 338 | 668 | 192 | 367 | 452 | 386 | 180 | **2,583** | 26,432 | 8.9% |
| **合計** | **1,965** | **3,166** | **1,879** | **985** | **1,132** | **1,573** | **1,177** | **11,739** | **99,445** | **10.6%** |

> **注意**：`beagiver-event-acl-waf`、`blog-acl-waf`、`intro-acl-waf` 上週無流量記錄（封鎖與允許皆為 0），可能屬於閒置或未連接至 CloudFront Distribution 的 ACL。

---

## AWS 管理規則說明（共用參考）

以下為本報告中出現的 AWS 管理規則群組，提供正體中文說明：

| 管理規則群組名稱 | 正體中文說明 |
|--------------|-----------|
| **AWSManagedRulesCommonRuleSet** (CRS) | 核心規則集，防禦常見 Web 攻擊，涵蓋 OWASP Top 10 漏洞，包含跨站腳本攻擊（XSS）、路徑遍歷、HTTP 通訊協定異常、惡意 Body 大小等。適用於大多數 Web 應用程式的基礎防護層。 |
| **AWSManagedRulesAmazonIpReputationList** | Amazon IP 信譽清單，封鎖 Amazon 內部威脅情報所標記的惡意 IP，包含已知殭屍網路節點、惡意爬蟲、滲透測試工具來源及其他威脅行為者 IP。 |
| **AWSManagedRulesSQLiRuleSet** | SQL 注入防護規則集，偵測並阻擋 SQL 注入攻擊（SQLi），防止攻擊者透過惡意 SQL 語句操控資料庫查詢，威脅資料外洩或資料庫損毀。 |
| **AWSManagedRulesKnownBadInputsRuleSet** | 已知惡意輸入規則集，阻擋已知的漏洞利用特徵，包含 Log4Shell（CVE-2021-44228）、Spring4Shell、SSRF（伺服器端請求偽造）、常見路徑穿越攻擊等已公開的 CVE 漏洞利用模式。 |
| **AWSManagedRulesWordPressRuleSet** | WordPress 特定規則集，防禦 WordPress CMS 相關漏洞，包含 WordPress 登入保護、XML-RPC 濫用防護、WordPress 特有路徑的未授權存取等。 |
| **AWSManagedRulesPHPRuleSet** | PHP 應用程式特定規則集，防禦 PHP 相關攻擊，包含 PHP 物件注入、file_include 路徑注入、PHP 設定覆蓋攻擊等 PHP 應用程式常見漏洞。 |
| **AWSManagedRulesBotControlRuleSet** | Bot 控制規則集，識別並管理自動化 Bot 流量，可辨識常見搜尋引擎爬蟲（允許）、惡意爬蟲、自動化攻擊工具（封鎖），支援 token 型挑戰機制。 |
| **AWSManagedRulesAnonymousIpList** | 匿名 IP 清單，識別使用匿名化服務（VPN 出口節點、Tor 出口節點、公開代理伺服器、開放 DNS 解析器）的請求，協助判斷流量來源的真實性。 |

---

## 1. beagiver-acl-waf

**描述**: beagiver waf acl | **預設 Action**: Block  
**週封鎖**: 2,975 | **週允許**: 2,504 | **封鎖率**: 54.3%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 1,282 | 43.1% | Amazon IP 信譽清單觸發 |
| managed-known-bad-input | 253 | 8.5% | 已知惡意輸入（Log4j 等 CVE） |
| managed-core | 133 | 4.5% | 核心規則集（OWASP Top 10） |
| 其他 / Default Action=Block | 1,307 | 43.9% | 未命中任何 Allow 規則，由預設 Block 動作攔截 |
| **合計** | **2,975** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | vulnerability-scan-whitelist | 自訂 | Allow | beagiver-waf-vulnerability-scan-whitelist | 漏洞掃描白名單，允許授權的弱點掃描工具 IP 存取 |
| 1 | bypass_static_files | 自訂 | Allow | beagiver-waf-bypass-static-files | 靜態資源略過規則，允許 CSS/JS/圖片等靜態檔案請求直接通過 |
| 2 | managed-core | AWS 管理 CRS | OverrideAction=None（繼承管理規則 Block） | beagiver-waf-managed-core | 核心規則集，防禦 OWASP Top 10 等常見 Web 攻擊 |
| 3 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None（繼承管理規則 Block） | beagiver-waf-managed-reputation-ip | Amazon IP 信譽清單，封鎖已知惡意 IP |
| 4 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None（繼承管理規則 Block） | beagiver-waf-managed-sql-injection | SQL 注入防護規則集 |
| 5 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None（繼承管理規則 Block） | beagiver-waf-managed-known-bad-input | 已知漏洞利用特徵（Log4Shell、Spring4Shell 等） |
| 6 | internal-ip-limitation | 自訂 | Allow | beagiver-waf-internal-ip-limitation | 內部 IP 白名單，允許內網或特定 IP 段直接通過 |
| 7 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count（僅計數，不阻擋） | beagiver-waf-managed-anonymous-ip | 匿名 IP 清單，識別 VPN/Tor/代理流量（Count 模式，不阻擋） |
| 8 | bot-white-list | 自訂 | Allow | beagiver-waf-bot-whitelist | 合法 Bot 白名單，允許 Google、Facebook 等已知搜尋引擎爬蟲 |
| 9 | anonymous-ip-captcha | 自訂 | Captcha | beagiver-waf-anonymous-ip-captcha | 對匿名 IP 流量（VPN/代理）要求通過 CAPTCHA 驗證 |
| 10 | request-rate-breaker | 自訂 | Count（計數觸發後由下一規則處理） | beagiver-waf-request-rate-breaker | 速率限制偵測器，統計超出速率的請求（Count 模式） |
| 11 | request-rate-captcha | 自訂 | Captcha | beagiver-waf-request-rate-captcha | 速率過高的請求觸發 CAPTCHA 挑戰 |
| 12 | malicious-user-agent | 自訂 | Block | beagiver-waf-malicious-user-agent | 封鎖已知惡意 User-Agent（如自動攻擊工具、惡意爬蟲特徵字串） |
| — | **Default Action** | — | **Block** | — | 所有未命中上述規則的請求一律封鎖 |

---

## 2. beagiver-event-acl-waf

**描述**: beagiver event waf acl | **預設 Action**: Block  
**週封鎖**: 0 | **週允許**: 0 | **狀態**: 上週無流量記錄（ACL 可能未連接 CloudFront Distribution）

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | bypass_static_files | 自訂 | Allow | beagiver-event-waf-bypass-static-files | 靜態資源略過規則 |
| 1 | managed-core | AWS 管理 CRS | OverrideAction=None | beagiver-event-waf-managed-core | 核心規則集（OWASP Top 10） |
| 2 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | beagiver-event-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 3 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | beagiver-event-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | beagiver-event-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 5 | internal-ip-limitation | 自訂 | Allow | beagiver-event-waf-internal-ip-limitation | 內部 IP 白名單 |
| 6 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | beagiver-event-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 7 | anonymous-ip-captcha | 自訂 | Captcha | beagiver-event-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 8 | request-rate-breaker | 自訂 | Count | beagiver-event-waf-request-rate-breaker | 速率限制偵測 |
| 9 | request-rate-captcha | 自訂 | Captcha | beagiver-event-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 3. blog-acl-waf

**描述**: blog waf acl | **預設 Action**: Block  
**週封鎖**: 0 | **週允許**: 0 | **狀態**: 上週無流量記錄（ACL 可能未連接 CloudFront Distribution）

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | request-rate-count | 自訂 | Count | blog-waf-request-rate-count | 速率統計規則（Count 模式，純計數） |
| 1 | bypass_static_files | 自訂 | Allow | blog-waf-bypass-static-files | 靜態資源略過 |
| 2 | managed-core | AWS 管理 CRS | OverrideAction=None | blog-waf-managed-core | 核心規則集（OWASP Top 10） |
| 3 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | blog-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 4 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | blog-waf-managed-sql-injection | SQL 注入防護 |
| 5 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | blog-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 6 | managed-wordpress-rules | AWS 管理 WordPress 規則集 | OverrideAction=None | blog-waf-managed-wordpress-rules | WordPress 特定漏洞防護 |
| 7 | managed-php-rules | AWS 管理 PHP 規則集 | OverrideAction=None | blog-waf-managed-php-rules | PHP 應用程式漏洞防護 |
| 8 | managed-bot-control | AWS 管理 Bot 控制規則集 | Count（僅偵測不阻擋） | blog-waf-managed-bot-control | Bot 流量識別與管理（Count 模式） |
| 9 | add-is-bot-header | 自訂 | Count + 插入 Header `X-WAF-IS-BOT: true` | blog-waf-add-is-bot-header | 標記 Bot 請求並加入自訂 Header，供後端應用程式辨識 |
| 10 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | blog-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 11 | add-is-anonymous-header | 自訂 | Count + 插入 Header `X-WAF-IS-ANONYMOUS: true` | blog-waf-add-is-anonymous-header | 標記匿名 IP 請求，供後端辨識 |
| 12 | block-admin-from-external-ip | 自訂 | Block | blog-waf-block-admin-from-external-ip | 封鎖來自外部 IP 的後台管理路徑存取（如 /admin、/wp-admin） |
| 13 | internal-ip-limitation | 自訂 | Allow | blog-waf-internal-ip-limitation | 內部 IP 白名單 |
| 14 | bot-white-list | 自訂 | Allow | blog-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 15 | anonymous-ip-captcha | 自訂 | Captcha | blog-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 16 | request-rate-captcha | 自訂 | Captcha | blog-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 4. clinic-acl-waf

**描述**: clinic waf acl | **預設 Action**: Block  
**週封鎖**: 2,214 | **週允許**: 1,310 | **封鎖率**: 62.8%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 430 | 19.4% | Amazon IP 信譽清單觸發 |
| managed-core | 17 | 0.8% | 核心規則集（OWASP Top 10） |
| managed-known-bad-input | 14 | 0.6% | 已知惡意輸入 |
| 其他 / Default Action=Block | 1,753 | 79.2% | 未命中任何 Allow 規則由預設 Block 攔截 |
| **合計** | **2,214** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | vulnerability_scan_whitelist | 自訂 | Allow | vulnerability_scan_whitelist | 漏洞掃描白名單，授權掃描工具 IP 直接通過 |
| 1 | bypass_static_files | 自訂 | Allow | clinic-waf-bypass-static-files | 靜態資源略過 |
| 2 | managed-core | AWS 管理 CRS | OverrideAction=None | clinic-waf-managed-core | 核心規則集（OWASP Top 10） |
| 3 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | clinic-waf-managed-reputation-ip | Amazon IP 信譽清單，封鎖已知惡意 IP |
| 4 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | clinic-waf-managed-sql-injection | SQL 注入防護 |
| 5 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | clinic-waf-managed-known-bad-input | 已知漏洞利用特徵（Log4j 等） |
| 6 | internal-ip-limitation | 自訂 | Allow | clinic-waf-internal-ip-limitation | 內部 IP 白名單 |
| 7 | surveycake-ip-allowlist | 自訂 | Allow | clinic-waf-surveycake-ip-allowlist | SurveyCake 問卷平台 IP 允許清單（特定合作夥伴白名單） |
| 8 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | clinic-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式，不阻擋） |
| 9 | bot-white-list | 自訂 | Allow | clinic-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 10 | anonymous-ip-captcha | 自訂 | Captcha | clinic-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA 驗證 |
| 11 | request-rate-breaker | 自訂 | Count | clinic-waf-request-rate-breaker | 速率限制偵測（Count 模式） |
| 12 | request-rate-captcha | 自訂 | Captcha | clinic-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| 13 | malicious-user-agent | 自訂 | Block | clinic-waf-malicious-user-agent | 封鎖已知惡意 User-Agent |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 5. giver-acl-waf

**描述**: giver waf acl | **預設 Action**: Block  
**週封鎖**: 1,800 | **週允許**: 18,549 | **封鎖率**: 8.8%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 288 | 16.0% | Amazon IP 信譽清單觸發 |
| managed-known-bad-input | 15 | 0.8% | 已知惡意輸入 |
| managed-core | 8 | 0.4% | 核心規則集 |
| 其他 / Default Action=Block | 1,489 | 82.7% | 未命中 Allow 規則由預設 Block 攔截 |
| **合計** | **1,800** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | detected-malicious-ip | 自訂 | Block | giver-waf-detected-malicious-ip | 偵測並封鎖特定已知惡意 IP（自訂 IP 黑名單） |
| 1 | malicious-ip-block | 自訂 | Block | giver-waf-malicious-ip-block | 另一組惡意 IP 封鎖規則（自訂 IP Set） |
| 10 | StaticFilePatterns | 自訂 | Allow | giver-waf-static-file-patterns | 靜態資源類型略過（依 URL 模式比對） |
| 20 | managed-core | AWS 管理 CRS | OverrideAction=None | giver-waf-managed-core | 核心規則集（OWASP Top 10） |
| 30 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | giver-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 40 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | giver-waf-managed-sql-injection | SQL 注入防護 |
| 50 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | giver-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 100 | internal-ip-limitation | 自訂 | Allow | giver-waf-internal-ip-limitation | 內部 IP 白名單 |
| 110 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | giver-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 120 | bot-white-list | 自訂 | Allow | giver-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 130 | anonymous-ip-captcha | 自訂 | Captcha | giver-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 140 | request-rate-breaker | 自訂 | Count | giver-waf-request-rate-breaker | 速率限制偵測 |
| 150 | request-rate-captcha | 自訂 | Captcha | giver-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| 160 | malicious-user-agent | 自訂 | Block | giver-waf-malicious-user-agent | 封鎖惡意 User-Agent |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 6. go-acl-waf

**描述**: go waf acl | **預設 Action**: Block  
**週封鎖**: 653 | **週允許**: 12,154 | **封鎖率**: 5.1%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 15 | 2.3% | Amazon IP 信譽清單觸發 |
| managed-known-bad-input | 7 | 1.1% | 已知惡意輸入 |
| 其他 / Default Action=Block | 631 | 96.6% | 未命中 Allow 規則由預設 Block 攔截 |
| **合計** | **653** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | bypass_static_files | 自訂 | Allow | go-waf-bypass-static-files | 靜態資源略過 |
| 1 | managed-core | AWS 管理 CRS | OverrideAction=None | go-waf-managed-core | 核心規則集（OWASP Top 10） |
| 2 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | go-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 3 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | go-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | go-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 5 | managed-wordpress-rules | AWS 管理 WordPress 規則集 | OverrideAction=None | go-waf-managed-wordpress-rules | WordPress 特定漏洞防護 |
| 6 | managed-php-rules | AWS 管理 PHP 規則集 | OverrideAction=None | go-waf-managed-php-rules | PHP 應用程式漏洞防護 |
| 7 | managed-bot-control | AWS 管理 Bot 控制規則集 | Count | go-waf-managed-bot-control | Bot 流量識別（Count 模式） |
| 8 | add-is-bot-header | 自訂 | Count + `X-WAF-IS-BOT: true` | go-waf-add-is-bot-header | 標記 Bot 請求加入 Header |
| 9 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | go-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 10 | add-is-anonymous-header | 自訂 | Count + `X-WAF-IS-ANONYMOUS: true` | go-waf-add-is-anonymous-header | 標記匿名 IP 請求加入 Header |
| 11 | block-admin-from-external-ip | 自訂 | Block | go-waf-block-admin-from-external-ip | 封鎖外部 IP 對後台管理路徑的存取 |
| 12 | internal-ip-limitation | 自訂 | Allow | go-waf-internal-ip-limitation | 內部 IP 白名單 |
| 13 | bot-white-list | 自訂 | Allow | go-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 14 | anonymous-ip-captcha | 自訂 | Captcha | go-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 15 | request-rate-breaker | 自訂 | Count | go-waf-request-rate-breaker | 速率限制偵測 |
| 16 | request-rate-captcha | 自訂 | Captcha | go-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 7. internal-acl-waf

**描述**: internal service acl waf | **預設 Action**: Block  
**週封鎖**: 872 | **週允許**: 602 | **封鎖率**: 59.2%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| Default Action=Block | 872 | 100% | 僅有一條 Allow 規則（內部 IP），所有非內部 IP 流量皆由預設 Block 攔截 |
| **合計** | **872** | 100% | |

> **注意**：`internal-acl-waf` 設計為最嚴格的白名單模式—僅允許特定內部 IP 通過，其餘所有流量一律封鎖。高封鎖率（59.2%）為預期行為，代表有大量非授權 IP 嘗試存取內部服務。

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | internal-ip-limitation | 自訂 | Allow | internal-ip-limitation | 僅允許指定的內部 IP 段（如 VPN、辦公室 IP）存取服務 |
| — | **Default Action** | — | **Block** | — | 所有非內部 IP 流量一律封鎖 |

---

## 8. intro-acl-waf

**描述**: intro waf acl | **預設 Action**: Block  
**週封鎖**: 0 | **週允許**: 212 | **狀態**: 上週有少量允許流量但無封鎖記錄

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | bypass_static_files | 自訂 | Allow | intro-waf-bypass-static-files | 靜態資源略過 |
| 1 | managed-core | AWS 管理 CRS | OverrideAction=None | intro-waf-managed-core | 核心規則集（OWASP Top 10） |
| 2 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | intro-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 3 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | intro-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | intro-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 5 | managed-bot-control | AWS 管理 Bot 控制規則集 | Count | intro-waf-managed-bot-control | Bot 流量識別（Count 模式） |
| 6 | add-is-bot-header | 自訂 | Count + `X-WAF-IS-BOT: true` | intro-waf-add-is-bot-header | 標記 Bot 請求加入 Header |
| 7 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | intro-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 8 | internal-ip-limitation | 自訂 | Allow | intro-waf-internal-ip-limitation | 內部 IP 白名單 |
| 9 | bot-white-list | 自訂 | Allow | intro-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 10 | anonymous-ip-captcha | 自訂 | Captcha | intro-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 11 | request-rate-breaker | 自訂 | Count | intro-waf-request-rate-breaker | 速率限制偵測 |
| 12 | request-rate-captcha | 自訂 | Captcha | intro-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 9. meet-acl-waf

**描述**: meet waf acl | **預設 Action**: Block  
**週封鎖**: 642 | **週允許**: 37,682 | **封鎖率**: 1.7%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 191 | 29.8% | Amazon IP 信譽清單觸發 |
| managed-known-bad-input | 13 | 2.0% | 已知惡意輸入 |
| managed-core | 4 | 0.6% | 核心規則集 |
| 其他 / Default Action=Block | 434 | 67.6% | 未命中 Allow 規則由預設 Block 攔截 |
| **合計** | **642** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | bypass_static_files | 自訂 | Allow | meet-waf-bypass-static-files | 靜態資源略過 |
| 1 | managed-core | AWS 管理 CRS | OverrideAction=None | meet-waf-managed-core | 核心規則集（OWASP Top 10） |
| 2 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | meet-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 3 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | meet-waf-managed-sql-injection | SQL 注入防護 |
| 4 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | meet-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 5 | internal-ip-limitation | 自訂 | Allow | meet-waf-internal-ip-limitation | 內部 IP 白名單 |
| 6 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | meet-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 7 | anonymous-ip-captcha | 自訂 | Captcha | meet-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 8 | bot-white-list | 自訂 | Allow | meet-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 9 | request-rate-breaker | 自訂 | Count | meet-waf-request-rate-breaker | 速率限制偵測 |
| 10 | request-rate-captcha | 自訂 | Captcha | meet-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 10. nabi-acl-waf

**描述**: nabi waf acl | **預設 Action**: Block  
**週封鎖**: 2,583 | **週允許**: 26,432 | **封鎖率**: 8.9%

### 規則封鎖統計（上週）

| 規則名稱 | 封鎖次數 | 佔比 | 備註 |
|---------|--------|------|------|
| managed-reputation-ip | 436 | 16.9% | Amazon IP 信譽清單觸發 |
| ip_reputation_lists | 55 | 2.1% | 自訂 IP 信譽黑名單 |
| managed-known-bad-input | 36 | 1.4% | 已知漏洞利用特徵 |
| managed-core | 12 | 0.5% | 核心規則集 |
| 其他 / Default Action=Block | 2,044 | 79.1% | 未命中 Allow 規則由預設 Block 攔截 |
| **合計** | **2,583** | 100% | |

### 規則完整清單

| Priority | 規則名稱 | 類型 | Action | Metric Name | 正體中文說明 |
|----------|---------|------|--------|-------------|-----------|
| 0 | detect_page_view | 自訂 | Count + 插入 Header `X-ROUTE-TYPE: page_view` | detect_page_view | 識別頁面瀏覽類請求並標記 Header，供後端分析流量類型 |
| 1 | label_static_file_route | 自訂 | Count + 插入 Header `X-ROUTE-TYPE: static_file_route` | label_static_file_route | 識別靜態檔案請求並標記 Header |
| 2 | label_ajax_route | 自訂 | Count + 插入 Header `X-ROUTE-TYPE: ajax_route` | label_ajax_route | 識別 AJAX API 請求並標記 Header |
| 3 | bypass_static_files | 自訂 | Allow | bypass_static_files | 靜態資源略過（Allow）|
| 4 | managed-core | AWS 管理 CRS | OverrideAction=None | nabi-waf-managed-core | 核心規則集（OWASP Top 10） |
| 5 | query-string-size-limit | 自訂 | Block | nabi-waf-query-string-size-limit | 封鎖 Query String 超過大小限制的請求（防禦 Buffer Overflow 或異常請求） |
| 6 | managed-reputation-ip | AWS 管理 IP 信譽清單 | OverrideAction=None | nabi-waf-managed-reputation-ip | Amazon IP 信譽清單 |
| 7 | managed-sql-injection | AWS 管理 SQLi 規則集 | OverrideAction=None | nabi-waf-managed-sql-injection | SQL 注入防護 |
| 8 | managed-known-bad-input | AWS 管理已知惡意輸入規則集 | OverrideAction=None | nabi-waf-managed-known-bad-input | 已知漏洞利用特徵 |
| 9 | internal-ip-limitation | 自訂 | Allow | nabi-waf-internal-ip-limitation | 內部 IP 白名單 |
| 10 | ip_reputation_lists | 自訂 | Block | ip_reputation_lists | 自訂 IP 信譽黑名單清單（封鎖特定已知惡意 IP Set） |
| 11 | managed-anonymous-ip | AWS 管理匿名 IP 清單 | Count | nabi-waf-managed-anonymous-ip | 匿名 IP 識別（Count 模式） |
| 12 | bot-white-list | 自訂 | Allow | nabi-waf-bot-whitelist | 合法搜尋引擎 Bot 白名單 |
| 13 | page_view_rate_breaker | 自訂 | Captcha | page_view_rate_breaker | 頁面瀏覽速率過高時觸發 CAPTCHA（防禦自動化頁面爬取） |
| 14 | scanners_and_probes | 自訂 | Block | scanners_and_probes | 封鎖掃描器與探測行為（偵測特定掃描工具的請求特徵） |
| 15 | http_flood | 自訂 | Block | http_flood | 封鎖 HTTP Flood DDoS 攻擊（速率型封鎖規則） |
| 16 | bad_bot | 自訂 | Block | bad_bot | 封鎖已知惡意 Bot 特徵（User-Agent/行為模式） |
| 17 | anonymous-ip-captcha | 自訂 | Captcha | nabi-waf-anonymous-ip-captcha | 匿名 IP 觸發 CAPTCHA |
| 18 | request-rate-breaker | 自訂 | Count | nabi-waf-request-rate-breaker | 速率限制偵測 |
| 19 | request-rate-captcha | 自訂 | Captcha | nabi-waf-request-rate-captcha | 速率過高觸發 CAPTCHA |
| 20 | geo-rate-limit-captcha | 自訂 | Count | nabi-waf-geo-rate-limit-captcha | 地理位置速率限制偵測（針對特定地區的速率計數） |
| 21 | geo-rate-limit-minor-countries | 自訂 | Challenge | nabi-waf-geo-rate-limit-minor-countries | 對次要地區（流量佔比低的國家）高速率請求觸發 JavaScript Challenge |
| 22 | malicious-user-agent | 自訂 | Block | nabi-waf-malicious-user-agent | 封鎖惡意 User-Agent（一般類別） |
| 23 | malicious-user-agent-no-area | 自訂 | Block | nabi-waf-malicious-user-agent-no-area | 封鎖惡意 User-Agent（無地區限制的廣泛封鎖規則） |
| 24 | manufacturer-ip-limitation | 自訂 | Allow | nabi-waf-manufacturer-ip-limitation | 製造商/合作夥伴 IP 允許清單 |
| — | **Default Action** | — | **Block** | — | 未命中規則一律封鎖 |

---

## 整體分析

### 封鎖趨勢（上週每日）

```
ACL               週一   週二   週三   週四   週五   週六   週日
beagiver         [ 509] [ 804] [ 913] [ 338] [ 144] [ 139] [ 128]
clinic           [ 407] [ 769] [ 213] [  17] [ 163] [ 272] [ 373]
nabi             [ 338] [ 668] [ 192] [ 367] [ 452] [ 386] [ 180]
giver            [ 201] [ 472] [ 397] [  36] [ 249] [ 143] [ 302]
internal         [ 195] [ 171] [  27] [  83] [  28] [ 294] [  74]
go               [ 139] [  89] [ 119] [ 124] [  33] [  50] [  99]
meet             [ 176] [ 193] [  19] [  20] [  63] [ 150] [  21]
─────────────────────────────────────────────────────────────────
全ACL合計        [1965] [3166] [1879] [ 985] [1132] [1573] [1177]
```

### 主要發現

1. **週三（5/28）beagiver 封鎖量最高（913 次）**：為全週最大單日封鎖事件，主要由 IP 信譽清單觸發（1,282 次/週）。
2. **週二（5/27）為全帳號封鎖高峰（3,166 次）**：多個 ACL 同步出現較高封鎖量，可能代表週二有大規模掃描或探測活動。
3. **IP 信譽清單（AmazonIpReputationList）為最主要的規則觸發來源**：跨 5 個 ACL 共封鎖 2,222 次（beagiver: 1,282 + clinic: 430 + nabi: 436 + giver: 288 + meet: 191）。
4. **Default Action=Block 佔總封鎖比例高**：多個 ACL 有大量「未命中規則但被預設封鎖」的流量，顯示這些 ACL 採用白名單策略（只允許特定流量），正常且安全。
5. **beagiver-event-acl-waf、blog-acl-waf、intro-acl-waf 上週無封鎖流量**：需確認是否仍連接至 CloudFront Distribution，若未使用應考慮清理（WAF ACL 每月 $5 費用）。

### 建議事項

| 優先級 | 建議 | 說明 |
|-------|------|------|
| 高 | 確認 3 個零流量 ACL 狀態 | `beagiver-event-acl-waf`、`blog-acl-waf`、`intro-acl-waf` 若未使用，可刪除節省費用 |
| 高 | 調查 internal-acl-waf 封鎖來源 | 封鎖率 59.2%（872/1,474）且無 WAF 管理規則，建議確認哪些 IP 被封鎖及是否為誤封 |
| 中 | 啟用 WAF Logging | 目前僅依賴 CloudWatch 彙總指標，啟用 WAF Access Log 可取得 IP、URI、封鎖原因等詳細資料 |
| 中 | 設定 CloudWatch Alarm | 針對 `beagiver-acl-waf` 和 `nabi-acl-waf` 設定封鎖量突增告警（如日均超過 3 倍） |
| 低 | 評估 malicious-user-agent 規則效果 | 多個 ACL 的 malicious-user-agent (Block) 規則週封鎖次數為 0，可確認規則定義是否仍有效 |

---

## AWS API 費用預估

### 本次盤查執行的 API 呼叫

| API 服務 | 操作 | 呼叫次數 | 說明 |
|---------|------|--------|------|
| WAFv2 | ListWebACLs | 2 | 列出所有 CLOUDFRONT ACL（含分頁） |
| WAFv2 | GetWebACL | 10 | 取得每個 ACL 的完整規則清單 |
| CloudWatch | GetMetricStatistics | ~60 | 查詢封鎖/允許統計（全部 ACL × 多個維度） |
| **合計** | | **~72** | |

### 費用試算

| 服務 | 定價 | 本次成本 |
|-----|------|---------|
| WAFv2 API（ListWebACLs、GetWebACL） | 控制平面 API 免費 | $0.00 |
| CloudWatch GetMetricStatistics | $0.01 / 1,000 API 請求 | ~$0.0007 |
| **本次盤查 API 費用合計** | | **≈ $0.001** |

### WAF 常態運行月費（參考）

| 項目 | 數量 | 單價 | 月費 |
|-----|------|------|------|
| Web ACL | 10 個 | $5.00 / ACL / 月 | $50.00 |
| 規則（管理規則群組視為 1 條） | 平均每 ACL ~6 條規則，合計 ~60 條 | $1.00 / 規則 / 月 | $60.00 |
| 請求處理費 | 約 111,184 請求 / 週 ≈ 447,000 請求 / 月 | $0.60 / 百萬請求 | $0.27 |
| **WAF 月費合計（估計）** | | | **~$110.27 / 月** |

> 若清除 3 個零流量 ACL（beagiver-event、blog、intro）可節省 WAF ACL 費用：$5 × 3 = **$15/月**，規則費約 $15-20/月，合計可節省約 **$30-35/月**。

---

## 資料來源與查詢說明

### 查詢環境

| 項目 | 值 |
|-----|---|
| AWS Profile | 104awsdev14-ro |
| AWS Account | 518375879370 |
| WAF Scope | CLOUDFRONT |
| CloudWatch Namespace | AWS/WAFV2 |
| Region | us-east-1 |
| 統計週期（Period） | 86400（日）/ 604800（週）|
| 分析 UTC 區間 | 2026-05-25T16:00:00Z ～ 2026-06-01T16:00:00Z |
| 對應台北時間 | 2026-05-26 00:00 ～ 2026-06-01 23:59 |

### 查詢 Dimension 說明

| Dimension 組合 | 用途 |
|--------------|------|
| WebACL={acl_name}, Rule=ALL | 取得該 ACL 的總封鎖/允許請求數 |
| WebACL={acl_name}, Rule={MetricName} | 取得該 ACL 特定規則的封鎖次數 |

### 資料限制

1. **CloudWatch 為彙總統計**：不含個別請求的 IP 位址、URI、User-Agent 等詳細資料，需啟用 WAF Logging 才能取得
2. **DefaultAction 封鎖無獨立 Metric**：被 ACL 預設 Block Action 攔截的請求，包含在 Rule=ALL 的計數中，但沒有獨立的 Dimension 可查詢，本報告以差值估算
3. **Count OverrideAction 規則不計入 BlockedRequests**：設定為 Count 的規則（如 managed-anonymous-ip、managed-bot-control）不會增加 BlockedRequests 計數，僅計入 CountedRequests（本次未查詢）
4. **CloudFront WAF 不含 Region=CloudFront Dimension**：CloudFront WAF 的 CloudWatch 指標不加 Region Dimension

### 查詢步驟摘要

1. `wafv2 list-web-acls`：列出帳號下所有 CLOUDFRONT ACL（2 次呼叫，含分頁）
2. `wafv2 get-web-acl`（×10）：取得每個 ACL 的完整規則定義
3. `cloudwatch get-metric-statistics`（×20）：查詢所有 ACL 的週 BlockedRequests 和 AllowedRequests（Rule=ALL）
4. `cloudwatch get-metric-statistics`（×40）：查詢各 ACL 主要規則的週 BlockedRequests（逐規則）

---

*報告產生時間：2026-06-04 Taipei Time*  
*資料來源：AWS CloudWatch（Namespace: AWS/WAFV2）、AWS WAFv2 API*  
*查詢執行者：Claude Code（claude-sonnet-4-6）with MCP AWS API*
