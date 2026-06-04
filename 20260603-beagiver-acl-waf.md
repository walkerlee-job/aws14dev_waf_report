# AWS WAF 封鎖分析報告 — beagiver-acl-waf

---

## 1. 執行摘要

| 項目 | 內容 |
|------|------|
| WAF ACL 名稱 | `beagiver-acl-waf` |
| 分析日期（台北時間） | 2026-06-03（週三） |
| 時間範圍（UTC） | 2026-06-02T16:00:00Z ～ 2026-06-03T16:00:00Z |
| 總封鎖次數 | **260** 次 |
| 總允許次數 | **514** 次 |
| 總請求次數 | **774** 次 |
| 封鎖率 | **33.6%** |
| 尖峰封鎖時段 | 17:00（173 次）、22:00（35 次） |
| 主要封鎖原因（Top 1） | ExploitablePaths_URIPATH（已知可利用路徑掃描） |
| 主要來源國家（Top 2） | 🇺🇸 美國（70.8%）、🇭🇰 香港（12.7%） |

### 整體評估

昨日 beagiver-acl-waf 共封鎖 260 次請求，封鎖率 33.6%，呈現**中等威脅等級**。最需關注的是 **17:00 的異常尖峰**：單一小時封鎖 173 次（佔全日 66.5%），主要由 `ExploitablePaths_URIPATH` 規則觸發，來源集中在美國 IP（171 次），研判為疑似自動化漏洞掃描工具針對已知可利用路徑發動的探測攻擊。此外，12:00 出現荷蘭 IP 偵察行為（28 次），22:00 香港 IP 觸發預設封鎖行為（33 次），建議近期調閱 Sampled Requests 確認攻擊特徵後評估是否需強化規則。

---

## 2. 封鎖前 5 名規則

| 排名 | 規則名稱（子規則） | 所屬規則群組 | 父規則 Priority | 封鎖次數 | 佔比 | 說明 |
|------|-------------------|--------------|-----------------:|---------:|-----:|------|
| 🥇 | `ExploitablePaths_URIPATH` | AWSManagedRulesKnownBadInputsRuleSet | 5 — managed-known-bad-input | **117** | 45.0% | 偵測 URI 路徑中包含常見可利用路徑（如 `/etc/passwd`、`/.env`、`/wp-config.php`、`/phpmyadmin` 等），封鎖對已知弱點路徑的探測與利用嘗試 |
| 🥈 | `AWSManagedReconnaissanceList` | AWSManagedRulesAmazonIpReputationList | 3 — managed-reputation-ip | **25** | 9.6% | 封鎖 Amazon 威脅情資中已知從事偵察活動（端口掃描、服務探測）的 IP 地址，通常與自動化掃描工具、資安研究/惡意探測活動有關 |
| 🥉 | `HostingProviderIPList` | AWSManagedRulesAnonymousIpList | 7 — managed-anonymous-ip（Count 模式）| **14** | 5.4% | 識別來自雲端/主機服務商 IP（AWS EC2、Azure VM、GCP 等）的請求，常被攻擊者用於隱藏真實來源；因父規則組設為 Count 覆蓋，子規則標記後由後續 CAPTCHA 規則或預設 Block 動作封鎖 |
| 4 | `AWSManagedIPReputationList` | AWSManagedRulesAmazonIpReputationList | 3 — managed-reputation-ip | **7** | 2.7% | 封鎖 Amazon 威脅情資資料庫中已知惡意 IP，包含殭屍網路 C&C 伺服器、已知攻擊者 IP、惡意網路基礎設施等 |
| 5 | `RestrictedExtensions_URIPATH` | AWSManagedRulesCommonRuleSet | 2 — managed-core | **6** | 2.3% | 封鎖 URI 路徑中包含危險副檔名的請求（如 `.php`、`.asp`、`.cgi`、`.jsp` 等），防止對伺服器端腳本路徑的未授權存取探測 |

