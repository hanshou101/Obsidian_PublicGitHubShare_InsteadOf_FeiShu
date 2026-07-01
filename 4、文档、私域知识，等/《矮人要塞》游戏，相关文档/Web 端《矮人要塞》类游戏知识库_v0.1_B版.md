下面给你一份**《Web 端矮人要塞复刻版》开发知识库 v0.1**。它不是“玩家攻略”，而是面向实现的系统拆解：哪些系统必须建模、哪些数据必须结构化、哪些地方适合先简化、哪些地方必须由授权本地数据自动抽取。

我按你提供的“已获授权、个人研究”前提处理；工程上仍建议把**玩法引擎、数据解析、文本/美术/音频资源**分层，方便后续替换、审计和自动化生成。

---

## 1. 版本基线：以 v53.15 / Premium 时代为准

截至 2026 年 7 月 1 日，官方 Bay 12 页面显示 Dwarf Fortress Classic 最新为 **53.15，发布于 2026 年 6 月 25 日**；该版本加入了 100 种灭绝的真实世界动物及其 animal people，并更新了部分 animal people 图形。([Bay 12 Games][1]) Wiki 的版本史也列出 v53.15 为 2026-06-25，对应“extinct animals”更新；v53.13 则加入了信使外交任务，例如议和、宣战、建立联系、改善或开启贸易、寻求盟友等。([矮人要塞维基][2])

官方开发页把 Dwarf Fortress 的长期目标定义为一个“幻想世界模拟器”：玩家不是只玩一局城建，而是在一个有长期历史、文明、人物、地点与事件演化的世界中扮演不同角色；项目由 Tarn Adams 与 Zach Adams 设计，开发起点可追溯到 2002/2006 年。([Bay 12 Games][3]) Steam 官方页则把当前商业版描述为带图形与教程的“最深、最复杂的世界模拟”，核心包括生成世界、管理殖民地、角色个性、天气、材料、身体部位战斗、诗歌/音乐/乐器/舞蹈生成，以及 200 多种岩石和矿物。([Steam商店][4])

---

## 2. 产品定义：不要复刻“界面”，要复刻“三层模拟”

Web 版如果只做“像素小人挖洞 + 仓库 + 工坊”，会像普通殖民地模拟；Dwarf Fortress 的灵魂是三层系统叠加。

第一层是**世界模拟**：世界生成会创建地理、历史、语言、文明兴衰、角色外貌与个性；生成结束后，世界活动仍继续，堡垒模式和冒险模式只是玩家对巨大世界的一小部分干预。([矮人要塞维基][5])

第二层是**局部堡垒模拟**：玩家在三维 z-level 地图上挖掘、建造、存储、生产、分配劳动、设置房间、管理粮食酒水、交易、医疗、军事、防御、贵族、司法、文化场所与居民心理。官方 Steam 页明确强调，玩家要“指挥矮人、保证食物与酒水、抵御攻击、制造从皮革到钢铁的各种材料”，并处理水闸、水与岩浆、酒馆、图书馆、神庙、蜂蜜、蜡、陶器、动物训练、装订等系统。([Steam商店][4])

第三层是**角色与历史连续性**：角色有身体、技能、属性、个性、价值观、偏好、关系、记忆、伤病、装备和社会身份；同一个世界可以在堡垒模式、冒险模式、传奇模式之间互相影响。Adventure Mode 是开放世界 roguelike 玩法，玩家能在生成世界中创建角色/队伍，访问城镇、洞穴、神殿、巢穴、文明地点，甚至重访废弃或退休的堡垒。([矮人要塞维基][6])

---

## 3. 顶层模式清单

| 模式                    | Web 复刻中的职责                       | MVP 优先级 |
| --------------------- | -------------------------------- | ------: |
| World Generation 世界生成 | 生成地形、生物群系、文明、语言、历史人物、遗迹、战争、神器、地点 |      P1 |
| Fortress Mode 堡垒模式    | 主游戏：殖民地管理、挖掘、生产、物流、心理、贸易、军事、防御   |      P0 |
| Adventure Mode 冒险模式   | 单角色/小队在同一世界探索、战斗、接任务、访问地点        |      P2 |
| Legends Mode 传奇模式     | 查询世界历史、人物、文明、战争、神器、地点、事件索引       |      P1 |
| Arena / 测试场           | 调试战斗、材料、身体部位、怪物、装备               | P1 开发工具 |

