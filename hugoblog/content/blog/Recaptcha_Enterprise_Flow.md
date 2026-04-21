+++
date = "2026-04-21T11:50:00+05:30"
title = "Google reCAPTCHA Enterprise / v3 前後端驗證流程"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "reCAPTCHA,Security,Web"
+++

## 申請

1. <https://www.google.com/recaptcha/about/> 建立專案。
2. 選 **reCAPTCHA Enterprise** 或 v3(大部分場景選 v3 無 UI challenge)。
3. 設定**通行網域白名單**:填 prod 與 dev/staging 網域;未列在名單的 domain token 產生會失敗。
4. 取得兩把 key:
   - `site key`(前端用,可公開)→ `__RECAPTCHA_SITE_KEY__`
   - `secret key`(後端用,**不能外洩**)→ `__RECAPTCHA_SECRET_KEY__`

## 前端產 token(Enterprise)

```html
<script src="https://www.google.com/recaptcha/enterprise.js?render=__RECAPTCHA_SITE_KEY__"></script>
<script>
grecaptcha.enterprise.ready(async () => {
  const token = await grecaptcha.enterprise.execute(
    '__RECAPTCHA_SITE_KEY__',
    { action: 'LOGIN' }                  // 自己定義,後端驗證會比對
  );
  // 把 token 帶進 login 請求
});
</script>
```

## 後端驗證

```http
POST https://www.google.com/recaptcha/api/siteverify
Content-Type: application/x-www-form-urlencoded

secret=__RECAPTCHA_SECRET_KEY__&response=<token>
```

成功:

```json
{ "success": true, "challenge_ts": "...", "hostname": "...", "score": 0.9, "action": "LOGIN" }
```

失敗(常見):

```json
{ "success": false, "error-codes": ["timeout-or-duplicate"] }
```

## 關鍵行為

- **Token 只能驗一次**:第二次呼叫 siteverify 會拿到 `timeout-or-duplicate`。所以後端別重試 verify,要重試就讓前端重新 `execute()`。
- Token 有時效(約 2 分鐘),前端產生後要儘快送到後端驗。
- `score` 越高越像人類(v3 / Enterprise),業務邏輯自己訂門檻(常見 0.5 或 0.7)。
- `action` 不比對也能過 `success: true`,但**建議嚴格比對**,避免 token replay 跨 action。

## Enterprise vs 免費 v3

- Enterprise 要綁 Google Cloud 專案,計費但分析能力更完整(risk analysis API)。
- 免費版 v3 適合低流量、單純驗證「像不像真人」。
