# Web 端《矮人要塞》类游戏知识库_v0.1_A版

> 用途：作为“干净室”功能规格、系统拆解、数据模型与 Web 实现路线图。本文只描述玩法、系统、数据结构、交互逻辑和工程拆分，不复制原作代码、素材、文案、图像、音频或专有数据表。

## 0. 版本锚点与资料口径

- 调研日期：2026-07-01，用户时区 Asia/Tokyo。
- 官方 Classic 当前版本：53.15，发布日期 2026-06-25。官方更新说明称该补丁加入约一百种现实灭绝动物，并为每种增加 animal person，同时改进既有 animal people 的装备/衣着显示。
- Steam/itch 付费版：2022-12-06 发布；Adventure Mode 于 2025-01-23 结束 beta 并上线。
- 本知识库优先参考：Bay 12 官方页面、Steam 新闻页、Dwarf Fortress Wiki 当前 v53.x 页面。Wiki 页面有些标注“从旧版本迁移，可能未完全更新”，所以实现时需要把“核心机制”与“版本细节”分开管理。

主要参考入口：

- Bay 12 Games 官方下载/更新页：https://www.bay12games.com/dwarves/
- Bay 12 Games 开发目标页：https://www.bay12games.com/dwarves/dev.html
- Steam 商店页：https://store.steampowered.com/app/975370/Dwarf_Fortress/
- Dwarf Fortress Wiki：World generation、Dwarf fortress mode、Adventurer mode、Legends、Object testing arena、Work orders、Labor、Stockpile、Stress、Thoughts and preferences、Combat、Health care、Trading、Military、Squad、Noble、Justice、Location、Zone、Farming、Metal industry、Water、Magma、Weather 等。

---

## 1. 一句话设计目标

《矮人要塞》的核心不是“建造地下基地”本身，而是：

> 在一个先被长期历史模拟塑造过的奇幻世界里，玩家通过有限的命令影响一个局部地点，由大量自主代理、物理/材料规则、经济物流、情绪记忆、战争外交与灾难事件共同产生涌现叙事。

Web 复刻应围绕这个目标拆分，而不是一开始追求全量规则。正确的工程顺序是：

1. 先实现稳定的 3D tile 世界、可暂停 tick 模拟、实体/物品/建筑数据层。
2. 再实现职业/任务/物流，因为这是堡垒模式的“血液循环”。
3. 再实现食物、酒、睡眠、房间、情绪，因为这是殖民地可持续性的核心。
4. 再接入材料、行业链、战斗、流体、温度、历史、外交、Adventure/Legends 等深层系统。

---

## 2. 游戏模式总览

### 2.1 World Generation / 世界生成

世界生成是所有模式的前置。它生成：

- 大世界地图：海洋、山脉、河流、湖泊、火山、气候区、雨量、排水、温度、海拔、植被、生物群系、善恶/野蛮度区域。
- 地质与资源：岩层、土壤、矿脉、宝石、含水层、地下洞窟、岩浆海、特殊地下层。
- 文明与历史：矮人、人类、精灵、哥布林等文明；聚落；战争；领袖；历史人物；神器；神祇；语言；传说；野兽与怪物。
- 可玩世界状态：玩家可以在同一生成世界中创建堡垒、进入冒险、查看 Legends；玩家行为会写回历史。

MVP 不需要先做完整世界生成。建议分三层：

- v0：固定小世界 + 手写地形/文明种子。
- v1：参数化随机地形 + 基础历史事件。
- v2：文明扩张、战争、神器、人物生命周期、站点生成。

### 2.2 Fortress Mode / 堡垒模式

玩家选择 embark 地点，配置初始七名矮人的技能、物资、工具与动物。进入地图后，玩家不是直接控制单个矮人，而是下达设计/建造/生产/行政命令；矮人按可达性、劳动许可、优先级和可用材料自主执行。核心目标是开放式的：生存、发展、积累财富、吸引移民与贵族、抵御威胁、挖掘更深处、创造故事。

核心循环：

1. 选择地点与初始物资。
2. 挖掘入口、临时仓库、工坊。
3. 采集木材/矿石/植物，建立食物和酒供应。
4. 设置卧室、餐厅、会议区、医院、神庙、酒馆、图书馆、公会等。
5. 用 manager/work orders 自动化生产。
6. 通过贸易、移民、军事、贵族、正义系统扩大复杂度。
7. 面对野兽、盗贼、围攻、地下怪物、精神崩溃、洪水、岩浆、饥荒、幽灵等“Fun”。

### 2.3 Adventure Mode / 冒险模式

