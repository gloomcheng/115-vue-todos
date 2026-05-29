# 07：資料庫檔案為什麼要放進 .gitignore

## 你會學到什麼

這一章只學一件事：

> 持久化資料庫保存的是資料；資料庫檔案不應該隨便 commit 到 git。

上一章我們已經把 browser SQLite 改成持久化：

```txt
SQLite database
  -> db.export()
  -> IndexedDB
```

這代表 refresh 後資料仍然存在。

這章接著補一個重要觀念：

> 一旦資料會被保存，就要開始思考資料安全。

## 目前 browser SQLite 會產生檔案嗎？

不會。

目前資料存在瀏覽器的 IndexedDB，不會在專案資料夾產生：

```txt
dev.db
todos.sqlite
```

所以你在 repo 裡不會看到一個 SQLite 檔案。

但是我們仍然要在 `.gitignore` 補上：

```gitignore
*.db
*.sqlite
*.sqlite3
```

原因是：

> 這不是為了現在的 IndexedDB，而是為了建立正確習慣：database file 是資料，不是 source code。

## 為什麼資料庫檔案不該 commit？

資料庫檔案可能包含：

- 使用者輸入的 TODO
- 個人筆記
- 測試帳號
- email
- token
- 內部資料
- 不該公開的 demo data

即使你覺得只是課堂練習，也可能不小心把真實資料打進去。

如果 `.db` 檔被 commit，再 push 到 GitHub，就會變成：

```txt
本機資料
  -> git commit
  -> GitHub
  -> 其他人可下載
```

這就是資安問題。

## source code 和 runtime data 的差別

可以 commit 的通常是 source code：

```txt
src/App.vue
src/lib/todoSqlite.ts
docs/*.md
package.json
```

不應該 commit 的通常是 runtime data：

```txt
dev.db
todos.sqlite
local.sqlite3
```

原因：

> source code 是大家一起維護的程式；runtime data 是每個環境自己產生的資料。

## 如果要提供範例資料怎麼辦？

不要直接 commit 自己本機的 `.db`。

比較好的做法是 commit：

```txt
schema.sql
seed.sql
seed.ts
migrations/
```

也就是：

- schema 描述資料表長什麼樣
- seed script 產生範例資料
- migration 描述資料庫如何升級

這樣同學可以自己產生資料庫，而不是共用你的本機資料庫檔案。

## 本章操作

打開 `.gitignore`，確認有：

```gitignore
# Local SQLite database files
*.db
*.sqlite
*.sqlite3
```

這代表：

> 如果專案資料夾未來產生 SQLite 檔案，git 會忽略它。

## 檢查自己是否理解

請回答：

1. 目前 browser SQLite 的資料存在哪裡？
2. 為什麼目前不會產生 `dev.db`？
3. 為什麼仍然要加 `.gitignore`？
4. database file 可能包含哪些敏感資料？
5. 如果要分享範例資料，為什麼 seed script 比 `.db` 檔更好？

## 本章重點

持久化資料庫解決的是：

> 資料 refresh 後要留下來。

`.gitignore` 解決的是：

> 本機資料庫檔案不要被誤提交、誤分享。
