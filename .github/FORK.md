# FORK.md — dist 分支相对上游(readest/readest main)的自主分歧清单

> Ground truth(两条命令,永不撒谎,以 git 为准):
> - 提交清单:`git log --no-merges upstream/main..origin/dist`
> - 文件足迹:`git diff upstream/main...origin/dist --stat`
>
> 最后审计:2026-11-26 — dist 独有提交 15 个,净足迹 8 文件(+382/−29)。
> 本文件只记录 git 查不到的部分:动机、退役条件、漂移后的处置。

## Infra(fork 独有,低冲突风险)

- **`.github/workflows/nightly-dist.yml`** — fork 专属 CI:每日 06:18(北京,实际触发
  受 GitHub 调度积压影响约 +5h)合并上游 → 双平台构建 → minisign 签名 → manifest
  组装(latest.json + release-notes.json)→ rolling `nightly` release(7 版滚动保留),
  发布时自动从 `.github/FORK.md` 刷新 release 正文。
  上游无此文件。App 内更新渠道指向本 release。
- **`.github/workflows/` 其余文件** = 冻结快照(GITHUB_TOKEN 不能推 workflow 文件,
  平台硬规则)。非 workflow 的 `.github`(模板/scripts)**自动流入**,不冻结。
  - 吸收基线:2026-11-23(WebKit PR-CI 更新);已知待吸收:2026-11-26 dependabot
    actions 组 bump(#7964),下次构建自动冻结并在 issue #1 通知。
  - 冻结时 bot 自动在 [issue #1](https://github.com/k6G52m4Dz75W/readest/issues/1)
    @owner(邮件通知)。
  - 吸收配方:`git checkout upstream/main -- .github/workflows && git checkout
    origin/dist -- .github/workflows/nightly-dist.yml`;新出现的 workflow 文件吸收后
    必须 API disable(全仓库仅 nightly-dist.yml 应为 active,当前 15 个 workflow)。
  - ⚠️ 上游的 `nightly.yml`/`release.yml` 是 disabled 镜像文件,改它们 = 白干。

## Behavior patches(易冲突,上游漂移后需重放)

| 文件 | 提交 | 动机 | 退役条件 |
|---|---|---|---|
| `src/utils/access.ts` | `90f528998` + `01e6125db` | 解锁 premium 功能门(AGPL 本地修改) | 上游商业模式变化 |
| `src/services/constants.ts` + `src-tauri/tauri.conf.json` | `0a62b906c` 系(`a1d59f1f7` `eefc29aeb` `56b4abd1d`) | 更新渠道指向 fork release;默认 nightly;pub_date 北京戳;manifest notes 编号 | 永久(自有发布渠道) |
| `src/utils/telemetry.ts` + `.env` | `c33ef356a` | 关闭遥测(PostHog 全路径杀) | 永久 |
| `__tests__/…/ControlPanelUpdateReset.test.tsx` | `7747eeaa9` | 测试期望跟随 fork 默认渠道(nightly) | 默认值再变时随之更新 |
| manifest notes 编号(含于 `56b4abd1d`) | 同上 | 适配兜底解析器(#7775 被拒,长期有效) | 我们自己改 notes 格式时重审 |
| `services/constants.ts` + `FoliateViewer.tsx` + `screenPages.ts` + `ControlPanel.tsx`(6 处守卫) | (本次提交) | Stable Page Numbers 默认关闭:屏幕页数为默认,显式开启才是 stable(e-ink 避开 stable 校准/重排抖动,#3271 线;固定排版书与滚动模式仍显示 stable 数,同上游显式 OFF 用户状态) | 上游翻转默认时删除 |

## Update protocol

- 每个新的 dist 独有提交落库时:**在此追加一行** + 落盘 agent 记忆并标注清楚
  (owner 指令 2026-11-26)。
- 每次吸收上游 workflow 后:刷新上方吸收基线日期,并在 issue #1 留吸收记录。
- 补丁随上游版本升级时:先本地跑对应测试(tsc + 相关 vitest 文件),fork 推送
  一律 `--no-verify`(pre-push 全量 vitest 本机有上游存量失败),但绕钩子前必须
  本地验证过相关测试。
- agent 提交带署名:`Co-Authored-By: ZCode (GLM-5.3-Flash) <noreply@z.ai>`。
