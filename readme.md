# Bug Fix Log - Cursor Talk to Figma MCP

This document records all bug fixes and improvements made to the Cursor Talk to Figma MCP project.

---

## Bug Fix #001: String Variable Setting Issue

**Date**: December 19, 2024  
**Time**: 14:30 UTC  
**Severity**: Critical  
**Status**: ✅ Fixed

### Issue Description

Users encountered a critical bug when attempting to set string-type variables in Figma through the MCP interface. Specifically, when trying to modify `typography/font_families/font_mono` variable to the value "san", the operation would fail with the error:

```
Error setting variable value: Reference variable not found: san
```

### Root Cause Analysis

The bug was caused by two interconnected issues in the MCP server implementation:

#### 1. Incorrect Parameter Schema Definition

**File**: `src/talk_to_figma_mcp/server.ts` (lines 2858-2863)

The `value` parameter schema was incorrectly defined to only accept color objects:

```typescript
// INCORRECT - Only supported color objects
value: z.object({
  r: z.number().optional(),
  g: z.number().optional(),
  b: z.number().optional(),
  a: z.number().optional()
}).optional().describe("The value for the variable"),
```

This schema validation rejected string values like "san", causing the parameter to be incorrectly passed to the `variableReferenceId` field instead of the `value` field.

#### 2. Incomplete Parameter Processing Logic

**File**: `src/talk_to_figma_mcp/server.ts` (lines 2870-2877)

The parameter processing logic only handled COLOR type variables:

```typescript
// INCOMPLETE - Only processed COLOR type
const formattedValue = valueType === "COLOR" && value
  ? {
      r: value.r || 0,
      g: value.g || 0,
      b: value.b || 0,
      a: value.a || 1
    }
  : value;
```

This caused STRING, FLOAT, and BOOLEAN type variables to be processed incorrectly.

### Solution Implementation

#### Fix 1: Updated Parameter Schema

**File**: `src/talk_to_figma_mcp/server.ts` (lines 2858-2868)

Replaced the restrictive color-only schema with a union type that supports all Figma variable types:

```typescript
// FIXED - Supports all variable types
value: z.union([
  z.string().describe("String value for STRING type variables"),
  z.number().describe("Number value for FLOAT type variables"),
  z.boolean().describe("Boolean value for BOOLEAN type variables"),
  z.object({
    r: z.number().optional(),
    g: z.number().optional(),
    b: z.number().optional(),
    a: z.number().optional()
  }).describe("Color object for COLOR type variables")
]).optional().describe("The value for the variable"),
```

#### Fix 2: Enhanced Parameter Processing

**File**: `src/talk_to_figma_mcp/server.ts` (lines 2875-2885)

Updated the processing logic to handle all variable types correctly:

```typescript
// FIXED - Proper handling for all types
let formattedValue = value;
if (valueType === "COLOR" && value && typeof value === "object") {
  formattedValue = {
    r: value.r || 0,
    g: value.g || 0,
    b: value.b || 0,
    a: value.a || 1
  };
}
// For STRING, FLOAT, BOOLEAN types, use raw values directly
// Figma API handles type conversion automatically
```

### Technical Details

#### Figma API Compatibility

The fix aligns with the official Figma Plugin API documentation:

- **STRING variables**: Accept raw string values (e.g., `"Roboto"`)
- **FLOAT variables**: Accept raw number values (e.g., `100`)
- **BOOLEAN variables**: Accept raw boolean values (e.g., `true`)
- **COLOR variables**: Accept RGBA objects (e.g., `{r: 0.5, g: 0.3, b: 0.8, a: 1}`)

#### Error Resolution

The fix resolves the following issues:

1. ✅ **Parameter validation errors**: String values now pass schema validation
2. ✅ **Type confusion**: Strings are no longer mistaken for variable reference IDs
3. ✅ **API compatibility**: Implementation matches Figma's official API usage patterns
4. ✅ **Functionality completeness**: All variable types (STRING, FLOAT, BOOLEAN, COLOR) are now supported

### Usage Examples

After the fix, users can now correctly set variables of all types:

```typescript
// Set string variable (typography/font_families/font_mono)
{
  variableId: "VariableID:380:4",
  valueType: "STRING",
  value: "san"
}

// Set number variable
{
  variableId: "VariableID:380:5",
  valueType: "FLOAT",
  value: 100
}

// Set boolean variable
{
  variableId: "VariableID:380:6",
  valueType: "BOOLEAN",
  value: true
}

// Set color variable
{
  variableId: "VariableID:380:7",
  valueType: "COLOR",
  value: {r: 0.5, g: 0.3, b: 0.8, a: 1}
}
```

### Files Modified

- `src/talk_to_figma_mcp/server.ts` - Updated parameter schema and processing logic for the `set_variable_value` tool

### Testing

The fix has been validated with:
- ✅ No syntax errors introduced
- ✅ Schema validation now accepts all supported data types
- ✅ Parameter processing correctly handles all variable types
- ✅ Compatibility with Figma's official Plugin API maintained

### Impact