P0 的意思不是“原作里最重要”，而是 Web MVP 不做它就不像 DF。P1 是第一轮扩展后必须进入的系统。P2 可以在核心引擎稳定后追加。

---

## 4. 核心数据模型：先建知识图谱，不要手抄百科

Dwarf Fortress 的内容规模极大，正确做法不是把 Wiki 页面手工复制成文档，而是建立**可自动更新的知识库/规则库**。原版内容大量通过 raw 文件定义，Wiki 说明这些对象文件位于 `/data/vanilla/*/objects/`，定义物品、材料、植物、动物和其他对象；vanilla 内容和 mods 使用同一类 token 格式。([矮人要塞维基][7]) Raw 文件是文本文件，常见对象类型包括 BODY、BUILDING、CREATURE、ENTITY、INORGANIC、ITEM、LANGUAGE、MATERIAL_TEMPLATE、PLANT、REACTION、TISSUE_TEMPLATE 等；DF 本体不是开源项目，大多数内容 modding 依赖 raw 文件和 token。([矮人要塞维基][8])

建议知识库拆成这些实体表：

| 实体                 | 关键字段                                                             |
| ------------------ | ---------------------------------------------------------------- |
| `World`            | seed、年龄、尺寸、文明列表、历史事件索引                                           |
| `RegionTile`       | biome、elevation、rainfall、drainage、savagery、evil/good、temperature |
| `LocalMap`         | embark 坐标、z-level 范围、tile 网格、含水层、洞穴层、矿脉                          |
| `Tile`             | 地形类型、材质、是否开挖、建筑、流体深度、温度、可通行性                                     |
| `Material`         | 类别、密度、硬度、剪切/冲击属性、熔点、价值、反应标签                                      |
| `CreatureDef`      | creature token、caste、体型、年龄、繁殖、技能、攻击、可驯化性                         |
| `BodyPlan`         | body part 树、组织层、器官、功能、连接关系                                       |
| `PlantDef`         | 生长环境、季节、可食/可酿/可纺/可制纸、产物                                          |
| `ItemDef`          | 类型、子类型、尺寸、材质约束、装备槽、价值                                            |
| `Unit`             | 位置、职业、技能、劳动、需求、心情、个性、身体、装备、关系                                    |
| `Job`              | job type、目标、材料需求、工具需求、优先级、执行者、状态机                                |
| `Building`         | 位置、尺寸、材料、任务队列、占用格、连接机制                                           |
| `Reaction`         | 输入、输出、技能、工坊、燃料、产物质量规则                                            |
| `Stockpile`        | 区域、过滤规则、容器策略、take/give 链接、优先级                                    |
| `Zone / Location`  | bedroom、meeting area、hospital、tavern、temple、library、guildhall    |
| `Squad`            | 成员、制服、训练计划、警戒、巡逻、攻击命令                                            |
| `Event`            | 时间、主体、客体、地点、类型、可显示文本、传奇索引                                        |
| `Thought / Memory` | 触发原因、情绪、压力影响、记忆重放、价值观/人格影响                                       |

token 层还应建立“注册表”：audio、biome、building、body、creature、entity、graphics、item、instrument、interaction、labor、language、material、plant、reaction、skill、syndrome、tissue、world 等类别。Wiki 的 token 页面把这些列为对象属性定义的主要类别。([矮人要塞维基][9])

---

## 5. 地图与物理层

Dwarf Fortress 的地图不是二维城市网格，而是多 z-level 的体素/瓦片世界。MoMA 对游戏的说明提到，游戏世界由 2D tiled blocks/text 生成，但堡垒实际包含多个 z-level，玩家在生成的大陆、海洋、山地与地下空间中建立矮人聚落。([The Museum of Modern Art][10])

Web 实现应把地图拆成四层：

| 层   | 内容                            |
| --- | ----------------------------- |
| 地质层 | 岩层、矿脉、宝石、土壤、黏土、沙、含水层          |
| 地形层 | 墙、地板、斜坡、楼梯、开放空间、河流、池塘、洞穴      |
| 建筑层 | 门、桥、墙、工坊、家具、陷阱、机械、轨道、农田       |
| 动态层 | 单位、物品、流体、温度、烟雾/蒸汽、污染物、血液、路径占用 |

