# Task Plan: 前端按需加载节点功能

## Goal
实现前端图数据的按需加载机制，解决初始加载全部节点导致浏览器卡死的问题，提供用户可控的关联层级选择功能。

## Current Phase
Phase 7 (Testing)

## Phases

### Phase 1: 需求分析与架构设计
- [x] 分析当前前端代码结构和数据流
- [x] 明确按需加载的具体需求
- [x] 设计新的数据流和组件交互方式
- **Status:** complete

### Phase 2: 状态管理改造 (useAppState.tsx)
- [x] 添加 `displayedNodeIds: Set<string>` - 已显示在图形中的节点 ID
- [x] 添加 `expansionDepth: 1 | 2 | 3` - 展开关联层级，默认为 1
- [x] 添加 `expandNode(nodeId: string)` 方法 - 展开节点及其 N 跳关联
- [x] 添加 `setExpansionDepth(depth: 1|2|3)` 方法
- [x] 修改初始加载：graph 数据保留但 displayedNodeIds 为空
- **Status:** complete

### Phase 3: 图形适配器改造 (graph-adapter.ts)
- [x] 添加 `addNodesToGraphology()` 方法 - 增量添加节点
- [x] 添加 `updateNodeDimmedState()` 方法 - 设置节点变暗状态
- [x] 添加 `clearDimmedState()` 方法 - 清除变暗状态
- **Status:** complete

### Phase 4: Sigma 渲染改造 (useSigma.ts)
- [x] 添加 dimmed 属性处理到 nodeReducer
- [x] 支持增量更新 Sigma 图（而非每次全量重建）
- **Status:** complete

### Phase 5: 左侧树形组件改造 (FileTreePanel.tsx)
- [x] 修改点击文件时的行为：调用 `expandNode()` 而非仅选中
- [x] 添加关联层级选择器 UI（1/2/3 层）
- [x] 显示当前已加载节点数量
- [x] 添加清除图形按钮
- **Status:** complete

### Phase 6: 右侧图形组件改造 (GraphCanvas.tsx)
- [x] 修改初始化逻辑：不立即渲染全量图
- [x] 监听 `displayedNodeIds` 变化，增量更新图形
- [x] 实现选中节点高亮、其他已显示节点变暗效果
- [x] 添加空图提示
- **Status:** complete

### Phase 7: 测试与验证
- [x] TypeScript 编译检查通过
- [x] 验证初始加载不卡死（空图）
- [x] 验证点击文件展开功能正常
- [x] 验证层级选择功能正常
- [x] 验证高亮/变暗效果正常
- [x] 验证多次展开后性能正常
- **Status:** complete

## Key Questions
1. ~~是否需要后端新增 API？~~ **否** - 前端已有全量数据，只需控制显示
2. 关联层级 (1/2/3) 指的是图遍历的跳数（使用现有 `getNodesWithinHops` 函数）
3. 已加载节点区分方式：
   - 选中节点及其关联：正常颜色 + 高亮边框
   - 其他已显示节点：降低透明度（dimmed）

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| 前端存储全量数据，按需显示 | 避免频繁请求后端，提升用户体验 |
| 使用现有 `getNodesWithinHops` 函数 | 代码复用，逻辑一致 |
| 增量更新 Sigma 图而非全量重建 | 提升性能，避免闪烁 |
| `displayedNodeIds` 存储已显示节点 | 便于追踪和增量更新 |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
| - | - | - |

## Notes
- 核心改动文件：
  - `gitnexus-web/src/hooks/useAppState.tsx` - 状态管理
  - `gitnexus-web/src/components/FileTreePanel.tsx` - 左侧树
  - `gitnexus-web/src/components/GraphCanvas.tsx` - 右侧图形
  - `gitnexus-web/src/lib/graph-adapter.ts` - 图转换
  - `gitnexus-web/src/hooks/useSigma.ts` - Sigma 渲染

## 技术方案详细设计

### 数据流设计
```
初始加载: graph(全量) + displayedNodeIds(空) → Sigma(空图)
点击文件: nodeId → getNodesWithinHops(nodeId, expansionDepth)
       → 更新 displayedNodeIds → 增量更新 Sigma → 高亮选中新节点
```

### 状态定义
```typescript
// useAppState.tsx 新增
displayedNodeIds: Set<string>;
expansionDepth: 1 | 2 | 3;
expandNode: (nodeId: string) => void;
setExpansionDepth: (depth: 1|2|3) => void;
```

### 视觉效果
- 选中节点：原色 + 高亮边框 (zIndex: 2)
- 关联节点：原色 + 普通状态
- 其他已显示节点：降低透明度至 0.3
- 未显示节点：hidden=true
