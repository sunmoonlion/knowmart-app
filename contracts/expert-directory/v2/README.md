# 专家公开目录 v2

提供方：knowmart `app/domain/knowbots/directory.py`，生成 `render()`；消费方：knowbench `KnowmartCatalog.snapshot()` 与 `expert_match.DirectorySnapshot`。

沿既有内部目录接口增加 `directory_version=2`；未传仍返回 v1。新响应必须完整，否则返回服务错误，不能把截断或供应商故障转成“没有专家”。提供精确专家版本、摘要、当前价格与公开能力声明；不公开方法正文、验收规则、输入绑定和 capability_bindings。新动态包使用 capabilities/v1，旧包如实使用 scope/v1。

当前边界是完整目录最多 1000 项，knowbench 匹配上下文最多 100000 字符；超过时报错，后续索引优化不能静默丢候选。手动指定时由 knowbench 钉住目录中的唯一版本后核范围。

此契约上线必须与 knowbench v2 客户端同版，原固定专家入口保持既有契约。工程回归不等于真实专家发布或模型语义验收。
