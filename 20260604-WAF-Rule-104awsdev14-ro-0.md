# WAF 規則完整報告
- **產生日期**：2026-06-04
- **AWS 帳號**：518375879370
- **AWS Profile**：104awsdev14-ro
- **Scope**：CLOUDFRONT（全球）
- **WAF 版本**：AWS WAF v2
- **統計期間**：2026-05-28 ～ 2026-06-04（近一週）

---

## 近一週阻擋總覽（2026-05-28 ～ 2026-06-04）

| Web ACL | 說明 | 近一週總封鎖數 |
|---------|------|:---:|
| nabi-acl-waf | nabi waf acl | **2,052** |
| giver-acl-waf | giver waf acl | **1,791** |
| clinic-acl-waf | clinic waf acl | **1,730** |
| beagiver-acl-waf | beagiver waf acl | **948** |
| internal-acl-waf | internal service acl waf | **778** |
| go-acl-waf | go waf acl | **566** |
| meet-acl-waf | meet waf acl | **212** |
| beagiver-event-acl-waf | beagiver event waf acl | 0 |
| blog-acl-waf | blog waf acl | 0 |
| intro-acl-waf | intro waf acl | 0 |
| **合計** | | **8,077** |

> **說明**：封鎖數包含規則動作封鎖（Block/Captcha/Challenge）及 Default Block 動作（不符合任何 Allow 規則的請求）。

---

## AWS 管理規則說明（正體中文）

| 規則集名稱 | 正體中文說明 |
|---|---|
| **AWSManagedRulesCommonRuleSet** | **通用核心規則集（CRS）**：防護 OWASP 十大常見 Web 攻擊，包含本地/遠端檔案包含（LFI/RFI）、跨站腳本（XSS）、路徑穿越、HTTP 協議違規、請求大小限制及伺服器端請求偽造（SSRF）偵測。部分規則可依需求覆寫為 Count 模式。 |
| **AWSManagedRulesAmazonIpReputationList** | **Amazon IP 信譽清單**：依據 Amazon 威脅情報封鎖已知惡意 IP，涵蓋偵察掃描 IP（`AWSManagedReconnaissanceList`）、Tor 出口節點（`AWSManagedTorExitNode`）及已知 DDoS 攻擊來源 IP（`AWSManagedIPDDoSList`）。 |
| **AWSManagedRulesSQLiRuleSet** | **SQL 注入規則集**：偵測並封鎖 SQL 注入攻擊嘗試，涵蓋 URI 路徑、查詢字串、Request Body 及 Cookie 等所有攻擊向量。 |
| **AWSManagedRulesKnownBadInputsRuleSet** | **已知惡意輸入規則集**：封鎖已知危險請求模式，包含 Log4j RCE 漏洞（`Log4JRCE`）、Java 反序列化 RCE（`JavaDeserializationRCE`）、SSRF（`SSRF_URIPATH`）及惡意巨集利用（`ExploitableMacros_QUERYARGUMENTS`）。 |
| **AWSManagedRulesWordPressRuleSet** | **WordPress 防護規則集**：針對 WordPress 應用程式常見攻擊，包含 XML-RPC 濫用、wp-login.php 暴力破解、WordPress REST API 用戶列舉及外掛/主題特有漏洞防護。 |
| **AWSManagedRulesPHPRuleSet** | **PHP 應用程式防護規則集**：偵測 PHP 程式碼注入、函數注入（`PHPHighRiskMethodsVariables_HEADER`/`_BODY`/`_QUERYARGUMENTS`）及 PHP 配置覆寫嘗試（`PHPObjectInjection`）。 |
| **AWSManagedRulesBotControlRuleSet** | **Bot 控制規則集**：識別並分類自動化流量（爬蟲、腳本工具、瀏覽器自動化等），提供 Label 標記供後續規則使用。本身設為 Count 模式時不阻擋，僅標記。 |
| **AWSManagedRulesAnonymousIpList** | **匿名 IP 清單**：識別使用匿名服務的流量，包含 VPN、代理伺服器、Tor 等匿名 IP（`AnonymousIPList`）及雲端/託管提供商 IP（`HostingProviderIPList`）。常用於觸發後續 CAPTCHA 規則。 |

---

## 1. beagiver-acl-waf