> **注意事項（HostingProviderIPList）：** `managed-anonymous-ip`（Priority 7）設有 `OverrideAction=Count`，該規則群組下所有子規則均以計數模式運行（請求不被直接封鎖，僅添加標籤）。CloudWatch 所記錄的 BlockedRequests 數據，實際上是匹配 HostingProviderIPList 後，由 Priority 9 的 `anonymous-ip-captcha`（Captcha 動作）CAPTCHA 挑戰失敗，或流至預設 Block 動作而被封鎖的請求數。

> **ManagedRuleGroupRule 封鎖數 vs Rule 維度驗證：**  
> - `managed-known-bad-input`（Rule）= 120 = ExploitablePaths(117) + ReactJSRCE(3) ✓  
> - `managed-reputation-ip`（Rule）= 32 = Reconnaissance(25) + IPReputation(7) ✓  
> - `managed-core`（Rule）= 6 = RestrictedExtensions(6) ✓  
> - 已命名規則合計 = 158；剩餘 **102 次**由預設 Block 動作（Default Action: Block）攔截

---

## 3. 每小時封鎖時序（台北 UTC+8）

| 時段（+8） | 封鎖 | 允許 | 總請求 | 封鎖率 | 備註 |
|:----------:|-----:|-----:|-------:|-------:|------|
| 00:00–01:00 | 1 | 0 | 1 | 100% | 散點流量 |
| 01:00–02:00 | 0 | 0 | 0 | — | 低流量 |
| 02:00–03:00 | 1 | 0 | 1 | 100% | 散點流量（LU IP）|
| 03:00–04:00 | 1 | 0 | 1 | 100% | 散點流量（BG IP）|
| 04:00–08:00 | 0 | 0 | 0 | — | 深夜低流量 |
| 08:00–09:00 | 1 | 0 | 1 | 100% | 散點流量 |
| 09:00–10:00 | 0 | 0 | 0 | — | — |
| 10:00–11:00 | 0 | 17 | 17 | 0% | 正常業務流量 |
| 11:00–12:00 | 0 | 0 | 0 | — | — |
| 12:00–13:00 | 28 | 0 | 28 | 100% | NL 偵察行為高峰（AWSManagedReconnaissanceList 24 次）|
| 13:00–14:00 | 0 | 71 | 71 | 0% | 正常業務流量 |
| 14:00–15:00 | 1 | 0 | 1 | 100% | 散點 NL IP |
| 15:00–16:00 | 0 | 82 | 82 | 0% | 正常業務流量 |
| 16:00–17:00 | 14 | 282 | 296 | 4.7% | 正常業務高峰開始，含少量 SG/JP/MY HostingProvider |
| **17:00–18:00** | **173** | **62** | **235** | **73.6%** | ⚠ **異常尖峰：ExploitablePaths 掃描（美國 IP 171 次）** |
| 18:00–19:00 | 1 | 0 | 1 | 100% | — |
| 19:00–20:00 | 1 | 0 | 1 | 100% | — |
| 20:00–21:00 | 3 | 0 | 3 | 100% | — |
| 21:00–22:00 | 0 | 0 | 0 | — | — |
| **22:00–23:00** | **35** | **0** | **35** | **100%** | ⚠ **HK IP 預設封鎖（33 次）** |
| 23:00–24:00 | 0 | 0 | 0 | — | — |
| **合計** | **260** | **514** | **774** | **33.6%** | |

### 每小時封鎖時序 ASCII 長條圖（每 █ = 10 次，▌ = 1-9 次）

```
時段    封鎖次數
────────────────────────────────────────
00:00 │▌                               │   1
01:00 │                                │   0
02:00 │▌                               │   1
03:00 │▌                               │   1
04:00 │                                │   0
05:00 │                                │   0
06:00 │                                │   0
07:00 │                                │   0
08:00 │▌                               │   1
09:00 │                                │   0
10:00 │                                │   0
11:00 │                                │   0
12:00 │███                             │  28
13:00 │                                │   0
14:00 │▌                               │   1
15:00 │                                │   0
16:00 │█▌                              │  14
17:00 │█████████████████▌  ⚠ 尖峰      │ 173
18:00 │▌                               │   1
19:00 │▌                               │   1
20:00 │▌                               │   3
21:00 │                                │   0
22:00 │████                            │  35
23:00 │                                │   0
────────────────────────────────────────
```

