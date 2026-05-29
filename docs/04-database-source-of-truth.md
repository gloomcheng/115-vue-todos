# 教學 04：從 localStorage 走到 Database

## 這一段課程的目標

前面我們已經做到：

1. 把 TODO UI 做到接近真實複雜度
2. 用單一職責拆 component
3. 用 `localStorage` 讓 refresh 後資料留下來

這一週開始要進入新的階段：

> 把資料從「瀏覽器自己的保存區」搬到「應用程式真正的資料來源」。

也就是 database。

這一段先不要急著寫 DB code。第一段課程要先讓學生理解：

- 為什麼 `localStorage` 不夠
- database 解決的是什麼問題
- 前端為什麼不直接碰 database
- API 在前端和 DB 中間扮演什麼角色
- state management 會因為 DB 出現而怎麼改變

## 開場：先回顧 localStorage

可以先問學生：

> 上週我們解決了什麼問題？

答案是：

> refresh 後資料不見。

我們的做法是：

```txt
Vue state -> watch -> localStorage
```

也就是：

- Vue state 改變
- `watch` 監聽 `todos`
- 把最新 todos 用 `JSON.stringify` 存進 `localStorage`
- App 重新載入時，再從 `localStorage` 讀回來

這個解法適合教學，因為它很直覺。

但是它不是正式產品完整解法。

## localStorage 的限制

這裡要讓學生自己感覺到問題。

可以這樣問：

> 如果我換一台電腦，TODO 還在嗎？

不在。

> 如果我換一個瀏覽器，TODO 還在嗎？

不在。

> 如果我清除瀏覽器資料，TODO 還在嗎？

不在。

> 如果我要讓同一組 TODO 給同學或同事一起看，可以嗎？

不行。

所以 `localStorage` 的限制是：

- 只存在同一台裝置
- 只存在同一個瀏覽器
- 清除瀏覽器資料就會消失
- 沒有登入使用者概念
- 沒有多人共享
- 沒有權限
- 不適合做備份
- 不適合跨裝置同步

這時候可以給一句話：

> localStorage 是瀏覽器自己的小抽屜，不是應用程式真正的資料庫。

## Database 解決什麼問題？

database 的角色是：

> 讓資料離開單一瀏覽器，成為系統共同承認的資料來源。

也就是說，TODO 不再只是「我這個瀏覽器上的資料」，而是「這個應用程式裡的資料」。

database 可以支援：

- 跨瀏覽器
- 跨裝置
- 使用者登入
- 多人共享
- 權限控制
- 備份
- 查詢
- 排序
- 分頁
- 資料關聯

這裡可以先不要講太多 SQL。重點是讓學生理解資料位置改變了。

## Source of Truth 的改變

localStorage 版本裡，前端幾乎就是資料來源。

```txt
App.vue todos
  -> localStorage
```

database 版本裡，真正的資料來源會變成 DB。

```txt
Database
  -> Backend API
  -> Vue App state
  -> Components
```

這裡要講一個很重要的觀念：

> 前端 state 是畫面目前拿到的一份資料；database 才是系統真正保存的資料。

所以未來 `todos` state 的角色會變成：

- App 啟動時，從 API 拿資料
- 使用者操作時，呼叫 API 修改 DB
- API 成功後，更新前端畫面

## 為什麼前端不直接連 DB？

學生常會問：

> Vue 可不可以直接連 database？

可以用這樣回答：

> 技術上某些情境可以，但一般 web app 不會讓前端直接碰 database。

原因是：

- DB 帳號密碼不能放在瀏覽器
- 瀏覽器程式碼使用者都看得到
- 權限檢查應該在後端
- 資料驗證應該在後端
- DB schema 不應該暴露給前端直接操作
- 多個 client 需要一致的 business rules

所以我們會加一層 backend API。

```txt
Vue
  -> HTTP API
  -> Backend service
  -> Database
```

教學句：

> 前端負責使用者互動，後端負責資料規則和資料保存。

## TODO App 的 API 設計

先從 CRUD 開始，不要一開始做太複雜。

可以設計成：

```txt
GET    /todos
POST   /todos
PATCH  /todos/:id
DELETE /todos/:id
```

每個 API 對應目前前端已經有的操作。

