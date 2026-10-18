---
author: "Sanjin"
handle: "@EqbymCi"
source: "https://x.com/EqbymCi/status/2106279257488216334"
date: 2026-10-03T07:04:42.000Z
type: article
likes: 65
reposts: 14
replies: 4
bookmarks: 123
views: 7007
---
# Verification Engineering 时代

*By Sanjin (@EqbymCi)*

![Banner](<Verification%20Engineering%20时代.assets/image-01.jpg>)

在 AI Native 组织里，Verification 正在成为最关键的工程环节之一，也是 loop engineering / graph engineering 能否真正闭环的基础。

AI 几分钟就能生成上千行代码，但接下来的问题越来越突出：

- 如何用确定性的检查，验证 AI 生成的代码是否符合预期？
- 如何设计高效的 CI/CD，让验证速度跟得上代码生成速度？
- 如何让 lint、type check、format、测试跑得更快，并持续监控耗时、失败率和误报？
- 如何设计真正有用的 review bot，让它发现问题，并给出可以验证的证据？

以往人和人结对编程，一个人一天提交十几次代码已经不少了。现在，从人和 agent 协作，到 agent 和 agent 协作，开发高峰期十几个 agent 同时提交代码。代码量、提交频率和验证需求一起上涨。

过去还能忍受的几分钟检查、偶发的测试失败、重复执行的 CI，都会被放大。一次慢几分钟，乘上几十次提交和多个 agent，就是大量等待和资源消耗。

这也让 Rust 工具链、增量检查、缓存和并行执行的价值变得更加明显。

但在一些传统公司，Verification 仍然缺少明确的负责人。考核更关注功能交付，人力又不断压缩。优化流水线、维护测试、治理误报这些事，很容易被当成“有空再做”的细节。

我认为，这已经值得单独设一个 Verification Engineer 岗位：持续优化验证系统的速度、覆盖范围和可信度。团队规模较小时，也至少应该明确这项职责由谁承担。

高喊 AI Native 转型，却连最基本的 infra 都没人维护，最后只会让代码生成得更快，问题堆积得更多。

以下是我在开发中的一些实践，以及值得进一步投入的方向。

**1. 给工具链“锈化”，缩短反馈时间**

例如使用 Biome 处理 lint 和 format，以及采用 Rolldown / Oxc 的 Vite 工具链。工具替换的收益要在自己的项目中测量，同时检查规则兼容性和迁移成本。可参考 [Biome](https://biomejs.dev/) 和 [Vite 的 Rolldown 集成说明](https://v7.vite.dev/guide/rolldown)。

还要关注缓存、增量检查和重复任务。给 agent 一个统一的验证入口，按依赖关系选择变更模块及受影响的下游，并复用已经验证过的结果。

缓存的输入需要覆盖依赖、配置和相关环境条件，同时保留全量回归兜底。整条流水线的反馈时间，才是最终需要优化的对象。

**2. 给程序化 E2E 分层**

按业务价值、变更风险和分支阶段划分验证范围。高频提交优先跑核心链路，完整回归放到合适的合并、发布或定时任务中，避免每次提交都承担全量测试成本。

多个 agent 分别改前后端时，可以把接口契约纳入检查。关键业务则需要明确哪些约束必须始终成立，例如重复请求不能重复扣款、无权限用户不能访问受保护资源。

这些约束应该成为可执行的验收条件，帮助测试覆盖边界情况。

**3. 定期让 agent 做真实操作巡检**

设置 cron job，让 agent-browser 模拟用户点击、操作，并保留录像。我在开发中会给各个微服务配置定期巡检，使用公司自部署、成本较低的视觉模型。

程序化 E2E 覆盖明确的预期，agent 巡检补充更接近真实使用的探索。两者配合，能提供不同角度的反馈。

" dogfood "

巡检发现的问题，还应尽量沉淀成可复现的步骤和稳定的回归用例，让一次探索的收益延续到后续每次提交。

**4. 减轻开发和测试环境的负担**

例如在适合的本地开发、测试场景中尝试 PGlite，减少独立 PostgreSQL 实例的启动和维护成本。PGlite 将 PostgreSQL 编译为 WASM，可作为嵌入式数据库运行，具体能力可以参考 [PGlite 官方说明](https://pglite.dev/)。

涉及生产数据库特性、并发行为和部署差异的部分，仍然用真实 PostgreSQL 验证。

当多个 agent 同时开发时，环境启动速度、资源占用和隔离能力，也会直接影响整个验证系统的吞吐量。

**5. 验证合并后的代码**

十几个 agent 各自的分支都是绿的，合在一起未必能跑。

Merge queue 会把待合并改动与最新主干、队列前面的改动一起执行所要求的检查。对于高频并行提交，这能提前发现组合后的问题。使用 GitHub Actions 时，还需要让相关工作流响应 merge_group 事件。参考 [GitHub merge queue 文档](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue)。

同一个 PR 有新提交时，也应考虑取消已经过时的检查，把计算资源留给仍有决策价值的版本。合并队列与发布任务则需要单独设计取消规则。

**6. 验证“测试能不能发现错误”**

实现和测试都由同一个 agent 根据同一套理解生成，可能一起理解错需求。

验收标准应该来自明确需求、历史故障和可信的独立参照。另一个 agent 来写测试有帮助，但不能保证没有相同盲点。

关键模块可以引入 mutation testing：故意改错一个条件或运算符，看看测试能否抓住。如果代码变错了，测试依然通过，就需要检查断言是否有效、边界是否覆盖。参考 [Stryker 对 mutation testing 的介绍](https://stryker-mutator.io/docs/)。

这类检查有额外计算成本，可以优先放在关键模块或定时任务中。覆盖率之外，还需要知道测试对错误有多敏感。

**7. 给 Verification 本身做可观测性**

我建议至少持续看这些指标：

- 反馈耗时 p50 / p95，以及排队时间；
- 缓存命中率和重复执行成本；
- 首次通过率、重试后通过率和重试成本；
- 不稳定测试的数量、负责人和修复进度；
- review bot 的有效问题比例与误报情况；
- 上线后仍然漏出的故障。

这组指标是对验证系统的管理建议。采集流水线、任务和运行结果时，可以参考 [OpenTelemetry 的 CI/CD 语义约定](https://opentelemetry.io/docs/specs/semconv/cicd/)。

尤其要把首次通过与重试后通过分开统计。Playwright 就区分了 passed、flaky 和 failed：首次失败、重试通过的测试会归为 flaky。参考 [Playwright 重试文档](https://playwright.dev/docs/test-retries)。

对不稳定测试，应记录原因、负责人和修复期限；临时隔离也要有退出条件。否则 agent 很容易学会反复重跑 CI，大家逐渐不再相信红灯。

验证的目标，是更快拿到可信反馈，并据此决定能否继续合并、发布和扩大流量。

当代码生成越来越便宜，如何快速、可靠地验证这些代码，正在成为新的工程瓶颈。

AI Native 的基础设施投入，也应该跟着这个瓶颈转移。