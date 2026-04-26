# uniapp-mini-program-skill

Codex skill for diagnosing and fixing uni-app mini-program issues, with a focus on WeChat and Alipay mini-program differences.

## What It Covers

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

## License

MIT