- **ID**：3f7edf3e-412d-4225-b24c-68e5f17eb507
- **描述**：beagiver waf acl
- **預設動作**：🚫 Block（未符合任何規則的請求全部封鎖）
- **CloudWatch Metric**：BeagiverWaf
- **近一週總封鎖**：**948 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | vulnerability-scan-whitelist | IP 白名單 | ✅ Allow | 允許弱點掃描工具 IP，確保安全測試不被阻擋 | — |
| 1 | bypass_static_files | 正規表達式 | ✅ Allow | 允許靜態資源路徑（`/errorpages`、`/static`、`/2023giverreview`、`/2024giverreview`）直接通過 | — |
| 2 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` 覆寫為 Count，其餘子規則仍為 Block | — |
| 3 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP（偵察掃描、Tor 出口節點、DDoS 攻擊來源） | **593** |
| 4 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 5 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護（Log4j RCE、SSRF 等） | **448** |
| 6 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP（internal-ipset）直接通過 | — |
| 7 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數，搭配第 9 條規則觸發 CAPTCHA | — |
| 8 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot（`awswaf:managed:aws:bot-control:bot:category:email_client`） | — |
| 9 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 對命中 `AnonymousIPList` 標籤的匿名 IP 顯示 CAPTCHA 驗證 | 0 |
| 10 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘內同一 IP 超過 700 次請求時計數標記 | — |
| 11 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘內同一 IP 超過 700 次請求時顯示 CAPTCHA | 0 |
| 12 | malicious-user-agent | 複合條件 | 🚫 Block | **惡意 User-Agent + 中國 IP**：符合惡意 UA 正則且來源為中國（CN）的請求一律封鎖 | 0 |

> 📊 **分析**：本週封鎖以惡意 IP 信譽（593次）和已知惡意輸入（448次，Log4j 等攻擊）為主，合計 1,041 次規則觸發。差異部分來自 Default Block 動作。

---

## 2. beagiver-event-acl-waf

- **ID**：1603723b-a9bd-437c-93ac-1b0d86d630b7
- **描述**：beagiver event waf acl
- **預設動作**：🚫 Block
- **CloudWatch Metric**：BeagiverEventWaf
- **近一週總封鎖**：**0 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | bypass_static_files | 正規表達式（OR） | ✅ Allow | 允許靜態檔案副檔名（svg/jpg/png/js/css/ico/gif/json/txt/html）及 `/css/`、`/img/`、`/index.html` 路徑通過 | — |
| 1 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` 覆寫為 Count | — |
| 2 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖已知惡意 IP | 0 |
| 3 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 4 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | 0 |
| 5 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 直接通過 | — |
| 6 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 7 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 對匿名 IP 標籤請求顯示 CAPTCHA 驗證 | 0 |
| 8 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 700 次請求計數 | — |
| 9 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次請求觸發 CAPTCHA | 0 |

---

## 3. blog-acl-waf