玩家在已生成世界中创建一个或一队冒险者，选择种族、文明、难度、技能、属性、装备、坐骑/宠物等，进入开放世界探索城镇、堡垒、荒野、洞穴、神庙、巢穴、遗迹；可询问传闻、招募同伴、战斗、旅行、重访旧堡垒、影响历史。当前版本中建造不可用，制作能力有限，重点是单体/小队 roguelike 探索、战斗、社交与世界交互。

### 2.4 Legends Mode / 传说模式

世界浏览器/史书模式。查看地图、文明、历史人物、地点、神器、艺术、结构、事件链。它是开发调试与玩家叙事回放都极其重要的系统。Web 版建议把 Legends 作为“世界事件数据库的可视化前端”来实现，而不是最后补。

### 2.5 Object Testing Arena / 对象测试竞技场

用于生成生物、装备、技能、队伍、液体、天气、温度等，测试战斗、材料、mod/raw 数据。Web 版建议从早期就实现一个“仿真沙盒”，供开发者调试 AI、战斗、流体和材料规则。

---

## 3. 顶层领域模型

### 3.1 World

字段建议：

- `worldId`, `seed`, `calendar`, `age`, `historyYears`
- `regionMap`: 大世界区域网格
- `biomes`, `climates`, `elevation`, `rainfall`, `drainage`, `temperature`, `savagery`, `alignment`
- `civilizations`, `entities`, `sites`, `historicalFigures`
- `artifacts`, `writtenWorks`, `events`
- `mythSystems`: 神、领域、诅咒、死灵、狼人/吸血鬼、特殊材料/互动
- `activeGames`: fortress/adventure timelines

### 3.2 LocalMap / EmbarkMap

- 3D tile map：`x,y,z`。
- 每个 tile：自然地形、构造物、建筑、液体、温度、草/树/植物、物品堆、污染物、可通行性、光照/室内外、是否挖掘过、是否被发现。
- 地图边界与大世界连接：河流从边界进入/流出，商队/敌军从边界生成，冒险模式从局部地图进入大世界旅行。

### 3.3 Tile

核心字段：

```ts
type Tile = {
  pos: Vec3;
  terrain: TerrainId;          // soil, stoneWall, floor, ramp, stair, openSpace, tree, waterSource...
  naturalMaterial?: MaterialId;
  construction?: ConstructionId;
  building?: BuildingId;
  fluid?: FluidState;          // water/magma depth 0-7, pressure, stagnant/salt, temperature
  temperature: number;
  contaminants: Contaminant[];
  items: ItemId[];
  creatures: CreatureId[];
  designation?: Designation;
  flags: TileFlags;            // outside, light, subterranean, revealed, hidden, damp, warm, flow, etc.
};
```

### 3.4 Material

材料是物品、建筑、战斗、温度、价值、加工链的共同基础。字段：

- 名称、颜色、价值、类别：stone/metal/wood/glass/cloth/leather/bone/plant/liquid/powder。
- 密度：决定重量，进而影响搬运速度、武器冲击、物品质量体验。
- 温度阈值：熔点、沸点、燃点、热伤害/冷伤害阈值。
- 机械属性：impact/shear/compressive/tensile/torsion/bending 的 yield、fracture、strain。
- 派生标签：fire-safe、magma-safe、weapon-grade、armor-grade、edible、brewable、economic-stone。
- 反应/配方用途：ore -> bar、flux、fuel、alloy、soap、dye、thread、cloth、meal 等。

### 3.5 Creature

字段：

- 生物种/种姓：species/caste、体型、寿命、速度、感官、是否会飞/游泳/水生/夜视/不吃不喝。
- 身体结构：body parts、组织层、器官、骨骼、肌肉、皮肤、神经、血液。
- 属性：strength、agility、toughness、endurance、recuperation、diseaseResistance、focus、willpower、empathy、spatialSense 等。
- 技能：labor skills、combat skills、social skills、performance skills、knowledge skills。
- 个体心理：personality traits、values、preferences、needs、thoughts、memories、stress、focus。
- 社交关系：family、friends、lovers、grudges、groups、guild、religion、citizenship。
- 状态：hunger、thirst、drowsiness、pain、bleeding、infection、injury、mood、job、path、burrow。
- 装备/库存：worn, held, inventory, owned rooms/items。

### 3.6 Item

字段：

- itemType：bar、boulder、block、log、weapon、armor、clothing、furniture、food、drink、seed、corpse、bodyPart、book、instrument、tool、container 等。
- material、quality、wear、temperature、stackCount、containedItems、owner、forbidden、dump、melt、trade、artifact。
- 价值公式：基础 item value × material value × quality multiplier × decoration/artifact/symbolic modifiers。
- 可达性：所在 tile/container/building/creature；是否被 job 预定。

