# knowmart-app

**knowmart**：分身（knowbot）的存放与「请」。现在只做**私有模式**——每个用户只看得到、只请得到自己的分身，一个人可以有多个；后端强制按所有者过滤，界面叫「我的分身」。对外上架给别人用、开发者资格与审核是持牌以后的事。

由 [tpl-app](https://github.com/sunmoonlion/tpl-app) 实例化（tpl-app 4b19daf，2026-10-16）：

```text
knowmart-app/
├── knowmart-backend/          FastAPI 后端（Admin / Web / Internal 三面一个工程）
├── knowmart-web-frontend/     Next.js 网页端
└── knowmart-admin-frontend/   Next.js 管理端
```

设计、规则、部署都在并列的 `k8s` 仓：先读 `../k8s/docs/project-guide/overall-architecture.md`，本应用的设计见 `../k8s/docs/dev-investment-agent/tree-build/SDD/modules/0018-advisor-market.md`（私有模式）与 `0026-knowlab.md`「一之二」；动代码前读 `../k8s/docs/dev-investment-agent/tree-build/rules/constraints.md`（含合规 `C-L*`）。构建与部署由 k8s 仓的发布链做，本仓不放部署脚本。

**现在只是空应用**：模板的骨架、认证与运行角色，没有业务功能。
