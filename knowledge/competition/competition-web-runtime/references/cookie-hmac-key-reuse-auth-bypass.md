# Cookie HMAC 密钥复用与声明信任

## 适用场景与信号

URL 中的访问token、已签名Cookie以及多个角色的Cookie存在密钥复用，服务端又直接把客户端声明作为权限依据。关键词包括HMAC key reuse、known-key session forgery、客户端声明信任与Cookie签名边界。

## 格式与算法关系

原案例格式为 `name = base64url(payload) . base64url(signature)`。签名输入可能是编码后的payload文本，而非解码JSON；HMAC-SHA256涉及密钥、消息的原始字节和Base64url无填充表示。验证同一个已知Cookie的签名关系，才能判断访问token是否被复用为签名密钥。

## 需要区分的边界

学生端Cookie和管理端Cookie的名称、用途、签名密钥与接受位置可能不同。原材料列举admin_session、admin_token、admin_auth、manage_token、backstage_session，以及admin、role、isAdmin、access、level、user、authenticated、type等权限字段。它们是案例中的候选名称与声明类别，不保证任何应用采用同样规则。

JSON紧凑序列化、字段大小写、Base64与Base64url、填充、签名大小写、Cookie path/Secure属性与浏览器状态会影响字节或接受结果。签名能够重现不等同服务端授予权限；HTTP 200也不等同管理员权限已被证明，需结合服务端身份与受保护数据的实际返回。

## 证据与修复

把公开输入来源、签名输入关系、角色Cookie边界、服务端接受结果和实际影响放在一起。原案例的靶场域名与示例token不随通用版分发。

原修复方向包括：独立服务端签名密钥、服务端会话授权、角色间密钥分离、iat/exp/typ等声明校验、签名解析失败的受控错误响应。

来源：原 `CTF-Sandbox-Orchestrator/competition-web-runtime/references/cookie-hmac-key-reuse-auth-bypass.md`。重编排为格式、信任与判断知识，不提供工具调用或浏览器执行入口。