### 3.7 Job / Task

任务是玩家意图和矮人行为之间的接口。来源：

- designations：挖掘、砍树、采集、平滑/雕刻、建造。
- workshops：单次任务、重复任务、配方任务。
- manager work orders：全局生产订单、条件检查。
- zones：捕鱼、取水、倾倒、牧场、训练、医院。
- needs：吃、喝、睡、社交、祈祷、娱乐、清洁。
- emergency：灭火、救伤、逃跑、战斗、婴儿/儿童照料、尸体处理。

调度建议：

1. Job 生成：由玩家命令、系统需求、建筑/工坊、AI 事件生成。
2. Job 验证：材料可用、工具可用、技能/劳动许可、路径可达、优先级。
3. Worker 竞价：空闲矮人按距离、技能、劳动、burrow、当前需求、危险评估选择。
4. 预定资源：锁定物品、建筑、tile，避免多个矮人抢同一资源。
5. 执行 tick：移动、拾取、加工、放置、产出、经验增长、事件写入。
6. 失败处理：取消、重试、公告、释放资源。

---

## 4. 堡垒模式核心系统清单

### 4.1 Embark / 开局

- 世界列表与时间线选择。
- Embark 地点筛选：海拔、地形、土壤、树木、河流、含水层、矿物、邻近文明、善恶、野蛮度、气候、温度、地下条件。
- 初始七矮人：技能点、职业、昵称、偏好、人格、关系。
- 物资：食物、酒、种子、工具、武器、动物、木材、石材、金属、布料、皮革。
- 可选挑战：无树、含水层、冰原、邪恶雨、火山、荒野、高围攻风险。

### 4.2 地图与挖掘

- Z-level 多层视图。
- 自然 tile：土壤、粘土、沙、岩壁、岩地板、斜坡、楼梯、开放空间、树、河床。
- 玩家 designations：挖通道、挖楼梯、挖斜坡、平滑、雕刻、砍树、采植物、移除构造物。
- 地质反馈：潮湿石壁提示水、温暖石壁提示岩浆。
- Cave-in：无支撑结构坍塌。
- 室内/室外判定：影响植物、天气、地下适应、光照。

### 4.3 建筑与构造

构造物：墙、地板、楼梯、斜坡、桥、道路、栅栏、栏杆、门、舱门、格栅、机关门、水闸、轨道。

建筑：工坊、炉、床、桌、椅、箱、柜、棺材、武器架、装甲架、靶场、井、贸易站、桥、陷阱、杠杆、机关、风车/水车、泵、磨坊、蜂箱、鸟巢箱、笼子、链条。

### 4.4 Stockpile / 仓储与物流

基础：

- Stockpile 按物品类别与材质过滤：食物、家具、尸体、垃圾、石头、木材、宝石、金属条、布料、皮革、武器、护甲、成品、弹药、动物等。
- Containers：barrel、bin、bag、pot、wheelbarrow。
- Links：take from / give to，限制工坊只从特定仓库取货。
- Hauling jobs：物品不在合适位置时生成搬运任务。

实现重点：

- 物品堆栈与容器递归会显著增加复杂度；MVP 可先允许单层容器。
- 搬运是性能大户；需要路径缓存、区域连通性缓存、批量任务合并。
- Stockpile tile 越多，扫描越重；Web 版应避免每 tick 全量扫描。

### 4.5 Labor / Skill / Work Details

- Labor 是是否允许做某类工作；Skill 是做该类工作的熟练度。
- Job 会由具备对应劳动许可且可达的空闲矮人执行，执行后提升相关技能。
- Work details 允许把劳动分配给指定角色或全体。
- 关键民用技能：Mining、Wood Cutting、Carpentry、Stoneworking、Masonry、Engraving、Growing、Brewing、Cooking、Butchery、Tanning、Weaving、Clothesmaking、Furnace Operating、Weaponsmithing、Armorsmithing、Mechanics、Architecture、Record Keeping、Organization、Appraisal、Medical skills。

### 4.6 Manager 与 Work Orders

Manager 解锁全局生产订单：

- 固定数量：make 10 beds。
- 条件订单：若 beds < 10 则 make bed；若 drinks < 200 则 brew drink。
- 周期检查：daily/monthly/seasonal。
- 材料筛选：指定材料或任意可用材料。
- Workshop profile：允许某工坊只接某些工匠/技能等级。

MVP 建议：

