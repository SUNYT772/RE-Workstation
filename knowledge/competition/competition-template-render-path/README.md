# 模板与渲染路径

适用问题：关键问题或产物位于路由解析、服务端渲染上下文、模板数据或 hydration 交接中。

## 主题与术语

SSR、模板、route loader、hydration、服务端/客户端渲染边界、预览/隐藏路由与模板到处理器的执行检查差异。

## 分析关注点

- Map the render chain in order: route resolution, loader or data fetch, template or component context, response HTML, hydration payload, and client takeover.
- Record host, route params, preview toggles, tenant or host switches, and server-only variables before mutating anything.
- Compare template gating with loader or handler enforcement.
- Preserve one successful render and one failing render path with the smallest delta.
- Reproduce the smallest request-to-render branch that proves the decisive behavior.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map Route To Render Context

- Record host, path, route match, loader, template, layout, hydration blob, and client boot chunk for the active view.
- Note whether the response is SSR HTML, static HTML plus hydration, edge-rendered content, or a template fragment used by another route.
- Keep server-only context and client-visible context separate.

### Trace Template And Enforcement Boundaries

- Show where permissions, feature flags, preview state, tenant selection, or host-based switches are applied.
- Compare template-level gating, loader-level gating, and backend handler enforcement instead of trusting any one layer.
- Record hidden fields, inline data, hydration JSON, meta tags, or alternate partials that expose the decisive branch.

### Reduce To The Decisive Render Path

- Compress the result to the smallest sequence: request -> route match -> loader or template context -> rendered output or hidden data -> resulting effect.
- State clearly whether the decisive weakness lives in route selection, template context construction, server-client hydration handoff, or mismatched enforcement between render and handler.
- If the task becomes mostly emitted bundle recovery or source map reconstruction, see related bundle knowledge.

## 细分参考与关联

- 参考：[template-render-path.md](references/template-render-path.md)：渲染、hydration 检查项与证据组织方式。

## 证据字段

- Route names, loader names, templates, layouts, hydration keys, and host or preview switches
- One success or failure pair that shows where render-layer behavior diverges
- One minimal request-to-render sequence that reaches the decisive branch

来源：原 `CTF-Sandbox-Orchestrator/competition-template-render-path/SKILL.md`。
