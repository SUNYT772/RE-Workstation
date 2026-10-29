# HTTP 观测与接口归纳

## 输入与上下文

请求方法、URL、headers、body、content-type与时间。

## 方法与判断

原记录结构保存 timestamp/method/url/headers/content/content_type；文本正文最多5000字符，截断属于证据范围限制。按 /v数字/、/api/、/game/、/login、/sign、/auth、/token 推断候选接口，仅是启发式。按方法与路径归并端点，保留重复请求的时序；原 response 部分为空，不能声称已记录响应。

## 结果结构

请求记录、候选端点集合、截断标识、正文编码、响应是否存在以及身份状态。

来源：原 `templates/mitmproxy/capture.py`。保留其分析方法和数据结构，整理为独立于具体工具的知识参考。