```ts
type WorkOrder = {
  recipeId: string;
  quantity: number | 'repeat';
  materialFilter?: MaterialFilter;
  conditions: WorkOrderCondition[];
  frequency: 'once' | 'daily' | 'monthly' | 'seasonal';
  status: 'active' | 'waiting' | 'suspended' | 'complete' | 'failed';
};
```

### 4.7 生产行业链

#### 木工/木材

- 来源：砍树。
- 用途：床、桶、箱、门、轮车、木炭、训练武器、建筑。
- 风险：无树地形导致床、桶、燃料早期困难。

#### 石工/石材

- 来源：挖掘产生 boulder/stone。
- 用途：门、桌、椅、家具、工艺品、建筑、机关、石块。
- 经济石：可被限制，防止把重要矿石/通量石随意加工。

#### 农业/食物/酒

- 地下农场：湿泥/土壤，核心作物如 plump helmet，能生吃、烹饪、酿酒。
- 地上农场：需暴露在地上或对应生物群系作物。
- 种子循环：吃生植物、酿酒、研磨、加工通常产生种子；烹饪植物通常不保留种子，所以需要厨房禁用关键种子/酒类烹饪。
- 酒是核心需求：矮人偏好饮酒，酒不足会严重影响效率和情绪。
- 食物来源：农业、贸易、捕鱼、狩猎、采集、畜牧繁殖、陷阱。

#### 动物/畜牧

- 牧场 zone：食草动物需要放牧。
- 繁殖：公母、成年、种群控制。
- 屠宰：肉、骨、皮、脂肪、头骨、蹄、角。
- 训练：宠物、猎犬、战犬、可骑乘/战兽。
- 笼子与陷阱：捕捉动物/敌人，动物园、屠宰、交易。

#### 纺织/皮革/衣物

- 植物纤维、羊毛、蛛丝 -> thread -> cloth -> clothing/bags/rope。
- 皮 -> leather -> armor/clothing/bags/quiver/backpack。
- 衣物磨损会导致坏心情；长期堡垒必须有纺织供应链。

#### 金属工业

- 采矿得到矿石，smelter 把矿石变成 metal bars。
- 非岩浆炉通常消耗 fuel；fuel 来源：木炭或 coke/refined coal。
- 合金：bronze、brass、electrum、steel 等。
- Steel 需要 iron、flux stone、fuel；适合高端武器/护甲。
- Magma smelter/forge 减少燃料压力，但需要岩浆安全材料和地形。

#### 玻璃/陶瓷/肥皂/灰烬

- Sand -> glass goods。
- Clay -> ceramic goods。
- Wood furnace -> ash；ashery -> lye/potash；soap 需要 lye + tallow/oil。
- Soap 对医院清洁和好心情都有价值。

### 4.8 房间、Zone、Location

- Zone：meeting area、pasture、water source、fishing、garbage dump、pen/pasture、hospital、sand/clay collection、pond、pit、training 等。
- Location：hospital、tavern、temple、library、guildhall。
- Room：bedroom、dining room、office、tomb 等，通常由家具定义。
- Room value：由面积、材质、家具、质量、雕刻、装饰决定；影响贵族要求、公会请愿、幸福感。

地点功能：

- Tavern：饮酒、吃饭、社交、表演、访客、谣言、偶发斗殴。
- Temple：祈祷、冥想；可献给具体神或不指定神。
- Library：书写、阅读、学者、书籍与知识传播。
- Guildhall：同职业矮人示范技能、社交、满足需求；职业人数达到阈值后可能请愿。
- Hospital：伤员休息和治疗，存储 thread、cloth、splints、crutches、plaster、buckets、soap。

### 4.9 心理、需求、记忆、压力

矮人心理不是单一 happiness 数值，而是多层：

- Personality traits：勇敢、愤怒、利他、纪律、冒险、焦虑等。
- Values：重视工艺、家庭、力量、和平、宗教、自然、知识等。
- Preferences：喜欢某材料、颜色、生物、食物、酒、物品类型。
- Needs：社交、祈祷、制作、学习、训练、娱乐、独处、家庭等；未满足会影响 focus，长期可能变成坏想法。
- Thoughts/emotions：经历即时事件后产生情绪，正面降低压力，负面提高压力。
- Memories：同一事件可在之后反复被回忆，再次影响压力与人格。
- Stress：累积压力决定长期心理健康；极端可能导致 tantrum、depression、insanity、berserk。

实现建议：

