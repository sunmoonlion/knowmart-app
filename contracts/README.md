# knowmart 提供的契约

本目录是 knowmart 对外契约的唯一可改真源（SDD 0027）。调用方在自己父仓的 `contracts/` 放一份锁（契约名、主版本、schema 路径、sha256），升级时对比摘要。

## 分身清单 `knowbot/v2`（SDD 0043 第三、四节）

- schema：`knowbot/v2/knowbot-manifest.schema.json`，摘要见同目录 `contract-manifest.json`；生成：`python -m app.domain.knowbots.schema 2`，后端测试核逐字一致
- 清单 v2 = 逻辑 + 作者挑选的数据（2026-10-28，decisions ⑭）：`bindings` 是作者挑选、交付时冻结的数据引用（资料、版本、摘要，要读法的钉读法，凭 `expert:<专家>@<版本>`），数据本身留在 knowbase；**没有数据需求声明**（每件委托要什么数据，由委托第 0 步用户自己的模型按专家逻辑和用户问题提出、用户确认，形状是 knowbase 的 `data-request/v1`，不进清单）；没有 `data_bindings`
- 跨字段规则以交付时的校验为准：同一份资料的同一版只绑一次；一份清单的绑定只冻结给一个专家版本
- 只管**外层**：格式标识 `schema`、名字与说明、版本、运行方、`bindings`、`rule_sources`（每条判断规则出自用户原文还是用户确认过的改写，`C-L1`）。`pack` 是可执行部分（步骤、验收、底稿的形状），由执行方工作台拥有、按自己的模型完整校验，这里只要求它是带 `pack_id`、`version`、非空 `workflow` 的对象，且 `pack.version` 等于外层 `version`；外层规范 JSON 不超过 1 MB
- 摘要是外层规范 JSON（键排序、紧凑分隔、UTF-8）的 sha256，写作 `sha256:<十六进制>`；同一分身同一版本只认一份内容
- 只收 v2（2026-10-28，decisions ⑰ 不留旧版兼容）：`knowbot/v1`（数据需求声明、knowlab 数据集绑定）契约已删，`parse_manifest` 不是 v2 就拒；以前交来的 v1 版本由迁移 `20261028_0010` 删掉，只剩 v1 版本的专家一并删，订单历史不动
- 不兼容的改动开新主版本目录

## 交付请求 `knowbot-delivery/v1`（0032 待定第一项、0043 第 4.3 节）

- schema：`knowbot-delivery/v1/knowbot-delivery.schema.json`，摘要见同目录 `contract-manifest.json`；生成：`python -m app.domain.knowbots.schema delivery`（真源 `app/domain/knowbots/delivery.py`），后端测试核逐字一致；knowlab 锁 `knowlab-app/contracts/knowmart-delivery-v1-lock.json`
- 字段：`delivery_id`、`owner_subject`、`knowbot_ref`、`manifest`（清单 v2）、`request_public`、`category`
- `category`：平台人员在 knowlab 交付时按方法内容选的二级类目，取值是 `app/domain/knowbots/catalog.py` 的类目表去掉「未分类」；带了就记在专家上，不带的不动（新专家照旧未分类，不能公开）
- 收清单时硬查所绑数据（清单 v2 的 `bindings`）：问 knowbase `POST /api/internal/v2/user-data/service/expert-bindings/check`（服务关系 `knowbase:user-data-purchase`），每份带凭的 `expert:<专家>@<版本>`，knowbase 连冻结时的许可一起核（签请专家的授权看的是冻结的那一档）。自用要每份还在、作者还能用（「只许本人用」的也可以）；公开要每份都「可以经专家给别人用」以上。不行回 409，`code` 是 `binding_gone`、`binding_disabled`、`binding_deleted`、`binding_not_shareable` 之一；knowbase 没配、连不上回 503
- 自用转公开：同一版、同一份清单，换一个交付单号再交一次带 `request_public: true`，重查许可过了就放开给别人；已经放开了的同一份再来当重放回 200（knowlab 超时重试）；内容不一样照旧 409 `knowbot_version_exists`
- 在售那一版所绑数据失效（删、禁用、许可改低）：后台每 10 分钟核一次（`app.tasks.market.check_bindings`），转「待处理」（版本状态 `held`，原因同上四个代码），别人请不了、主人照样能请，作者在小店「我的专家」里看得到原因；数据好了自动回在售

