---
name: scnet-model-git-upload
description: Use when uploading a private model to SCNet 模型管理 via git+LFS (gitlab.scnet.cn:9002/model/<user>/personnal.git), when git-lfs push is slow (proxy detour), or when push rc=0 but objects 404.
tasks: ""
---

# SCNet 模型上传（git + LFS）

平台「模型管理」的 API 上传方式：私有模型仓库是 git LFS 仓库，把模型文件拷进去 push 即完成上传。

## 前置

- `git lfs install`（git-lfs 3.x 实测可用）
- 仓库地址与认证从平台「模型管理 → 模型详情」页复制（形如下面的 clone 命令，**含个人 token，别写进任何会进 git 的文件**）

## 步骤（平台原文流程）

```bash
git lfs install
git clone https://<用户名>:<TOKEN>@gitlab.scnet.cn:9002/model/<用户名>/personnal.git
cd personnal
cp -rf /work/my_model_dir/* .
git add -A .
git commit -m "commit message"
git push
git config --global lfs.activitytimeout 3600   # 大模型必设（--global 写系统盘会丢，建议改仓库级）
```

## 🚀 关键提速：LFS 上传绕开代理直连内网（26-10-19 实测 8→27 MB/s）

LFS 对象的实际存储是**平台内网 MinIO**：`oss.scnet.cn = 10.1.114.201:9000`（batch API 下发的 href 是 `http://oss.scnet.cn:9000/gitlab-lfs-prod/<oid分片>`）。容器默认 `http_proxy` 会把这条内网流量绕道 Squid（10.1.100.10:3120）再折回内网，**限速 ~8 MB/s**。

```bash
no_proxy='oss.scnet.cn' NO_PROXY='oss.scnet.cn' git push origin main   # 直连内网，实测 3.4×
```

- 容器到 `oss.scnet.cn:9000` 的内网 TCP 直连是通的（TCP 实测 OK），只有 gitlab 主站（公网 IP 221.11.21.199，太原联通）必须走代理——但那只有 KB 级元数据流量
- **适用判断**：先 `timeout 6 bash -c 'exec 3<>/dev/tcp/oss.scnet.cn/9000'` 测直连，通就加 no_proxy
- 物理位置：gitlab 前端 geo 在太原联通；oss 内网 IP 看不出区域——模型建在哪个「使用区域」存储就在哪侧，跨区域机器直连走平台骨干

## 🚨 push rc=0 ≠ 对象已落存储

实测出现过：push 成功 + ref 更新，但 `git lfs fetch` 报 404（oss 存储同步延迟）。验证方法（不下载对象）：

```bash
oid=$(git show HEAD:"<file>" | grep -oE 'oid sha256:[a-f0-9]+' | cut -d: -f2)
size=$(git show HEAD:"$f" | grep -oE 'size [0-9]+' | awk '{print $2}')
curl -sk -X POST 'https://<用户名>:<TOKEN>@gitlab.scnet.cn:9002/model/<用户名>/personnal.git/info/lfs/objects/batch' \
  -H 'Content-Type: application/vnd.git-lfs+json' -d '{"operation":"download","objects":[{"oid":"'$oid'","size":'$size'}],"transfer":"basic"}'
# 响应含 "actions"/download href = 已落存储；404/缺失 = 再等等或 git lfs push --object-id 重传
```

**别急着 `git lfs prune`**——验证通过前本地对象是唯一副本，prune 掉就只能重新上传。

## 其他实测坑

- **`git reset --hard` 会触发 LFS smudge 下载**（还可能 404 中断 checkout）——操作远端已有对象的仓库时加 `GIT_LFS_SKIP_SMUDGE=1`
- **`git clean -fd` 会删掉刚 mkdir 的空目录**——放文件的子目录要在每次 reset/clean 之后建
- **同盘搬运用 `mv` 别用 `cp`**：parastor 上 mv 瞬时完成；cp 会让源+工作树+LFS 对象三份并存，494G 配额直接爆
- 分片大模型推荐**逐分片 mv→add→commit→push→验证→删工作树文件**，空间峰值只有单分片×2
- `git config --global` 写系统盘（模板区，重启丢）；`git lfs prune` 前确认对象已推远端
- 上传中镜像若在保存（模板区只读），临时文件别放 /root

## 关联

- 快传通道：`scnet-fasttrans-transfer`（容器内 65012 不通，只能外部机器推）
- 文件归属判断：`scnet-instance-file-scope`
- 模型管理功能总览：https://www.scnet.cn/help/docs/mainsite/ai/model-management/index.html
