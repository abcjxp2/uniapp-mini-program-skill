# uniapp-mini-program-skill

> **🎯 为什么有这个项目？**<br>
> 这不是一个通用框架，而是一份 **“踩坑急救包”** 。<br>
> 它把支付宝/微信小程序开发中那些让人火大、反复试错、浪费token的固定坑，沉淀成AI可以直接读懂的规则。<br>
> **目标**：让你少生一场气，让AI少烧一次算力。

Codex skill for diagnosing and fixing uni-app mini-program issues, with a focus on WeChat and Alipay mini-program differences.

## What It Covers

> **Keywords**: `uniapp` `miniprogram` `tabbar-jumping` `CE1000` `wechat-vs-alipay` `ai-skill`

- `pages.json`, `switchTab`, custom tabBar, and lifecycle rules.
- WeChat custom tabBar and Alipay customize tabBar differences.
- Alipay CE1000 and red-screen component errors.
- `dist/build` pollution such as `pages 2`, `components 2`, `uni_modules 2`.
- `uni_modules` compiled runtime dependency checks.
- Alipay `requestPayment` `resultCode` and toast object handling.

## Install

Copy the skill folder into your Codex skills directory:

```bash
cp -R uniapp-mini-program ~/.codex/skills/
```

Then restart Codex or reload skills.

## Usage

Ask Codex about uni-app mini-program issues, for example:

```text
支付宝小程序报 CE1000，帮我检查产物依赖
```

```text
uni-app 微信和支付宝 tabBar 切换闪烁，帮我排查
```

## 如何贡献

欢迎提交 Issue 和 Pull Request！详见 [CONTRIBUTING.md](./CONTRIBUTING.md)

> **💬 真实心路**：[为什么这个项目值得存在](./docs/why-this-exists.md)

## License

MIT
