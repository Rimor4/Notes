### vim-surround

1. **添加 Surround（ys 操作）**
    
    - `ys{motion}{surrounding}`: 在指定的文本对象周围添加指定的符号。
        - 例如，`ysiw)`：在当前单词两边添加圆括号 `()`。
        - `ys$"`：从光标到行尾添加双引号 `""`。
2. **替换 Surround（cs 操作）**
    
    - `cs{old}{new}`: 替换现有的符号。
        - 例如，`cs"'`：将双引号 `"` 替换为单引号 `'`。
        - `cs)(`：将圆括号 `()` 替换为方括号 `[]`。
3. **删除 Surround（ds 操作）**
    
    - `ds{surrounding}`: 删除指定的符号。
        - 例如，`ds"`：删除光标所在文本的双引号 `"`。
        - `ds(`：删除光标所在文本的圆括号 `()`。
4. **添加 Surround 到整行（yss 操作）**
    
    - `yss{surrounding}`: 对整行内容添加符号。
        - 例如，`yss}`：将整行包裹在大括号 `{}` 中。

#### 常用 Surround 符号

- `(` 或 `)`：圆括号
- `{` 或 `}`：大括号
- `[` 或 `]`：方括号
- `<` 或 `>`：尖括号
- `"`：双引号
- `'`：单引号
- `` ` ``：反引号
- `t`：任意 HTML 标签。例如，`yss<em>` 会在整行文本上添加 `<em>` 标签。

#### 高级使用

1. **为 HTML 标签添加 Surround**
    
    - `ysiw<t`：在当前单词前后添加 HTML 标签 `<tag>...</tag>`。
    - 然后你会进入插入模式，可以输入你需要的标签名。
2. **改变 HTML 标签 Surround**
    
    - `cst<newtag>`：将旧的 HTML 标签替换为新的标签。
    - 例如，`cst<p>` 将当前 HTML 标签更改为 `<p>` 标签。
3. **在文本之间插入 Surround**
    
    - `ys` 后可以配合任意的文本对象命令，比如 `yss` 对整行操作，或 `ysiw` 对当前单词操作。