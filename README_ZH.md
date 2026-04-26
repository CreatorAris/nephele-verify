<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake.svg" />
  <img alt="github contribution snake animation" src="https://raw.githubusercontent.com/CreatorAris/CreatorAris/dist/github-snake.svg" />
</picture>

# Nephele Verify

[Nephele Workshop](https://nephele.arisfusion.com) 数字存证（`.nep`）的独立验证页。单文件 HTML、零构建、零后端、零埋点。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![No Build](https://img.shields.io/badge/build-none-green.svg)](#自建镜像)
[![Mirrorable](https://img.shields.io/badge/mirrorable-yes-purple.svg)](#自建镜像)
[![GitHub stars](https://img.shields.io/github/stars/CreatorAris/nephele-verify.svg)](https://github.com/CreatorAris/nephele-verify/stargazers)
[![GitHub last commit](https://img.shields.io/github/last-commit/CreatorAris/nephele-verify.svg)](https://github.com/CreatorAris/nephele-verify/commits)

[English](README.md) · [verify.arisfusion.com](https://verify.arisfusion.com)

</div>

## 这是什么

本仓库是 [verify.arisfusion.com](https://verify.arisfusion.com) 的源码 —— 用于验证 Nephele Workshop 生成的 `.nep` 存证包的页面。

公开它（MIT、单文件 HTML）是出于三个原因：

1. **算法透明**。任何人都能读懂验证逻辑，并对照 [技术审计文档](https://nephele.arisfusion.com/docs/security/audit) 进行核对。
2. **可镜像**。即使 `arisfusion.com` 哪天消失了，验证器仍然必须可用。任何人都可以自建镜像。
3. **可离线运行**。法院、律所、档案机构可以直接打开本地 `index.html` 在断网环境下完成验证。

## 验证什么

`.nep` 是一个 ZIP 容器，里面有 manifest、文件哈希、可选的 Merkle proof、以及一份 RFC 3161 时间戳 token。本页面在浏览器端完成以下校验：

- **文件哈希** —— 重新计算每个受保护文件的 SHA-256，与 manifest 对比。
- **Merkle 根** —— 从叶子哈希重建 Merkle 树并验证根哈希。
- **TSA 时间戳** —— 解析 RFC 3161 二进制 token（ASN.1 / DER），提取 `genTime` 和签发机构，并确认 token 中的摘要与 Merkle 根一致。
- **URL 取证包** —— 验证截图哈希、HTML 抓取哈希、TLS 证书快照、以及对 URL 证据根做的时间戳 token。

所有密码学操作通过浏览器的 Web Crypto API 完成。JSZip 从 CDN 加载用于读取 ZIP。**不上传任何文件内容到服务器。**

## 不验证什么

- **不**对 TSA 的证书链做信任根验证。如需做完整证书链信任校验，请用 OpenSSL `ts -verify` 命令对原始 `.tsa` token 验证；本页面会展示必要信息（签发机构名、`genTime`、被签摘要）以便离线核验。
- 时间戳的有效性只能证明"在 TSA 断言的时间点，某个摘要被时间戳过"，无法证明"文件内容在该时间点之前就存在"。

## 自建镜像

无构建步骤、无环境变量、无配置项。

```bash
# 静态文件，任何能托管 HTML 的地方都行
cp index.html /var/www/your-domain/

# 或者直接本地打开
open index.html       # macOS
start index.html      # Windows
xdg-open index.html   # Linux
```

唯一的外部请求是 JSZip 的 CDN 加载（`cdnjs.cloudflare.com`）。如果需要完全离线版本，把 `<script src=...>` 替换为本地 vendored 的 `jszip.min.js` 即可。

## 架构

```
.nep ZIP 文件
    -> JSZip 解出 manifest.json + 文件哈希 + (proof.tsa | proof.json)
    -> Web Crypto API 重新计算每个文件的 SHA-256
    -> 重建 Merkle 树，对比根哈希
    -> ASN.1 / DER 解析器拆开 TSA token
    -> 渲染最终结论 + 各组件状态
```

全部塞在同一个 HTML 文件里，view-source 即可读完。

## 反馈

验证器自身的 bug —— 漏判、误判、ASN.1 边界情况、翻译错别字 —— 在本仓库提 issue。欢迎 PR：本仓库就是 `verify.arisfusion.com` 的代码源头，合并即上线。

Nephele Workshop 桌面端本身的功能需求 —— 桌面端代码是闭源的，请通过 [官网](https://nephele.arisfusion.com) 上的联系方式提交，不要发到本仓库。

## License

MIT，见 [LICENSE](LICENSE)。可自由 fork、镜像、嵌入、修改。

## 相关仓库

- [nephele-core-audit](https://github.com/CreatorAris/nephele-core-audit) —— Nephele Workshop 客户端的可审计代码子集（rights / packer / validator）
- [nephele-wisp](https://github.com/CreatorAris/nephele-wisp) —— 浏览器侧伴侣（Chrome / Edge 扩展）
