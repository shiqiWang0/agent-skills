---
name: mock-crud-integration
description: 基于当前项目 mock-server 风格处理通用 CRUD 联调的技能。用于后端接口未成熟时，前端先基于 mock-server 独立完成列表、详情、创建、更新、删除、导入导出等接口联调；先对齐前端 hooks、请求响应契约和关键接口行为，再补齐 mock-server 路由、返回结构、异常分支与联调校验。用户提到“CRUD 联调”“CURD 联调”“补 mock 接口”“按前端请求补增删改查 mock”“先用 mock-server 跑通接口”“后端没好先前端自联调”时使用。
---

# Mock CRUD 联调技能

目标：在后端接口还没稳定或还没开发完时，按当前仓库的真实接口消费方式，快速补齐一套能支撑前端独立联调的 CRUD mock，并保证后续真实后端只要契约一致，就能低成本切换。

这里说的“契约一致”不只是字段格式一致，还包括接口行为一致，例如：

- method 一致：`PUT` / `PATCH` / `DELETE`
- 包结构一致：是否走 `response.data.data`
- 状态码和空响应一致：`200` / `201` / `204`
- 创建、更新、删除后的返回内容一致
- 分页、搜索、排序、导入导出等参数和行为一致

## 适用范围

- 基于 `mock-server/*.js` 新增或改造模块接口
- 前端已有 query hooks / types，需要反推接口契约
- 需要先用本地 mock-server 跑通页面，再切真实后端
- 后端接口未完成，但前端希望先独立完成 CRUD 自联调
- 场景包含列表、详情、创建、更新、删除，也可带导入、导出

## 先看哪里

1. 找前端消费入口：
   - `src/controllers/API/queries/**`
   - `src/controllers/API/helpers/constants.ts`
   - 页面或 store 中对 query/mutation 的调用
2. 找 mock-server 样例：
   - 优先参考 `mock-server/skill-api.js`
3. 如果是技能编辑器类似场景，再看：
   - `docs/ai-development/skill-api.md`
   - `docs/ai-development/skill-edit/technical-design.md`

## 当前项目已经确定的 mock 风格

- 用 `express` + `cors` + `express.json()`
- 路由前缀统一为 `/api/v1`
- 业务数据放在 `response.data.data`
- 列表接口常见格式：
  - `res.json({ data: [...] })`
- 详情/创建/更新接口常见格式：
  - `res.json({ data: { ... } })`
- 删除接口可返回：
  - `res.json({ data: null })`
- 导入接口常用 `multipart/form-data`
- 导出接口直接返回二进制，并设置下载响应头

不要擅自把返回值改成裸数组或裸对象，除非前端调用处已经确认不是按 `response.data.data` 取值。

## 联调目标边界

这个 skill 的目标不是“临时造点假数据把页面点亮”，而是让 mock 成为真实后端的前置契约实现。

因此默认要求：

- 前端可以在 mock-server 上完成自联调
- 真实后端接入时尽量只切 base URL 或少量接口实现
- 不允许因为 mock 写得太随意，导致真实联调时大面积返工

如果当前只能确定字段、还不能确定接口行为，要在结果里明确标注风险，不要假装已经完成可切换契约。

## 标准工作流

### 1. 先反推契约，不要先写接口

至少确认这些信息：

- Base URL key 和具体路径
- 每个接口的 HTTP method
- path params / query params / body / formData
- 前端最终读取的是哪一层字段
- 列表字段是否需要 query 层二次映射
- 是否需要 `version`、分页字段、状态字段
- 是否存在前端兜底逻辑，避免 mock 行为与真实行为冲突

如果 query 层存在映射函数，mock 返回要贴近“后端原始格式”，不要直接返回页面内部格式。

除了字段结构，还要确认这些行为语义：

- 创建后列表是否应立即可见
- 更新后是否返回最新详情
- 删除后返回 `null`、空对象还是 `204`
- 列表是否需要分页、过滤、排序
- 详情是否需要完整快照还是摘要字段

### 2. 设计最小可用数据模型

优先抽这三层：

- `seed list`：列表初始数据
- `detail factory`：根据 id 生成详情
- `store`：内存态 `Map` 或对象

推荐模式：

```js
const listSeed = [{ id: "item-1", name: "demo" }];

const createDetail = (id, name = "demo") => ({
  id,
  name,
  version: 1,
});

const store = new Map();
```

这样可以同时支撑：

- 列表刷新
- 创建后立刻可见
- 更新后详情回读
- 删除后列表收敛

### 3. 按 CRUD 闭环补路由

基础五件套：

1. `GET /xxx` 列表
2. `GET /xxx/:id` 详情
3. `POST /xxx` 创建
4. `PUT/PATCH /xxx/:id` 更新
5. `DELETE /xxx/:id` 删除

