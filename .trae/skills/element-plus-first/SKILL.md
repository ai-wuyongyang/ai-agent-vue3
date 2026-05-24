---
name: "element-plus-first"
description: "优先使用 Element Plus 组件实现页面效果。Invoke when implementing UI pages, layouts, forms, tables, dialogs, drawers, navigation, or other visual interactions in this project."
---

# Element Plus First

## 目的

在当前项目中实现页面效果时，优先使用 `Element Plus` 现成组件与设计能力，减少重复造轮子，保持界面风格统一、交互一致、开发效率更高。

## 触发时机

当出现以下任务时，优先启用本技能：

- 实现新页面
- 调整页面布局
- 搭建表单、表格、弹窗、抽屉、标签页、菜单、分页
- 实现搜索区、筛选区、卡片区、侧边栏、头部导航
- 做常见状态展示，如空状态、加载态、消息提示、确认框
- 用户要求“做个页面效果”“改个 UI”“补个后台管理界面”

## 核心规则

1. 优先选用 `Element Plus` 官方组件，而不是手写基础 UI。
2. 能通过组合 `Element Plus` 组件实现的效果，不优先自定义原子组件。
3. 布局优先考虑：
   - `el-container`
   - `el-header`
   - `el-aside`
   - `el-main`
   - `el-row`
   - `el-col`
   - `el-space`
   - `el-card`
4. 表单与数据录入优先考虑：
   - `el-form`
   - `el-input`
   - `el-select`
   - `el-date-picker`
   - `el-radio`
   - `el-checkbox`
   - `el-switch`
   - `el-upload`
   - `el-button`
5. 数据展示优先考虑：
   - `el-table`
   - `el-descriptions`
   - `el-tag`
   - `el-statistic`
   - `el-avatar`
   - `el-empty`
   - `el-skeleton`
6. 反馈与交互优先考虑：
   - `el-dialog`
   - `el-drawer`
   - `el-popconfirm`
   - `el-tooltip`
   - `el-message`
   - `el-notification`
   - `el-loading`
7. 导航与结构优先考虑：
   - `el-menu`
   - `el-breadcrumb`
   - `el-tabs`
   - `el-pagination`
   - `el-dropdown`
8. 图标优先使用 `@element-plus/icons-vue`。
9. 样式层面以少量定制为主，避免大面积覆盖官方组件默认行为。
10. 如果 `Element Plus` 无法覆盖目标效果，再补充轻量自定义结构，并说明原因。

## 实施原则

- 先判断页面需求能否由现有组件直接拼装完成。
- 优先通过组件属性、插槽、组合方式实现需求。
- 非必要不要直接手写按钮、输入框、下拉框、弹层等基础控件。
- 非必要不要引入第二套 UI 组件库。
- 保持 `Element Plus` 的尺寸、间距、状态语义一致。

## 回答与实现偏好

- 给出实现方案时，优先推荐 `Element Plus` 组件组合方案。
- 写代码时，默认直接落地 `Element Plus` 版本，而不是先写纯 `div` 占位版本。
- 如果用户没有特别指定视觉库，不主动改用其他 UI 库。

## 例外情况

以下情况允许不以 `Element Plus` 为主：

- 用户明确要求不用 `Element Plus`
- 设计稿要求高度定制，官方组件不适配
- 某个复杂交互用官方组件实现成本明显更高
- 现有页面已经基于其他特定组件体系实现，强行切换会造成风格割裂

## 示例

### 示例 1

用户说：“做一个搜索加表格的管理页面”

优先方案：

- 用 `el-form` 做搜索区
- 用 `el-table` 做列表
- 用 `el-pagination` 做分页
- 用 `el-button` 做操作按钮

### 示例 2

用户说：“做一个详情弹窗”

优先方案：

- 用 `el-dialog` 作为容器
- 用 `el-descriptions` 或 `el-form` 展示内容
- 用 `el-button` 做底部操作区

### 示例 3

用户说：“做一个后台首页布局”

优先方案：

- 用 `el-container`、`el-aside`、`el-header`、`el-main`
- 菜单用 `el-menu`
- 卡片区用 `el-card`