```ts
type Thought = {
  sourceEventId: string;
  valence: -3|-2|-1|0|1|2|3;
  stressDelta: number;
  focusDelta?: number;
  tags: string[]; // ateWithoutTable, sleptInBedroom, sawDeath, prayed, admiredCraft, wasRainedOn...
  createdAt: GameTime;
  memoryStrength: number;
};
```

### 4.10 贵族、行政、法律

基础职务：

- Expedition leader / Mayor：社交、会见外交人员、处理不满。
- Manager：work orders。
- Bookkeeper：库存精度。
- Broker：贸易、估价、交涉。
- Sheriff / Captain of the Guard：正义系统。
- Militia Commander/Captain：军事组织。
- Messenger：当前版本中有更多外交/贸易相关任务。

贵族系统：

- 高阶贵族要求卧室、餐厅、办公室、墓室，房间价值和家具数量有要求。
- Demands：个人要求。
- Mandates：生产某物或禁止出口某类物品；违背可能触发惩罚。
- Justice：处理破坏家具、斗殴、违反命令、杀人等。需要 jail、链条/笼子、卫队。

### 4.11 军事与防御

军事数据：

- Squad：共享日程和命令的军队单位。
- Uniform：武器、盾、头盔、胸甲、手套、靴子、弹药、背包、水袋。
- Schedule：训练、待命、巡逻、站岗、休息。
- Barracks：训练、睡眠、个人装备、队伍装备。
- Orders：kill target、station、patrol、defend burrow、alert。

战斗模型：

- 攻击类型：劈砍、刺击、钝击、咬、踢、摔、投掷、射击、抓取、摔跤。
- 命中流程：发现目标 -> 接近/射程 -> 攻击选择 -> 闪避/格挡/招架 -> 命中部位 -> 护甲/衣物/组织层计算 -> 伤害、疼痛、出血、断肢、昏迷、死亡。
- 材料影响：密度、剪切/冲击强度、武器接触面积、穿透面积、护甲层。
- 训练事故也会写入故事。

防御：

- 门、桥、闸门、陷阱、笼陷阱、武器陷阱、落石/落水/岩浆、堡垒射击孔、壕沟、护城河、诱导通道。
- 2025 Siege Update 后，原作围攻 AI 更复杂；Web 版可分阶段实现：v1 简单入侵路径，v2 携带攻城器械/破坏行为，v3 多路线/地下/营救同伴。

### 4.12 医疗

医院 location 需要：床、桌、traction bench、容器，以及 thread、cloth、splints、crutches、plaster powder、buckets、soap。

治疗流程：

1. Recover wounded：救回伤员。
2. Diagnose：诊断。
3. Clean：清洗污染物，最好有清洁水和肥皂。
4. Surgery：处理坏死组织/器官损伤。
5. Suture：用 thread 缝合切口、神经等。
6. Set bones：接骨。
7. Dress wounds：用 cloth 包扎。
8. Traction：复杂/重叠骨折牵引固定。
9. Immobilization：夹板或石膏。
10. Crutch：永久或长期行动障碍。

MVP 可先实现：受伤 -> 诊断 -> 清洗/包扎 -> 休养；高级版本再实现组织层和神经损伤。

### 4.13 流体、温度、天气

流体：

- Water/Magma 深度 0-7。
- 水有清洁/污浊、淡水/盐水、冻结、压力、流动等状态。
- 压力可使水向上流动；泵可重置/制造压力；U-bend、泵等可用于淡化。
- Magma 类似水但通常无压力，泵送时可施压；遇水生成 obsidian；温度极高，需要 magma-safe 材料。

温度：

- 材料受熔点/沸点/燃点影响。
- 火、烟、蒸汽、岩浆雾、水雾、污染物影响生物与物品。
- 冷区会降雪，露天水会冻结；地下水可保持液态。

Web 实现建议：

- v0 不做全局热扩散，只做 tile 局部状态。
- v1 只做 water depth cellular automata。
- v2 加压力、泵、冻结。
- v3 加 magma、温度、相变、烟/蒸汽。

### 4.14 威胁、灾难、死亡与幽灵

威胁来源：

- 野生动物、邪恶区域生物、盗贼、绑架者。
- 哥布林/敌对文明围攻。
- 地下洞窟生物、forgotten beast、titan、demon、undead、necromancer、werebeast、vampire。
- 自然灾害：洪水、岩浆、坍塌、冻水、饥荒、脱水、瘟疫、失温/高温。
- 内部灾难：压力螺旋、tantrum spiral、贵族惩罚、酒馆斗殴、训练事故、宠物过多、尸体堆积、幽灵。

死亡系统：