### 時序觀察

- **低水位時段（00:00–09:00）**：每小時封鎖 0–1 次，業務流量近乎為零，僅有少量散點探測請求
- **業務高峰開始（10:00–16:00）**：允許請求明顯增加（10:00=17、13:00=71、15:00=82、16:00=282），封鎖仍屬低水位
- **異常尖峰（17:00）**：封鎖從前一小時 14 次暴增至 173 次（增幅 **+1136%**），允許請求同時有 62 次，顯示是外部攻擊而非流量清空。ExploitablePaths 子規則占此時段 116/173 = **67%**，來源美國 IP 占 171/173 = **99%**，高度集中
- **22:00 次尖峰**：HK IP（33 次）未被任何管理規則命中，全數由預設 Block 動作封鎖，可能為正常用戶（HK 訪客無法通過 beagiver 允許規則）或探測流量

---

## 4. 來源國家分布

| 排名 | 國家 | 代碼 | 封鎖次數 | 佔比 | 主要觸發規則 |
|------|------|:----:|---------:|-----:|------|
| 1 | 🇺🇸 美國 | US | **184** | 70.8% | `ExploitablePaths_URIPATH`（17:00 爆發）、`ReactJSRCE_BODY`、Default Action |
| 2 | 🇭🇰 香港 | HK | **33** | 12.7% | Default Action Block（無管理規則命中，預設封鎖）|
| 3 | 🇳🇱 荷蘭 | NL | **29** | 11.2% | `AWSManagedReconnaissanceList`（24）、`RestrictedExtensions_URIPATH`（4）|
| 4 | 🇸🇬 新加坡 | SG | **6** | 2.3% | `HostingProviderIPList` 標記後封鎖 |
| 5 | 🇯🇵 日本 | JP | **2** | 0.8% | `AWSManagedIPReputationList` |
| 6 | 🇲🇾 馬來西亞 | MY | **2** | 0.8% | `HostingProviderIPList` 標記後封鎖 |
| 7 | 🇱🇺 盧森堡 | LU | **2** | 0.8% | `ExploitablePaths_URIPATH`、Default Action |
| 8 | 🇧🇬 保加利亞 | BG | **2** | 0.8% | `AWSManagedIPReputationList` |

> **出現在 list-metrics 但昨日封鎖為 0 的國家（代表有允許流量或無當日請求）：**  
> FI（芬蘭）、FR（法國）、ZA（南非）、RO（羅馬尼亞）、AR（阿根廷）、BY（白俄羅斯）、IN（印度）、KR（韓國）、GB（英國）、BR（巴西）、UA（烏克蘭）、IR（伊朗）、SE（瑞典）、CA（加拿大）、RU（俄羅斯）

### 國家封鎖分布 ASCII 橫向長條圖（每 █ = 10 次，▌ = 1-9 次）

```
🇺🇸 US │██████████████████▌             │ 184 (70.8%)
🇭🇰 HK │███▌                            │  33 (12.7%)
🇳🇱 NL │███                             │  29 (11.2%)
🇸🇬 SG │▌                               │   6 ( 2.3%)
🇯🇵 JP │▌                               │   2 ( 0.8%)
🇲🇾 MY │▌                               │   2 ( 0.8%)
🇱🇺 LU │▌                               │   2 ( 0.8%)
🇧🇬 BG │▌                               │   2 ( 0.8%)
```

---

## 5. 異常事件分析

### 事件 A：17:00 ExploitablePaths 掃描爆發

