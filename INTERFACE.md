# 内容工坊 × 独立采集工具 接口

协议：content-studio.collection  
版本：1.0  
网页版本：2.0.0  
更新：2026-10-20

## 边界

采集工具在独立项目运行，网页只导入已经生成的JSON。网页不会打开采集链接、下载图片、补全正文或启动采集。只支持手动文件/粘贴导入；无自动同步API。

来源作者、来源账号与网页自己的发布账号是不同概念。导入时由用户选择共享、经营资料或账号归属，不让文件替用户更改自己的账号配置。

## 顶层字段

- schema（必填）：固定content-studio.collection。
- schemaVersion（必填）：字符串1.0。未知版本整批拒绝，不猜测兼容。
- batchId（必填）：生产者稳定的批次ID，最长300字符。
- collectedAt（必填）：带时区的ISO 8601日期时间，例如2026-10-20T12:00:00Z。
- producer（必填）：name与version两个非空字符串。
- sample（可选，默认false）：整批是否示例；只要批次或条目标示为示例，入库素材就保留示例标识。
- items（必填）：1至500条。文件与粘贴文本单次最大8MB。

## 每条item

- externalId（必填）：来源内稳定ID，不能每次随机变化。与platform和type联合去重。
- type（必填）：note笔记、account来源账号资料、image图片元数据、text文字资料。
- platform（可选）：小红书（默认）、抖音、X。本轮重点小红书。
- title（必填）：非空，最长300字符。
- url（可选）：来源http/https链接，缺失为空字符串；非网页协议拒绝。
- content（可选）：实际采集的正文、转写或说明，最大40000字符；没采集到就留空，不能把标题扩写成全文。
- author（可选）：来源作者名称，不是网页目标发布账号。
- publishedAt（可选）：带时区ISO日期，未知留空。
- metrics（可选）：views、likes、comments、saves、shares。已知值为非负数字，未知省略或null，0仅表示明确测得为零。
- tags（可选）：最多50个字符串标签。
- rights（可选）：unknown、owned、authorized、reference（默认，仅作研究参考）。不得把公开可见等同于有权搬运。
- sample（可选）：本条示例标记。

顶层与item允许扩展字段；item原始对象保存到素材raw。采集软件的收藏夹、设备、原始图片URL等可作为扩展字段存放，但本版不自动处理。不要包含登录Cookie、密码或密钥。

## 导入与重复处理

1. 解析与校验整批，不写入。
2. 用户选择归属，查看新增、重复、冲突、示例数量和缺失字段。
3. 校验账号、经营资料、任务与素材的引用关系。
4. 确认导入时检查本地数据是否自预览后改变；如改变则重新预览。
5. 备份导入前原记录，然后一次写入。格式、版本、引用或保存失败均不部分入库。

相同platform、type、externalId作为同一来源实体。相同内容重复导入跳过；原始条目变化则新增“采集观察”历史快照，不覆盖人工编辑的素材正文与归属。历史观察可在素材历史中查看，不假装已自动更新全部页面指标。同一批次重复执行不会重复入库。

## 完整度

素材完整度以正文、来源链接、作者、发布时间、采集时间、至少一个已知互动指标六项计算。它仅表示资料完备程度，不是内容质量、真实性或爆款概率评分。

## 网页备份2

schema：content-studio.backup；schemaVersion：数字2；appVersion：2.0.0；exportedAt：ISO时间；records：完整StudioRecord数组。

StudioRecord有id、kind、data、version、createdAt、updatedAt。kind包括profile、topic、reference、draft、asset、metric，以及business、account、material、task、research、review、imagePlan、snapshot、importBatch。

关键关联：account.businessId；material.scope/businessId/accountIds；task.accountId/businessId/materialIds；draft.taskId/accountId/materialIds；imagePlan.taskId/draftId；review.taskId/materialIds/fromReviewId（fromReviewId在新任务中）；snapshot.recordId/recordVersion/value。

同时接受旧版schemaVersion为数字1的records备份及原始记录数组。恢复为合并：同ID同内容跳过，不同内容另存ID，并重映射导入数据的依赖。不会覆盖当前已有记录。未知记录类型、无效日期、重复ID或断裂引用整批拒绝。旧版已经删除的选题/草稿引用作为legacyOrphans保留在升级基线，不伪造丢失内容。

本地原存储键content-studio:github:v1不变。升级前备份键追加:before-v2，最近一次导入前备份追加:before-import。页面可导出完整备份及这些原始记录。快照只读，不直接回滚；要恢复时通过备份合并预览。

## 联调文件

- collection-example.json：明确标识为示例，不含真实采集。
- collection-schema.json：JSON Schema Draft 2020-12，可供采集项目校验输出。

协议修改须升级版本、记录变更、保留迁移测试，并由两个项目联调；不依赖共享代码目录或同一个浏览器会话。
