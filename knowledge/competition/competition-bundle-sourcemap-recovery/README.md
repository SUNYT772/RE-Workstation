# 构建产物与源码映射

适用问题：实际运行结构需要从构建产物、源码映射、分块表或混淆加载流程中还原。

## 主题与术语

JavaScript 构建产物、source map、构建清单、chunk 注册表、混淆加载流程、隐藏路由与 API 还原。

## 分析关注点

- Start from the served artifact set: entry HTML, build manifest, bootstrap bundle, chunk map, and source maps.
- Record chunk ids, route chunks, loader functions, endpoint strings, and config keys before broad manual deobfuscation.
- Reconstruct the smallest runtime graph that explains which asset executes now.
- Keep served artifact truth separate from repository source unless parity is proven.
- Reproduce the smallest asset-to-runtime boundary that proves the decisive behavior.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Map The Served Artifact Set

- Record entry HTML, script tags, preload hints, manifest files, asset map, chunk registry, and source map URLs.
- Note framework-specific artifacts such as route manifests, client reference manifests, or lazy-loader tables when present.
- Keep emitted filenames, hash suffixes, and route ownership tied together.

### Reconstruct Runtime Structure

- Follow bootstrap code, chunk loaders, module registry, string decoders, and lazy import boundaries.
- Use source maps, manifest files, and stable symbol clusters to recover route names, API calls, feature flags, and hidden panels.
- Distinguish build-time intent from the bundle that is actively served now.

### Reduce To The Decisive Bundle Path

- Compress the result to the smallest sequence: served asset -> loader path -> module or symbol -> runtime effect.
- State clearly whether the decisive weakness lives in manifest drift, chunk loading, hidden route code, string decoding, or stale source assumptions.
- If the task shifts from built assets to SSR or template enforcement, see related template-render knowledge.

## 细分参考与关联

- 参考：[bundle-sourcemap-recovery.md](references/bundle-sourcemap-recovery.md)：产物与去混淆检查项、证据组织方式。

## 证据字段

- Served filenames, chunk ids, manifest entries, source map paths, recovered symbols, and endpoint strings
- The exact executing bundle or module that proves the runtime branch
- One minimal asset-to-runtime sequence that reaches the decisive effect

来源：原 `CTF-Sandbox-Orchestrator/competition-bundle-sourcemap-recovery/SKILL.md`。
