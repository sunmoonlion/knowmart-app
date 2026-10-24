# knowmart 提供的契约

本目录是 knowmart 对外契约的唯一可改真源（SDD 0027）。调用方在自己父仓的 `contracts/` 放一份锁（契约名、主版本、schema 路径、sha256），升级时对比摘要。

## 分身清单 `knowbot/v1`

- schema：`knowbot/v1/knowbot-manifest.schema.json`，摘要见同目录 `contract-manifest.json`
- 生成：由 `knowmart-backend` 的外层模型 `app/domain/knowbots/manifest.py` 生成（`python -m app.domain.knowbots.schema`），后端测试核文件与模型逐字一致，不要手改
- 生产方：knowlab（工程化、验收后交付）；存放方：knowmart；执行方：knowbench 工作台

只管**外层**：格式标识 `schema`、名字与说明、版本、`data_needs`（数据需求声明：请专家的人要提供哪几样数据，SDD 0042 第五节）、`data_bindings`（旧格式：这一版能读的用户数据）、`rule_sources`（每条判断规则出自用户原文还是用户确认过的改写，`C-L1`）。`pack` 是可执行部分（步骤、验收、底稿的形状），由执行方工作台拥有、按自己的模型完整校验，这里只要求它是带 `pack_id`、`version`、非空 `workflow` 的对象。

JSON Schema 表达不了的跨字段规则以 knowmart 交付时的校验为准：`pack.version` 必须等于外层 `version`；同一数据集一版里只能绑一次；声明里每样的 `key` 不重、`max_count` 不小于 `min_count`、各样 `max_count` 合起来不超过 64、`tables` 只能写给 Excel / CSV；绑定上的 `need` 必须是声明过的 `key`；**写了声明的清单交到 knowmart 时不带绑定**（专家不绑数据，测试绑定只留在 knowlab）；外层规范 JSON 不超过 1 MB。

`data_needs` 为空、绑定上没写 `need` 时，这两样不进规范 JSON：2026-10-23 加这两样之前交付的清单，摘要一字不变。摘要是外层规范 JSON（键排序、紧凑分隔、UTF-8）的 sha256，写作 `sha256:<十六进制>`；同一分身同一版本只认一份内容。

不兼容的改动开新主版本目录，并留双版本并存期。

## 交付与停用（内部接口，服务关系 `knowmart:deliver`）

knowlab 调，只认 knowlab 的服务身份（SDD 0027、0037 第十八之二节）：

- `POST /internal/v1/knowmart/deliveries`：交一版过了两道门的专家（`delivery_id`、`owner_subject`、`knowbot_ref`、`manifest`）。按交付单号幂等：同一单号内容一样回 200，不一样回 409。版本号按清单里的记（knowlab 测试通过时定，可以有空号）。可带 `request_public: true`（knowlab 运营审核定了公开）：卖家发布主体与类目资质都过的，这一位进上架审核，不过的留自用、不报错；回包多 `listing_status`。
- `POST /internal/v1/knowmart/owners/{owner}/knowbots/{knowbot_id}/versions/{version}/stop`：knowlab 停用某一版（那边用户或运营停的）。只认这个所有者的，别人的与没有的都是 404；停过的再停照样 204。
- `PUT /internal/v1/knowmart/owners/{owner}/sources/{knowbot_ref}/unlisted/{version}`，body `{"outcome": "declined" | "rejected"}`：knowlab 交来没上架的那一版的结局（用户不要 / 运营审核没过），详情页写明空号那一版。按版本逐条写，结局是终态，重发、乱序都无害（Astra 评审 F08）；这个引用已交付过而主人不是这个人的回 409。
