# Sileo 越狱源脚手架

一个开箱即用的 Sileo / APT 静态越狱源模板。无需 Linux、无需 dpkg，**Windows /macOS/ Linux 上只用 Python 3 即可维护**。

## 目录结构



```
sileo-repo/

├── Release                  # 仓库元数据（源名、架构等），脚本会自动刷新 Date 与校验和

├── Packages                 # 包索引（脚本自动生成，勿手改）

├── Packages.gz / .bz2 / .xz # 压缩索引（自动生成）

├── CydiaIcon.png            # 源图标（方形，替换成你自己的）

├── sileo-featured.json      # Sileo 源页面顶部推荐横幅

├── index.html               # 源落地页（自动识别地址，含"添加到 Sileo"按钮）

├── .nojekyll                # GitHub Pages 必需（跳过 Jekyll 处理）

├── debs/                    # ★ 把你的 .deb 全部放到这里（可建子目录）

├── depictions/              # Sileo 原生 Depiction（JSON），每个包一个

├── assets/                  # banner、header、截图等图片

│   ├── banner.png           # 推荐横幅图（16:9，建议 1920x1080）

│   ├── header.png           # 包介绍页头图

│   └── screenshots/         # 插件截图

├── scripts/

│   ├── update-repo.py       # ★ 核心：扫描 debs/ 生成索引、更新 Release

│   └── make\_assets.py       # 占位素材生成器（仅需运行一次）

└── .github/workflows/

&#x20;   └── update.yml           # GitHub Actions：push deb 后自动重建索引
```

## 快速开始（3 步）

### 1. 修改源信息

编辑 `Release`，把 `Origin`、`Label`、`Description` 改成你自己的源名称和介绍：



```
Origin: MyRepo          ← 源名称

Label: MyRepo           ← 源名称（与 Origin 保持一致即可）

Description: ...        ← 源简介

Architectures: iphoneos-arm64 iphoneos-arm
```

> `iphoneos-arm64`
>
>  \= 现代 Rootless 越狱（Dopamine、palera1n、NathanLR 等）；
> `iphoneos-arm`
>
>  \= 旧版 Rootful 越狱。只做 Rootless 可只保留 
>
> `iphoneos-arm64`
>
> 。

### 2. 放入 .deb

把打包好的 `.deb` 文件丢进 `debs/` 目录（支持子目录分类）。

### 3. 生成索引



```
python scripts/update-repo.py
```

脚本会自动：解析每个 deb 的 control 信息 → 生成 `Packages` 及三种压缩包 → 在每个条目里写入 `Filename / Size / MD5sum / SHA1 / SHA256` → 更新 `Release` 的 `Date` 和校验和。

以后每次增删 deb，重跑这一条命令即可。

## 本地预览测试

Sileo 只能通过 HTTP/HTTPS 拉取源，不能用 `file://`。在仓库根目录启动临时服务器：



```
python -m http.server 8080
```

手机与电脑同一局域网时，源地址为 `http://电脑局域网IP:8080/`（仅测试用，正式源必须 HTTPS）。

浏览器打开 `http://localhost:8080/` 可预览落地页。

## 部署到 GitHub Pages（免费 + HTTPS，推荐）



1. 在 GitHub 新建仓库（例如 `your-repo`），把本目录全部文件推送到 `main` 分支：



```
cd sileo-repo

git init

git add .

git commit -m "init sileo repo"

git branch -M main

git remote add origin https://github.com/你的用户名/your-repo.git

git push -u origin main
```



1. 打开仓库 **Settings → Pages**，Source 选择 **Deploy from a branch**，分支选 `main` / 根目录 `(root)`，保存。

2. 等待一两分钟，你的源地址就是：



```
https://你的用户名.github.io/your-repo/
```



1. 以后更新：把新 deb 放进 `debs/` 后 push，`.github/workflows/update.yml` 会在云端自动重建 Packages 索引并提交，无需本地运行脚本。

> `.nojekyll`
>
>  必须保留，否则 GitHub Pages 会忽略以 
>
> `.`
>
>  开头的文件及部分路径。

## 其他部署方式

任意静态文件服务器均可：Nginx、Apache、宝塔面板、Cloudflare Pages、Vercel、对象存储（S3/OSS）开静态网站托管等。