| 項目 | 詳情 |
|------|------|
| 觸發規則 | `ExploitablePaths_URIPATH`（AWSManagedRulesKnownBadInputsRuleSet）、`ReactJSRCE_BODY` |
| 發生時段 | 2026-06-03 17:00–18:00（台北時間） |
| 封鎖次數 | 173 次（KnownBadInputs 子規則：119 次；其餘：54 次）|
| 來源國家 | 🇺🇸 美國（171/173 = 98.8%） |
| 封鎖占全日比例 | 66.5% |
| 評估 | **威脅等級：高** — 1 小時內 173 次封鎖，為當日每小時平均值（10.8 次）的 **16 倍**。`ExploitablePaths_URIPATH` 命中 116 次，代表攻擊者大量探測 beagiver 應用程式中已知可利用的敏感路徑（如 `.env`、`wp-config.php`、`phpmyadmin` 等），高度集中於美國 IP，極可能為自動化掃描工具（如 Shodan、Nuclei、Metasploit）發起的弱點探測。`ReactJSRCE_BODY`（3 次）同時出現，顯示攻擊者亦嘗試 React.js RCE 漏洞利用，攻擊意圖明確。|

**行動建議：** 立即調閱 17:00 的 Sampled Requests，確認觸發的具體 URI 路徑，若為針對 beagiver 特定路徑的定向攻擊，考慮將攻擊 IP 加入 IP Set 封鎖清單。

---

### 事件 B：22:00 香港 IP 預設封鎖群

| 項目 | 詳情 |
|------|------|
| 觸發機制 | Default Action Block（預設封鎖） |
| 發生時段 | 2026-06-03 22:00–23:00（台北時間） |
| 封鎖次數 | 33 次（HK）+ 2 次（其他規則）= 35 次 |
| 來源國家 | 🇭🇰 香港（33/35 = 94%） |
| 評估 | **威脅等級：中** — 22:00 時段出現 33 次來自香港 IP 的封鎖，特別之處在於這些請求**未被任何管理規則（Managed Rules）命中**，而是直接觸發了 WAF 的預設封鎖動作（Default Action: Block）。此情況可能有兩種解讀：(1) 正常 HK 用戶嘗試存取 beagiver，但 WAF 預設封鎖策略過嚴，導致合法流量被誤封；(2) 探測流量嘗試不含明顯攻擊特徵的一般請求，被預設 Block 攔截。建議調閱 Sampled Requests 確認 URI 和 User-Agent，以判斷是否為正常用戶訪問。|

---

### 事件 C：12:00 荷蘭 IP 偵察活動

| 項目 | 詳情 |
|------|------|
| 觸發規則 | `AWSManagedReconnaissanceList`（24 次）、`RestrictedExtensions_URIPATH`（4 次）|
| 發生時段 | 2026-06-03 12:00–13:00（台北時間） |
| 封鎖次數 | 28 次（全數來自 NL IP）|
| 來源國家 | 🇳🇱 荷蘭（NL）|
| 評估 | **威脅等級：中** — 荷蘭 IP 在 12:00 觸發 `AWSManagedReconnaissanceList` 規則 24 次，該規則封鎖 Amazon 威脅情資中已知從事偵察活動的 IP。荷蘭是許多 VPS 提供商（如 Hetzner、TransIP）和安全研究機構的所在地，這些 IP 可能屬於以下類型：安全掃描服務（Shodan、Censys）、滲透測試框架、或惡意偵察活動。同時出現的 4 次 `RestrictedExtensions_URIPATH` 命中，顯示有試圖存取敏感副檔名路徑的行為，與偵察活動特徵吻合。|

---

## 6. 共通性分析

### 攻擊類型分析

昨日攻擊態樣可歸納為三種主要類型：

1. **已知弱點路徑掃描（45.0%）**：`ExploitablePaths_URIPATH` 命中 117 次，為當日最主要威脅。攻擊工具對 beagiver 應用探測 CMS 相關路徑、框架特定弱點路徑及常見設定檔路徑。

2. **IP 信譽威脅（12.3%）**：`AWSManagedReconnaissanceList`（25 次）+ `AWSManagedIPReputationList`（7 次）合計 32 次，來自 Amazon 已知惡意 IP 清單，NL 偵察 IP 是主要貢獻者。

3. **匿名/主機提供商 IP（5.4%）**：`HostingProviderIPList` 命中 14 次，來自雲端 IP 的請求經 CAPTCHA 失敗或預設封鎖。

4. **預設封鎖（39.2%，共 102 次）**：大量請求未被任何具名管理規則命中，直接觸發 Default Action Block。這部分包含 HK IP（33 次，22:00）和部分其他流量，可能反映 beagiver WAF 預設封鎖策略過於嚴格，或確實有正常用戶流量被誤封。