- 尸体、部位、血迹、骨、灵魂/幽灵、墓室、棺材、纪念碑。
- 未安葬/未纪念可能导致鬼魂骚扰。
- 死亡事件影响见证者压力和艺术/雕刻内容。

### 4.15 Strange Mood / 神秘心情与神器

流程：

1. 某矮人进入 strange mood。
2. 占用对应技能相关工坊。
3. 按隐含材料列表收集 1-3 个基础组件与若干装饰材料。
4. 材料不足时在工坊显示线索。
5. 完成后生成半随机 artifact，通常提升相关技能到 legendary 或更高；possessed mood 例外可能不给经验。
6. 失败可能导致发疯、死亡、暴力事件。

这是“物品系统 + 工坊 + 心理 + 叙事”的交叉样板，应作为中期重点实现。

---

## 5. Adventure Mode 设计拆解

### 5.1 角色创建

- 世界选择：必须存在可控主流人口文明；不同文明提供不同站点、装备和技能池。
- 种族：矮人、人类、精灵、哥布林、动物人、实验体等，取决于世界与文明状态。
- 文明：决定初始地点、文化知识、语言、装备、社会关系。
- Destiny：Ordinary/Hero/Chosen，不改变世界功能，但影响引导、起始条件、同伴招募。
- 难度：属性点、技能点、装备点。
- 属性：身体与灵魂属性。
- 技能：武器、格斗、防御、移动、观察、社交、表演、阅读、制作等。
- 装备、坐骑、宠物、小队成员。

### 5.2 世界探索

- 局部 tile 移动与大地图 fast travel。
- 城镇、堡垒、森林聚落、黑暗堡垒、洞穴、神庙、巢穴、遗迹、玩家旧堡垒。
- 传闻/任务：询问困扰、寻找怪物、神器、地点、人物。
- 社交：问路、招募、交易、威胁、称赞、演奏、讲故事。
- 战斗：高度细粒度身体部位战斗。
- 休息、饥渴、睡眠、携带重量、攀爬、游泳、潜行、追踪。
- 退休/死亡/写回历史。

MVP 可实现：单角色 + 局部地图 + 战斗 + 对话/传闻 + fast travel；暂不做建造。

---

## 6. Legends Mode 设计拆解

Legends 是事件溯源与数据可视化系统。每个世界事件应有结构化记录：

```ts
type HistoricalEvent = {
  id: string;
  time: GameTime;
  type: string;
  actors: EntityRef[];
  site?: SiteId;
  region?: RegionId;
  artifact?: ArtifactId;
  payload: Record<string, unknown>;
  publicKnowledge: boolean;
  discoveredBy?: string[];
};
```

主要页面：

- World overview：年代、地图、文明总览。
- Civilizations：领袖、战争、贸易、站点、灭亡/兴起。
- Sites：堡垒、城镇、洞穴、塔、黑暗堡垒、神庙、巢穴。
- Historical figures：出生、关系、杀戮、职位、迁徙、死亡、转化。
- Artifacts：创建者、材料、装饰、持有者、盗窃、丢失、战争。
- Events timeline：过滤、搜索、图谱。
- Maps：生物群系、政治、地形、邪恶/善良区域、文明边界。

---

## 7. Web 技术架构建议

### 7.1 前端

- 渲染：Canvas 2D 起步；地图大、层级多时可切 WebGL/Pixi/Phaser。
- UI：React/Svelte/Vue 均可；重点是命令面板、对象详情、列表过滤、暂停/速度控制、警报日志。
- 地图视图：正交 top-down + Z-level 切换。不要一开始做复杂 3D。
- 状态同步：前端只发玩家命令，模拟状态由 engine 产出 snapshot/diff。

### 7.2 模拟引擎

推荐放在 Web Worker 或后端：

- Tick scheduler：暂停、单步、速度倍率。
- ECS 或 hybrid ECS：Tile/Item/Creature/Job/Building 分表。
- Deterministic RNG：所有随机都基于 seed 与事件上下文，便于复现和调试。
- Event log：所有重大事件写入历史；UI announcement 由事件派生。
- Pathfinding：分层 A* + 连通区缓存 + job 距离估算。
- Simulation LOD：远处/非活动区域降频。

### 7.3 存档

- Snapshot + event log。
- 大地图/局部地图分 chunk 保存。
- Entity tables 单独保存；Tile chunks 压缩。
- 可导出 Legends JSON/XML。

### 7.4 性能雷区

- 每 tick 全量扫描 stockpile、items、jobs。
- 每个矮人独立全图 A*。
- 流体每 tick 全图 cellular automata。
- 深层容器递归与物品权限检查。
- 个体心理回忆过多。
- 战斗日志过细导致 UI 卡顿。

