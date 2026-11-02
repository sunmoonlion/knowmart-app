# 2026-11-02 第二轨：WP-10 动态清单契约

feat/dynamic-methods 后端 a2333c49b4a83f5be03b49654dde3986ee43bb95，新增 v3 动态清单外层及父仓 schema；v2 schema 逐字不变、规范摘要与基线相同。使用原交付/绑定/版本不可变流程，完整方法规则由 knowbench 校验。动态执行尚未开放，不是可上线功能。

完整本地 PostgreSQL 回归 531 passed / 5 skipped，/tmp/luna-wp10-mart-full.txt；改动文件 ruff/pyright 通过。全仓既有 4 个类型、17 个 lint 问题，隔离解包原基线逐条重跑无新增；见后端 CHECKPOINT.md。

前置 731cf3d 已将 jsonschema 4.26.0 纳入 dev 与锁，清单实例 schema 校验不再因缺库静默跳过；已有运行依赖版本不变。

`uv run --locked pytest -q tests/test_knowbot_manifest_v2.py tests/test_knowbot_contract.py`：23 passed；相关文件 ruff 通过。输出 /tmp/luna-dynamic-contract-gates-mart.txt。这是现有契约回归，不是动态能力或生产业务验收。

knowlab/knowbench 的 v3 锁同步交付，不能靠伪造非空 workflow 绕过 v2 骨架。未发布；不修改 qwen 正在发布的候选或部署源。后续接任务计划和执行，再重新孵化作者确认的财务专家版本。
