# 教學 05：前端 scope 裡的 SQLite 第一課

## 這一課只教兩個學習點

這堂課不要一次講太多。

只教：

1. `localStorage` 是 key-value，SQLite 是 table
2. Vue state 可以從 table 查出來，而不是從一包 JSON 讀出來

先不要講：

- 後端 API
- 登入
- server database
- vector database
- optimistic update
- repository pattern
- migration
- sync conflict
- Pinia
- ORM

這堂課的目標不是把 TODO App 完整改成 SQLite，而是讓學生先理解：

> 資料可以從「一包 JSON」變成「一張表」。

## 為什麼這還是前端課？

這裡使用 SQLite，不是要把課程變成後端課。

我們先把 SQLite 當成「瀏覽器裡的本機資料庫」來理解。

資料流仍然是前端 scope：

```txt
Vue component
  -> Vue state
  -> SQLite in browser
```

沒有 server。

沒有登入。

沒有 API。

只是把保存資料的方式，從 `localStorage` 換成 SQLite。

## 學習點一：localStorage 是 key-value

上一堂課的保存方式大概是：

```ts
localStorage.setItem('vue-todos', JSON.stringify(todos.value))
```

可以畫成：

```txt
localStorage

key        | value
-----------|-------------------------
vue-todos  | "[{...}, {...}, {...}]"
```

這代表整份 TODO list 被轉成一個字串，放在一個 key 裡。

這很適合第一階段教學，因為很直覺。

但它也有明顯限制：

- 只能存字串
- 每次都像是在存一整包 JSON
- 沒有 table 概念
- 沒有 schema 概念
- 不適合練習資料查詢

教學句：

> localStorage 像一個貼標籤的小抽屜：給一個 key，放一個字串。

## 學習點二：SQLite 是 table

SQLite 會把資料放進 table。

TODO 資料可以想成：

```txt
todos table

id | title          | completed | priority | category
---|----------------|-----------|----------|---------
1  | Learn Vue      | 0         | high     | Course
2  | Add validation | 0         | medium   | Course
3  | Split component| 1         | high     | Teaching
```

這時候資料不再是一整包字串，而是一列一列的 row。

教學句：

> localStorage 存一包字串；SQLite 把資料放進表格。

## 從 Todo type 對應到 table

先看前端現在的 TypeScript type：

```ts
type Todo = {
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

這是前端資料形狀。

SQLite 裡可以對應成：

```sql
CREATE TABLE todos (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  completed INTEGER NOT NULL,
  priority TEXT NOT NULL,
  category TEXT NOT NULL,
  due_date TEXT NOT NULL,
  created_at TEXT NOT NULL
);
```

這是資料保存形狀。

這裡先不要講很深的 SQL 型別，只要讓學生理解：

- `number` 大致對應 `INTEGER`
- `string` 大致對應 `TEXT`
- `boolean` 在 SQLite 常用 `0` / `1`
- 前端 `dueDate` 到 DB 可能會變成 `due_date`

教學句：

> TypeScript type 是前端怎麼看資料；SQLite table 是資料怎麼被保存。

## 這堂課只示範 SELECT

這堂課不要一開始就做完整 CRUD。

只示範讀取：

```sql
SELECT * FROM todos;
```

然後把查到的 rows 放回 Vue state：

```ts
const todos = ref<Todo[]>([])

const loadTodos = async () => {
  todos.value = await listTodosFromSqlite()
}
```

重點不是 `listTodosFromSqlite` 裡面完整怎麼寫。

重點是這個觀念：

```txt
SQLite table
  -> SELECT
  -> rows
  -> Vue state
  -> template render
```

教學句：

> Vue 畫面仍然吃 todos state，只是 todos 的來源從 localStorage 換成 SQLite 查詢結果。

## 課堂口語版

可以這樣講：

> 上週我們把 todos 存成一包 JSON，放進 localStorage。
>
> 今天我們先不碰後端，也不碰 API。
>
> 我們只做一個觀念轉換：同樣是前端本機資料，能不能不要只存一包字串，而是把它放進一張 table？
>
> localStorage 像小抽屜，SQLite 像表格。
>
> Vue component 不需要知道底下是 localStorage 還是 SQLite。畫面最後都還是吃 `todos` state。
>
> 差別是：這次 `todos` 不是從 JSON parse 出來，而是從 table SELECT 出來。

## 課堂練習

請學生完成三件事，不寫完整 App：

1. 把 `Todo` type 畫成 table
2. 寫出 `CREATE TABLE todos`
3. 寫出 `SELECT * FROM todos`

加問：

> 哪些欄位應該進 table？哪些不應該？

應該進 table：

- title
- description
- completed
- priority
- category
- dueDate
- createdAt

不應該進 table：

- search
- statusFilter
- priorityFilter
- categoryFilter
- hideCompleted

原因：

> TODO 本身是資料；搜尋和篩選只是目前畫面怎麼看資料。

## 這堂課的結尾

用這句收：

> 今天先不要追求完整 CRUD。我們只要理解：資料可以從一包 JSON，變成一張 table。前端還是用 state 畫畫面，但 state 的來源可以換掉。

下一堂課再做：

- INSERT
- UPDATE
- DELETE
