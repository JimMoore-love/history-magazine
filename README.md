# 《列传》· 中国历史人物志

一期一人，按朝代归档的中国历史人物志期刊。全部为**单文件 HTML**（插画、地图全部 base64 内嵌），
可离线打开、可打印成 PDF，**不依赖任何 CDN 或外链资源**。

线上站点：**http://139.196.5.123/history-magazine/**
本仓库即线上站点的**唯一源**：仓库根目录 = 站点根目录。

---

## 目录结构

```
.
├── index.html                  ← 刊库首页（按朝代分类 + 分页，内嵌各期封面缩略图）
├── _home.tpl.html              ← 刊库首页外壳 + 渲染脚本（占位符 5 个，便于日后直接改壳）
├── <主题>-<期号>/
│   └── index.html              ← 单期（整期一个文件，含全部插画与地图）
└── .github/workflows/deploy.yml ← CI/CD：push 即上线
```

期号从 `1` 起，不补零；`slug` 用英文，格式 `<主题>-<期号>`。

## 已出期数

| 期 | 辑题 | 人物 | 朝代 | slug |
|---|---|---|---|---|
| 01 | 太和 | 元宏 | 北朝 · 北魏 | `beiwei-taihe-1` |
| 02 | 气吞万里 | 刘裕 | 南朝 · 刘宋 | `nanchao-liusong-2` |
| 03 | 元嘉 | 刘义隆 | 南朝 · 刘宋 | `liuyilong-yuanjia-3` |
| 04 | 台城 | 萧衍 | 南朝 · 南梁 | `liangwu-xiaoyan-4` |
| 05 | 沧海 | 曹操 | 东汉末 · 曹魏 | `caocao-canghai-5` |
| 06 | 宣和 | 赵佶 | 北宋 | `songhuizong-xuanhe-6` |
| 07 | 昭烈 | 刘备 | 三国 · 蜀汉 | `liubei-zhaolie-7` |
| 08 | 河阴 | 尔朱荣 | 北朝 · 北魏 | `erzhu-heyin-8` |
| 09 | 轮台 | 刘彻 | 西汉 | `hanwudi-luntai-9` |
| 10 | 烏臺 | 苏轼 | 北宋 | `sushi-wutai-10` |
| 11 | 藍關 | 韩愈 | 唐 | `hanyu-languan-11` |
| 12 | 沙丘 | 秦始皇 | 秦 | `shihuang-shaqiu-12` |
| 13 | 朱仙 | 岳飞 | 南宋 | `yuefei-zhuxian-13` |

---

## 发布方式（CI/CD）

**不要手动 scp 上传。** 把改动 push 到 `main` 即可，GitHub Actions 会自动完成整条链路：

```
push 到 main
   ↓
配置 SSH（私钥来自仓库 Secrets）
   ↓
上传前先在服务器端 tar 备份到 /root/hm-backup/
   ↓
tar 管道上传到 /usr/share/nginx/html/history-magazine/
   ↓
服务器端 sha1sum 逐文件比对（本地 ≠ 线上则整个 job 失败）
   ↓
公网 curl 实测 HTTP 状态码
```

### 必须配置的 Secrets

| 名字 | 说明 |
|---|---|
| `DEPLOY_SSH_KEY` | 部署用 SSH 私钥（OpenSSH 格式，ed25519） |
| `DEPLOY_HOST` | 服务器 IP / 域名 |
| `DEPLOY_USER` | 登录用户 |
| `DEPLOY_PATH` | 站点根目录绝对路径 |

对应的**公钥**需加到服务器 `~/.ssh/authorized_keys`。密钥专用（`github-actions-deploy@liezhuan`），
不要复用个人登录密钥，便于随时吊销。

### 手动触发

`Actions` → `Deploy 到 ali-claw` → `Run workflow`（`workflow_dispatch`）。

---

## 新增一期的流程

1. 本地生成单期工程 → 产出 `index.html` 单文件 + 封面缩略图；
2. 把单期放进 `<slug>/index.html`；
3. 重跑刊库首页生成脚本（按朝代分组、排序、分页全自动）；
4. `git add -A && git commit -m "第 11 期《XX》· XXX" && git push`；
5. 等 Actions 变绿 —— sha1 比对不过就是红的。

## 校验哲学

- **不信上传工具的退出码**（`scp` 曾静默失败、`tar` 管道也可能只传一半），只信**服务器端算出来的 sha1**；
- **备份永远在上传之前**，且备份文件留在服务器 `/root/hm-backup/`；
- 部署失败**不写摘要**，宁可红着让人看见，也不要"看起来成功"。

## 许可

内容为历史科普写作，插画为 AI 生成。代码与页面结构可自由参考。
