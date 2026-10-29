# 固件布局

适用问题：需要还原固件镜像的布局、启动、更新与可达设备行为之间的关系。

## 主题与术语

固件镜像、分区表、bootloader、内核/init、启动链、更新包、提取文件系统、嵌入配置与设备信任边界。

## 分析关注点

- Preserve the original image, extracted partitions, unpacked filesystems, and patched copies as separate artifacts.
- Map outer container, partition table, bootloader, kernel, rootfs, config, and update metadata before editing anything.
- Track the boot or update chain in order instead of jumping straight to the most interesting file.
- Record keys, signatures, offsets, partition boundaries, and init entrypoints in one compact evidence chain.
- Reproduce the decisive secret, branch, or reachable service from the smallest extracted path.

## 技术关系与判断

以下条目说明对象关系、验证证据与判断边界，可按问题组合参照。

### Establish Image Layout

- Identify container type, partition headers, compression, filesystem type, and any appended or nested images.
- Record offsets, sizes, hashes, mount points, and partition names before extraction mutates anything.
- Separate bootloader, kernel, initramfs, rootfs, config blobs, and update metadata as different layers.

### Trace Boot Or Update Flow

- Map how control moves from bootloader to kernel to init to services, or from update package to verifier to installer.
- Note which credentials, certificates, passwords, seeds, or config files are consumed at each stage.
- Distinguish checked-in firmware intent from the live behavior the extracted files actually support.

### Reduce To The Decisive Path

- Show the smallest chain from image boundary to service exposure, auth bypass, debug interface, credential recovery, or flag artifact.
- Keep extracted filesystems, derived configs, and patch experiments separate from pristine inputs.
- If the challenge becomes mostly about native crash behavior or exploit primitives after extraction, see related reverse knowledge.

## 细分参考与关联

- 参考：[firmware-layout.md](references/firmware-layout.md)：镜像布局、启动链检查项与证据组织方式。

## 证据字段

- Partition offsets, hashes, filesystem types, mount paths, boot entrypoints, and update metadata
- Extracted secrets, config paths, init scripts, service units, and credentials tied to the stage that consumes them
- Original images, extracted layers, mounted views, and patched copies as separate artifacts

来源：原 `CTF-Sandbox-Orchestrator/competition-firmware-layout/SKILL.md`。
