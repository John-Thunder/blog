---
title: Fully Disable Windows Defender
---

# 完全關閉 Windows Defender / Fully Disable Windows Defender

**注意！不建議關閉防毒軟體，建議只在實驗環境中執行**

**注意！不建議關閉防毒軟體，建議只在實驗環境中執行**

**注意！不建議關閉防毒軟體，建議只在實驗環境中執行**

網路上的教學大部分都是透過修改 registry 註冊表關閉 Windows Defender 或是去 Windows Security 中暫時關閉。

但這些方法往往都不是最有效的方式，在最新版可能無效/重開機就又回來了。

本篇文章是關於如何在 Windows 10 完全關閉 Defender，分為兩部分：
* 關閉主要功能 (e.g. 即時掃描)
* 關閉排程掃描

## 關閉 Windows Defender 主要功能


以下是在 Version 21H1 (OS build  19043) 有用的方式
1. 請先關閉防竄改保護，在防竄改保護啟用的狀態，Windows Defender 是無法完全關閉的。
	
	在設定中的「更新與安全性」→「Windows Defender」
	![](/assets/Disable-TamperProtection.png)
2. 透過執行(快捷鍵: WIN+R)，輸入 gpedit.msc 開啟群組原則編輯器，只要是 Windows 10 專業版/企業版都有，家用版沒有 gpedit.msc，如果找不到 gpedit.msc 請確認系統版本。
3. 將本機電腦原則展開至：
    中文系統: **電腦設定** → **系統管理範本** → **Windows 元件** → **Microsoft** **Defender** **Antivirus**
	英文版: **Computer Configuration** → **Administrative** **Templates** → **Windows** **Components** → **Microsoft** **Defender** **Antivirus**
	> 由於微軟換過 Defender 全名，所以在不同系統可能是 Windows Defender
	
4. 在右邊視窗找到 **關閉Windows Defender** 或是 **Turn Off Windows Defender** 

5. 點選兩下啟用編輯並選擇啟用並套用設定，如下圖：
	![](/assets/Defender-GPO.png)
6. **進行重開機，之後就完全關閉了**


##  關閉 Windows Defender 排程掃描
完全關閉 Windows Defender 之後系統固定排程掃描，如果你的系統有定期更新病毒碼，可能導致你的實驗環境中的檔案莫名遭到刪除/隔離。

**注意！不建議日常關閉防毒軟體，只建議在實驗環境中執行**

1. 透過執行(快捷鍵: WIN+R)，輸入 taskschd.msc 
2. 展開左側 **Task Scheduler Library** > **Windows** > **Windows Defender** ，右鍵選擇 disable(停用)
	![](/assets/DisableDefenderScheduleScan.png)