流体系统最少要支持水、岩浆、深度 0–7、下落、横向扩散、压力近似、阻挡物、蒸发/冷却/冻结的抽象。水闸是关键建筑：Wiki 说明 floodgate 用于控制水或岩浆流动，可由拉杆、压力板等机制开闭；关闭时表现为墙，打开后允许液体、物体和生物通过。([矮人要塞维基][11])

---

## 6. 堡垒模式主循环

堡垒模式的主循环可以实现成确定性 tick 系统：

```text
GameTick:
  1. 读取玩家命令：designations / buildings / stockpiles / zones / alerts
  2. 更新地图：挖掘完成、建筑占用、流体扩散、温度变化
  3. 生成任务：挖掘、搬运、工坊、农田、医疗、军事、清洁、喂养
  4. 单位 AI 选任务：劳动权限、技能、工具、距离、危险、需求、burrow
  5. 执行任务状态机：移动 -> 取材料 -> 到工位 -> 工作 -> 产出 -> 释放
  6. 更新物品物流：stockpile 过滤、容器、腐烂、禁用/倾倒/融化
  7. 更新心理：需求、想法、记忆、压力、社交、满意度
  8. 更新外部事件：商队、移民、使者、野兽、围攻、外交消息
  9. 更新战斗/伤病：攻击、命中部位、伤口、疼痛、失血、医疗任务
  10. 写入 announcements 与 legends 事件日志
```

这里最关键的是：玩家并不直接控制矮人移动，而是通过命令生成 job。劳动系统决定矮人能否执行某类 job；Wiki 说明 labors 控制哪些工作能/不能做，技能和属性影响效率，矮人是半自主的，jobs 来自 designation、zone、workshop 和 manager。([矮人要塞维基][12])

---

## 7. 劳动、任务、工坊、订单

这是 Web 复刻成败的核心。建议实现四级调度：

1. **Player Intent**：玩家划定挖掘、砍树、采集、建造、制作。
2. **Job Generator**：系统把意图变成可执行 job。
3. **Job Claiming**：符合劳动、工具、路径、burrow、优先级的单位领取。
4. **Job Execution**：单位按状态机取材、移动、工作、产出、释放资源。

工坊是生产系统中心。Wiki 说明 workshops 用来把材料加工成有用或有价值的物品；技能影响质量与速度；典型工坊多为 3×3；工坊任务会寻找原料、搬运、工作、生成物品，产物通常先留在工坊，之后由 hauling job 运走，工坊杂乱会拖慢任务。([矮人要塞维基][13])

Manager / Work Orders 是自动化层。Wiki 说明 work orders 通过 manager 实现精细自动化，可设置一次性或重复订单、条件、最大工坊数；人口超过一定规模时需要 manager 验证订单。([矮人要塞维基][14]) Web 版 MVP 可以先做“全局订单 + 简单条件”，随后再做复杂条件，例如“少于 10 个酒桶就酿酒”。

---

## 8. 物流：Stockpile 是游戏的血管

Stockpile 不只是“仓库格子”，而是 DF 的物流规则系统。Wiki 说明 stockpiles 用来让矮人存放不同类型物品，搬运 job 会把物品送到合适 stockpile；类别包括弹药、动物、护甲、条块、布、尸体、成品、食物、家具、宝石、皮革、垃圾、石头、武器、木材和自定义规则。([矮人要塞维基][15])

Web 版必须支持：

| 功能                     | P0 / P1 |
| ---------------------- | ------: |
| stockpile 区域绘制         |      P0 |
| 按物品类型过滤                |      P0 |
| 食物、石头、木材、成品、垃圾基础分类     |      P0 |
| barrel/bin/bag 容器      |      P1 |
| take from / give to 链接 |      P1 |
| 优先级与最大桶/箱数量            |      P1 |
| 量大后路径优化、批量 hauling     |      P1 |
| 禁用、倾倒、融化、交易标记          |      P1 |

一个好的 MVP 判断标准：玩家挖出石头后，空闲矮人能把石头搬到 stone stockpile；木匠能从 wood stockpile 取木材做床；床完成后能被搬到卧室安装。

---

## 9. 食物、饮料、农业、畜牧

DF 中矮人需要食物与酒水，农业是长期堡垒的基础。Wiki 说明 farming 能生产食物、酒、布和纸；农田需要土壤或泥化岩石，并区分地表/地下作物与生物群系限制。([矮人要塞维基][16])