**Attack / VulnerabilityCategory 維度交叉驗證：**
- `Attack=KnownBadInputs`：120 次（與 managed-known-bad-input 規則層一致）
- `Attack=RestrictedExtensions`：6 次（與 managed-core 規則層一致）
- `VulnerabilityCategory=HostingProviderIPList`：24 次
- `VulnerabilityCategory=AWSManagedIPReputationList`：7 次
- `VulnerabilityCategory=AWSManagedReconnaissanceList`：4 次
- `VulnerabilityCategory=AWSManagedIPDDoSList`：0 次（無 DDoS 攻擊）
- **未偵測到的攻擊類型**：`Attack=GenericLFI` = 0（無 LFI 攻擊）、SQLi = 0、XSS = 0

### 時間模式

- **持續性 vs 集中性**：昨日呈現**集中性攻擊**特徵。17:00 單一小時占全日 66.5%，具有明顯突發性，符合自動化掃描工具的行為模式（短時間內大量 HTTP 請求）。
- **業務流量相關性**：允許請求主要集中在 13:00–17:00（業務高峰），封鎖亦在此時段後開始增加，顯示攻擊者選擇在業務流量活躍時段發動，可能是為了混入正常流量。
- **夜間低活動**：00:00–09:00 期間僅有散點封鎖（4 次），無持續性攻擊跡象。

### 地理模式

- **美國 IP 占比 70.8%**：美國 IP 主導封鎖事件，但多數攻擊來自雲端/主機提供商 IP（非真實美國用戶）。17:00 的 ExploitablePaths 攻擊幾乎全來自美國 IP，推測為部署在美國 AWS/Azure/GCP 上的攻擊工具。
- **荷蘭 IP 偵察**：NL IP 在 12:00 的集中偵察活動（28 次），典型的 Shodan/Censys 掃描或 VPS 攻擊節點模式。
- **香港 IP 預設封鎖**：22:00 HK IP（33 次）的預設封鎖值得關注，需確認是否為合法用戶遭誤封。

### 規則效果評估

| 規則 | 效果評估 |
|------|---------|
| managed-known-bad-input | ✅ **有效** — 攔截 120 次 ExploitablePaths/ReactJSRCE 攻擊 |
| managed-reputation-ip | ✅ **有效** — 攔截 32 次已知惡意/偵察 IP |
| managed-core | ✅ **有效** — 攔截 6 次危險副檔名路徑掃描 |
| managed-anonymous-ip（Count 模式）| ⚠️ **僅計數** — OverrideAction=Count，子規則不直接封鎖；14 次 HostingProviderIPList 封鎖依賴後續 CAPTCHA 或預設 Block |
| malicious-user-agent | ℹ️ **未觸發** — 昨日無命中（CN IP 封鎖 = 0）|
| managed-sql-injection | ℹ️ **未觸發** — 無 SQLi 攻擊 |
| Default Action（Block）| ⚠️ **需評估** — 102 次預設封鎖，其中包含 HK IP 33 次，需確認是否有合法用戶被誤封 |

---

## 7. 建議事項

### 短期建議（立即 ～ 2 週）

1. **調閱 17:00 Sampled Requests**  
   立即在 AWS WAF 主控台查閱 2026-06-03 17:00–18:00 期間 `ExploitablePaths_URIPATH` 的抽樣請求，確認：具體被探測的 URI 路徑、攻擊 IP 範圍、User-Agent 特徵。若來源 IP 固定，建議加入自訂 IP Block Set。

2. **確認 22:00 HK IP 封鎖性質**  
   調閱 22:00 的 Sampled Requests，判斷 HK IP 封鎖是否為合法 beagiver 用戶（如 HK 業務員、合作方）。若為合法用戶，考慮在 WAF 中添加地理允許規則或將 HK IP 段加入白名單。

3. **調閱 12:00 NL 偵察 Sampled Requests**  
   確認荷蘭 IP 是否為已知安全掃描服務（Shodan、Censys）或未知威脅來源，評估是否需要針對 `AWSManagedReconnaissanceList` 命中的 IP 採取額外行動。

