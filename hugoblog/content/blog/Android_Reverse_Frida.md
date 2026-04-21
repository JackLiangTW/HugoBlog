+++
date = "2026-04-21T10:10:00+05:30"
title = "Android 逆向入門:Frida + Magisk + LSPosed 工具分工"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Android,逆向,Frida,Magisk"
+++

> **重要提醒**
>
> 本文介紹的工具與手法屬於**資安 / 逆向測試**領域,僅限用於:
>
> - 自己或客戶**授權**的 App(如 CTF、滲透測試、自家產品強化)
> - 學習用的公開教材、練習 App(例如 `InsecureBankv2`、`DIVA`、OWASP MSTG 測試 App)
>
> 操作時請注意:
>
> - 未經授權對他人 App 進行逆向、繞過保護、封包竄改,可能觸犯當地法律(著作權法、個資保護法、妨害電腦使用罪…),請遵守當地法規
> - 執行請在隔離的測試裝置 / 模擬器,**不要拿主力手機或個人銀行 App**
> - **本文內容僅供技術交流,風險自負**,作者不為讀者依此操作所致之法律、財務、裝置損失負責

## 工具分工

| 工具             | 角色 |
|------------------|------|
| **apktool**      | 靜態拆 APK:AndroidManifest、smali、resources |
| **Jadx**         | 反編譯 dex → 可讀 Java;入門必備 |
| **JEB Decompiler** | 商用 Jadx 強化版(smali → Java 還原更好) |
| **Frida**        | 動態 hook:attach 到執行中的 process 改行為 |
| **Magisk**       | 取得 root,並提供模組系統 |
| **LSPosed**      | Magisk 上的 Xposed 實作,Java/Kotlin hook |
| **Shamiko**      | 在 Magisk 上隱藏 root,逃過 SafetyNet |

## Frida 常用指令

前置:裝置要有 **frida-server** 在跑(root 裝置),App 要開著在前景。

```bash
# 監聽所有 onClick 事件,看點擊哪個 class
frida-trace -U -F -j '*!onClick'

# 監聽某 package 所有方法
frida-trace -U -F -j 'com.paymaya.common.base*!*'

# attach 到執行中的特定 package(不 spawn)
frida -U -F -l hook.js

# spawn 並 early-attach
frida -U -f com.target.app -l hook.js --no-pause
```

寫個簡單 hook(`hook.js`):

```js
Java.perform(() => {
  const MainActivity = Java.use('com.target.MainActivity');
  MainActivity.checkLicense.implementation = function () {
    console.log('[+] bypass checkLicense');
    return true;           // 永遠回 true
  };
});
```

## 入門學習資源

- 胡立《安卓 APK 反編譯系列》— <https://blog.huli.tw/2023/04/27/android-apk-decompile-intro-1/>
- r0ysue《Frida SO 逆向實戰》GitHub:<https://github.com/r0ysue/FridaAndrioidNativeBeginnersBook>
- Android 逆向工具總覽:<https://github.com/WuFengXue/android-reverse>
- Magisk Trust User Certs / JustTrustMe 原理:<https://uint128.com/2021/11/03/>
- Frida 自動化技巧:<https://blog.csdn.net/pdcfighting/article/details/125419046>

## SSL Pinning 繞過 3 常見手法

1. **apktool + network_security_config**:改 `AndroidManifest` + `res/xml/network_security_config.xml` 讓 App 信任 user CA;重新簽名安裝。
2. **Magisk 模組 Move Certificates**:把使用者 CA 移到系統 CA,全 App 生效,不用改每個 App。
3. **Frida Hook**:動態把 `checkServerTrusted` / `verify` 改成 return true;`JustTrustMe` 或 `frida-android-unpinning` 腳本最常用。

## Magisk / LSPosed / Shamiko 安裝順序

1. 先 root 裝置(rootAVD 或官方 boot.img patch)。
2. Magisk App → Modules → 裝 **LSPosed**(Zygisk 版)→ reboot。
3. 模組被 Play Protect 偵測時,裝 **Shamiko**:Magisk Settings 開啟 Zygisk + DenyList → 把要藏 root 的 App 加進來 → Shamiko 會自動套用。
4. 某些金融 / 遊戲 App 仍會被抓,嘗試關 Shamiko 的 whitelist(藏全域)或降版。

## 常見風險提醒

- 逆向自己或客戶授權的 App;他人 App 請遵守當地法律。
- root / 模組組合常會造成 SafetyNet / Play Integrity 失效,影響支付、銀行類 App 正常使用。
- Frida 的 payload 可能被 App 反 frida 檢測(`libfrida-agent.so` pattern),要改名或用 **stalker / emulated** mode 迴避。
