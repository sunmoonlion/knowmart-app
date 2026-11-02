# 2026-11-02 第二轨：契约验证门

feat/dynamic-methods 当前只完成契约测试开发依赖，尚未扩展动态清单。后端 731cf3d 将 jsonschema 4.26.0 纳入 dev 与锁，清单实例 schema 校验不再因缺库静默跳过；已有运行依赖版本不变。

`uv run --locked pytest -q tests/test_knowbot_manifest_v2.py tests/test_knowbot_contract.py`：23 passed；相关文件 ruff 通过。输出 /tmp/luna-dynamic-contract-gates-mart.txt。这是现有契约回归，不是动态能力或生产业务验收。

后续与 knowlab 编译、knowbench 运行契约一起处理动态包版本和新旧运行方兼容，不能靠伪造非空 workflow 绕过 v2 骨架。未发布；不修改 qwen 正在发布的候选或部署源。