- **ID**：a2e8855e-360e-4828-b641-1fe7149d0a61
- **描述**：blog waf acl（WordPress 部落格）
- **預設動作**：🚫 Block
- **CloudWatch Metric**：BlogWaf
- **近一週總封鎖**：**0 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | request-rate-count | 複合速率 | 📊 Count | 對特定 URI 模式做速率計數（排除白名單路徑） | — |
| 1 | bypass_static_files | 正規表達式（OR） | ✅ Allow | 允許靜態檔案及 WordPress 資源路徑（`/wp-content/uploads`、`/wp-includes/js`、主題/外掛 assets/vendors 目錄）通過 | — |
| 2 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` 覆寫為 Count | — |
| 3 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖已知惡意 IP | 0 |
| 4 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 5 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | 0 |
| 6 | managed-wordpress-rules | AWS 管理規則 | 🚫 Block | **AWSManagedRulesWordPressRuleSet**：WordPress 專屬攻擊防護（wp-login 暴力破解、XML-RPC 濫用等） | 0 |
| 7 | managed-php-rules | AWS 管理規則 | 🚫 Block | **AWSManagedRulesPHPRuleSet**：PHP 程式碼/函數注入防護 | 0 |
| 8 | managed-bot-control | AWS 管理規則 | 📊 Count | **AWSManagedRulesBotControlRuleSet**：Bot 流量識別，覆寫為 Count 僅標記 | — |
| 9 | add-is-bot-header | Label 比對 | 📊 Count | Bot 流量插入 `X-WAF-IS-BOT: true` Header | — |
| 10 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 11 | add-is-anonymous-header | Label 比對 | 📊 Count | 匿名 IP 插入 `X-WAF-IS-ANONYMOUS: true` Header | — |
| 12 | block-admin-from-external-ip | 複合條件 | 🚫 Block | 封鎖外部 IP 存取 WordPress 管理介面（`/wp-admin`、`/wp-login.php`、`/wp-json/wp/v2/users`、`/wp-json/oembed`），僅允許內部 IP 及 `admin-ajax.php` 通過 | 0 |
| 13 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 14 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 15 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP 顯示 CAPTCHA | 0 |
| 16 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | 0 |

---

## 4. clinic-acl-waf

- **ID**：be8118b1-8eb8-4319-be64-ea98669deec8
- **描述**：clinic waf acl
- **預設動作**：🚫 Block
- **CloudWatch Metric**：ClinicWaf
- **近一週總封鎖**：**1,730 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | vulnerability_scan_whitelist | IP 白名單 | ✅ Allow | 允許弱點掃描工具 IP 通過 | — |
| 1 | bypass_static_files | 正規表達式 | ✅ Allow | 允許 `clinic_web_static_file_route_patterns` 定義的靜態資源路徑通過 | — |
| 2 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` 覆寫為 Count | — |
| 3 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP | **559** |
| 4 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 5 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護（Log4j RCE、SSRF 等） | **7** |
| 6 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 7 | surveycake-ip-allowlist | IP 白名單 | ✅ Allow | 允許 SurveyCake 問卷服務 IP（surveycake-iplist），確保問卷回呼不被阻擋 | — |
| 8 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 9 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 10 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP 顯示 CAPTCHA | 0 |
| 11 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 700 次計數（不阻擋） | — |
| 12 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | 0 |
| 13 | malicious-user-agent | 複合條件 | 🚫 Block | 惡意 UA 正則 + 中國 IP 雙重條件封鎖 | 0 |

> 📊 **分析**：本週 1,730 次封鎖中，IP 信譽規則貢獻 559 次，已知惡意輸入 7 次，其餘約 1,164 次主要來自 Default Block（非內部 IP 請求）。

---

## 5. giver-acl-waf

- **ID**：ecb66ed9-7d5b-411f-883b-457fc0533d2d
- **描述**：giver waf acl
- **預設動作**：🚫 Block
- **CloudWatch Metric**：GiverWaf
- **近一週總封鎖**：**1,791 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | detected-malicious-ip | IP 封鎖清單 | 🚫 Block | 封鎖已偵測到的惡意 IP（detected_malicious_ip_set），動態更新的已知攻擊來源清單 | 0 |
| 1 | malicious-ip-block | IP 封鎖清單 | 🚫 Block | 封鎖額外惡意 IP 清單（Malicious-ipset），靜態惡意 IP 黑名單 | 0 |
| 10 | StaticFilePatterns | 正規表達式 | ✅ Allow | 允許 `cf-static-file-path-patterns` 定義的 CloudFront 靜態資源路徑通過 | — |
| 20 | managed-core | AWS 管理規則 | 🔍 排除部分 | **AWSManagedRulesCommonRuleSet**：排除（Exclude）`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`（等同停用這些規則） | — |
| 30 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP | **295** |
| 40 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 50 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | **6** |
| 100 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 110 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 120 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 130 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP CAPTCHA 驗證 | 0 |
| 140 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 700 次計數 | — |
| 150 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次 CAPTCHA | 0 |
| 160 | malicious-user-agent | 複合條件 | 🚫 Block | 惡意 UA 正則 + 中國 IP 雙重條件封鎖 | 0 |

> 📊 **分析**：本週 1,791 次封鎖中，IP 信譽規則貢獻 295 次，其餘約 1,490 次主要來自 Default Block 動作（非內部 IP 請求直接被預設封鎖）。

---

## 6. go-acl-waf

