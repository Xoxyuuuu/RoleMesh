# RoleMesh 交接说明（HANDOVER）

> 本文件用于交接与联调验收。**每次更新代码，请同步更新第二节「变更记录」和第三节「功能实现状态」**，
> 让接手的人一眼看出：这次改了什么、哪些还是 Mock、哪些已经接了真实接口。

---

## 一、当前交接版本

| 项目 | 内容 |
| --- | --- |
| 仓库 | https://github.com/Xoxyuuuu/RoleMesh |
| 分支 | `main` |
| 交接 commit | `c925f5b` Update README.md |
| 交接时间 | 2026-10-24 16:24（GMT+8） |
| 前端文件 | `index.html` —— 单文件、原生 HTML/CSS/JS，6379 行 / 346,201 字节 |
| 前端版本日期 | 2026-10-23 15:28（与本地最新工作副本内容逐字节一致，仅换行符差异） |
| 后端 | 暂无代码；仅 `openapi.yaml` 接口草案 |

**结论：前端为最新可演示版本；联调所需的后端契约文档随本次交付一并提供（见第四节待办）。**

---

## 二、变更记录

> 新改动加在表格**最上面一行**。写清楚「改了什么 + 影响哪个页面/文件」。

| 日期 | 改了什么 | 影响文件 |
| --- | --- | --- |
| 2026-10-24 | **新增登录 / 注册页**：手机号 / 邮箱双 Tab + 6 位验证码 + 60s 重发倒计时 + 协议勾选 + 「先逛逛」游客入口；顶栏登录态（用户胶囊 / 退出）；「我的」页同步登录名与脱敏账号。**仍为纯前端模拟**：验证码固定 `123456`，无任何网络请求 | `index.html`、`openapi.yaml`（新增 4 个 `/auth/*` 草案）、`HANDOVER.md` |
| 2026-10-24 | 远程重建提交历史，推「前端 + 页面文档（筛选后）」；README 精简两版 | `index.html`、4 份页面文档、`README.md` |
| 2026-10-23 | 前端整体视觉改版（莫兰迪配色）；新增/调整发起项目、申请、邀请、消息、画像相关交互 | `index.html` |
| 2026-10-23 | 新增后端契约文档：接口规格（由前端反推）、状态机与校验规则、契约字段对齐表、动态数据清单；`openapi.yaml` 落 23 个接口草案 | `openapi.yaml`、4 份后端文档 |
| 2026-10-20 | 前端 demo 与项目文档更新，补 docx 版本 | `index.html`、文档 |
| 2026-10-17 | 初始提交：前端 demo + 字段映射文档 | `index.html`、`字段交接-前端映射表.md` |

---

## 三、功能实现状态（Mock / 真实接口）

**全局结论：当前 0 个页面接入真实接口，全部为前端内置静态数据。**

判断依据（可直接复核）：`index.html` 中不存在任何 `fetch(` / `XMLHttpRequest` / `axios` 调用，
数据全部是文件内 `const PROJECTS`、`const TALENTS`、`let MY_APPLICATIONS` 这类写死的数组。
**唯一例外是登录态**：用 `sessionStorage` 的 `rm.session` 记住是否登录（刷新不丢），
但这只是前端本地标记，不是真实鉴权——手改这个值就能"登录"。其余业务数据仍在内存里，**刷新即重置**。

