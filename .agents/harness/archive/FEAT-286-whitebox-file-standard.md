# FEAT-286 白盒关卡文件规范

Status: done (2026-10-14)

用户选择在 TheManTest 内直接搭正式关卡的白盒，并要求制定文件规范、提交远端。正式规范位于 `Documentation/LevelDesign/WhiteboxFileStandard.md`，规定地图所有者目录、专属与共享资源、命名、外部 Actor 文件和 Perforce changelist 检查。此任务只写文档，没有修改地图、资产或玩法。

项目远端使用 Perforce。GitHub `origin` 虽存在，但不是当前发布渠道；Git 仅用于本地 WIP 安全检查点。`Documentation/LevelDesign/WhiteboxFileStandard.md` 是 Perforce 变更 120307 的唯一文件，已提交并从服务器回读，SHA256 与本地一致。`.agents/` 被 `.p4ignore` 排除，本地 harness 中补充了 Perforce 发布规则与白盒规范索引，不在该变更中。

2026-10-14 follow-up: user explicitly requested pre-created Level01 type folders and then actual placeholder files. Five `Content/Maps/Level01/{Blueprint,Meshes,Materials,Textures,Data}/.p4keep` text files now make those folders syncable through Perforce. WhiteboxFileStandard.md revision 2 documents this optional pre-creation workflow and says to delete each placeholder in the same changelist as the first real asset. Perforce change **120313** contains exactly those five additions plus the documentation edit; server readback of the documentation matches local bytes. No `.umap`, `.uasset`, gameplay or map edits.
