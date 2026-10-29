# Bundle And Sourcemap Recovery Checklist

## First Pass

- Entry HTML, script tags, preload links, manifests, chunk registries, source map URLs
- Bootstrap bundle, lazy chunks, route chunks, loader helpers, string decoders
- Framework clues: route manifest, client reference manifest, build id, asset map

## Chain To Reconstruct

- Served asset selected
- Bootstrap or loader resolves chunk or module
- Module registry or source map reveals structure
- Hidden route, API call, or branch is recovered
- Runtime effect is reproduced from the emitted asset set

## 关联证据

- Asset side: filenames, hashes, chunk ids, manifest entries, source map path
- Recovery side: recovered symbol, route, endpoint, loader helper, or decoded string
- Effect side: rendered panel, hidden route, accepted request, or client behavior

## 常见误判

- Trusting repository source over the currently served artifact set
- Opening huge minified bundles before checking manifests and source maps
- Recovering names without proving which bundle path actually executes at runtime

来源：原 `CTF-Sandbox-Orchestrator/competition-bundle-sourcemap-recovery/references/bundle-sourcemap-recovery.md`。
