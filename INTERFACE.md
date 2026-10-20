# 内容工坊 × 独立采集工具 接口

协议：content-studio.collection  
版本：1.0 / 1.1（兼容读取）  
网页版本：2.1.0  
更新：2026-10-20

## 边界

采集工具在独立项目运行，网页只导入已经生成的JSON。网页不会打开采集链接、下载图片、补全正文或启动采集。只支持手动文件/粘贴导入；无自动同步API。

来源作者、来源账号与网页自己的发布账号是不同概念。导入时由用户选择共享、经营资料或账号归属，不让文件替用户更改自己的账号配置。

## 顶层字段

- schema（必填）：固定content-studio.collection。
- schemaVersion（必填）：字符串1.0或1.1。未知版本整批拒绝，不猜测兼容。
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


## 第二轮扩展：采集结果 1.1

仍使用 content-studio.collection。1.0 文件继续有效；新采集工具建议输出 schemaVersion 字符串 1.1。

- items[].authorId：可选，小红书作者稳定 ID（最长300字符），作品用于匹配主页 externalId；不能把显示名称当稳定ID。
- items[].format：可选，建议“图文”“视频”“未知”，未提供时显示体裁未提供。
- items[].metrics.followers：可选，非负数或 null。对于 type=account 表示采集时账号粉丝数；对于笔记如提供则表示笔记涨粉，不用于判断作者低粉。
- 账号条目 externalId 为主页 ID。笔记 externalId 为笔记 ID，authorId 为所属作者。collectedAt 为真实采集时点，publishedAt 为作品实际发布时间。
- 只关联用户确认属于该博主的素材，不凭同名作者自动归并。最新采集观察指标供对标研究使用，人工正文与归属保留。
- 对标默认只分析已导入样本；缺少粉丝数不参与低粉筛选，缺少发布日期不参与近期筛选。粉丝采样时点与作品发布时点不同，不推定发布时粉丝数。

## 主页采集请求 1.0

网页导出给独立采集项目；不是网页内执行采集。schema 固定 content-studio.collection-request，schemaVersion 字符串1.0。

```json
{"schema":"content-studio.collection-request","schemaVersion":"1.0","requestId":"unique-id","requestedAt":"2026-10-20T12:00:00Z","platform":"小红书","profileUrl":"https://www.xiaohongshu.com/user/profile/example","authorId":"example","range":{"recentDays":30},"include":["account","notes"],"expectedResultSchema":"content-studio.collection/1.1"}
```

采集工具按授权和站点规则生成 collection 1.1；网页仍通过文件预览、选择归属、合并导入，随后在对标页关联作品。网页不根据请求文件伪造采集成功。

## Excel与复盘观察

新增 performance 记录，属于用户自己的 accountId，不要求 draftId。字段包含 mode(initial/review)、periodStart/periodEnd、capturedAt、origin、files、notes、summary、trends、audience、images、warnings、digest、aiAnalysis、suggestions、verified、sample。

- notes 保存标题、发布日期、体裁、规范指标及 raw 原始列；所有数字缺失为 null，不补0。ctr 统一为百分数，例如源0.088显示8.8；源12%显示12。原始列仍保留。
- summary 是 name/value/unit 的周期汇总；trends 是 name/date/value/unit 的每日序列；audience 是画像与来源占比。三个集合互不相加；平均点击率不把每日比率简单相加。
- 笔记表头可位于第一行之后，支持“笔记标题”“首次发布时间”“体裁”“曝光”“观看量”“封面点击率”“点赞”“评论”“收藏”“涨粉”“分享”“人均观看时长”“弹幕”等。未知列保留在raw；未识别工作表明确提示，未入库。
- Excel仅支持.xlsx；单文件8MB、一次最多8个文件、每表5000数据行、最多30个工作表。识别后需要核对再入库，不存原始工作簿字节。
- 没有趋势表时不把笔记发布时间冒充统计周期，由用户对照后台填写；趋势存在时按实际日期提出周期，仍需核对。
- 截图每批最多6张，单原件12MB，压缩副本合计1.2MB；完整备份包含副本，非原始图片。历史快照只存图片名称说明。
- 同一账号相同观察指纹不重复；不同导出时点只要数值变化就另存，不覆盖以往观察。指纹不含文件名或AI文本。
- 新任务及复盘档案可带reportId，必须与目标账号一致，备份冲突恢复时与其他关联一起重映射。

## 截图识别结果 1.0

结果只进当前预览，用户必须核对数字、单位、周期再确认入库。截图本身不是可核验的OCR文本；模型不能确定的值返回null，曲线无标注时不得猜数。

```json
{"schema":"content-studio.screenshot","schemaVersion":"1.0","summary":[{"name":"近7日总粉丝数｜文件名","value":null,"unit":"人"}],"audience":[{"name":"女性比例｜文件名","value":null,"unit":"%"}],"notes":[],"analysis":"证据、观察、限制与建议","suggestions":["一个可验证的实验"]}
```

summary/audience条目为name/value/unit；百分数89%写89。notes结构为title、publishedAt、format、metrics（非负数或null），可带raw原始字段。不同截图周期需写入指标名称，避免近7日和近30日合并。仅识别截图不能恢复图中未标注的每日明细。

## AI复盘／对标结果

```json
{"analysis":"包含笔记标题、指标或证据ID的观察；推测与限制明确区分","suggestions":["可执行操作与衡量方法"]}
```

默认外部AI复制提示词和材料包，再导回JSON。可选兼容Chat Completions完整HTTPS地址、模型和密钥，用户点击发送后才提交本次材料；截图用image_url内容块，需要视觉模型。配置只在页面内存，不导出、不持久化；服务需允许跨域，未配置不自动发送。对标AI结果为待人工判断的建议，复盘数据需勾选核对才能保存。

## 飞书导出与桥接请求 1.0

默认生成含BOM的UTF-8 CSV，可在飞书多维表格手动导入；CSV不等于自动写入。字段为博主、主页、账号粉丝数、作品标题、作品链接、发布日期、点赞、收藏、评论、样本标记；对可能的公式开头做转义。

可选用户自己的HTTPS桥接服务：网页POST application/json。

```json
{"schema":"content-studio.feishu-sync","schemaVersion":"1.0","requestId":"uuid","competitor":{"id":"stable-local-id","externalId":"author-id","profileUrl":"https://www.xiaohongshu.com/user/profile/example","name":"示例","followers":null,"sample":true},"notes":[{"id":"stable-local-id","externalId":"note-id","title":"示例作品","sample":true}]}
```

实际请求的competitor与notes包含当前档案和选中素材的全部字段。桥接服务负责目标表配置、飞书授权、字段映射、权限、CORS和写入回执；按externalId/本地id执行幂等更新，避免重复新增。不要在静态网页或公开仓库保存app_secret/token。

成功回执必须是HTTP成功且JSON包含provider=feishu与synced=true；否则网页显示失败或未确认，不自动重试。用户应到目标表核对。未配置服务不会向飞书发送数据。桥接实现属于独立集成，不在本网页项目中伪造。
