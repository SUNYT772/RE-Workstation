# Firmware Layout Checklist

## First Pass

- Identify outer container, partition headers, compression, checksums, and appended images
- Record partition names, offsets, sizes, filesystem types, mount points, and hashes
- Separate bootloader, kernel, initramfs, rootfs, config, and update payloads

## Boot Or Update Chain

- Boot ROM or vendor bootstrap
- Bootloader or secure boot stage
- Kernel and initramfs
- Root filesystem init path and services
- Update verifier, installer, and post-install hooks

## 关联证据

- Boundary facts: offsets, sizes, signatures, hashes, partition names
- Consumed state: keys, certs, passwords, default configs, scripts, or service files
- Reachable effect: debug service, auth branch, update bypass, or recovered artifact

## 常见误判

- Editing extracted files before recording pristine offsets and hashes
- Mixing config recovered from one partition with behavior sourced from another without proving the link
- Treating an extracted secret as decisive without showing where the boot or update flow actually consumes it

来源：原 `CTF-Sandbox-Orchestrator/competition-firmware-layout/references/firmware-layout.md`。
