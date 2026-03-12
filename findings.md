# Findings & Decisions

## Requirements
- 初始进入时不加载任何节点和边到右侧图形
- 用户点击左侧树中的文件时，展开该节点及其相关节点和边，高亮显示
- 其余的已加载节点保持变暗状态
- 前端设计一个选项区域，可选择关联层级（1/2/3层，默认为1层)

## Research Findings

### 当前代码结构

**前端入口:**
- `gitnexus-web/src/main.tsx` - React 入口
- `gitnexus-web/src/App.tsx` - 主应用组件

**状态管理:**
- `gitnexus-web/src/hooks/useAppState.tsx` - 全局状态核心
  - `graph`: KnowledgeGraph 类型，存储所有节点和关系
  - `selectedNode`, `highlightedNodeIds`: 选择/高亮状态
  - `viewMode`: 'onboarding' | 'loading' | 'exploring'

**当前加载流程（一次性加载全部）:**
1. `server-connection.ts` 的 `fetchGraph()` 通过 `/api/graph` 获取所有数据
2. `App.tsx` 的 `handleServerConnect()` 创建 KnowledgeGraph 并添加所有节点
3. `graph-adapter.ts` 的 `knowledgeGraphToGraphology()` 转换全部节点到 Graphology
4. Sigma.js 渲染全部节点

**关键组件:**
- `FileTreePanel.tsx` - 左侧文件树，从 `graph.nodes` 过滤 Folder/File 节点
- `GraphCanvas.tsx` - 右侧图形展示
- `useSigma.ts` - Sigma.js + Graphology WebGL 渲染

**类型定义 (`types.ts`):**
```typescript
type NodeLabel = 'Project' | 'Package' | 'Module' | 'Folder' | 'File' | 'Class' | 'Function' | 'Method' | 'Variable' | 'Interface' | 'Enum' | 'Decorator' | 'Import' | 'Type' | 'CodeElement' | 'Community' | 'Process';

interface KnowledgeGraph {
  nodes: GraphNode[],
  relationships: GraphRelationship[],
  addNode: (node: GraphNode) => void,
  addRelationship: (relationship: GraphRelationship) => void,
}
```

### 现有关键函数（可复用）

**graph-adapter.ts:**
- `getNodesWithinHops(graph, nodeId, maxHops)` - 获取 N 跳范围内的节点 ID 集合
- `filterGraphByDepth()` - 按深度过滤节点显示
- `filterGraphByLabels()` - 按标签过滤节点显示

**useSigma.ts:**
- `setGraph()` - 设置 Sigma 图
- `focusNode()` - 聚焦到指定节点

### SigmaNodeAttributes 属性
```typescript
interface SigmaNodeAttributes {
  x: number;
  y: number;
  size: number;
  color: string;
  label: string;
  nodeType: NodeLabel;
  filePath: string;
  hidden?: boolean;      // 控制节点显示/隐藏
  highlighted?: boolean; // 控制节点高亮
  zIndex?: number;       // 控制渲染层级
}
```

## Technical Decisions
| Decision | Rationale |
|----------|-----------|
| 前端保留全量数据，仅控制显示 | 用户可能随时需要加载新节点，避免重复请求 |
| 使用 `displayedNodeIds` Set 跟踪已显示节点 | 高效判断节点是否应该显示 |
| 点击文件时查找 N 跳关联节点 | 满足用户查看关联代码的需求 |
| 增量更新 Sigma 图而非全量重建 | 提升性能，避免闪烁 |
| 使用 `dimmed` 属性区分变暗状态 | 复用现有 hidden 机制，扩展视觉效果 |

## Issues Encountered
| Issue | Resolution |
|-------|------------|
| - | - |

## Resources
- 核心文件路径:
  - `gitnexus-web/src/hooks/useAppState.tsx`
  - `gitnexus-web/src/components/FileTreePanel.tsx`
  - `gitnexus-web/src/components/GraphCanvas.tsx`
  - `gitnexus-web/src/lib/graph-adapter.ts`
  - `gitnexus-web/src/hooks/useSigma.ts`
  - `gitnexus-web/src/core/graph/graph.ts`
  - `gitnexus-web/src/core/graph/types.ts`

## Visual/Browser Findings
- 暂无

## 实现细节

### Phase 2: useAppState.tsx 改动
```typescript
// 新增状态
const [displayedNodeIds, setDisplayedNodeIds] = useState<Set<string>>(new Set());
const [expansionDepth, setExpansionDepth] = useState<1|2|3>(1);

// 新增方法
const expandNode = useCallback((nodeId: string) => {
  // 1. 从 graph.relationships 获取 N 跳关联节点
  // 2. 更新 displayedNodeIds
  // 3. 设置 selectedNode
}, [graph, expansionDepth]);
```

### Phase 3: graph-adapter.ts 改动
```typescript
// 新增：增量添加节点到 Graphology 图
export const addNodesToGraphology = (
  graphologyGraph: Graph,
  knowledgeGraph: KnowledgeGraph,
  nodeIds: Set<string>,
  existingPositions: Map<string, {x, y}>
): void => { ... }

// 新增：设置节点变暗状态
export const setNodeDimmed = (
  graph: Graph,
  nodeIds: Set<string>,
  dimmed: boolean
): void => { ... }
```

### Phase 5: FileTreePanel.tsx 改动
```typescript
// Filters 区域添加关联层级选择器
<div className="mt-6 pt-4 border-t border-border-subtle">
  <h3>Expansion Depth</h3>
  <div className="flex gap-1.5">
    {[1, 2, 3].map(d => (
      <button onClick={() => setExpansionDepth(d)}>{d} layer</button>
    ))}
  </div>
</div>

// 点击文件时调用 expandNode
const handleNodeClick = (treeNode) => {
  if (treeNode.graphNode) {
    expandNode(treeNode.graphNode.id); // 而非仅 setSelectedNode
  }
};
```

### Phase 6: GraphCanvas.tsx 改动
```typescript
// 监听 displayedNodeIds 变化，增量更新图
useEffect(() => {
  if (!graph || displayedNodeIds.size === 0) {
    // 空图 - 清空或保持空状态
    return;
  }
  // 增量添加新节点
  addNodesToGraphology(sigmaGraph, graph, displayedNodeIds, nodePositions);
  sigma.refresh();
}, [displayedNodeIds]);
```
