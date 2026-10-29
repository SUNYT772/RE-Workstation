# 运行路由

适用问题：需要判断实际请求由哪个节点、代理规则或头部驱动的路由分支提供服务。

## 主题与术语

反向代理、Host/forwarded headers、vhost、WebSocket upgrade、路径前缀重写、base URL 推导与多节点路由。

## 分析关注点

- Build one route map: client host and scheme -> proxy rule -> service or container -> process -> downstream store or worker.
- Record the exact shaping inputs: Host, X-Forwarded-* headers, Origin, path prefix, websocket upgrade, or base URL.
- Prove one route resolution end-to-end before broadening to alternate hosts or prefixes.
- Re-run the same request with one routing input changed at a time.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Route Inputs

- Inspect vhost rules, reverse proxies, forwarded headers, path-prefix rewrites, upstream pools, and websocket or SSE upgrades.
- Note which parts of the request influence routing or app behavior: host, scheme, port, path, prefix, cookie scope, or origin.
- 公网外观域名、云主机名与独立 VPS 节点均可参与多节点路由；其归属与作用依据实际配置、监听与流量证据判断。

### Trace Route To Live Consumer

- Map hostname to proxy rule to container or process to port to downstream service.
- Compare checked-in proxy intent against live listeners, mounted configs, runtime env, and observed traffic.
- Keep headers, proxy config, and live request traces tied together in one evidence chain.

### Prove The Decisive Deviation

- Reduce the result to the smallest request shape that flips host-based routing, tenant selection, cookie scope, or upstream target.
- Distinguish route resolution from application auth logic; prove where each decision really happens.
- If the problem shifts from routing to general web state or container runtime drift, see related parent knowledge.

## 细分参考与关联

- 参考：[runtime-routing.md](references/runtime-routing.md)：路由检查项、头部差异矩阵与证据组织方式。
- If the hard part is parser differentials, transfer-framing ambiguity, or proxy-backend request smuggling behavior, see [请求归一化与解析差异](../competition-request-normalization-smuggling/README.md).

## 证据字段

- Hostnames, proxy snippets, header sets, path prefixes, listener ports, and route-specific cookies
- The exact request shape that reaches the decisive backend or branch
- One compact host -> proxy -> service -> process map for the active path

来源：原 `CTF-Sandbox-Orchestrator/competition-runtime-routing/SKILL.md`。
