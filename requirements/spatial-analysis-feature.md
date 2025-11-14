# 空间关系分析功能需求文档

## 概述

实现一个空间感知功能，类似于 Figma 中按住 `Option/Alt` 键时显示节点间距离的交互效果。给定一个节点，分析该节点及其所有子节点与相邻元素的空间关系信息。

## 功能描述

### 输入参数
- `nodeId` (string): 目标节点的ID
- `includeChildren` (boolean, 可选): 是否包含子节点分析，默认为 true
- `maxDepth` (number, 可选): 最大递归深度，默认为无限制

### 输出格式

返回一个数组，包含目标节点及其所有子节点的空间关系信息：

```typescript
interface SpatialAnalysisResult {
  nodeId: string;
  name: string;
  type: string;
  depth: number; // 在层级树中的深度，根节点为0
  boundingBox: {
    x: number;
    y: number; 
    width: number;
    height: number;
  };
  leftElement?: {
    name: string;
    nodeId: string;
    relation: "parent" | "sibling" | "child" | "other";
    type: string;
  };
  rightElement?: {
    name: string;
    nodeId: string;
    relation: "parent" | "sibling" | "child" | "other";
    type: string;
  };
  upElement?: {
    name: string;
    nodeId: string;
    relation: "parent" | "sibling" | "child" | "other";
    type: string;
  };
  downElement?: {
    name: string;
    nodeId: string;
    relation: "parent" | "sibling" | "child" | "other";
    type: string;
  };
  leftDistance: number;   // 到左侧元素的距离，-1表示无相邻元素
  rightDistance: number;  // 到右侧元素的距离，-1表示无相邻元素
  upDistance: number;     // 到上方元素的距离，-1表示无相邻元素
  downDistance: number;   // 到下方元素的距离，-1表示无相邻元素
}
```

### 示例输出

```json
[
  {
    "nodeId": "parent123",
    "name": "主容器",
    "type": "FRAME",
    "depth": 0,
    "boundingBox": {
      "x": 100,
      "y": 100,
      "width": 200,
      "height": 150
    },
    "leftElement": {
      "name": "左侧组件",
      "nodeId": "left456",
      "relation": "sibling",
      "type": "FRAME"
    },
    "rightElement": {
      "name": "右侧组件", 
      "nodeId": "right789",
      "relation": "sibling",
      "type": "FRAME"
    },
    "upElement": {
      "name": "页面容器",
      "nodeId": "page001",
      "relation": "parent",
      "type": "PAGE"
    },
    "downElement": null,
    "leftDistance": 20,
    "rightDistance": 15,
    "upDistance": 30,
    "downDistance": -1
  },
  {
    "nodeId": "child123",
    "name": "标题文本",
    "type": "TEXT",
    "depth": 1,
    "boundingBox": {
      "x": 110,
      "y": 110,
      "width": 180,
      "height": 24
    },
    "leftElement": {
      "name": "主容器",
      "nodeId": "parent123", 
      "relation": "parent",
      "type": "FRAME"
    },
    "rightElement": {
      "name": "主容器",
      "nodeId": "parent123",
      "relation": "parent", 
      "type": "FRAME"
    },
    "upElement": {
      "name": "主容器",
      "nodeId": "parent123",
      "relation": "parent",
      "type": "FRAME"
    },
    "downElement": {
      "name": "描述文本",
      "nodeId": "child456",
      "relation": "sibling",
      "type": "TEXT"
    },
    "leftDistance": 10,
    "rightDistance": 10,
    "upDistance": 10,
    "downDistance": 16
  }
]
```

## 算法要求

### 1. 空间计算逻辑

#### 相邻元素判定
- **左侧元素**: 右边界 <= 当前节点左边界，且纵向有重叠的最近元素
- **右侧元素**: 左边界 >= 当前节点右边界，且纵向有重叠的最近元素  
- **上方元素**: 下边界 <= 当前节点上边界，且横向有重叠的最近元素
- **下方元素**: 上边界 >= 当前节点下边界，且横向有重叠的最近元素

#### 距离计算
- 计算两个节点边界之间的最短距离
- 如果节点重叠，距离为0
- 如果某个方向无相邻元素，距离为-1

### 2. 关系类型判定

- **parent**: 相邻元素是当前节点的父节点
- **sibling**: 相邻元素与当前节点有相同父节点
- **child**: 相邻元素是当前节点的子节点
- **other**: 其他关系（如跨层级的元素）

### 3. 性能优化

- 使用空间索引（如四叉树）优化最近邻查找
- 支持大量子节点的高效处理
- 避免重复计算相同节点的边界信息

## 技术实现

### MCP工具接口

```typescript
server.tool(
  "analyze_spatial_relationships",
  "分析节点及其子节点的空间关系",
  {
    nodeId: {
      type: "string",
      description: "目标节点ID"
    },
    includeChildren: {
      type: "boolean", 
      description: "是否包含子节点分析",
      default: true
    },
    maxDepth: {
      type: "number",
      description: "最大递归深度",
      default: -1
    }
  },
  async (args) => {
    // 实现逻辑
  }
);
```

### Figma插件端实现

在 `src/cursor_mcp_plugin/code.js` 中添加对应的处理函数：

```javascript
case "analyze_spatial_relationships":
  return await analyzeSpatialRelationships(params);
```

## 使用场景

### 1. 设计规范检查
- 验证元素间距是否符合设计系统规范
- 检查对齐和分布是否合理

### 2. 布局分析
- 分析复杂组件的内部空间结构
- 识别布局问题和优化机会

### 3. 自动化调整
- 基于空间关系自动调整元素位置
- 批量优化间距和对齐

### 4. 设计审查
- 快速了解设计的空间上下文
- 生成空间关系报告

## 测试用例

### 基础测试
1. 单个节点无子节点的空间分析
2. 包含多层嵌套子节点的复杂组件分析
3. 边界条件：重叠节点、超出画布的节点

### 性能测试  
1. 大量子节点（1000+）的处理性能
2. 深层嵌套（10+层级）的处理效率
3. 复杂布局的计算时间

### 准确性测试
1. 距离计算的精度验证
2. 关系类型判定的正确性
3. 边界重叠情况的处理

## 交付物

1. MCP工具实现
2. Figma插件端处理函数
3. TypeScript类型定义
4. 单元测试用例
5. 使用文档和示例