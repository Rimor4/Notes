
> 可用debug发现： <span style="background:#fff88f">逻辑错误</span>
> 难用/不可用debug发现（算法错误）：<span style="background:#ff4d4f">逻辑错误</span>
### Credit
1.  **`if{}    else if{}    else{}`** 代替 `if{}   if{}   else{}`
2. if else 情况考虑完整
### Readability
1. **初始化sum = 0**;
2. <span style="background:#fff88f">逻辑错误</span> （句子结束时无空格但是words++）
### Wordle
1. **可变大小的数组无法初始化**`int status[wordsize] = {0}`~~错误~~
2. do while 语句中**无法定义变量**
### Tide_man
1. <span style="background:#ff4d4f">逻辑错误</span>（1）rank1: Alice;  rank2: Bob;  rank3: Charlie    则preferences: 1>2; 1>3; 2>3
2. <span style="background:#ff4d4f">逻辑错误</span>（2）locked【j】【i】
### Volume
1. while(fread(   )) 将把之后要读取的文件内容提前读取（即**while循环在fread读取到\0时才结束**）
### Filter-more
1. **round函数接受浮点数**，输出四舍五入的整数
2. <span style="background:#fff88f">逻辑错误</span> [Sobel 算子](https://en.wikipedia.org/wiki/Sobel_operator)应用于图像时，在遍历计算Gx，Gy时就<span style="background:#d3f8b6">迭代</span>了image中的像素，导致后续算子计算所用原像素错误。
3. <span style="background:#fff88f">逻辑错误</span> 计算Gx，Gy时内核数组的<span style="background:#d3f8b6">下标</span>{ <font color="#4bacc6">不一定非得用二维数组</font> }不会超过2！）与图像像素下标不同
4. **相似的变量**GxRed GxGreen GxBlue打错   【解决办法：**搜索**】

### Recover
1. <span style="background:#d4b106">逻辑障碍</span> 先读取块，再找jpg

### Speller
1. malloc 分配内存时 需要确保为字符串的**结尾符`\0`分配一个字节**的内存空间;     `char *new_word = malloc((LENGTH + 1) * sizeof(char));`
2. free( )是安全的，不需要 free 后检查指针为NULL (free的是内存，不会把指针指向null) ; 

### Mario.py
1. print函数 同一行打印要加参数 **end=""**
### DNA.py
1. 调用csv文件的列名： **reader.fieldnames**  （本质是字符串列表）
```python
 STR = []
    for i in range(1, len(reader.fieldnames)): # fieldnames表示csv文件的列名
        STR.append(longest_match(seq, reader.fieldnames[i]))
```
2. 调用读取的csv字典列表 (data) ：
```python
data[i][reader.fieldnames[j]] """i是第i行（第i个字典）fieldnames[j] 是j列的键 """
```

### Fiftyville.sql
1. "IN"  不要错用成  "="


### Finance
1. return render_template（）要与if method = "POST"同级
 因为默认get路径，先进入else ：  return render_template才能在html中进入post
2. **generate_password_hash(password) 传入的参数不带引号**（password已经是字符串，不需要再加引号）
3. 大胆用终端创建TABLE：保存了在计算机 .db文件中
4. 处理好if elif else 父子关系
5.  **变量名避免与函数名相同**！
6. flask 可在html中使用 {% for... %}来输出多行表格
7. lookup函数返回的是字典，而不是一个price值
8. Python字典中**没有的键不能引用**
9.  sqlite中没有"INDEX BY"语法（不需要引用索引）
10. 同一个路由下， **GET与POST中的变量不能共用**（共用变量声明时放在`if request.method == "POST"` 前
11. 
```python
symbols = db.execute(
        "SELECT DISTINCT symbol FROM transactions WHERE user_id = ?",                 session["user_id"]
    )   # 返回字典列表！
    ...
    render_template("sell.index",symbols=symbols)
    ...
    {% for symbol in symbols %}
    <option value="{{ symbol['symbol'] }}">{{ symbol['symbol'] }}</option>
    {% endfor %}
```
sell_symbol尽管只有一个值，但还是字典，注意对于字典的取值，避免传递的是==字典字符串==
也可以将**字典列表**：
```python
    symbols = [symbol['symbol'] for symbol in symbols] # 后面的symbols是原symbols
```
变成**字符串列表**
12. SQL创建TABLE中的外键约束时写错外表名称（users->user）导致main.user不存在
13. Python中使用  `?` 占位符插入sql表格时，**参数不加括号**
```python
	db.execute(
                "INSERT INTO SELL (user_id, symbol, shares, price, sell_time)                 VALUES (?, ?, ?, ?, ?)",
                session["user_id"], sell_symbol, sell_shares, sell_price, sell_time
            )
```
14. html `<input>` 中可以用`type="number"` 来获取数字
15. float.is_integer(shares)可**判断浮点数是不是整数**；而不是去判断整数是不是浮点数（整数是一种特殊的浮点数）