+++
date = "2026-04-21T10:00:00+05:30"
title = "AdminJS Action / Guard / 自訂 component 筆記"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "AdminJS,NestJS,Admin"
+++

## 一句話

AdminJS 是 NestJS / Node.js 的 Admin Panel 框架,資料來源相容 TypeORM、Mongoose、Prisma 等。核心是**用資料模型自動生成 CRUD UI**,再用 action 擴充。

## Action 類型

| 類型               | 作用對象           | 情境 |
|--------------------|--------------------|------|
| Record action      | 單筆記錄           | 編輯、手動觸發 |
| Resource action    | 整個 resource      | 匯出 CSV、批次匯入 |
| Bulk action        | 選中的多筆         | 批次改狀態 |

## 常用 hook 與參數

```ts
actions: {
  approve: {
    actionType: 'record',
    guard: 'Confirm?',                         // 前端會跳確認框
    component: false,                          // 不進自訂畫面,直接送 request
    handler: async (request, response, context) => {
      // 1) context.record 已 parsed
      // 2) 在 before 用 throw 可以直接中止並彈錯
      if (context.record.params.status === 'done') {
        throw new Error('Already approved');
      }
      await service.approve(context.record.params.id);
      return {
        record: context.record.toJSON(context.currentAdmin),
        notice: { message: 'Approved', type: 'success' },
      };
    },
    after: async (response) => { /* 改顯示 */ return response; },
  }
}
```

## 自訂前端 component

- 可 import AdminJS 內建 UI kit(`@adminjs/design-system`)。
- 在 component 裡用 `api.resourceAction` 呼叫同 app 的其他 resource action,帶上 query / params。
- 讓 bulk action 結束後 `api.refreshList()` 重刷表而不整頁 reload。

## Filter / List 的 UX 小撇步

- `closeFilter` 選項在 action 完成後自動收合篩選欄,避免表格被擠到畫面底。
- `showInDrawer: true` 把 form 放 side drawer,比 modal 順。
- 單筆 row 若不想讓人點進 show 畫面 → `showFilter: false` 配合 `isVisible` 做遮罩。

## 常踩到

- Mongoose populate + AdminJS:關聯欄位要在 `properties` 裡註冊成 `reference`,不然只會顯示 ObjectId 字串。
- 自訂 component 的 React 版本要和 AdminJS 指定版本一致,否則 hooks 亂掉。
