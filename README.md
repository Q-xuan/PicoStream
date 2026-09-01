# wetink

给本地 agent 用的 Astro Nano 风格流式体感 CSS。无构建、无框架。  
Drop-in streaming CSS for a local agent. No build. No framework.

主题：**石上墨迹**。粉墙黛瓦、朱红游廊、苔石干径、漏窗框景。不贴园林纹样。

## 演示

GitHub Pages：

- 玩场：https://q-xuan.github.io/wetink/prototype/
- harness：https://q-xuan.github.io/wetink/prototype/harness.html

`?theme=light|dark`，`?lang=zh|en`，`?cut=1`。harness 另有 `?tape=dsh|headless|pi|paste`，`?speed=0.5|1|2|0`。

本地：仓库根目录 `python3 -m http.server 8080`，打开 http://127.0.0.1:8080/prototype/ 。

推 `main` 会打下一个 patch tag（没有 tag 则从 `v0.1.0` 起）、打包 zip / npm tarball、发 Release、部署 Pages，包已在 npm 上则再发一版。手动：Actions → release → Run workflow，可填 `vX.Y.Z`。本地打 tag：`git tag vX.Y.Z && git push origin vX.Y.Z`。若 Actions 里 `github-pages` 环境要批准，点一次即可。

第一次上 npm 要本机发一版（之后 CI 用 Trusted Publisher，不必再放 token）。包名是 `wetink`。

```
npm publish --access public
```

然后到 npmjs.com → wetink → Trusted Publisher，填 `Q-xuan` / `wetink` / `release.yml`，允许 publish。有 `NPM_TOKEN` secret 也可以，CI 会走 token。

## 引入

```html
<link rel="stylesheet" href="wetink.css" />
<html class="wet-page">
```

npm（上架后）：

```
npm i wetink
```

```html
<link rel="stylesheet" href="node_modules/wetink/wetink.css" />
```

CDN 跟 npm，不必另发：

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/wetink/wetink.css" />
<link rel="stylesheet" href="https://unpkg.com/wetink/wetink.css" />
```

字体按需加载 Inter + Noto Serif SC / Lora；不加载则走系统回退。亮/暗：`html.dark` 或 `data-theme="dark"`。语言：`html lang`。`:lang(en)` 时正文衬线改走 Lora，匾额字距收紧并大写。短匾由接入方按语言填写，CSS 不写死文案。

## 契约

```html
<article class="wet-thread">
  <section class="wet-turn" data-role="user">
    <p class="wet-meta">问</p>
    <p class="wet-voice">...</p>
  </section>

  <section class="wet-turn" data-role="assistant" data-stream="typing">
    <p class="wet-meta">答</p>
    <details class="wet-think">
      <summary>最新一行活摘要</summary>
      <p>全文题跋，默认折起。</p>
    </details>
    <div class="wet-tools">
      <details class="wet-tool" data-state="done">
        <summary><b>read</b> · file.ts</summary>
      </details>
    </div>
    <div class="wet-voice">
      <p data-tail>正文用 Lora。<span class="wet-caret" aria-hidden="true"></span></p>
      <pre class="wet-code" data-lang="ts">const ok = true</pre>
    </div>
  </section>
</article>
```

| 属性 | 值 |
| --- | --- |
| `data-role` | `user` · `assistant` · `system` · `tool` |
| `data-stream` | `thinking` · `typing` / `live` · `tool` · `settled` · `steer` · `error` |
| `data-state`（工具） | `running` · `done` · `error` |
| `data-kind`（工具，可选） | `read` · `search` · `list` · `write` · `exec` |
| `data-tail` | 标在还在长的那个节点上 |

规则：旧回合保持 `settled`，只有当前回合可以湿墨。有 `data-tail` 时，同回合其余正文退一级，只有尾巴满墨。思考默认折起一行；湿着用 caret 和墨色，caret 跟在可见行末，不先撑开再塌。`<p class="wet-think">` 仍可用。连续工具包进 `.wet-tools`，槽位常驻。栏只有回合左边那一条。开合只动三角，井跟字齐。改口走 `steer`，不走青紫断。干径是细黛线，不是第二根苔柱。未完成的表可打 `data-open`。新块可加 `wet-enter`。caret 放在最后一个还在长的节点里。

短匾（接入方按 `lang` 填，属性仍是英文）：

| | 问 | 答 | 思 | 写 | 器 | 毕 | 改 | 断 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `zh` | 问 | 答 | 思 | 写 | 器 | 毕 | 改 | 断 |
| `en` | Ask | Reply | Think | Write | Tool | Done | Steer | Break |

玩场：`?theme=light|dark`，`?lang=zh|en`，`?cut=1`。

## 接到真实流

不能把这份 CSS 丢进 DeepSeek Harness 或 Pi Web。他们是 React / 自绘壳，DOM 不是 `data-stream`。

能测体感的路径：用他们的事件流喂这份契约。

- 回放：[`prototype/harness.html`](prototype/harness.html)
- `?tape=dsh|headless|pi|paste`，`?speed=0.5|1|2|0`
- DSH：`assistant/chunk`（`reasoning-delta` / `text-delta` / `tool-call-delta`）+ `tool/call` + `tool/result`
- Pi：`message_update.assistantMessageEvent`（`thinking_delta` / `text_delta` / `toolcall_*`）+ `tool_execution_*`
- 自己的 dump：一行一条 JSON，拖进页面或粘贴。DSH 的 `stream-json` / `session.jsonl` 和包一层 `session_event` 都能读

本环境没有 API 密钥，跑不了他们的真网页。回放的是线协议，不是他们的皮肤。

## 文件

- [`wetink.css`](wetink.css) — 交付物
- [`package.json`](package.json) — npm 包名，无构建
- [`DESIGN.md`](DESIGN.md) — 定调
- [`AGENTS.md`](AGENTS.md) — 接入方 / agent 的梯子和验证地板
- [`prototype/index.html`](prototype/index.html) — 玩场
- [`prototype/harness.html`](prototype/harness.html) — DSH / Pi 事件回放
- [`prototype/fixtures/`](prototype/fixtures/) — 标本

约束：第一性原理 · 马斯克原型 · 轻量化。