MVP 最小系统：

| 系统    | 最小实现                                 |
| ----- | ------------------------------------ |
| 饥饿/口渴 | unit 定时增加需求；找食物/酒水 job               |
| 农田    | plot + 作物 + 季节 + 种子 + 成熟时间           |
| 酿酒    | plant -> drink + seed，依赖 still       |
| 烹饪    | raw food -> prepared meal，依赖 kitchen |
| 畜牧    | pasture zone、屠宰、蛋、奶、剪毛可后置            |
| 腐烂    | 食物无容器/无仓库时腐败，产生 miasma 可后置           |

动物训练是 DF 的重要扩展系统。Wiki 说明 animal trainer 可驯服野生动物，或把某些驯服物种训练为战斗/狩猎动物；训练需要 animal training zone 或 pasture，部分动物可根据 `[TRAINABLE]`、`[TRAINABLE_HUNTING]`、`[TRAINABLE_WAR]` 等 token 进入不同训练路径。([矮人要塞维基][17])

---

## 10. 房间、地点、文化系统

DF 的房间不是纯装饰，它影响心理、贵族需求、社交、访客和长期文化。Location 系统包括 hospital、library、tavern、temple、guildhall；Wiki 说明这些 location 从 Places 菜单查看，通过 zone 指定，可设置访问限制、职业、物品数量，部分房间可分配给 location 使用。([矮人要塞维基][18])

优先级建议：

| 场所           | 功能            | 优先级 |
| ------------ | ------------- | --: |
| Bedroom      | 睡眠、满意度、贵族要求   |  P0 |
| Dining room  | 进餐 thought、社交 |  P0 |
| Meeting area | 空闲聚集、社交       |  P0 |
| Hospital     | 诊断、治疗、病床、医疗物资 |  P1 |
| Tavern       | 饮酒、表演、访客、租房   |  P1 |
| Temple       | 祈祷、神祇需求、压力管理  |  P1 |
| Guildhall    | 技能演示、请愿、职业社群  |  P1 |
| Library      | 书籍、学者、抄写、知识   |  P2 |

Guildhall 需要特别保留，因为它把社交、技能学习与请愿连接起来。Wiki 说明 guildhall 是矮人展示技能和满足社交需求的 location，达到成员数后会请愿建立；演示能教学技能，访客在允许时也能演示。([矮人要塞维基][19])

---

## 11. 个性、需求、想法、压力

这是 DF 与普通经营游戏的核心区别之一。每个单位应该有：

| 模块                 | 字段                       |
| ------------------ | ------------------------ |
| 基础需求               | 饥饿、口渴、疲劳、社交、祈祷、学习、创造、武艺等 |
| Personality Facets | 勇敢、焦虑、友善、纪律、贪婪、愤怒倾向等     |
| Values             | 家庭、友情、法律、权力、传统、自然、工艺、战争等 |
| Preferences        | 喜欢的材料、动物、颜色、食物、酒、物品类型    |
| Thoughts           | 近期情绪事件                   |
| Memories           | 可反复触发压力变化的记忆             |
| Stress / Focus     | 长期心理状态、效率和崩溃风险           |

Wiki 说明 thoughts 是矮人在堡垒模式中经历事件后产生的积极或消极感受，近期情绪会影响 stress；即使当下没有事件，矮人也可能通过 memories 重新体验过去事件，使一次经历反复产生影响。([矮人要塞维基][20]) Thoughts and preferences 页面还说明，个人资料分为 thoughts、memories、traits、values、preferences、needs 等，负面压力严重时矮人可能取消工作并发呆，临时失常可发展成永久疯狂。([矮人要塞维基][21])

价值观应建成数值系统。Wiki 说明 personality value 范围为 -50 到 +50，文化价值会影响个体价值；values 与 traits 会决定 needs，以及这些 needs 未满足时对 focus 的影响。([矮人要塞维基][22])

---

## 12. 贸易、外交、贵族、司法

贸易系统最低要支持 trade depot、商队、搬货、估价、交换、利润判断、请求下一年商品。Wiki 说明贸易需要友好文明、可到达的 trade depot，broker 会有帮助；交易流程包括把货物移到 depot、请求 broker 或任意矮人交易、评估价值、处理重量、接受/还价/赠送。([矮人要塞维基][23])

