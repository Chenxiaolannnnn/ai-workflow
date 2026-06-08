---
name: skyunion-weekly-fetch
description: 从 IGG Skyunion 内部系统（task.skyunion.net 任务管理 + eip.skyunion.net 需求管理）批量抓取指定人员的任务单和需求单，读取详情内容（任务说明、备注、历史记录），用于生成周报或工作汇总。当用户说"查一下XX上周的任务"、"帮我看看任务系统里的工作内容"、"从需求单/任务单查上周工作"、"生成周报前先查任务系统"时，必须使用此技能。即使用户只说"帮我写周报"但提到了任务系统、需求单、工号等内部人员信息，也应主动使用此技能。
---

# Skyunion 任务&需求抓取技能

## 概述

通过浏览器中运行的 JS（`evaluate_script` / DevTools MCP），调用 Skyunion 内部 API 批量获取任务单和需求单，再从详情页 DOM 提取完整内容（任务说明、备注、历史操作）。整个过程不依赖截图，全程 API + JS DOM 读取。

---

## 前置条件

1. 用户已在浏览器中登录对应系统（task / EIP）。若未登录，导航到登录页等用户完成授权。
2. 需要在**对应域名的页面**下执行 JS fetch，因为两个系统有独立 Cookie：
   - 任务系统：确保当前页面是 `task.skyunion.net`
   - 需求系统：确保当前页面是 `eip.skyunion.net`

---

## Step 1：查找用户内部 ID

在 `task.skyunion.net` 任意页面执行：

```js
async () => {
  const resp = await fetch(`/v2/api/common/user/list?keyword=姓名关键字`);
  const data = await resp.json();
  return data.data; // 返回用户列表，找 id 字段
}
```

记录目标用户的数字 ID（如 `11866`），后续搜索用。

---

## Step 2：搜索任务单

在 `task.skyunion.net` 页面执行，获取执行人的全部任务后**在 JS 内过滤时间范围**：

```js
async () => {
  // 获取全部任务（不带时间过滤，避免服务端分页丢数据）
  const fetchPage = async (page) => {
    const body = `page%5Bpage%5D=${page}&page%5Bpagesize%5D=100&sort=2&handler_id%5B%5D=${USER_ID}`;
    const resp = await fetch('/v2/api/task/search_new', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'X-Requested-With': 'XMLHttpRequest'
      },
      body
    });
    const data = await resp.json();
    return {
      list: data.data?.list || [],
      total: data.data?.pagination?.total || 0
    };
  };

  const first = await fetchPage(1);
  const totalPages = Math.ceil(first.total / 100);
  const rest = totalPages > 1
    ? await Promise.all(Array.from({length: totalPages - 1}, (_, i) => fetchPage(i + 2)))
    : [];
  const allTasks = [first.list, ...rest.map(r => r.list)].flat();

  // 在 JS 内过滤：上周创建 或 上周有更新
  const weekStart = new Date('YYYY-MM-DD').getTime() / 1000; // 替换为实际日期
  const weekEnd   = new Date('YYYY-MM-DD').getTime() / 1000;

  return allTasks
    .filter(t => {
      const created = new Date(t.t_dateline).getTime() / 1000;
      const updated = new Date(t.t_updated_dt).getTime() / 1000;
      return (created >= weekStart && created < weekEnd)
          || (updated >= weekStart && updated < weekEnd);
    })
    .map(t => ({
      id: t.t_id,
      sign: t.sign,           // ← 详情访问必须有 sign
      title: t.t_title,
      status: t.t_status_text,
      handler: t.t_handler_id_text,
      reporter: t.t_reporter_id_text,
      created: t.t_dateline,
      updated: t.t_updated_dt,
      expected: t.t_expected_dt
    }));
}
```

> **注意**：`sign` 字段是访问详情页的校验码，缺少它会弹出"校验码错误"并无法读取内容。必须从 `search_new` 返回结果中取，不能猜测。

---

## Step 3：读取任务详情

对每条任务，在 `task.skyunion.net` 页面执行（可 `Promise.all` 并行）：

```js
async (id, sign) => {
  const resp = await fetch(
    `/task_detail.php?action=view&sign=${sign}&task_id=${id}`
  );
  const html = await resp.text();
  const doc = new DOMParser().parseFromString(html, 'text/html');
  const body = doc.body.innerText;

  // 提取任务说明区域（从"任务说明"到"历史记录"之间）
  const lines = body.split('\n').map(l => l.trim()).filter(l => l.length > 0);
  const descIdx = lines.findIndex(l => l.includes('任务说明'));
  const histIdx = lines.findIndex(l => l.includes('历史记录'));
  const descSection = lines
    .slice(descIdx > -1 ? descIdx : 0, histIdx > -1 ? histIdx : lines.length)
    .join('\n')
    .substring(0, 2000);

  // 提取历史记录（含备注文本）
  const histSection = body.match(/历史记录[\s\S]{0,3000}/)?.[0] || '';

  return { id, descSection, histSection };
}
```

**备注格式**（在 descSection 中）：
```
[ID:12203772][2026-06-06 16:00:00]  陈晓枫(xiaofengchen02)，0.00小时
备注正文内容...
```