## 调用方的锁

本仓现在不锁别家的契约（2026-10-28 起清单 v2 不再带数据需求声明，原来锁 knowbase `knowbot-data-needs/v2` 的那把删了）。

小店调 knowbase `user-data/v2`（SDD 0043 第 4.3 节）：写知识条目时凭用户本人令牌读 `GET /api/internal/v2/user-data/columns/knowmart`（用户在 knowbase 总栏交给 knowmart 的资料），成交后以服务关系 `knowbase:user-data-purchase` 调 `POST /api/internal/v2/user-data/purchases` 给买家建使用权（不复制）。用到的字段少，没有放锁；`user-data/v1` 已不再调用。

## 交付与停用（内部接口，服务关系 `knowmart:deliver`）

knowlab 调，只认 knowlab 的服务身份（SDD 0027、0037 第十八之二节）：

- `POST /internal/v1/knowmart/deliveries`：交一版过了两道门的专家（请求体是契约 `knowbot-delivery/v1`，见上一节）。按交付单号幂等：同一单号内容一样（含申请公开与否、类目）回 200，不一样回 409。版本号按清单里的记（knowlab 测试通过时定，可以有空号）。可带 `request_public: true`（knowlab 运营审核定了公开）：只有这样交来的版本才给别人看、给别人请，没带的那一版只给主人（Fable 评审 P2-2）；卖家发布主体与类目资质都过的，这一位进上架审核；没进的回包 `listing_status` 照实写为什么：`category_needed`（卖家还没在 knowmart 选类目）、`not_qualified`（资质没过）、`shop_suspended`（卖家小店被平台暂停）。
- `POST /internal/v1/knowmart/owners/{owner}/knowbots/{knowbot_id}/versions/{version}/stop`：knowlab 停用某一版（那边用户或运营停的）。只认这个所有者的、knowlab 交来的（引用以 `knowlab-` 开头），别的与没有的都是 404；停过的再停照样 204。
- `POST /internal/v1/knowmart/owners/{owner}/knowbots/{knowbot_id}/versions/{version}/withdraw-public`：knowlab 收回某一版的公开（自用转公开撤回撞上已收下时调）。只撤掉这一版的 `public_allowed`，保留作者的自用上架；这一位没有别的可公开版本且条目非平台下架时，一并退回自用（`listing_status='self'`、`visibility='private'`）；撤过的再撤照样 204。

- `GET /internal/v1/knowmart/owners/{owner}/sources/{knowbot_ref}/versions`：knowlab 取回它交来的各版现在怎样（`version`、`status` 为 active / held / stopped、`held_reason`、`public_allowed`），在 knowlab 专家栏照实写待处理与原因；只收 `knowlab-` 开头的引用，别人的回空。
- `PUT /internal/v1/knowmart/owners/{owner}/sources/{knowbot_ref}/unlisted/{version}`，body `{"outcome": "declined" | "rejected"}`，只收 `knowlab-` 开头的引用：knowlab 交来没上架的那一版的结局（用户不要 / 运营审核没过），详情页写明空号那一版。按版本逐条写，结局是终态，重发、乱序都无害（Astra 评审 F08）；这个引用已交付过而主人不是这个人的回 409。


## 内部接口：AI 请专家与需求（服务关系 `knowmart:catalog`、`knowmart:demand`、`knowmart:summon`）

- `GET /internal/v1/knowmart/owners/{owner}/catalog`：执行方以服务身份替用户找专家，只回本人可用（自用或公开且未停店）的专家版本列表。
- `POST /internal/v1/knowmart/owners/{owner}/demands`：执行方以服务身份替用户发需求，匿名进入需求大厅。
- `POST /internal/v1/knowmart/knowbots/{knowbot_id}/versions/{version}/summon`：执行方为用户请专家时调用，系统记一笔成交（同一用户同一版按部分唯一索引只记一次，价格按条目标价，自己请自己的不记）。