### 中期建議（2 週 ～ 2 個月）

4. **設定 CloudWatch Alarm**  
   針對 `Rule=ALL, BlockedRequests` 設置每小時 > 50 次的 CloudWatch Alarm（SNS 通知），以便即時發現類似 17:00 的異常尖峰。

5. **評估 managed-anonymous-ip 的 OverrideAction**  
   目前 `managed-anonymous-ip`（Priority 7）設為 Count 模式，考慮是否將特定高風險子規則（如 `HostingProviderIPList`）的動作改為 Block，以直接封鎖來自雲端 IP 的請求，而不依賴後續 CAPTCHA 機制。

6. **評估預設封鎖策略對正常用戶的影響**  
   beagiver WAF 的 Default Action 為 Block，需評估是否有正常用戶（特別是 HK、海外訪客）因無法通過任何允許規則而被誤封。考慮添加地理允許規則，或調整 bypass_static_files 規則的範圍。

7. **啟用 WAF Logging（S3 或 CloudWatch Logs）**  
   目前無法從 CloudWatch 取得 IP 層級、URI 層級的詳細攻擊資訊。建議啟用 WAF 完整日誌記錄，以便進行更細緻的威脅分析。

### 長期建議（2 個月以上）

8. **建立自動化 WAF 日報系統**  
   使用 EventBridge + Lambda 每日自動執行 CloudWatch 查詢，產生標準化報告並發送到 Slack/Email。

9. **WAF Logging + Athena + QuickSight 分析管道**  
   啟用 WAF 完整日誌（送至 S3），建立 Athena 查詢用於深層分析（IP 分布、URI 熱點、時序分析），並建立 QuickSight Dashboard 供團隊長期監控。

10. **定期評估 AWS Managed Rules 版本更新**  
    AWS 定期更新 Managed Rules 規則庫，建議每季度確認各規則群組是否有新規則更新，以及是否需要調整 Override Actions。

---

## 8. WAF ACL 規則完整清單

> **ACL 名稱：** `beagiver-acl-waf`  
> **ARN：** `arn:aws:wafv2:us-east-1:518375879370:global/webacl/beagiver-acl-waf/3f7edf3e-412d-4225-b24c-68e5f17eb507`  
> **容量：** 1213 WCU