外交系统从 v53.13 起更重要：官方更新说明提到信使可执行议和、宣战、建立联系、改善贸易、开启贸易和寻求盟友等任务。([Bay 12 Games][1]) 因此 Web 版知识库应把“文明关系”抽象为独立模型，而不是只做固定商队事件。

贵族与行政职位要拆开。Wiki 说明 nobles and administrators 是堡垒中的职位：管理员执行具体任务，如管理 work orders、贸易、军事指挥、库存记录；高级贵族会提出 demands 和 legally-binding mandates，未满足会触发司法系统。([矮人要塞维基][24])

司法系统至少需要案件、犯罪、证人、审讯、定罪、处罚、牢房/链条。Wiki 说明 justice 由 sheriff 或 captain of the guard 执行，用来处理公民或访客的犯罪行为，例如违抗上级、破坏家具、打架等。([矮人要塞维基][25])

---

## 13. 战斗、身体、伤病、医疗

DF 战斗系统必须从“HP 条”改为“身体部位 + 材料 + 伤口”。Steam 官方页强调，战斗模拟包含技能、身体部位、材料属性、瞄准攻击、摔跤、疼痛、恶心、毒素等。([Steam商店][4])

实现上建议：

```text
Attack:
  attacker skill + weapon skill + weapon material + attack verb
  vs
  defender dodge / block / parry + armor coverage + armor material
  ->
  target body part
  ->
  tissue layers damaged
  ->
  wound effects: bleeding, pain, function loss, infection, unconsciousness, death
```

Wiki 的 combat 页面说明防御包括护甲、闪避、盾牌、格挡；任何持有物都可能成为武器，武器技能和战斗技能会影响结果。([矮人要塞维基][26]) Wound 页面列出喉部动脉、眼睛、肺、心脏、肠胃、脊椎等器官受伤后的不同后果，例如失明、窒息、致命失血、疼痛、恶心、呕吐、瘫痪等。([矮人要塞维基][27]) Armor 页面说明材料对护甲非常重要，非金属护甲通常较弱，钝击可隔着护甲造成骨折，护甲使用技能影响负重惩罚。([矮人要塞维基][28])

医疗系统要由诊断驱动。Wiki 说明 hospital location 需要床、桌子、traction bench、箱子，以及 thread、cloth、splints、crutches、plaster、bucket、soap 等物资；医生职责包括 diagnosis、surgery、bone doctor、doctor，伤者需要先诊断再治疗。([矮人要塞维基][29])

---

## 14. 军事、防御、围攻

军事系统最低要支持 squad、制服、训练、警戒、巡逻、驻扎、攻击命令、burrow。Wiki 说明军事是防御围攻、巨兽、泰坦和地下威胁的关键，核心由 squads 和 scheduling 组成。([矮人要塞维基][30]) 军事 FAQ 还强调 schedule 与 alert、月份相关，可用于商队护卫、burrow、防线、巡逻等。([矮人要塞维基][31])

需要特别关注 2025 Siege Update。官方版本史说明 v53.01 是 Siege Update，入侵者可以挖掘、破坏、建造并进行计划，攻城器械也得到改进。([矮人要塞维基][2]) PC Gamer 对该更新的报道补充说，哥布林可使用攻城槌，巨魔可用镐和方块，敌方小队会冲锋、保持距离、伏击、绕开危险，工程师能铺地板、楼梯、桥并从地下挖掘；玩家侧则有快速发射弩炮、改良投石机/弩炮等防御工具。([PC Gamer][32])

Web 版可以分三阶段：

| 阶段 | 内容                                  |
| -- | ----------------------------------- |
| P0 | 野兽/敌人寻路到单位，近战攻击，简单陷阱                |
| P1 | squad、训练、制服、射击、巡逻、burrow、桥门水闸防御     |
| P2 | Siege Update 风格敌方工程、破门、挖掘、攻城器械、战术小队 |

---

## 15. Strange Mood、神器、传奇事件

Strange Mood 是 DF 叙事感的关键。Wiki 说明矮人会周期性进入 strange mood，停止其他活动，尝试制造 artifact；失败会导致疯狂或死亡。触发通常需要至少 20 个 eligible creatures、没有当前 mood、未超过 artifact 限制；mood 单位会占用工坊、按顺序收集材料，完成后制造神器并获得技能提升，possessed 类型例外。([矮人要塞维基][33])