| 使用者操作 | 現在的前端 function | 之後對應 API |
| --- | --- | --- |
| 載入頁面 | `loadTodos()` | `GET /todos` |
| 新增 TODO | `addTodo(draft)` | `POST /todos` |
| 完成 / 取消完成 | `toggleTodo(id)` | `PATCH /todos/:id` |
| 刪除 TODO | `deleteTodo(id)` | `DELETE /todos/:id` |
| 清除完成項目 | `clearCompletedTodos()` | 多次 `DELETE` 或 batch API |

這樣學生會知道：

> API 不是突然冒出來的，它是目前前端行為的遠端版本。

## DB Schema 初稿

可以先用一張簡單的 `todos` table：

```sql
todos
- id
- title
- description
- completed
- priority
- category
- due_date
- created_at
- updated_at
```

對應到現在前端的 `Todo` type：

```ts
export type Todo = {
  id: number
  title: string
  description: string
  completed: boolean
  priority: Priority
  category: string
  dueDate: string
  createdAt: string
}
```

這裡可以讓學生比較命名差異：

- 前端常用 camelCase：`dueDate`
- DB 常用 snake_case：`due_date`

這會帶出一個實務問題：

> API 要不要負責轉換資料格式？

第一階段可以先保持簡單，但要讓學生知道這是之後會遇到的設計點。

## State Management 會怎麼變？

localStorage 版本：

```ts
const todos = ref<Todo[]>(loadTodos())

watch(todos, (newTodos) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(newTodos))
})
```

database 版本會變成：

```txt
onMounted
  -> fetch todos from API
  -> set todos.value

addTodo
  -> POST API
  -> update todos.value

toggleTodo
  -> PATCH API
  -> update todos.value

deleteTodo
  -> DELETE API
  -> update todos.value
```

所以 state management 會多出幾個狀態：

- `isLoading`
- `error`
- `isSaving`
- API 成功
- API 失敗
- 是否重試

這是從本機 state 走向遠端資料一定會遇到的問題。

教學句：

> 一旦資料在遠端，state management 就不只是資料本身，還包含 loading、error 和同步狀態。

## 第一段不要做 optimistic update

第一段先用簡單、穩定、好理解的方式：

```txt
使用者操作
  -> 呼叫 API
  -> API 成功
  -> 更新畫面
```

先不要一開始就講 optimistic update。

因為 optimistic update 會多出 rollback、失敗處理、同步衝突，對第一段太早。

這一段先讓學生理解：

> 資料從 localStorage 搬到 DB 後，所有修改都變成 async。

## 課堂口語版

可以這樣講：

> 上週我們把資料存進 localStorage，所以 refresh 後資料還在。
>
> 但 localStorage 只是在這台電腦、這個瀏覽器裡留下資料。
>
> 如果這個 TODO App 變成真正產品，使用者會期待換電腦也看得到、登入後也看得到、團隊成員也可以共享。
>
> 這時候資料就不能只放在瀏覽器裡，而要放到 database。
>
> 所以今天開始，我們要把 TODO App 的資料來源從 localStorage 改成 database。
>
> 但前端不會直接連 DB。前端會透過 API 跟後端說：「我要資料」、「我要新增」、「我要修改」、「我要刪除」。
>
> database 是真正保存資料的地方，Vue state 是畫面目前拿到的一份資料。

## 第一段課堂練習

先不寫完整後端，可以先請學生做設計練習：

1. 找出目前有哪些操作會改變 todos
2. 把每個操作對應到一個 API endpoint
3. 寫出 todos table 需要哪些欄位
4. 討論哪些 state 是前端 UI state，哪些 state 是 DB data

範例答案：

```txt
UI state:
- search
- statusFilter
- priorityFilter
- categoryFilter
- hideCompleted

DB data:
- todos
- todo.title
- todo.description
- todo.completed
- todo.priority
- todo.category
- todo.dueDate
```

這裡要強調：

> 不是所有 state 都要存進 database。

搜尋字串、目前選到哪個 filter，通常只是畫面狀態。  
TODO 本身才是需要長期保存的資料。

## 這一段的 commit 目標

這個 commit 只新增課程說明，不做程式實作。

建議 commit message：

```txt
docs: add database source of truth lesson
```

下一段再開始做 API client 或 mock API。