- **ID**：26ecd99f-ce28-4aee-8727-d167b69e5995
- **描述**：go waf acl（WordPress 部落格，帶 PHP 防護）
- **預設動作**：🚫 Block
- **CloudWatch Metric**：GoWaf
- **CAPTCHA 免疫時間**：600 秒
- **近一週總封鎖**：**566 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | bypass_static_files | 正規表達式（OR） | ✅ Allow | 允許靜態檔案副檔名、`/errorpages/`、`/wp-content/uploads/`、`/wp-includes/js/`、WordPress 主題/外掛 assets/vendors 通過 | — |
| 1 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` 覆寫為 Count | — |
| 2 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP | **13** |
| 3 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 4 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | **5** |
| 5 | managed-wordpress-rules | AWS 管理規則 | 🚫 Block | **AWSManagedRulesWordPressRuleSet**：WordPress 專屬攻擊防護（wp-login 暴力破解、XML-RPC 濫用、用戶枚舉防護） | **13** |
| 6 | managed-php-rules | AWS 管理規則 | 🚫 Block | **AWSManagedRulesPHPRuleSet**：PHP 程式碼注入/函數注入防護 | 0 |
| 7 | managed-bot-control | AWS 管理規則 | 📊 Count | **AWSManagedRulesBotControlRuleSet**：Bot 流量識別，覆寫為 Count | — |
| 8 | add-is-bot-header | Label 比對 | 📊 Count | Bot 流量插入 `X-WAF-IS-BOT: true` Header | — |
| 9 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 10 | add-is-anonymous-header | Label 比對 | 📊 Count | 匿名 IP 插入 `X-WAF-IS-ANONYMOUS: true` Header | — |
| 11 | block-admin-from-external-ip | 複合條件 | 🚫 Block | 封鎖外部 IP 存取 WordPress 管理介面（`/wp-admin`、`/wp-login.php`、`/wp-json/wp/v2/users`），允許 `admin-ajax.php` 通過 | **1** |
| 12 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 13 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 14 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP CAPTCHA（免疫時間 600 秒） | 0 |
| 15 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 300 次計數（閾值較嚴格） | — |
| 16 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | 0 |

> 📊 **分析**：本週 566 次封鎖中，IP 信譽 13 次、WordPress 攻擊 13 次、已知惡意輸入 5 次、WordPress 管理介面保護 1 次，其餘來自 Default Block。

---

## 7. internal-acl-waf

- **ID**：4833f500-eb14-4fd3-a877-1f45f135dc81
- **描述**：internal service acl waf（內部服務專用）
- **預設動作**：🚫 Block（僅允許內部 IP）
- **CloudWatch Metric**：internal-waf
- **近一週總封鎖**：**778 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | internal-ip-limitation | IP 白名單 | ✅ Allow | **唯一規則**：僅允許內部 IP（internal-ipset）通過。所有非內部 IP 的請求均被預設 Block 動作封鎖，實現完整的存取控制隔離 | — |

> 📊 **分析**：778 次封鎖**全部來自 Default Block 動作**，即所有來自非內部 IP 的請求均被預設封鎖。代表有 778 次外部請求嘗試存取內部服務端點。

---

## 8. intro-acl-waf

- **ID**：590fce66-f400-447d-b31f-e1b1a07361ad
- **描述**：intro waf acl（企業介紹頁面）
- **預設動作**：🚫 Block
- **CloudWatch Metric**：IntroWaf
- **近一週總封鎖**：**0 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | bypass_static_files | 正規表達式（OR） | ✅ Allow | 允許 js/css 檔案、`/errorpages/` 及各產品介紹頁靜態目錄（`intro_beagiver__`、`intro_blog__`、`intro_meet__`、`intro_talentmarket__`、`intro_lms__`、`intro_salary__`、`intro_personalbrand__`、`intro_senior__`）通過 | — |
| 1 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`CrossSiteScripting_BODY` 覆寫為 Count | — |
| 2 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖已知惡意 IP | 0 |
| 3 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 4 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | 0 |
| 5 | managed-bot-control | AWS 管理規則 | 📊 Count | **AWSManagedRulesBotControlRuleSet**：Bot 流量識別 | — |
| 6 | add-is-bot-header | Label 比對 | 📊 Count | Bot 流量插入 `X-WAF-IS-BOT: true` Header | — |
| 7 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 8 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 9 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 10 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP CAPTCHA 驗證 | 0 |
| 11 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 300 次計數（嚴格閾值） | — |
| 12 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | 0 |

---

## 9. meet-acl-waf

- **ID**：6ee5a35e-b00c-4cf0-92a1-8e93976972f4
- **描述**：meet waf acl
- **預設動作**：🚫 Block
- **CloudWatch Metric**：MeetWaf
- **近一週總封鎖**：**212 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | bypass_static_files | 正規表達式（OR） | ✅ Allow | 允許靜態檔案副檔名（svg/jpg/png/js/css/ico/gif/json/txt）及 `/errorpages`/`static/` 路徑通過 | — |
| 1 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER` 覆寫為 Count | — |
| 2 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP | **191** |
| 3 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 4 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護 | **6** |
| 5 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 6 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 7 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP CAPTCHA 驗證 | 0 |
| 8 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 9 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 700 次計數 | — |
| 10 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | 0 |

