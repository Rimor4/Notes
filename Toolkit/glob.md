## [glob语法](https://code.visualstudio.com/docs/editor/glob-patterns#_glob-pattern-syntax)

VS Code 支持以下 glob 语法：

- `/`分隔路径段
- `*`匹配路径段中的零个或多个字符
- `?`匹配路径段中的一个字符
- `**`匹配任意数量的路径段，包括没有
- `{}`对条件进行分组（例如`{**/*.html,**/*.txt}`匹配所有 HTML 和文本文件）
- `[]`**声明**要匹配的字符范围（`example.[0-9]`匹配`example.0`, `example.1`, ...）
- `[!...]`否定要匹配的字符范围（`example.[!0-9]`匹配`example.a`, `example.b`, 但不匹配`example.0`）

**注意：**路径由 分隔`/`，甚至在 Windows 上也不分隔`\`。但应用时，全局模式将匹配同时包含斜杠和反斜杠的路径。