| 页面 / 功能 | 数据来源 | 状态 | 对应接口（`openapi.yaml`） |
| --- | --- | --- | --- |
| 首页 · 项目推荐列表 | `PROJECTS` 静态数组 | 🟡 Mock | `GET /projects`、`GET /projects/{id}` |
| Mesh · 人才匹配 | `TALENTS` 静态数组 | 🟡 Mock | `GET /talents`、`GET /talents/{id}` |
| 项目详情（角色/成员/动态） | `PROJECTS` + `ACTIVITIES` | 🟡 Mock | `GET /projects/{id}`、`GET /projects/{id}/activities` |
| 发起项目 / 编辑项目 | 表单写入内存变量 | 🟡 Mock（无持久化） | `POST /projects`、`PUT /projects/{id}` |
| 申请加入（提交 / 撤回） | `MY_APPLICATIONS`、`realApplications` | 🟡 Mock（仅内存） | `POST /projects/{id}/applications`、`DELETE /applications/{id}` |
| 收到的申请（同意 / 拒绝） | `RECEIVED_APPLICATIONS` | 🟡 Mock | `GET /projects/{id}/applications`、`/agree`、`/reject` |
| 邀请人才（发起 / 撤回 / 接受 / 拒绝） | `MY_INVITATIONS`、`RECEIVED_APPLICATIONS` | 🟡 Mock | `POST /invitations`、`DELETE /invitations/{id}`、`/accept`、`/reject` |
| 消息列表 / 聊天发送 | `CONVERSATIONS` | 🟡 Mock（发送只追加到内存） | `GET /conversations`、`GET/POST /conversations/{id}/messages` |
| 我的 · 个人画像 | `MY_PROFILES` | 🟡 Mock | `GET /me/profiles`、`POST /me/profiles`、`PUT /me` |
| 我的 · 我发起/我参与 | `MY_OWN_PROJECTS`、`MY_JOINED_PROJECTS` | 🟡 Mock | `GET /me/projects/owned`、`/me/projects/joined` |
| 我的 · 数据概览 | 由上述数组前端聚合 | 🟡 Mock | `GET /me/summary` |
| 登录 / 注册（手机号 / 邮箱 + 验证码） | 前端校验 + 固定验证码 `123456`；登录态存 `sessionStorage` | 🟡 Mock（无鉴权，可绕过） | `POST /auth/code`、`POST /auth/login`、`POST /auth/logout`、`GET /auth/me` |
| 游客模式（先逛逛） | 仅置本地标记，不做接口 | 🟡 Mock | 无（接真实接口后可取消或保留做试用） |

统计：`openapi.yaml` 共 27 个路径 / 32 个接口操作，**已接入 0 个**。

图例：🟢 已接真实接口 · 🟡 Mock（前端假数据）· ⛔ 未实现

---

## 四、联调前待补清单

1. **5 份后端契约文件需一并提交**：`openapi.yaml`、`后端接口规格-由前端反推.md`、
   `后端业务规则-状态机与校验.md`、`契约字段对齐表.md`、`需后端接口的动态数据清单.md`。
   → 不提交的话，后端只看到前端代码，无法对齐字段和状态机。
2. **前端没有请求层**：建议新增 `api.js` + 一个 `USE_MOCK = true` 开关，把图上的 Mock 调用统一收口，
   后续接真实接口只改开关，不动页面逻辑。登录页的替换点已经留好：`authSendCode()` 换成 `POST /auth/code`，
   `authSubmit()` 里的 `setTimeout` 换成 `POST /auth/login`。
3. **登录还没有接入后端**：当前验证码写死 `123456`、登录态存 `sessionStorage`，手改即可伪造。
   接 `/auth/login` 后要改成存 token，并给其余接口带上 `Authorization: Bearer`。
4. **无业务数据持久化**：刷新即重置，联调时会看不出「数据有没有真的存进去」。
5. **契约字段待拍板**：`契约字段对齐表.md` 中带 ⚠️ 的行（状态枚举映射、档位→数值、硬性/加分技能拆分等）
   需后端与产品确认后才能定 DTO。
6. **本地 git 与远程历史已分叉**：本地是 `edfc930 → e9da455 → 63884b1`，远程是 `aa1f98e → b7be100 → c925f5b`，
   直接 `git push` 会被拒。建议先备份本地工作区，再以远程 `main` 为准重新关联（`git fetch && git reset --hard origin/main`）。
7. **Word 版本未入库**：`契约字段对齐表.docx`、`需后端接口的动态数据清单.docx`、`页面功能说明-大白话版.docx`
   仅本地留存，内容与对应 `.md` 一致，需要时再补。

---

## 五、更新约定

- 每次提交前，先更新本文件：**第二节加一行变更记录**；若某个功能从 Mock 换成真实接口，**第三节对应行改成 🟢 并写上接口路径**。
- commit message 统一格式：`feat|fix|docs(范围): 说明`，例如 `feat(messages): 接入会话列表接口`。
- 只改样式/文案的提交，也要在第二节留一行，注明「无接口变化」。