**历史操作格式**（在 histSection 中）：
```
历史记录
操作时间    操作人    内容
2026-06-06 16:00:00  陈晓枫(xiaofengchen02)  添加备注(ID:12203772)
2026-06-04 16:26:28  系统  状态变更：已分派 => 已解决
2026-06-01 11:12:51  吕伟涛(weitaolv)  任务重分派：吕伟涛 => 陈晓枫
```

---

## Step 4：搜索需求单

**切换到 `eip.skyunion.net` 页面**后执行（EIP 每页固定 15 条，用 `p` 参数翻页）：

```js
async () => {
  const fetchPage = async (page) => {
    const body = [
      'sponsor_status=1%2C2',
      'id=', 'task_id=',
      'search_product_module=0-0-0',
      'product_id=0',
      'module_id%5B%5D=0', 'module_id%5B%5D=0',
      'status%5B%5D=0',
      'has_upload=0',
      'creator=', 'sponsor=',
      'stime=', 'etime=',   // 留空，JS 里过滤
      'keyword=',
      'sort=4',             // 按最近更新时间
      'cmd=search',
      '_MID_=295',
      `maker%5B%5D=${USER_ID}`,
      `p=${page}`
    ].join('&');

    const resp = await fetch('/reqm/requirement/ajax', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'X-Requested-With': 'XMLHttpRequest'
      },
      body
    });
    const arr = JSON.parse(await resp.text());
    const scriptCmd = arr.find(d => d.cmd === 'script');
    const match = scriptCmd?.data.match(/DbGrid\.update\((\{.*\})\)/s);
    return match ? JSON.parse(match[1]).data : [];
  };

  // EIP 总数需先取第一页，从 pagination 命令里读
  // 实践发现总数固定在页面标题附近，可先取5页覆盖75条
  const pages = await Promise.all([1,2,3,4,5].map(fetchPage));
  const allRows = pages.flat();

  const weekStart = new Date('YYYY-MM-DD').getTime() / 1000;
  const weekEnd   = new Date('YYYY-MM-DD').getTime() / 1000;

  // 去重并过滤时间范围
  const seen = new Set();
  return allRows
    .filter(row => {
      if (seen.has(row.id?.value)) return false;
      seen.add(row.id?.value);
      const created = parseInt(row.order_time?.value || 0);
      const updated = parseInt(row.dateline?.value || 0);
      return (created >= weekStart && created < weekEnd)
          || (updated >= weekStart && updated < weekEnd);
    })
    .map(row => ({
      id: row.id?.value,
      title: row.title?.value,
      status: row.status?.display?.replace(/<[^>]+>/g, ''),
      creator: row.creator_e_id?.display,
      maker: row.maker_e_id?.display,
      created: row.order_time?.display,
      updated: row.dateline?.display,
      taskId: row.task_id?.display?.replace(/<[^>]+>/g, '')
    }));
}
```

---

## Step 5：读取需求详情

在 `eip.skyunion.net` 页面执行（只能在 EIP 域名下读，跨域无法访问）：

```js
async (reqId) => {
  const resp = await fetch(`/reqm/requirementDetail/detail/${reqId}`);
  // 注意：EIP 需求详情页是 iframe 内渲染，直接 fetch 可能拿到外壳页
  // 改用 requirement/ajax 接口更可靠：
  const body = `cmd=getDetail&id=${reqId}&_MID_=295`;
  const resp2 = await fetch('/reqm/requirement/ajax', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
      'X-Requested-With': 'XMLHttpRequest'
    },
    body
  });
  const arr = JSON.parse(await resp2.text());
  return arr; // 按需解析
}
```

> 如 API 返回整页 HTML，则改用与任务系统相同的 DOMParser 方式解析。

---

## 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| "校验码错误"弹窗 | 访问任务详情未带 `sign` | 从 search_new 取 `sign` 字段，不能自行构造 |
| API 返回 `500000 Something went wrong` | 参数格式错误 | 检查 Content-Type 是否为 `application/x-www-form-urlencoded`，以及必填字段是否齐全 |
| EIP 需求搜索返回 HTML 而非 JSON | 在 task 域名下跨域调用 EIP | 切换到 eip.skyunion.net 页面再执行 |
| `search_new` 只返回1条 | `dateline[]` 参数配合 `p_id_operate=1` 只筛"≥某日创建"，非区间 | 不传日期参数，全量拉取后 JS 过滤 `t_dateline` 和 `t_updated_dt` |
| EIP 分页不生效 | 误用 `page` 参数（task 系统参数名） | EIP 用 `p=2`、`p=3` 翻页 |

---

## 执行顺序速查

```
1. task.skyunion.net 已登录？
   └─ 查用户 ID（/v2/api/common/user/list）
   └─ 搜索任务（search_new）→ 全量 → JS 过滤时间段
   └─ 并行读取各任务详情（task_detail.php + DOMParser）

2. eip.skyunion.net 已登录？
   └─ 搜索需求（/reqm/requirement/ajax）→ 5页 → JS 过滤
   └─ 如需详情，在 EIP 域名下读取

3. 汇总结果，供后续生成周报
```
