---
title: 如何真的讀懂 ATT&CK® Evaluations Round2
---

# 前言

這篇文章是我在奧義智慧(CyCarrier)工作時完成，將同樣的內容同步轉載在個人部落格做為紀錄
[Link](https://medium.com/@cycarrier.tw/第二屆-att-ck-大亂鬥-天下第一武道會-d2dd79ba5621)

![ATT&CK T-shirt 與 CyCraft合照](/assets/mewithCyCraft.png)
## 建議先理解痛苦金字塔模型 (pyramid of pain)

痛苦金字塔模型是在描述攻擊方的痛點，防守方採取怎樣的防禦層次會使攻擊方更痛苦。

ATT&CK 框架主要在描述攻擊方的 TTP (Tactics, Techniques and Procedures) 與 Tools ，當防守方善用 ATT&CK 框架來做為偵測基準，攻擊方會很痛苦。

這也是為何近年全世界資安圈都十分推崇 ATT&CK 框架。

實務上雖然有些資安產品主力偵測/阻擋仍是依靠 IoC (e.g. Domain, IP, Hash)。

但長遠下來真正能夠穩定的捕捉駭客，仍要從 TTP 面向著手。

例如：許多防毒軟體都能夠偵測到 Mimikatz 這個檔案或是變種，但當出現 Fileless 版本或變種的 Mimikatz 時，有些資安廠商就無法處理，因為他們偵測的是 IoC 特徵，而不是 TTP 這類高階的攻擊手法。

![https://detect-respond.blogspot.com/2013/03/the-pyramid-of-pain.html](/assets/pyramidofpain.png)

## 不是所有的 Technique 都得一擊必殺

在正式看 Evaluation 結果之前必須先理解這句話，ATT&CK 整理了許多攻擊者曾用到的 Technique，但這些 Technique 也可能會被一般人使用，例如: File and Directory Discovery (T1083) 在日常操作系統維運人員常常會用 dir/ls 這類指令。

所以當你在看各家廠商表達 ATT&CK ID，這部分有時可清楚指出明確的攻擊手法 (e.g. T1003 Credential Dumping)，有時應該當作補充資訊 (e.g. T1083 File and Directory Discovery)，這些補充資訊讓分析人員能夠快速理解告警/資料的含意。

## 有些 Technique 難以靠規則 (rule-based) 偵測

有些行為簡易的攻擊手法，可依靠一條 Log，一筆 IoC，不需額外的情境 (situation) 資訊等，也不需要上下文 (context) 即可偵測。所以 Technique 基本上可以分為兩種類型，第一類只需要簡單 IoC 就可以識別，第二類需要攻擊行為的上下文，且攻擊者有多種方式實作這個 Technique，因此某些 Technique 是很難真的能夠 100% 的保證偵測。

舉例來說，像是下列這兩個 Techniques，就不是簡單靠 Pattern/IoC 可以識別的攻擊手法，所以可以透過這種項目來觀察資安產品的能力，有武功高強的，才能精準偵測：
- Component Object Model Hijacking (T1122) — 可能被 Hijacking (劫持) 的 COM 目標元件實在是太多，而且新軟體安裝也可能做出類似的行為。傳統上偵測是否遭遇惡意 Hijacking 這件事需牽扯很複雜的邏輯判斷，像是上飛機帶尖銳物品的人不一定是要劫機，但如果拿尖銳物品靠近/威脅機組人員這就是最可能要劫機了。這種 Technique 需要上下文，了解情境才能判斷是不是 Hijacking 這件事，以這次評測各家沒有一家廠商能夠”明確”指出這件事。

- Credential Dumping (T1003) — 讀取 lsass.exe 記憶體、dump 網域控制器的 NTDS 資料庫、讀取 SAM 裡面的Credential，這些都是同個 Technique 。當越多種攻擊方式能夠做到同個 Technique，也就沒人能夠保證這事件能夠 100% 偵測。

# 評測（Evaluation)

MITRE ATT&CK 評測小組負責紅隊攻擊與驗證結果，給予正確的偵測類別標籤 (Detection Categories) 與修飾詞 (Modifiers)。最令人佩服的創舉是 MITRE 會完整揭露各廠商的測試結果與產品畫面，讓全體公眾均受益，持續進步精進。

## 評測使用到的偵測類別 (Detection Categories)

當進行測試時，ATT&CK 評測小組會給予每個測試項目 5+1 種偵測類別與 6 種測修飾詞作為該項目的測試結果。我們先討論偵測類別，屬於在現場發生的有 5 種，MSSP 則是離開評測現場後再交付給 MITRE (注意，MSSP這是可以選擇性參加，不是每一家廠商都有參與 MSSP 這個項目)。

廠商的偵測結果必須呼應到攻擊方的測試項目內容，評測小組才會給予適當的偵測類別標籤 。倘若廠商無法偵測，或偵測內容與攻擊方的測試內容不相關，則評測小組會給予 None (無法偵測，或是不認有分)。所以 ATT&CK 的評測結果上，都經過評測小組驗證，並不存在誤報這件事。

因此，廠商在評測現場能獲得的有效偵測類別共 4 種，針對 General /Tactic /Technique 這 3 種較高階的偵測類別，MITRE 會再給予 Alert (告警) 這個修飾詞標籤，下面圖例可看到有特殊紅色系列背景做區隔 ，而 Telemetry 不適用 Alert 修飾，評測小組不會給予告警(Alert)標籤做修飾。

![https://d1zq5d3dtjfcoj.cloudfront.net/round2_graphic.png](https://d1zq5d3dtjfcoj.cloudfront.net/round2_graphic.png)

為了方便大家快速理解，這邊以 [APT29 SubStep 3.B.2](https://attackevals.mitre.org/technique_comparison.html?round=APT29&step_tid=3.B.2_T1088&vendors=) 為例，以相同 SubStep 但不同的 Main Detection ，讓大家能快速理解 Main Detection 到底是什麼，請記住下列攻擊手法，以下舉例全都是使用相同手法不同廠商的偵測。

> 攻擊手法: 執行提升權限的 PowerShell payload

> 評測標準: 能夠看到 High integrity level 的 PowerShell.exe 是從 control.exe 執行的(control.exe 是從 sdclt.exe 執行的)

### Technique(偵測到攻擊手法)

自動將原始資料處理/分析過後加上描述，能夠對應到測試中攻擊步驟的 ATT &CK Technique 或是等價的描述，除了偵測到權限提升這件事還能夠描述對應的 Technique 是甚麼，這是資訊精準度最高的Detection類型。

範例:

一個 Technique Detection 標記了ATT&CK Technique “Bypass UAC” ，這個 Detection 同時關連到來源使用者執行了rcs.3aka3.doc

![https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png](https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png)

[https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png](https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png)

### Tactic (偵測到攻擊策略但不曉得確切手法)

自動將 Raw Data 處理/分析過後加上描述，能夠對應到測試中攻擊步驟的 ATT &CK Tactic 或是等價的描述，在此我們這篇文章的舉例只要能夠看到資料並且描述 TA0004, Privilege Escalation (權限提升) 即可認為是 Tactic Detection。

範例: 

一個 Tactic Detection ， 偵測到有 Privilege Escalation (權限提升) 方式執行起來的可疑 PowerShell，但並未標示具體手法 Bypass UAC。

![https://d1zq5d3dtjfcoj.cloudfront.net/Carbon%20Black-APT29-3799.png](https://d1zq5d3dtjfcoj.cloudfront.net/Carbon%20Black-APT29-3799.png)

[https://d1zq5d3dtjfcoj.cloudfront.net/Carbon Black-APT29-3799.png](https://d1zq5d3dtjfcoj.cloudfront.net/Carbon%20Black-APT29-3799.png)

### General  (偵測到可疑/惡意)

可以自動偵測到惡意的行為並且做標註，但沒指出 Tactic 或 Technique。

範例:

發現惡意的 PowerShell 並且用不同顏色的視窗標記惡意的 PowerShell

![https://d1zq5d3dtjfcoj.cloudfront.net/Cybereason-APT29-7389.png](https://d1zq5d3dtjfcoj.cloudfront.net/Cybereason-APT29-7389.png)

[https://d1zq5d3dtjfcoj.cloudfront.net/Cybereason-APT29-7389.png](https://d1zq5d3dtjfcoj.cloudfront.net/Cybereason-APT29-7389.png)

### Telemetry(遙測，紀錄原始資料)

原始資料沒有做任何加工，沒有告警，必須是當下測試的攻擊步驟所產生的相關資料。只能做簡單的Label 標記，不能有進一步的邏輯/處理。相對於Technique，Telemetry 是資訊精確度最低的 Detection 類型，因為通常是UI上的部分資料，甚至是廠商產品內部原始Log，如DB欄位、XML或是JSON。

範例： 

能夠看到 High integrity level 的 powershell 是被 control.exe 呼叫的。


![https://d1zq5d3dtjfcoj.cloudfront.net/Symantec-APT29-7397.png](https://d1zq5d3dtjfcoj.cloudfront.net/Symantec-APT29-7397.png)

[https://d1zq5d3dtjfcoj.cloudfront.net/Symantec-APT29-7397.png](https://d1zq5d3dtjfcoj.cloudfront.net/Symantec-APT29-7397.png)

### MSSP (資訊安全代管服務)

在台灣基本上就等同 SOC 的服務。只要是從 MSSP/MDR 各類監管服務送出，並且能夠指出測試過程中攻擊步驟皆為 MSSP，此項目通常會經過人工判讀，會跟資安團隊人數與人員素質有關，寫報告跟作文能力比較難以量化(廠商可以根據這次評測採用人海戰術)，因此 MSSP 這個 Detection Type 比較考驗各家廠商 MDR 團隊的服務能量與技術能力。

## 評測使用到的修飾詞 (Modifiers)

這次評測總共出現 6種修飾詞，這邊只挑重要的，常見的修飾詞來講，避免文章過長。

### Alert (有通報分析人員應注意)


在評測中 Alert 的定義是: 指出惡意行為給分析人員並且帶有分數、強調、通知等提示訊息，皆會被加上此 Modifier，並且不可以是 Telemetry (遙測)的 Modifier。

> Alert 我想可以榮登 Round2 (第二屆) 裡面最容易被誤解的 Modifier ，先來句超級讓人容易腦筋瞬間卡死的話: **Technique/Tactic/General 的 Detection 不一定帶有 Alert，但如果有 Alert 對應的一定是這三種 Detection 之一。**

也因為 Alert 這個詞，導致有些不了解 ATT&CK Evaluation 的人，有了誤解， 以為Alert 越多越不好，會有誤判、打擾使用者。

這次參賽的廠商對於 Alert 可以分為兩種類型:

1. Technique 被歸類為 Alert 的廠商: CyCraft (奧義智慧), Bitdefender, FireEye (火眼), Cylance, 與 SentinelOne
這類廠商的設計理念比較像是，不論 Technique 還是 Tactic 這些都是給予等級顯示給分析人員參考，所以在這次評測才會都被上了 Alert 這個 Modifier。
在寫這篇文章的時候做了下統計，有趣的是這類廠商反而在這次評測佔大多數。至於這些廠商如何實際重大危脅的時候告警使用者，有些廠商的做法是當 Technique 威脅等級大於一定的分數就會發訊息給客戶，其餘威脅等級低的 Alert 則依舊當作是提供分析人員補充的資訊。

2. 不是所有的 Technique/Tactic 都帶有 Alert的廠商: CrowdStrike, TrendMicro (趨勢科技), Symantec(賽門鐵克)
這類廠商的設計理念比較像是，將告警分級，將資訊充分的，通知資安人員，並產生告警；若資訊比較不充分，縱使 Telemetry 的資料裡面有 Technique/Tactic 補充訊息，沒有發出告警給資安人員。MITRE 利用 Alert 修飾來確認該偵測結果有讓資安人員注意到有駭客入侵。

![統計 21 家廠商同時做到告警偵測 Alert + Technique 的比例
](/assets/statistic_21_vendors_alerts.png)

哪種類型好並沒有一定的標準，至於如何挑選以及利用這次評測在文章的後面“該如何利用 ATT&CK Evaluations ”一節會提到。
但當你清楚了解有這兩種產品策略存在時，就不會誤解了。

### Correlated (資料能夠與攻擊者所做的其他事關聯)

這個相對好解釋，只要 Detection Type 能夠關聯到前面的攻擊步驟並讓分析人員能夠理解，就會被賦予 Correlated。這邊就用相同的圖片直接來解釋，這張圖關聯出來 sdclt.exe 是被 cmd.exe 執行的，而 cmd.exe 是被 rcs.3aka3.doc 執行的。這樣能夠關連到攻擊者先前的攻擊步驟的就是 Correlated。


![https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png](https://d1zq5d3dtjfcoj.cloudfront.net/CyCraft-APT29-2208.png)

### Configuration Change (曾現場要求更動設定)

這個 Modifier 分為兩種 UX Change、Detection Change

* UX Change：廠商在測試當下認定產品有記錄到攻擊相關資料，對評測小組提出疑義，經調整產品介面相關的設定，才得以在非標準產品(系統內部)的介面顯示出相關資料給使用者(評測小組)看，這就會被加上 Configuration Change-UX。
* Detection Change： 廠商在測試當下無法偵測，並且在知道攻擊方(MITRE) 打了什麼攻擊手法後要求重新再攻擊一次，重新測試結果後突然有了新的 Main Detection (Technique /Tactic/ Telemetry)，修改無論僅調整參數、或新增產品規則都會被歸類在此 。

對於此 Modifier 需要注意的是，評測結果雖有紀錄更動次數，但並沒有具體記錄廠商所修改/新增的設定，所以這是看評測結果的人需要多考量的事。


### Delayed(延遲)

有分為兩種，但不論哪一種都是在描述評測過程中晚收到資料。

Manuel : 工人智慧，MSSP/MDR 團隊分析後給出報告後所產生內容都會被歸類在此。

Processing: 經過機器學習分析、或 MDR 分析師處理後，分析出較複雜的邏輯而且比較晚提供給使用者會被加上 Delayed(Processing)。
# 該如何利用 ATT&CK Evaluations

來到這邊文章也要做結尾了，ATT&CK Evaluation 提供了公平的測試並且不評分，從技術的角度去截圖描述各家廠商的表現。所以能夠讓大家以同一個技術視角 (ATT&CK 駭侵框架) 去挑選最適合自己情況的產品，而不會在什麼都不了解的情況下就找了許多不適合自己的產品來做 POC。

## 挑選最適合自己的產品、廠商

那麼如何挑選最適合自己的廠商可能又要寫很長一篇，所以先就這次可以從評測中觀察出來的指標來看。

* 資安防禦策略：
這個詞含括的範圍很廣，但這邊想提的是你應該先知道自己目前比較不足的項目是在哪裡，找出你在意且有危害的 Technique/Tactic 後來去看對應的幾項評測結果是不是能夠符合你的需求，如果正面表列太難，可以反過來先刪選你不太需要的。

*事件分析能量：
如果你的資安人員很少或是能力還沒提升上來，建議先挑選產品畫面是資安人員能夠理解的。這個評測的好處就是各家廠商的畫面都出來了，讓你可以先篩選出自己認為完全看不懂的產品。會這樣說的原因是，許多廠商都有 Telemetry，但看懂這些沒有帶任何註記的資料其實是相當難以閱讀有些甚至不是給人看的。

* 資安託管成熟度：
如果 MSSP/MDR Service 是你購買產品的考量的點，建議將原廠是否能夠提供支援納入評估之中，這次考試全都是原廠提供監控服務也可以說是最了解自己產品的人去作監控，如果原廠都做不好更何況是其他人，可以將有把 MSSP 加入評測的廠商一次做比較看誰寫的最詳細或是最符合你的胃口。

## 實際舉例
這邊會舉例各種情況，但為了簡化情境複雜度，所以這邊假設的情境都會以在意相同 Technique/Tactic 來做舉例，這邊舉例較為複雜的 APT29 substep 20.B.1 (Created Kerberos Golden Ticket using Invoke-Mimikatz)，經典的 Windows Active Directory 橫向移動手法，這項較難以偵測也是 AD 被攻陷的單位之痛。

評估時請不要忘記注意 Configuration Change ，因為要考量實際使用是不是能有相同設定。
過程中強烈建議使用官方網站的工具 [Technique Comparison Tool](https://attackevals.mitre.org/technique_comparison.html?round=APT29&step_tid=20.B.1_T1097&vendors=CyCraft,CrowdStrike,Cybereason,Elastic,F-Secure,FireEye,GoSecure,Kaspersky,McAfee,Microsoft,Palo%20Alto%20Networks,Secureworks,SentinelOne,Symantec,Trend%20Micro,VMware%20Carbon%20Black)

### 舉例一：資安成熟度不高。
**我專注本業，需要倚賴 MSSP/MDR，IT/資安僅一位**

先挑出來有 MSSP/MDR 的廠商
(注意部分廠商並沒有參加 MSSP 評測: Bitdefender, Endgame, Hansight, Malwarebytes, McAfee, Reaqta 等)

篩選完畢後，可以按照這個思路去整理考慮順序:

1. 能夠明確偵測出來 Technique/Tactic  的廠商
2. 有 General  Detection 提示有可疑行為
3. 最次是 MSSP 有告警 Pass The Ticket 這行為的廠商
4. 如果完全沒有上述幾項，可以先考慮排出在清單之外

為什麼 MSSP 有告警卻排在第三順位呢? 因為這次評測各家廠商是知道攻擊方打了什麼攻擊的，相當於你知道考試要考甚麼，只要在公布後的幾天內交出正確答案與截圖就會拿到 MSSP。而能夠明確指出來 Technique/Tactic 是最優秀也證明 MSSP 是能夠看到的，至於 General 也算是可以接受的。

### 舉例二: 我是專職的分析師，給我最詳細的資料我時間很多！
**我是專職的分析師，給我最詳細的資料，我時間很多！**

單位具有足夠技術人力資源，或技術狂熱份子想要接收到每一筆超級細項的資料，個人覺得省錢其實可以考慮裝個微軟免費工具 Sysmon 就搞定了。

如果你是這種人，可以按照這個思路去整理考慮順序:

1. 整理每個 substep 各廠商的 Telemetry 的覆蓋率。
2. 你在意的 substep 各廠商能不能提供有用的 Detection 資料
3. 找自己喜歡而且能夠讀懂的產品介面

對於技術狂熱份子，第二點尤其重要。如果只是看到指令(process cmd-line, powershell cmd-line…)卻沒看到系統層級行為(API Call, file event…)，對於資料收集其實是沒意義的，駭客變化一下指令就可能看不懂了。

### 舉例三: 我是資安人員，但我同時要處理許多資安事件
**我是資安營運人員，有明確告警再通知我，我時間寶貴！**

這類的人沒辦法 24小時都在電腦前面只看 Raw Data，他們需要把時間花在其他資安維運上。

這類相較第一個舉例差異在，他是專門的資安人員並且是要處理資安事件的，但現實上時間有限，他必須專注在有效告警，沒辦法像舉例二花大量時間在看 Raw Data。

這類的情況，可以按照這個思路去整理考慮順序:
1. 先找出可提供 MSSP/MDR 的廠商(可參閱舉例一)
2. 找出能夠標出較多 Technique/Tactic 的廠商，這項可以大大減少閱讀 Raw Data 的時間
3. 你在意的 substep 這些廠商有沒有標記 Technique/Tactic/General
4. 找自己喜歡而且能夠讀懂的產品介面

# 結語

ATT&CK Evaluation 不只是讓所有廠商的產品攤開來，讓我們了解哪樣的產品可能是適合自己的。同時也是讓各家廠商進入一個類競賽的環境能夠持續進步，可以看到有些廠商有持續進步而，有些廠商故步自封。

但也因為這樣的測試方式，廠商有許多不同的角度去計算凸顯自己的強項，雖見第二輪的標籤分類尚有不完美之處，但相較第一輪的標籤方式已是大大進步的。

老實說，21大門派資安防禦的切入點都不完全相同，因此各產品設計重點也各有強項，青菜蘿蔔本來就各有所好，攤在一起測試本來就不盡公平，但是 MITRE 這樣以紅藍隊情境測試方式，應該是業界目前公認，不公平中最公平的了，更何況人家是 MITRE，而你是甚麼蔥。寫這篇文章是希望能夠寫一篇技術文章讓大家能夠理解這次評測，也希望甲方在挑選產品時能夠更輕鬆不被迷惑，乙方也能夠省些時間減少進入 PoC 大亂鬥的次數。希望文明的進步，能夠革除鬥蟋蟀的陋習 !

---

延伸閱讀:

[Using ATT&CK for CTI Training  MITRE ATT&CK®](https://attack.mitre.org/resources/training/cti/)

[ATT&CK Evaluations: Understanding the Newly Released APT29 Results](https://medium.com/mitre-attack/attack-apt29-results-released-cd30b3686ad9)

[Quantifying the MITRE ATT&CK Round 2 Evaluation](https://medium.com/@jonathan.ticknor1/quantifying-the-mitre-att-ck-round-2-evaluation-fa9c39765a26)

參考資料:

[MITRE ATT&CK® EVALUATIONS](https://attackevals.mitre.org/APT29/detection-categories.html)

[MITRE ATT&CK® EVALUATIONS](https://attackevals.mitre.org/technique_comparison.html?round=APT29&step_tid=3.B.2_T1088&vendors=CyCraft,Bitdefender,CrowdStrike,Cybereason,Elastic,F-Secure,FireEye,GoSecure,HanSight,Kaspersky,Malwarebytes,McAfee,Microsoft,Palo%20Alto%20Networks,ReaQta,Secureworks,SentinelOne,Symantec,Trend%20Micro,VMware%20Carbon%20Black)