| Priority | 規則名稱 | 類型 | Action / OverrideAction | Metric Name | 說明 |
|:--------:|---------|------|:-----------------------:|-------------|------|
| 0 | `vulnerability-scan-whitelist` | 自訂（IP Set） | **Allow** | beagiver-waf-vulnerability-scan-whitelist | 弱點掃描白名單 IP 清單，允許內部安全掃描工具的請求通過，避免合法安全測試被 WAF 封鎖 |
| 1 | `bypass_static_files` | 自訂（Regex Match） | **Allow** | beagiver-waf-bypass-static-files | 允許符合特定路徑 regex 的靜態資源請求（`/errorpages`、`/static`、`/2023giverreview`、`/2024giverreview/`），確保靜態前端資源不被封鎖 |
| 2 | `managed-core` | ManagedRuleGroup（AWSManagedRulesCommonRuleSet）| OverrideAction: **None** | beagiver-waf-managed-core | AWS 核心規則集（CRS），防護常見 Web 應用弱點。部分規則已覆蓋為 Count：`NoUserAgent_HEADER`、`GenericRFI_QUERYARGUMENTS`、`SizeRestrictions_BODY`、`SizeRestrictions_Cookie_HEADER`（避免誤封正常業務請求）。封鎖規則：`GenericLFI_QUERYARGUMENTS/BODY/URIPATH`（本地檔案引入）、`RestrictedExtensions_URIPATH`（危險副檔名路徑） |
| 3 | `managed-reputation-ip` | ManagedRuleGroup（AWSManagedRulesAmazonIpReputationList）| OverrideAction: **None** | beagiver-waf-managed-reputation-ip | Amazon IP 信譽清單，封鎖 Amazon 威脅情資資料庫中已知惡意 IP：`AWSManagedIPReputationList`（殭屍網路 C&C、已知攻擊者）、`AWSManagedReconnaissanceList`（偵察掃描 IP）、`AWSManagedIPDDoSList`（DDoS 攻擊者 IP） |
| 4 | `managed-sql-injection` | ManagedRuleGroup（AWSManagedRulesSQLiRuleSet）| OverrideAction: **None** | beagiver-waf-managed-sql-injection | AWS SQL 注入防護規則集，偵測並封鎖 HTTP 請求中的 SQL 注入攻擊特徵（Query 參數、Request Body、URI 路徑、標頭） |
| 5 | `managed-known-bad-input` | ManagedRuleGroup（AWSManagedRulesKnownBadInputsRuleSet）| OverrideAction: **None** | beagiver-waf-managed-known-bad-input | 已知惡意輸入規則集，封鎖已知惡意輸入模式：`ExploitablePaths_URIPATH`（已知可利用路徑）、`ReactJSRCE_BODY`（React.js RCE 漏洞利用）、`JavaDeserializationRCE`（Java 反序列化 RCE）、`SSRF_QUERYARGUMENTS`（SSRF 攻擊）等 |
| 6 | `internal-ip-limitation` | 自訂（IP Set） | **Allow** | beagiver-waf-internal-ip-limitation | 內部 IP 白名單，允許內部服務 IP 直接通過，無需經過後續安全規則檢查 |
| 7 | `managed-anonymous-ip` | ManagedRuleGroup（AWSManagedRulesAnonymousIpList）| OverrideAction: **Count** ⚠️ | beagiver-waf-managed-anonymous-ip | 匿名 IP 清單（計數模式），識別並標記來自匿名服務的 IP：`AnonymousIPList`（Tor、匿名代理、VPN）、`HostingProviderIPList`（AWS EC2/Azure VM/GCP 等雲端 IP）。**設為 Count 模式，不直接封鎖，僅添加 Label 供後續規則使用** |
| 8 | `bot-white-list` | 自訂（Label Match） | **Allow** | beagiver-waf-bot-whitelist | Email 用戶端 Bot 白名單，允許帶有 `awswaf:managed:aws:bot-control:bot:category:email_client` 標籤的請求通過，避免正常電子郵件服務被封鎖 |
| 9 | `anonymous-ip-captcha` | 自訂（Label Match） | **Captcha** | beagiver-waf-anonymous-ip-captcha | 針對匿名 IP 標籤（`awswaf:managed:aws:anonymous-ip-list:AnonymousIPList`）的請求發起 CAPTCHA 驗證挑戰；CAPTCHA 通過則放行，失敗則封鎖 |
| 10 | `request-rate-breaker` | 自訂（Rate Based）| **Count** | beagiver-waf-request-rate-breaker | 速率計數規則，偵測每 IP 在 5 分鐘內超過 700 次請求；設為 Count 模式（僅計數，不封鎖），可能為後續 Captcha 規則的參考依據 |
| 11 | `request-rate-captcha` | 自訂（Rate Based）| **Captcha** | beagiver-waf-request-rate-captcha | 速率 CAPTCHA 規則，對每 IP 在 5 分鐘內超過 700 次請求的來源發起 CAPTCHA 驗證，防止高頻自動化請求 |
| 12 | `malicious-user-agent` | 自訂（AND：Regex + Geo Match）| **Block** + Label | beagiver-waf-malicious-user-agent | 封鎖來自中國（CN）且 User-Agent 符合惡意特徵 regex 的請求，添加 `malicious-user-agent` 標籤。結合地理與 UA 特徵的精準封鎖規則 |
| — | **Default Action** | — | **Block** | — | 所有不符合任何規則的請求一律封鎖（預設拒絕策略）。昨日約 **102 次**請求由此機制封鎖 |

---

## 9. 資料來源與查詢說明

### 查詢環境

| 項目 | 值 |
|------|-----|
| AWS Profile | `104awsdev14-ro` |
| Scope | CLOUDFRONT（全球） |
| Region | `us-east-1`（CloudFront WAF 固定區域） |
| Namespace | `AWS/WAFV2` |
| Period | 3600 秒（每小時） |
| 分析時間（UTC） | 2026-06-02T16:00:00Z ～ 2026-06-03T16:00:00Z |
| 分析日期（台北 UTC+8） | 2026-06-03 00:00 ～ 23:59 |