**硬性要求：必须支持 HTTPS**（Sileo 不接受自签证书与纯 HTTP 的正式源）。Nginx 示例要点：正常静态托管 + 配置证书（Let's Encrypt / 宝塔一键 SSL）即可，无需特殊 MIME 设置。

## 在 Sileo 中添加源



* 点击落地页上的 "添加到 Sileo" 按钮（链接格式为 `sileo://source/https://你的地址/`）；

* 或在 Sileo 中：**Sources → 右上角 + → 输入源 URL**。

## 打包插件（Theos）

在 Theos 工程目录：



```
\# Rootless（现代越狱，推荐）

make package THEOS\_PACKAGE\_SCHEME=rootless

\# Rootful（旧越狱）

make package
```

生成的 deb 在 `packages/` 目录，放进本源的 `debs/` 即可。

`control` 文件推荐字段：



```
Package: com.yourname.mytweak     # 唯一 ID，反向域名格式

Name: MyTweak                     # 显示名称

Version: 1.0.0

Architecture: iphoneos-arm64      # rootless 用 arm64；rootful 通用包可用 iphoneos-arm

Description: 一句话简介

Maintainer: 你的名字 \<you@example.com>

Author: 你的名字 \<you@example.com>

Section: Tweaks                   # Tweaks / Themes / Utilities / System ...

Depends: mobilesubstrate         # 按实际依赖填写

Icon: https://源地址/assets/icon.png

SileoDepiction: https://源地址/depictions/com.yourname.mytweak.json
```

## 自定义原生 Depiction

Sileo 原生 Depiction 是 JSON，在 control 中用 `SileoDepiction` 字段指向其 URL。

`depictions/com.example.mytweak.json` 是完整示例，包含：标题、Markdown 正文、截图轮播、版本 / 日期表格、外链按钮、更新日志 Tab。

常用视图 class：



| class                                              | 用途                 |
| -------------------------------------------------- | ------------------ |
| `DepictionHeaderView` / `DepictionSubheaderView`   | 大 / 小标题            |
| `DepictionMarkdownView`                            | Markdown / HTML 正文 |
| `DepictionImageView`                               | 单张图片               |
| `DepictionScreenshotsView`                         | 可横滑、点击放大的截图轮播      |
| `DepictionTableTextView`                           | 键值行（版本、日期等）        |
| `DepictionTableButtonView` / `DepictionButtonView` | 跳转按钮               |
| `DepictionSeparatorView` / `DepictionSpacerView`   | 分割线 / 空白           |
| `DepictionRatingView` / `DepictionReviewView`      | 评分 / 评价            |

完整规范见官方文档：[https://developer.getsileo.app/native-depictions](https://developer.getsileo.app/native-depictions)

## 图片素材规格



| 素材                      | 建议尺寸            | 说明                    |
| ----------------------- | --------------- | --------------------- |
| `CydiaIcon.png`         | 512×512（方形）     | 源图标                   |
| `sileo-featured` banner | 1920×1080（16:9） | 官方建议不要在图内放文字          |
| Depiction `headerImage` | 约 1280×360      | 包页顶部头图                |
| 截图                      | 设备原比例竖图         | 配 `accessibilityText` |

所有图片 URL 必须是完整的 HTTPS 地址，替换文件后重跑 `update-repo.py`（图片本身不进 Packages，但 deb 内的 control 字段变了才需要重建）。

## 常见问题



* **Sileo 刷新报错 / 找不到包**：确认源地址以 `/` 结尾、`Release` 与 `Packages` 在根目录、已通过 HTTPS 访问、重跑过 `update-repo.py`。

* **deb 是 zstd 压缩脚本报错**：执行 `pip install zstandard` 后重试（gz/xz/bz2 无需任何依赖）。

* **需要 GPG 签名吗**：Sileo 对 HTTPS 源不强制要求签名，本模板为无签名仓库，可直接使用；如需签名可自行用 `apt-ftparchive`/gpg 扩展。

* **更新后 Sileo 仍显示旧内容**：Sileo 有缓存，可在 Sources 页下拉刷新，或删除源后重新添加。

* **想先看看成品长什么样**：双击打开 `index.html` 可预览落地页（但 "添加到 Sileo" 按钮需部署后才有效）。