This fix ensures that all Figma variable types can be properly set through the MCP interface, resolving the critical string variable setting issue that was preventing users from modifying typography and other string-based design tokens.

---

## Bug Fix #002: 颜色变量设置序列化问题及工具重构

**Date**: September 12, 2025  
**Time**: 16:57 CST  
**Severity**: Critical  
**Status**: ✅ Fixed

### Issue Description

用户在设置颜色变量时遇到严重问题，颜色对象被错误地序列化为字符串`"[object Object]"`，导致变量设置失败。具体表现为：

```
{
  "variableId": "VariableID:391:3",
  "valueType": "COLOR",
  "value": "[object Object]"
}
Error setting variable value: Invalid color value
```

对于`colors.gray.50 (#f9fafb)`这样的颜色，RGB值(0.976, 0.980, 0.984)无法正确传递到Figma。

### Root Cause Analysis

问题的根本原因是参数组装和序列化过程中的复杂性：

#### 1. 复杂的Union类型参数
原始的`set_variable_value`工具使用了复杂的union类型参数，在MCP传输过程中容易导致对象序列化错误。

#### 2. 类型混淆
单一工具处理多种数据类型，容易在参数传递过程中发生类型转换错误。

#### 3. 缺乏类型安全
没有针对特定变量类型的验证机制，错误只能在运行时发现。

### Solution Implementation

基于[Figma插件文档](https://www.figma.com/plugin-docs/working-with-variables/)的最佳实践，我们将`set_variable_value`重构为4个专门的MCP工具：

#### 重构方案：专门化工具设计

**新增工具**:
1. `set_color_variable` - 专门处理颜色变量
2. `set_float_variable` - 专门处理数值变量  
3. `set_string_variable` - 专门处理字符串变量
4. `set_boolean_variable` - 专门处理布尔变量

#### Fix 1: 服务器端实现

**文件**: `src/talk_to_figma_mcp/server.ts`

添加了4个新的专门工具：

```typescript
// 颜色变量工具 - 独立参数避免序列化问题
server.tool("set_color_variable", {
  variableId: z.string(),
  r: z.number().min(0).max(1),
  g: z.number().min(0).max(1), 
  b: z.number().min(0).max(1),
  a: z.number().min(0).max(1).optional().default(1),
  // ...
});

// 其他3个专门工具类似实现...
```

#### Fix 2: 插件端实现

**文件**: `src/cursor_mcp_plugin/code.js`

实现了对应的4个专门处理函数：

```javascript
// 颜色变量专门处理函数
async function setColorVariable(params) {
  const variable = await figma.variables.getVariableByIdAsync(variableId);
  
  // 严格的类型验证
  if (variable.resolvedType !== "COLOR") {
    throw new Error(`Variable ${variableId} is not a COLOR variable`);
  }
  
  // 直接处理颜色值，无需序列化
  const colorValue = { r, g, b, a };
  variable.setValueForMode(mode, colorValue);
}
```

#### Fix 3: 类型系统更新

更新了`FigmaCommand`和`CommandParams`类型定义以支持新工具。

### Technical Improvements

#### 1. 消除序列化问题
- 颜色值现在作为独立的r, g, b, a参数传递
- 避免了对象序列化为`[object Object]`的问题

#### 2. 类型安全增强  
- 每个工具只处理特定类型的变量
- 运行时类型验证确保变量类型匹配
- 清晰的错误信息便于调试

#### 3. API简化
- 不再需要`valueType`参数
- 每个工具的参数结构明确
- 减少了参数组装错误的可能性

### 使用示例

#### 修复前 (容易出错)
```javascript
{
  "variableId": "VariableID:391:3",
  "valueType": "COLOR", 
  "value": "[object Object]"  // ❌ 序列化错误
}
```

#### 修复后 (类型安全)
```javascript
{
  "variableId": "VariableID:391:3",
  "r": 0.976,    // #f9 / 255
  "g": 0.980,    // #fa / 255  
  "b": 0.984,    // #fb / 255
  "a": 1
}
```

### Files Modified

- `src/talk_to_figma_mcp/server.ts` - 添加4个专门的MCP工具
- `src/cursor_mcp_plugin/code.js` - 实现4个专门的处理函数

### Backward Compatibility

- 旧的`set_variable_value`工具保留但标记为DEPRECATED
- 提供迁移指南帮助用户切换到新工具
- `create_variable`工具的提示信息更新为推荐新工具

### Testing & Validation

- ✅ 颜色变量设置成功，无序列化错误
- ✅ 所有4种变量类型都能正确处理
- ✅ 类型验证机制工作正常
- ✅ 向后兼容性保持良好

### Impact

这次重构从根本上解决了变量设置的可靠性问题：
- 🎯 **彻底解决颜色变量序列化问题**
- 🛡️ **提供类型安全的API接口**  
- 🚀 **提升开发体验和错误诊断能力**
- 📚 **符合Figma官方最佳实践**

现在用户可以安全、可靠地设置任何类型的Figma变量，特别是解决了困扰用户的颜色变量问题。

---

*This log will be updated with future bug fixes and improvements.*