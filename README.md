本规范制定的三个目标：**简洁**、**统一**、**开放**，关于如何设计良好风格的 Restful API，Github 有一份[满分答案](https://docs.github.com/en/rest/reference/actions)，熟读三遍，其义自现。

本规范将使用尽可能简单的表述方式从以下几个部分分别作出约定：

- 基础约定
- 创建类接口
- 查询类接口
- 文件类接口
- 敏感类接口
- 图表类接口

> 注意本规范是笔者在所在公司内部制定的规范整理而来的接口规范模板，仅供你在制定规范时候参考，在落地时应根据团队实际情况作出调整，欢迎对本规范进行补充

## **1. 基础约定**

**1.1 接口路径以 /api 或 /[version]/api 开头**

正确：[/api/tasks](/api/tasks) 或 [/v2/api/tasks](/v2/api//tasks)

错误：[/biz/tasks](/biz/tasks) 或 [/biz/api/tasks](/biz/api/tasks)

注意：一个产品无论后端有多少个服务组成也应该只有一个 API 入口

**1.2 接口路径以 api/aa-bb/cc-dd 方式命名**

正确：[/api/task-groups](/api/task-groups)

错误：[/api/taskGroups](/api/taskGroups)

**1.3 接口路径使用资源名词而非动词，动作应由 HTTP Method 体现，资源组可以进行逻辑嵌套**

正确：POST [/api/tasks](/api/tasks) 或 [/api/task-groups/1/tasks](/api/task-groups/1/tasks) 表示在 id 为 1 的任务组下创建任务

错误：POST [/api/create-task](/api/create-task)

**1.4 接口路径中的资源使用复数而非单数**

正确：[/api/tasks](/api/tasks)

错误：[/api/task](/api/task)

**1.5 接口设计面向开放接口，而非单纯前端业务**

要求我们在给结构路径命名时候面向通用业务的开放接口，而非单纯前端业务，以获取筛选表单中的任务字段下拉选项为例

正确：[/api/tasks](/api/tasks)

错误：[/api/task-select-options](/api/task-select-options)

虽然这个接口暂时只用在表单的下拉选择中，但是需要考虑的是在未来可能会被用在任意场景，因此应以更通用方式提供接口交由客户端自由组合

**1.6 规范使用 HTTP 方法和响应码**

- **常见方法**
  - 获取列表 GET /api/tasks
  - 获取单个 GET /api/tasks/1
  - 删除单个 DELETE /api/tasks/1
  - 创建单个 POST /api/tasks
  - 差量修改单个 PATCH /api/tasks/1
  - 全量修改单个 PUT /api/tasks/1
- **常见响应**
  - 200：创建成功，通常用在同步操作时
  - 202：创建成功，通常用在异步操作时，表示请求已接受，但是还没有处理完成
  - 400：参数错误，通常用在表单参数错误
  - 401：授权错误，通常用在 Token 缺失或失效，注意 401 会触发前端跳转到登录页
  - 403：操作被拒绝，通常发生在权限不足时，注意此时务必带上详细错误信息
  - 404：没有找到对象，通常发生在使用错误的 id 查询详情
  - 5XX：服务器错误

**1.7 基础外层数据结构**

- 不分页数据

  ```javascript
  {
    code: 20000
    status: 200;
    message: "请求成功",
    data: {
      id: 1,
      name: '任务1'
    }
  }
  ```

- 分页数据

  ```javascript
  {
    code: 20000
    status: 200;
    message: "请求成功",
    data: {
      items: [{
        id: 1,
        name: '任务1'
      }, {
        id: 2,
        name: '任务2'
      }],
      total: 2
    }
  }
  ```

需要注意的是其中 `code` 表示业务编码，`status` 表示 HTTP 状态码，如此设计的原因是部分场景下前后端之间存在不可控的网关或代理（比如某些网关有一些流量控制策略会导致直接返回 403 状态码，此时客户端无法分辨 403 是网关的还是业务方），类似这类情况下为了能够让客户端正确分辨业务方的处理结果则需要在响应体加上 `status`，而 `code` 表示的业务编码是为了帮助工程师更容易定位问题，它并不是必须的，这取决你们团队风格。

> 尽管在响应体中体现了状态码，但这并不代表所有 HTTP 就可以全部返回 200 了，无论如何在条件范围内请尽可能使用标准的 HTTP 状态码

**1.8 请求和响应字段采用 `aa_bb_cc` 方式命名**

```javascript
// 正确
{
  role_ids: [11,12,35],
}
// 错误
{
  roleIds: [11, 12, 35],
  RoleIds: [11, 12, 35],
  ROLE_IDS: [11, 12, 35]
}
```

**1.9 时间字段以 ISO 8601 格式返回 ：`YYYY-MM-DDTHH:MM:SSZ`**

**1.10 常见业务字段约定**

名称：name

状态：status

创建时间：created_at

更新时间：updated_at

**1.11 空数组应该使用 []，而不是 null**

```javascript
// 正确
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    role_ids: [],
  }
}
// 错误
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    role_ids: null
  }
}
```

**1.12 前后端传输过程以标准 JSON 格式，避免反复正反序列化**

```javascript
// 正确
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    role_ids: [11,12,35],
  }
}
// 错误
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    role_ids: '[11,12,35]'
  }
}
```

## **2. 创建类接口**

**2.1 创建完成后直接返回 id**

```javascript
{
  code: 20000
  status: 200;
  message: "创建成功",
  data: {
     id: 1,
  }
}
```

**2.2 关联关系只以 id 为标识，其它字段不应依赖客户端，**

以创建用户为例：POST [/api/users](/api/users)

```javascript
// 正确
{
  username: 'user1'
  password: 'xxxx',
  role_ids: [1,2,3]
}

// 错误
{
  username: 'user1'
  password: 'xxxx',
  role_ids: [{
    id: 1,
    name: '角色1'
  }, {
    id: 2,
    name: '角色2'
  }, {
    id: 3,
    name: '角色3'
  }]
}
```

**2.3 参数缺失错误以数组形式返回，并附带用户友好的提示**

```javascript
{
  code: 40000
  status: 400;
  message: "参数错误",
  data: {
    errors: [{
       field: 'name',
       message: '缺失'
    }]
  }
}
```

## **3. 查询类接口**

**3.1 排序使用 sort 和 order**

例如 GET [/api/](/api/user/1)[tasks?](/api/user/1)[sort=create_time&order=descend](/api/users?sort_key=create_time&sort_order=descend)

其中 `order` 为 `descend` 时表示降序，为 `ascend` 时表示升序

**3.2 分页使用 page 和 per_page**

例如 GET [/api/tasks?page=1&per_page=10](/api/tasks?page=1&per_page=10)

其中 `page` 从 1 开始，而不是 0，如果没有传递 `per_page` 和 `page` 参数表示不分页获取所有数据

**3.3 普通筛选使用健值对，举筛选使用数组合并拼接，多列模糊查询使用 `keyword` 关键词，时间区间使用 `start_xx_time` 和 `end_xx_time`**

例如 GET [/api/tasks?creator=ming](/api/tasks?creator=ming) 表示查询所有 ming 用户创建的任务

例如 GET [/api/tasks?keyword=ming](/api/tasks?keyword=ming) 表示查询任意列包含 ming 关键词的任务

例如 GET [/api/tasks?status=pending,complete](/api/tasks?status=pending,complete) 表示查询状态为阻塞和完成的任务

例如 GET [/api/tasks?start_create_time=2022-04-20T00:00:00-02:00Z&end_create_time=2022-04-25T23:59:59-02:00Z](/api/tasks?start_create_time=2022-04-20T00:00:00-02:00Z&end_create_time=2022-04-25T23:59:59-02:00Z) 表示查询 2022.04.22-2022.4.25 之间创建的任务

**3.4 应该返回所有关联数据展开详情**

```javascript
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    username: 'ming'
    role_ids: [1,2,3],
    // 注意这里会展开角色信息，便于客户端显示
    roles: [{
       id: 1,
       name: '角色1'
    }, {
       id: 2,
       name: '角色2'
    }, {
       id: 3,
       name: '角色3'
    }]
  }
}
```

**3.5 状态应该使用有语义的枚举，而不是纯数字**

```javascript
// 正确
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: '训练任务'
    status: 'training'  // 'pending' |'training' | 'complete' | 'error'
  }
}
// 错误
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: '训练任务'
    status: 0
  }
}
```

**3.6 合理自然嵌套结构而不是平铺**

```javascript
// 正确
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: '训练任务'
    creator_id: 1,
    creator_name: 'ming'
  }
}
// 错误
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: '训练任务'
    creator: {
      id: 1,
      username: 'ming'
    }
  }
}
```

**3.7 删除接口应酌情提供批量删除**

例如 DELETE [/api/tasks/1](/api/tasks/1)表示删除 id 为 1 的任务

例如 DELETE [/api/tasks?ids=1,2,3](/api/tasks?ids=1,2,3) 表示批量删除 id 为 1 或 2 或 3 的任务（如果列表数据量较大，容易沉淀无用数据的应该提供批量删除功能，比如任务、文件、日志等）

## **4. 文件类接口**

**4.1 提供统一文件上传接口，应该支持单个和批量上传所有类型文件**

- 单文件上传 [/api/files](/api/file)

  ```javascript
  // 请求，注意这里是 FormData
  {
    file: File
  }

  // 响应
  {
    code: 20000
    status: 200;
    message: "上传成功",
    data: {
      id: 'bb313c99',
      url: '/files/bb313c99.pdf'
      name: '合同.pdf' // 原文件的名称
    }
  }
  ```

- 多文件上传[/api/multiple-files](/api/multiple-files)

  ```javascript
  // 请求，注意这里是 FormData
  {
    files: [File, File]
  }

  // 响应
  {
    code: 20000
    status: 200;
    message: "上传成功",
    data: [{
      id: 'bb313c99',
      url: '/files/bb313c99.pdf'
      name: '合同1.pdf' // 原文件的名称
    }, {
      id: 'bb313c88',
      url: '/files/bb313c88.pdf'
      name: '合同2.pdf' // 原文件的名称
    }]
  }
  ```

**4.2 文件路径应该由服务端至少补全至根路径**

```javascript
// 正确
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: 'ming'
    avatar: '/files/bb313c99.png',
    // 或
    avatar: 'https://cdn.xxx.com/apps/1/files/bb313c99.png',
  }
}
// 错误
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    id: 1,
    name: 'ming'
    avatar: 'bb313c99.png'
  }
}
```

**4.3 对于使用到文件的接口应该使用文件 id 或地址而非 FormData**

```javascript
// 正确
{
  name: 'new task',
  file_id: 'bb313c99',
  // 或
  file_url: '/files/bb313c99.pdf',
}
// 错误
{
  name: 'new task',
  file: File
}
```

注意：应该先由 POST [api/files](/api/files) 上传完文件拿到文件 id 或地址后再执行后续操作

## **5. 敏感类接口**

**5.1 涉及到用户隐私的应该对相关字段做加密处理，**

以登录为例：POST [/api/login](/api/login])

```javascript
// 正确
{
  username: 'ming'
  password: 'U2FsdGVkX1/AnXKSBDbztNBfp4czlZxQ++3jRtNZhY0=',
}

// 错误
{
  username: 'ming'
  password: 'ming@xxx.com',
}
```

注意：本规范不约定使用何种加密算法，请视实际场景而定

## **6. 图表类接口**

**6.1 曲线图、柱状图**

```javascript
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    x_axis: ['2022.04.20','2022.04.21', '2022.04.22']
    series: [{
      name: '新增用户'，
      data： [1,0,2],
      color: '#f5f5f5' // 可选，如果加上的话会使用该色值
    }, {
      name: '活跃用户'，
      data： [1,0,3],  // 注意，没有数据时候也要使用 0 填充，应该和 x_axis 一一对应
      color: '#f5f5f5'
    }]
  }
}
```

**6.2 饼图**

```javascript
{
  code: 20000
  status: 200;
  message: "请求成功",
  data: {
    series: [{
      name: '上线用户',
      value: 1890,
      color: '#f5f5f5' // 可选，如果加上的话会使用该色值
    }, {
      name: '下线用户',
      value: 2000,
      color: '#f5f5f5'
    }]
  }
}
```
