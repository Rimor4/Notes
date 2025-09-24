 ![[Pasted image 20231205211319.png]]

![[Pasted image 20231205212826.png]]


编写一个 SQL 查询来列出 Bradley Cooper 和 Jennifer Lawrence 主演的所有电影的标题(AS 可以省略)
```sql
SELECT m1.title
  FROM movies AS m1
  JOIN stars AS s1 ON m1.id = s1.movie_id
  JOIN people AS p1 ON p1.id = s1.person_id
  JOIN stars AS s2 ON m1.id = s2.movie_id
  JOIN people AS p2 ON p2.id = s2.person_id
 WHERE p1.name = "Johnny Depp"
   AND p2.name = "Helena Bonham Carter";
```

编写一个 SQL 查询来列出主演凯文·培根也主演的电影的所有人员的姓名
凯文·培根本人不应包含在最终的名单中
```sql
SELECT DISTINCT p.name
  FROM people AS p
  JOIN stars AS s1 ON p.id = s1.person_id        # 获得了所有主演的信息
  JOIN stars AS s2 ON s1.movie_id = s2.movie_id  # 获得所有在同一部电影中的主演信息
  JOIN people AS kb ON s2.person_id = kb.id      # 连接了在同一部电影中主演的人员的信息
 WHERE p.name != 'Kevin Bacon'
   AND kb.name = 'Kevin Bacon'
   AND kb.birth = 1958;
```



SQLite 支持 **ALTER** TABLE 的有限子集。SQLite 中的 ALTER TABLE 命令允许对现有表进行以下更改：可以<u>重命名</u>；可以<u>重命名列</u>；可以向其中<u>添加一列</u>；或者可以从中<u>删除一列</u>。
```sql
-- 创建新表
CREATE TABLE new_table (
    existing_column1 INTEGER,
    existing_column2 TEXT,
    new_column_name INTEGER
);

-- 将数据从旧表复制到新表，同时更改列名
INSERT INTO new_table (existing_column1, existing_column2, new_column_name)
SELECT existing_column1, existing_column2, old_column_name
FROM your_table_name;

-- 删除旧表
DROP TABLE your_table_name;

-- 将新表重命名为旧表的名字
ALTER TABLE new_table RENAME TO your_table_name;

-- 更改列名
ALTER TABLE your_table_name
RENAME COLUMN old_column_name TO new_column_name;

```
**insert or replace**:如果不存在就插入，存在就更新
**insert or ignore**:如果不存在就插入，存在就忽略