优化策略：

- Dirty regions：只有变更区域重算。
- Job index：按 recipe/material/tile 分类索引。
- Reservation system：资源预定避免反复取消。
- Path cache：同区域同目标批量复用。
- Fluid active frontier：只更新有变化的液体边界。
- Event compaction：低价值事件聚合，高价值事件完整保存。

---

## 8. 开发里程碑

### Milestone 0：可视化与数据骨架

- 3D tile chunk、Z-level 视图、暂停/速度、命令日志。
- 静态矮人/物品/建筑显示。
- 沙盒生成小地图。

### Milestone 1：最小堡垒循环

- 挖掘、砍树、建墙/地板/楼梯。
- Job/worker 调度。
- Stockpile、搬运。
- Carpenter/Stoneworker 工坊与基础家具。
- 食物/酒简化需求。

### Milestone 2：可持续殖民地

- 农场、酿酒、厨房。
- 睡眠、卧室、餐厅、会议区。
- 基础 thought/need/stress。
- 移民波、贸易商队。

### Milestone 3：行业与行政

- Manager、work orders、bookkeeper、broker。
- 金属工业、纺织、皮革、动物。
- Room value、贵族要求。

### Milestone 4：危险与防御

- 野兽、盗贼、敌对文明袭击。
- Squad、训练、装备、战斗。
- 医院与伤病。
- 陷阱、门、桥、基础围攻。

### Milestone 5：世界生成与历史

- 参数化世界生成。
- 文明、站点、历史人物、事件日志。
- Legends browser。
- Retire/abandon fortress。

### Milestone 6：深层模拟

- 流体压力、泵、冻结、岩浆、温度、火。
- Strange moods、artifacts、雕刻/艺术叙事。
- 高级心理、记忆、宗教、公会、图书馆。

### Milestone 7：Adventure Mode

- 角色创建、小队、fast travel、局部地图探索。
- 任务/传闻、社交、战斗、重访旧堡垒。
- 与 Legends/世界历史双向写入。

---

## 9. 功能优先级矩阵

| 域 | P0 必做 | P1 中期 | P2 深化 |
|---|---|---|---|
| 地图 | 3D tile、Z 切换、挖掘 | 洞窟、含水层、地质层 | 岩浆海、隐藏地下文明 |
| 矮人 AI | Job 执行、吃喝睡 | 需求、压力、社交 | 记忆、人格变化、创伤 |
| 物流 | Stockpile、搬运、预定 | Links、容器、wheelbarrow | Minecart、复杂路线 |
| 工坊 | 手动任务 | Work orders、条件 | Workshop profile、复杂配方 |
| 农业 | 地下作物、酿酒 | 地上作物、厨房 | 季节、肥料、腐烂 |
| 工业 | 木/石 | 金属、纺织、皮革 | 玻璃、陶瓷、肥皂、纸书 |
| 社会 | 卧室/餐厅 | 酒馆/神庙/公会 | 图书馆、访客、学术 |
| 军事 | 简单敌人和攻击 | Squad、装备、训练 | 围攻 AI、攻城器械 |
| 医疗 | 受伤休养 | 医院流程 | 组织层、感染、残疾 |
| 流体 | 简单水深 | 压力/泵/冻结 | 岩浆/温度/蒸汽/火 |
| 世界 | 固定地图 | 随机地形 | 历史、文明、语言 |
| Legends | 事件日志 | 浏览器 | 地图导出、知识发现 |
| Adventure | 暂不做 | 单角色探索 | 小队、任务、世界写回 |

---

## 10. 可直接给编码工具的 TypeScript 骨架