如果当前模块有导入导出，再补：

6. `POST /xxx/import`
7. `GET /xxx/export/:id`

### 4. 保证错误分支可联调

至少保留：

- 404：资源不存在
- 400：必填参数缺失
- 500：兜底异常

建议提供统一辅助函数：

```js
const getItemOrThrow = (id) => {
  const item = store.get(id);
  if (!item) {
    const error = new Error(`Item not found: ${id}`);
    error.status = 404;
    throw error;
  }
  return item;
};
```

### 5. 自检前端联调链路

至少验证这几个动作：

1. 首次打开页面，列表能正常渲染
2. 点击某项，详情能回填
3. 新建后无需刷新即可出现在列表
4. 更新后详情与列表状态一致
5. 删除后列表消失，详情态不报错
6. 异常分支不会把页面打崩

## 落地规则

### 列表接口

- 返回后端风格字段名
- 若前端 query 层会映射字段，mock 不要提前映射
- 时间字段保持数值时间戳，避免 UI 格式化异常

### 详情接口

- 返回页面保存所需的完整快照
- 能一次返回全量时，不要拆散成多个请求

### 创建接口

- 创建成功后要能立即被列表接口读到
- `id` 尽量本地生成，推荐 `randomUUID()`
- 返回值结构尽量与详情接口一致

### 更新接口

- 优先按前端实际请求体做全量覆盖或明确 merge
- 如果前端回传 `version`，mock 也应保留并递增
- 不要默默丢弃前端会再次依赖的字段

### 删除接口

- 删除前先校验存在性
- 返回 `data: null` 最稳妥

### 导入导出接口

- 导入：确认字段名，通常是 `file`
- 导出：设置
  - `Content-Type: application/zip`
  - `Content-Disposition: attachment; filename="xxx.zip"`

## 编码模板

可直接按下面结构扩展：

```js
const listSeed = [];
const store = new Map();

const clone = (value) => JSON.parse(JSON.stringify(value));

const createDetail = (id, payload = {}) => ({
  id,
  name: payload.name || "未命名",
  version: payload.version ?? 1,
});

const seed = () => {
  listSeed.forEach((item) => {
    store.set(item.id, createDetail(item.id, item));
  });
};

const getOrThrow = (id) => {
  const item = store.get(id);
  if (!item) {
    const error = new Error(`Item not found: ${id}`);
    error.status = 404;
    throw error;
  }
  return item;
};

app.get(`${API_PREFIX}/items`, (_req, res) => {
  res.json({ data: Array.from(store.values()).map(clone) });
});

app.get(`${API_PREFIX}/items/:id`, (req, res, next) => {
  try {
    res.json({ data: clone(getOrThrow(req.params.id)) });
  } catch (error) {
    next(error);
  }
});

app.post(`${API_PREFIX}/items`, (req, res, next) => {
  try {
    const id = `item-${randomUUID()}`;
    const detail = createDetail(id, req.body);
    store.set(id, detail);
    res.status(201).json({ data: clone(detail) });
  } catch (error) {
    next(error);
  }
});

app.put(`${API_PREFIX}/items/:id`, (req, res, next) => {
  try {
    const current = getOrThrow(req.params.id);
    const nextDetail = {
      ...current,
      ...req.body,
      id: req.params.id,
      version: (req.body.version ?? current.version) + 1,
    };
    store.set(req.params.id, nextDetail);
    res.json({ data: clone(nextDetail) });
  } catch (error) {
    next(error);
  }
});

app.delete(`${API_PREFIX}/items/:id`, (req, res, next) => {
  try {
    getOrThrow(req.params.id);
    store.delete(req.params.id);
    res.json({ data: null });
  } catch (error) {
    next(error);
  }
});
```

## 处理策略

- 优先做“最小但闭环”的 mock，不先追求完美字段覆盖
- 页面如果依赖列表摘要字段和详情完整字段，允许两套结构，但要共享同一个 store 来源
- mock 要模拟真实后端的命名和包裹层，不要为了前端方便改契约
- 发现前端 hook 写法和文档不一致时，以实际代码为准
- 发现字段不确定时，先搜索调用链，不要猜
- 用户如果只是说“后端还没好，先把页面联调起来”，默认按“前端可独立自联调，后续可平滑切真实接口”的目标来做
- 如果只能保证字段一致、不能保证行为一致，要明确告知“这只是 UI mock，不是可平滑替换的接口 mock”

## 输出要求

执行这类任务时，最终至少给出：

1. 改了哪些 mock 路由
2. 对齐了哪些前端契约
3. 还缺哪些真实后端语义
4. 做了哪些本地验证，哪些还没验证

## 参考资料

- 通用契约检查表：`references/crud-contract-checklist.md`