> 📊 **分析**：本週 212 次封鎖中，IP 信譽規則貢獻 191 次（佔比 90%），已知惡意輸入 6 次，其餘來自 Default Block。

---

## 10. nabi-acl-waf

- **ID**：3bf577f1-e6b2-4e61-8c6c-f2e822ff7016
- **描述**：nabi waf acl（含進階路由標記、IP 信譽清單及地理限速）
- **預設動作**：🚫 Block
- **CloudWatch Metric**：NabiWaf
- **近一週總封鎖**：**2,052 次**

| 優先級 | 規則名稱 | 類型 | 動作/效果 | 說明 | 近一週封鎖數 |
|:---:|---|---|---|---|:---:|
| 0 | detect_page_view | 正規表達式（NOT） | 📊 Count | 偵測非靜態/API 路由的頁面瀏覽請求，插入 `X-ROUTE-TYPE: page_view` Header | — |
| 1 | label_static_file_route | 正規表達式 | 📊 Count | 識別靜態資源路由，插入 `X-ROUTE-TYPE: static_file_route` Header | — |
| 2 | label_ajax_route | 正規表達式 | 📊 Count | 識別 API 路由（`/api/`、`/v2/`），插入 `X-ROUTE-TYPE: ajax_route` Header | — |
| 3 | bypass_static_files | 正規表達式 | ✅ Allow | 允許靜態資源路徑（`/errorpages`、`/static`、`/_nuxt`、`/asset/img`、`/asset/audio`、`/event/img`、`/css`、`/img`、`/js`、`/pcImg/`）直接通過 | — |
| 4 | managed-core | AWS 管理規則 | 🔍 部分 Count | **AWSManagedRulesCommonRuleSet**：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`、`SizeRestrictions_QUERYSTRING` 覆寫為 Count | — |
| 5 | query-string-size-limit | 大小限制 | 🚫 Block | 封鎖 Query String 超過 4096 bytes 的請求，防護超大參數注入攻擊 | 0 |
| 6 | managed-reputation-ip | AWS 管理規則 | 🚫 Block | **AWSManagedRulesAmazonIpReputationList**：封鎖 Amazon 威脅情報已知惡意 IP | **225** |
| 7 | managed-sql-injection | AWS 管理規則 | 🚫 Block | **AWSManagedRulesSQLiRuleSet**：SQL 注入攻擊防護 | 0 |
| 8 | managed-known-bad-input | AWS 管理規則 | 🚫 Block | **AWSManagedRulesKnownBadInputsRuleSet**：已知惡意輸入防護（Log4j RCE、SSRF 等） | **162** |
| 9 | internal-ip-limitation | IP 白名單 | ✅ Allow | 允許內部 IP 通過 | — |
| 10 | ip_reputation_lists | IP 封鎖清單（OR） | 🚫 Block | 封鎖自定義信譽名單（reputation_list_set_ipv4 / ipv6），客製化惡意 IP 封鎖清單 | **118** |
| 11 | managed-anonymous-ip | AWS 管理規則 | 📊 Count | **AWSManagedRulesAnonymousIpList**：匿名 IP 計數 | — |
| 12 | bot-white-list | Label 比對 | ✅ Allow | 允許 email client Bot | — |
| 13 | page_view_rate_breaker | 速率限制（Scope-Down） | 🧩 Captcha | 5 分鐘內同一 IP 頁面瀏覽（page_view Label）超過 100 次觸發 CAPTCHA，精準過濾頁面爬蟲 | — |
| 14 | scanners_and_probes | IP 封鎖清單（OR） | 🚫 Block | 封鎖已知掃描器和探測工具 IP（scanners_probes_set_ipv4 / ipv6） | 0 |
| 15 | http_flood | IP 封鎖清單（OR） | 🚫 Block | 封鎖已知 HTTP 洪水攻擊來源 IP（http_flood_set_ipv4 / ipv6） | 0 |
| 16 | bad_bot | IP 封鎖清單（OR） | 🚫 Block | 封鎖已知惡意 Bot IP（bad_bot_set_ipv4 / ipv6） | — |
| 17 | anonymous-ip-captcha | Label 比對 | 🧩 Captcha | 匿名 IP CAPTCHA 驗證 | — |
| 18 | request-rate-breaker | 速率限制 | 📊 Count | 5 分鐘 700 次計數 | — |
| 19 | request-rate-captcha | 速率限制 | 🧩 Captcha | 5 分鐘 700 次觸發 CAPTCHA | — |
| 20 | geo-rate-limit-captcha | 速率限制（地理）| 📊 Count | 非台灣（TW）來源，5 分鐘超過 700 次計數 | — |
| 21 | geo-rate-limit-minor-countries | 速率限制（地理）| 🔐 Challenge | 非主要地區（排除 TW/CN/SG/US/HK/JP），5 分鐘 100 次以上觸發 Silent JS Challenge | — |
| 22 | malicious-user-agent | 複合條件 | 🚫 Block | 惡意 UA 正則（malicious_user_agents）+ 中國 IP 雙重條件封鎖 | — |
| 23 | malicious-user-agent-no-area | 正規表達式 | 🚫 Block | 無地區限制的惡意 UA 封鎖（malicious_user_agent pattern set），所有 IP 適用 | — |
| 24 | manufacturer-ip-limitation | IP 白名單 | ✅ Allow | 允許製造商/合作夥伴 IP（manufacturer-ipset）通過 | — |

> 📊 **分析**：本週 2,052 次封鎖中，IP 信譽 225 次、已知惡意輸入 162 次（Log4j 等攻擊）、自定義 IP 信譽清單 118 次，合計規則觸發 505 次，其餘 ~1,547 次來自 Default Block 動作及其他規則（geo-rate、CAPTCHA 等）。

---

## 費用預估

### 本次盤查 AWS API 呼叫統計

| 服務 | 操作 | 呼叫次數 | 單價 | 小計 |
|------|------|:---:|------|------|
| AWS WAFv2 | ListWebACLs | 6 | $0.00（免費） | $0.00 |
| AWS WAFv2 | GetWebACL | 10 | $0.00（免費） | $0.00 |
| AWS STS | GetCallerIdentity | 1 | $0.00（免費） | $0.00 |
| AWS IAM | ListAccountAliases | 1 | $0.00（免費） | $0.00 |
| Amazon CloudWatch | ListMetrics | 1 | $0.01 / 1,000次 | < $0.001 |
| Amazon CloudWatch | GetMetricStatistics | **74** | $0.01 / 1,000次 | < $0.001 |
| **合計** | | **93** | | **~$0.001** |

### WAF 持續運行費用（參考）

| 項目 | 費率 | 說明 |
|------|------|------|
| Web ACL 數量費 | $5.00 / 月 / ACL | 10 個 ACL = $50.00/月 |
| 規則費 | $1.00 / 月 / 規則 | 依實際規則數計費 |
| AWS 管理規則 | $1.00 / 月 / 規則集 | 依啟用的規則集數計費 |
| 請求費 | $0.60 / 100萬次請求 | 依實際流量計費 |
| Bot Control | $10.00 / 月 + $1.00 / 100萬請求 | 啟用 BotControlRuleSet 的 ACL |

> **本次盤查 API 費用約 $0.001 USD（不到 1 美分），可忽略不計。**

---

*報告產生：2026-06-04 | WAF Scope: CLOUDFRONT (global) | AWS Account: 518375879370 (104awsdev14-ro) | Region: us-east-1*