Web 版要把 artifact 作为独立实体：

```text
Artifact:
  id
  name
  creator_unit
  created_year_tick
  base_item
  material_layers
  decorations
  image/story description
  historical_event_id
  location/site
  ownership
```

最小玩法：mood 触发 → 占工坊 → 请求材料 → 成功产出高价值物品 → 写入事件日志。失败后再接入精神崩溃、死亡、暴力、传奇记录。

---

## 16. 冒险模式与传奇模式

Adventure Mode 不应先做完整，因为它依赖世界生成、地点、战斗、对话、声望、任务、物品、身体系统。当前 Wiki 说明冒险模式允许玩家创建角色/队伍，可选矮人、人类、精灵、哥布林、亡灵实验体、animal person 等，访问地点、接任务、探索洞穴/神殿/巢穴/城镇，重访废弃或退休堡垒；当前版本 building 不可用，crafting 也较有限。([矮人要塞维基][6])

Legends Mode 则更适合早做，因为它本质上是世界数据库查询器。建议每个系统都写 event：

```text
HistoricalEvent:
  time
  event_type
  actors[]
  entities[]
  site
  region
  artifact
  cause
  result
  display_template
```

只要事件系统设计好，Legends Mode 就是可视化索引：人物传记、文明年表、战争、怪物、神器、地点历史。

---

## 17. Web 技术架构建议

推荐架构：

| 层               | 技术选择                                                              |
| --------------- | ----------------------------------------------------------------- |
| Simulation Core | TypeScript + ECS/SoA；性能瓶颈可迁移 Rust/WASM                            |
| Rendering       | Canvas 2D 起步；大地图/动画转 WebGL                                        |
| Workers         | Web Worker 跑模拟、寻路、流体，主线程只渲染 UI                                    |
| Save            | IndexedDB + 压缩快照 + 事件日志                                           |
| Determinism     | seed RNG；所有随机事件带 seed 与 tick                                      |
| Data            | raw parser -> normalized JSON -> schema validation                |
| UI              | React/Svelte/Vue 任一；仿真层不可依赖 UI 框架                                 |
| Modding         | JSON/raw-like DSL；版本迁移器                                           |
| Debug           | tick inspector、job inspector、path overlay、fluid overlay、event log |

核心性能策略：

```text
不要每 tick 全图扫描。
使用 dirty chunks。
地图按 chunk 分块，例如 32x32x1 或 32x32x4。
流体、温度、路径、stockpile 都用增量更新。
单位 AI 不要每帧重算全局最优，只做周期性 job search。
寻路缓存按 burrow / traffic / gate state 失效。
```

---

## 18. 数据抽取流水线

因为你们目标是“知识库 + VibeCoding 工具可理解”，最推荐这样做：

```text
licensed_df_install/
  data/vanilla/*/objects/*.txt
  data/vanilla/vanilla_procedural/*
  data/art/*
  data/sound/*

pipeline/
  01_parse_raws.ts
  02_resolve_tokens.ts
  03_validate_refs.ts
  04_emit_json.ts
  05_generate_markdown_kb.ts
  06_generate_tests.ts
```

输出结构：

```text
/kb
  /00_version_baseline.md
  /01_core_loop.md
  /02_world_generation.md
  /03_map_tiles_fluids.md
  /04_units_minds_bodies.md
  /05_jobs_labors_workshops.md
  /06_stockpiles_logistics.md
  /07_farming_food_animals.md
  /08_rooms_locations_culture.md
  /09_trade_diplomacy_nobles_justice.md
  /10_combat_military_health.md
  /11_adventure_legends.md
  /12_data_schema.md
  /13_mvp_roadmap.md

/data
  materials.json
  creatures.json
  plants.json
  items.json
  reactions.json
  bodies.json
  tissues.json
  entities.json
  languages.json
  skills.json
  labors.json
```

v52.01 引入 Lua scripting，官方更新说明提到它让 mod 制作和未来高级系统更容易，并把部分 procedural object generation 的算法/数据暴露给 modding，例如 forgotten beasts、curses、divine items、necromancers、experiments、evil weather 等；vanilla procedural scripts 位于 `data/vanilla/vanilla_procedural`。([矮人要塞维基][2]) 这意味着 Web 版知识库不应只解析传统 raw，还应预留 procedural generation 脚本/规则层。

---