### 查詢 Dimension 說明

| Dimension 組合 | 用途 |
|---------------|------|
| `WebACL=beagiver-acl-waf, Rule=ALL` | 全 ACL 總封鎖/允許基準值 |
| `WebACL=beagiver-acl-waf, Rule={MetricName}` | 各命名規則的封鎖次數（Rule 維度） |
| `WebACL=beagiver-acl-waf, ManagedRuleGroup={}, ManagedRuleGroupRule={}` | 管理規則群組子規則的封鎖次數 |
| `WebACL=beagiver-acl-waf, Country={Code}` | 各來源國家的封鎖次數 |
| `WebACL=beagiver-acl-waf, Attack={Value}` | 攻擊類型維度（交叉驗證） |
| `WebACL=beagiver-acl-waf, VulnerabilityCategory={Value}` | 弱點類別維度（交叉驗證） |

### 資料限制

1. **CloudWatch 為彙總統計**：CloudWatch Metrics 僅提供每小時聚合計數，無法查看個別請求的來源 IP、URI、User-Agent 等詳細資訊。需搭配 WAF Sampled Requests 或完整 WAF Logging 進行深層分析。

2. **ManagedRuleGroup Count OverrideAction**：`managed-anonymous-ip`（Priority 7）設為 Count 覆蓋，其子規則（HostingProviderIPList、AnonymousIPList）不直接封鎖請求，而是添加標籤供後續規則處理。CloudWatch 顯示的 BlockedRequests 數據為這些請求在後續規則（anonymous-ip-captcha / Default Action）中最終被封鎖的次數，**非由該子規則本身直接封鎖**。

3. **Country 維度**：Country 維度僅代表封鎖事件的請求來源 IP 地理位置，因大量攻擊使用雲端/VPS IP，實際攻擊者所在地可能與 IP 地理位置不同。

4. **預設封鎖（102 次）歸因**：Rule=ALL 總封鎖 260 次，已命名規則總和 158 次，差值 102 次由 Default Action Block 或未出現在 list-metrics 的規則（如 anonymous-ip-captcha CAPTCHA 失敗）所貢獻，無法從 CloudWatch 進一步細分。

### 查詢步驟

1. `aws wafv2 list-web-acls` — 取得所有 Web ACL 清單
2. `aws wafv2 get-web-acl` — 取得 beagiver-acl-waf 完整規則設定
3. `aws cloudwatch list-metrics --dimensions Name=WebACL,Value=beagiver-acl-waf` — 取得所有可用 Dimension 組合（BlockedRequests + AllowedRequests）
4. `aws cloudwatch get-metric-statistics`（共 49 次查詢，分 3 批次執行）— 依序查詢 Rule、ManagedRuleGroupRule、Country、Attack、VulnerabilityCategory 維度的 BlockedRequests 及 AllowedRequests

---

### AWS API 費用預估

| 服務 | API 操作 | 呼叫次數 | 費用（us-east-1） |
|------|---------|:-------:|:----------------:|
| AWS WAF v2 | `GetWebACL` | 1 | 免費（WAF 服務費用內）|
| AWS WAF v2 | `ListWebACLs` | — | 免費 |
| CloudWatch | `ListMetrics` | 2 | $0.00（含 10,000 免費額度）|
| CloudWatch | `GetMetricStatistics` | 49 | $0.00（含 10,000 免費額度）|
| **合計** | — | **52 次 API 呼叫** | **< $0.01**（在免費額度內）|

> CloudWatch API 定價：$0.01 / 1,000 次請求，每月前 10,000 次免費。本次 52 次查詢完全在免費額度內。

---

*報告產生時間：2026-06-04 Taipei Time*  
*資料來源：AWS CloudWatch Metrics（AWS/WAFV2 namespace）、AWS WAFv2 GetWebACL API*  
*分析工具：Claude Code (AWS MCP API)*
