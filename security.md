# Security Guide

## 编码相关

1. 所有文件保存为 UTF-8 NO BOM 格式，换行符格式 Unix (\n)。

2. MongoDB 查询非 UTF-8 时会抛出异常，因此查询前需要判断输入内容是否是 UTF-8 编码。

   ```php
   mb_check_encoding($input, 'UTF-8') // BOOL
   ```

3. 字符串操作时，使用 mbstring 系列函数，避免中文字符被切割导致的安全和逻辑问题。注意，字符串替换不需要使用 mbstring 实现（前提是编码是 UTF-8）。

4. 正则匹配时，使用 /u 修饰符进行 UTF-8 编码匹配。

## 数据库相关

1. 对所有查询，都需要确保用户输入数据的类型符合设计，避免用户输入数组产生的注入漏洞。

2. 输入检查在 Model 层面进行。

3. 若使用用户传入的值构造 MongoID，需要先验证格式，无效构造会产生异常。

   ```php
   MongoId::isValid($input) // BOOL
   ```

4. 对于所有插入数据库的值应当检查长度，并对过长的值截断或报错。

## 前端相关

1. 输出到用户的内容需要反复检查是否存在 XSS 可能性，对于没有 escape 的需要 escape，但不要多次 escape。注意，Twig 模板输出时会自动进行一次 escape。

   + 对于输出到 HTML 的部分，至少需要过滤 `<`，`>`，`&`
   
   + 对于输出到 HTML 属性的部分，至少需要过滤 `"`，`'`
   
   + 对于输出到 `script` 标签内的部分，使用 `json_encode` 进行编码，并务必指定 `JSON_HEX_TAG` 掩码

2. 用户以 HTML 内容输入时，使用 `htmlpurifier` 进行 HTML 过滤。

3. 所有 POST 或可能发生状态改变的 GET 请求（如 logout），需要检查 CSRF token。