```ts
export type GameTime = { year: number; month: number; day: number; tick: number };
export type Vec3 = { x: number; y: number; z: number };

export interface GameState {
  world: WorldState;
  activeSite?: SiteState;
  creatures: Record<string, Creature>;
  items: Record<string, Item>;
  buildings: Record<string, Building>;
  jobs: Record<string, Job>;
  events: HistoricalEvent[];
  rngState: string;
}

export interface WorldState {
  id: string;
  seed: string;
  time: GameTime;
  regions: RegionTile[];
  civilizations: Record<string, Civilization>;
  sites: Record<string, SiteSummary>;
  historicalFigures: Record<string, HistoricalFigure>;
}

export interface SiteState {
  id: string;
  type: 'fortress' | 'town' | 'forest_retreat' | 'dark_fortress' | 'cave' | 'lair';
  map: ChunkedTileMap;
  stockpiles: Record<string, Stockpile>;
  zones: Record<string, Zone>;
  locations: Record<string, Location>;
  military: MilitaryState;
  economy: EconomyState;
}

export interface Creature {
  id: string;
  speciesId: string;
  name: string;
  pos: Vec3;
  citizenOf?: string;
  attributes: Record<string, number>;
  skills: Record<string, SkillState>;
  needs: NeedState[];
  thoughts: Thought[];
  stress: number;
  focus: number;
  inventory: string[];
  wounds: Wound[];
  currentJobId?: string;
  laborSettings: Record<string, boolean>;
  burrows: string[];
}

export interface Job {
  id: string;
  type: string;
  source: 'designation' | 'workshop' | 'workOrder' | 'zone' | 'need' | 'emergency';
  targetPos?: Vec3;
  requiredLabor?: string;
  requiredSkill?: string;
  requiredItems: ItemRequirement[];
  reservedBy?: string;
  priority: number;
  status: 'open' | 'reserved' | 'inProgress' | 'suspended' | 'complete' | 'cancelled';
}
```

---

## 11. 规则实现原则

1. **先做可解释模拟，再做真实细节。** 原作细节极深，但 Web 项目的第一需求是稳定、可调试、可迭代。
2. **每个系统都写入事件。** 事件不仅用于提示，也用于 Legends、调试、回放和存档压缩。
3. **所有资源都必须可预定。** 工坊、物品、床、医院器械、目标 tile、敌人都应有 reservation。
4. **所有 AI 决策都必须能解释。** UI 中显示“为什么这个任务没人做”：无材料、无路径、无劳动、被 burrow 限制、危险、资源已预定。
5. **把“版本细节”做成数据。** 生物、材料、配方、作物、建筑、职业、技能、需求、思绪，都应由 JSON/raw-like 数据驱动。
6. **保留 Arena。** 每新增一个复杂规则，都能在 Arena 中单独测试。

---

## 12. 初始开发 Backlog

### Epic A：Map Engine

- A1 chunked 3D tile map。
- A2 Z-level renderer。
- A3 tile flags：outside/light/subterranean/revealed。
- A4 designations：mine/channel/stair/chop/gather。
- A5 pathfinding + passability。

### Epic B：Agent & Job Engine

- B1 creature table。
- B2 job creation/reservation/execution。
- B3 hauling。
- B4 hunger/thirst/sleep。
- B5 skill gain。

### Epic C：Fortress Economy

- C1 stockpiles。
- C2 workshops。
- C3 recipes。
- C4 manager work orders。
- C5 trade depot and caravans。

### Epic D：Wellbeing

- D1 bedrooms/dining rooms。
- D2 simple happiness/stress。
- D3 preferences。
- D4 needs/focus。
- D5 death/corpses/memorials。

### Epic E：Combat

- E1 hostile creature spawn。
- E2 melee attack。
- E3 wounds and death。
- E4 squads。
- E5 ranged combat。

### Epic F：World & Legends

- F1 world seed and region map。
- F2 civilizations/sites。
- F3 historical events。
- F4 Legends browser。
- F5 Adventure entry。

---

## 13. 重要不确定项与调研缺口

- 原作很多细节是硬编码、wiki 逆向、版本迁移结果，不能把所有 wiki 条目当成当前版本绝对事实。
- 战斗公式、心理公式、流体/温度的精确数值若要 1:1，需要专门做实验或对公开资料进行二次整理。
- 官方仍在更新，特别是 v52-v53 之后有 Lua、diplomacy、siege、dino/extinct creature 等变化，应把版本差异作为数据层管理。
- Web 版应避免一开始追求“完整复刻”，否则会被流体、心理、战斗、物流四个系统同时拖垮。

---

## 14. 最小可玩规格草案

第一版 Web Demo 的定义：

- 生成一个 64×64×16 的局部地图，有地表、土层、岩层、小河、树木、矿石。
- 7 个矮人，有名字、技能、饥渴、睡眠、简单压力。
- 玩家可暂停、挖掘、砍树、建造、放置仓库、放置工坊、下生产任务。
- 矮人能自动执行：挖矿、搬运、制作床/桌/椅/门/酒桶、种植 plump-helmet-like 作物、酿酒。
- 有卧室、餐厅、会议区；缺桌吃饭/睡地板会产生坏想法。
- 秋季来一支商队，可交易。
- 第一年后有移民。
- 有一个小型敌袭；可组建 squad 进行简单近战。
- 所有重大事件写入 Legends-like timeline。

这个 Demo 已经足以让 VibeCoding 工具围绕真实系统开始增量实现，而不是只做 UI 壳。