## 19. MVP 路线图

### Milestone 0：数据与引擎骨架

完成 raw-like 数据 schema、地图 chunk、单位 ECS、job queue、save/load、事件日志、基本 UI。没有这个，后面系统会变成硬编码泥潭。

### Milestone 1：最小堡垒

玩家能选择 embark，看到 z-level 地图，指定挖掘，矮人自动挖洞、搬运石头、建 stockpile、造工坊、做床、建卧室、吃饭喝酒、睡觉。Quickstart 页面把早期堡垒流程拆成 embark、技能装备、劳动、挖掘、库存、楼梯、meeting area、食物、农业、砍树、饮料、牧场、工坊、酿酒、交易、移民、卧室、管理员、餐厅、陷阱、金属工业、军事等步骤，可作为 MVP 任务拆解依据。([矮人要塞维基][34])

### Milestone 2：经济闭环

加入 farming、brewing、cooking、wood/stone/craft、trade depot、broker、merchant、work orders、containers、stockpile links。目标是堡垒可持续运行 1–2 年。

### Milestone 3：个性与灾难

加入 thoughts、needs、stress、memories、moods、artifacts、injury、hospital、death、burial、miasma、nobles、mandates、justice。目标是出现 DF 式叙事：不是“输了”，而是“因为某个矮人失去宠物、贵族强制生产、医院缺肥皂、战士失血导致连锁崩溃”。

### Milestone 4：军事与世界压力

加入 squads、training schedules、uniforms、ambush、siege、megabeast、diplomacy、messengers、civilization relations。目标是外部世界能对堡垒施压。

### Milestone 5：世界史、冒险、传奇

加入完整 worldgen、historical figures、sites、civilizations、legends browser、adventure mode 角色创建与探索。目标是同一世界多模式连续。

---

## 20. “像 DF”的验收测试

下面这些测试比 UI 是否像原版更重要：

| 测试           | 通过标准                                                                       |
| ------------ | -------------------------------------------------------------------------- |
| 挖掘闭环         | 玩家指定挖掘；有 pick 且启用 mining 的单位领取；到达目标；tile 从 wall 变 floor/open；产出 stone item |
| 工坊生产         | 木匠从 stockpile 取 wood，在 carpenter workshop 做 bed，产物留在工坊，随后被搬走               |
| 饮料压力         | 酒不足时矮人产生 drink job；长期不足导致负面 thought / focus 降低                             |
| 农业循环         | 种子 → 作物 → 酿酒/食物 → 种子回收，能支撑人口增长                                             |
| 卧室心理         | 有床和房间的矮人比睡地板获得更好 thought                                                   |
| 材料战斗         | 铜剑、钢剑、木棍对不同护甲产生不同伤口                                                        |
| 身体部位         | 攻击眼睛导致视力损失；腿伤影响移动；肺伤影响呼吸；失血可死亡                                             |
| 医疗           | 伤者被搬到 hospital；先 diagnosis，再按伤口生成 suturing/surgery/splint 等 job            |
| Strange Mood | 合格人口后触发 mood；占工坊；请求材料；成功生成 artifact；失败产生严重后果                               |
| 贸易           | 商队抵达 depot；玩家搬货；broker 估价；交易改变双方库存                                         |
| 围攻           | 敌人从地图边缘进入，寻路到目标；桥/门/陷阱/水闸影响路径与伤害                                           |
| 传奇日志         | 重大事件可在 Legends 查询到：神器诞生、死亡、战争、怪物袭击、文明关系变化                                  |

---

## 21. 最关键的取舍

第一版不要追求“100% 内容数量”，而要追求“100% 规则可扩展”。例如 53.15 加了 100 种灭绝动物，如果你们的数据管线正确，它们只是新增 creature records；如果系统硬编码动物行为，就会每次更新都崩。([Bay 12 Games][1])

优先复刻这些“结构性规则”：

1. 多 z-level tile map。
2. 半自主单位 + job/labor 调度。
3. stockpile 物流。
4. workshop/reaction 生产。
5. material-driven item system。
6. body-part combat。
7. thought/need/stress 心理。
8. event/legend 历史记录。
9. raw/token 数据驱动。
10. save/load 与确定性模拟。

只要这十个成立，Web 版就具备 DF 的骨架；内容量可以靠授权数据解析、原创数据补充和后续模块逐步扩展。

