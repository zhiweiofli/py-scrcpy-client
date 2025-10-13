# PyAV 10.0.0 降级可行性评估报告 / PyAV 10.0.0 Downgrade Feasibility Evaluation

## 执行概要 / Executive Summary

**结论 / Conclusion**: 不建议将 PyAV 从当前版本 12.0.0 降级到 10.0.0  
**Recommendation**: Downgrading PyAV from current version 12.0.0 to 10.0.0 is **NOT RECOMMENDED**

---

## 1. 当前状态 / Current State

### 项目配置 / Project Configuration
- **当前 PyAV 版本 / Current PyAV Version**: 12.0.0
- **Python 版本要求 / Python Version Requirements**: >=3.9, <3.13
- **支持的 Python 版本 / Supported Python Versions**: 3.9, 3.10, 3.11, 3.12

### PyAV 使用情况 / PyAV Usage
项目中仅在以下位置使用 PyAV：
The project uses PyAV only in the following locations:

```python
# scrcpy/core.py
from av.codec import CodecContext
from av.error import InvalidDataError

# Usage in __stream_loop method:
codec = CodecContext.create("h264", "r")
packets = codec.parse(raw_h264)
frames = codec.decode(packet)
frame = frame.to_ndarray(format="bgr24")
```

---

## 2. 兼容性问题 / Compatibility Issues

### 2.1 Python 版本兼容性 / Python Version Compatibility

**关键发现 / Critical Finding**:
- **PyAV 10.0.0 仅支持 / PyAV 10.0.0 only supports**: Python 3.7, 3.8, 3.9, 3.10
- **PyAV 10.0.0 不支持 / PyAV 10.0.0 does NOT support**: Python 3.11, 3.12

**影响分析 / Impact Analysis**:
如果降级到 PyAV 10.0.0，必须同时将 Python 版本要求从 `>=3.9,<3.13` 修改为 `>=3.9,<3.11`，这意味着：

If downgrading to PyAV 10.0.0, the Python version requirement must be changed from `>=3.9,<3.13` to `>=3.9,<3.11`, which means:

1. ❌ **失去 Python 3.11 和 3.12 支持 / Loss of Python 3.11 and 3.12 support**
   - Python 3.11 和 3.12 是目前最新且性能改进的版本
   - Python 3.11 and 3.12 are the latest versions with significant performance improvements
   
2. ❌ **破坏向后兼容性 / Breaking backward compatibility**
   - 已使用 Python 3.11 或 3.12 的用户无法升级
   - Users already using Python 3.11 or 3.12 cannot upgrade
   
3. ❌ **违背 Python 社区趋势 / Against Python community trends**
   - Python 3.9 将于 2025-10 结束支持
   - Python 3.9 will reach end-of-life in October 2025
   - Python 3.10 将于 2026-10 结束支持
   - Python 3.10 will reach end-of-life in October 2026

### 2.2 版本时间线 / Version Timeline

```
PyAV Version Support Matrix:
┌──────────┬──────┬──────┬──────┬──────┬──────┬──────┐
│ PyAV Ver │ Py37 │ Py38 │ Py39 │ Py310│ Py311│ Py312│
├──────────┼──────┼──────┼──────┼──────┼──────┼──────┤
│  10.0.0  │  ✓   │  ✓   │  ✓   │  ✓   │  ✗   │  ✗   │
│  11.0.0  │  ?   │  ✓   │  ✓   │  ✓   │  ✓   │  ✗   │
│  12.0.0  │  ✗   │  ✓   │  ✓   │  ✓   │  ✓   │  ✓   │
└──────────┴──────┴──────┴──────┴──────┴──────┴──────┘

Current Project: Py39, Py310, Py311, Py312 ✓
With PyAV 10:   Py39, Py310 only ✗
```

---

## 3. API 变化分析 / API Changes Analysis

### 3.1 使用的 API / APIs Used

本项目使用的 PyAV API 非常有限：
The project uses a very limited set of PyAV APIs:

1. `CodecContext.create(codec_name, mode)` - 创建编解码器上下文
2. `codec.parse(data)` - 解析视频流数据
3. `codec.decode(packet)` - 解码数据包
4. `frame.to_ndarray(format)` - 转换为 numpy 数组
5. `InvalidDataError` - 异常处理

### 3.2 API 稳定性 / API Stability

根据 PyAV 官方文档和发布说明：
According to PyAV official documentation and release notes:

- ✅ PyAV 10.0.0 到 12.0.0 之间，上述核心 API 保持稳定
- ✅ The core APIs listed above remain stable between PyAV 10.0.0 and 12.0.0
- ⚠️ 主要变化集中在：编解码器选项、容器处理、高级过滤器
- ⚠️ Major changes focus on: codec options, container handling, advanced filters
- ✅ 基本的解码流程（本项目使用的）无重大变化
- ✅ Basic decoding workflow (used by this project) has no major changes

**结论 / Conclusion**: 从 API 角度看，降级在技术上可行但不是必需的。
**Conclusion**: From an API perspective, downgrading is technically feasible but not necessary.

---

## 4. 测试结果 / Test Results

### 4.1 当前版本测试 / Current Version Tests
使用 PyAV 12.0.0 运行测试：
Tests with PyAV 12.0.0:

```
13 passed, 322 warnings in 8.43s
✅ All tests passed successfully
```

### 4.2 降级测试 / Downgrade Tests
由于 Python 3.12 环境不支持 PyAV 10.0.0，无法直接测试。
Cannot directly test due to Python 3.12 environment not supporting PyAV 10.0.0.

需要在 Python 3.9 或 3.10 环境中测试。
Would require testing in Python 3.9 or 3.10 environment.

---

## 5. 成本收益分析 / Cost-Benefit Analysis

### 5.1 降级成本 / Downgrade Costs

1. **开发成本 / Development Cost**: 低 / Low
   - API 改动较小
   - API changes are minimal
   
2. **测试成本 / Testing Cost**: 中等 / Medium
   - 需要在多个 Python 版本测试
   - Need to test across multiple Python versions
   
3. **维护成本 / Maintenance Cost**: 高 / High
   - ❌ 失去 Python 3.11/3.12 支持
   - ❌ Loss of Python 3.11/3.12 support
   - ❌ 需要维护旧版本兼容性
   - ❌ Need to maintain old version compatibility
   - ❌ 社区生态系统不兼容风险增加
   - ❌ Increased risk of ecosystem incompatibility
   
4. **用户影响 / User Impact**: 高 / High
   - ❌ 已有用户无法升级
   - ❌ Existing users cannot upgrade
   - ❌ 新用户被迫使用旧版 Python
   - ❌ New users forced to use older Python

### 5.2 降级收益 / Downgrade Benefits

**无明显收益 / No Clear Benefits**:
- PyAV 10.0.0 没有提供任何 12.0.0 不具备的关键功能
- PyAV 10.0.0 does not provide any critical features that 12.0.0 lacks
- 性能差异可忽略不计
- Performance differences are negligible
- 没有重大 bug 修复需求
- No critical bug fixes required

---

## 6. 风险评估 / Risk Assessment

### 高风险因素 / High Risk Factors

1. **生态系统不兼容 / Ecosystem Incompatibility** (严重 / CRITICAL)
   - numpy 2.x 可能与 PyAV 10.0.0 存在兼容性问题
   - numpy 2.x may have compatibility issues with PyAV 10.0.0
   
2. **依赖冲突 / Dependency Conflicts** (高 / HIGH)
   - 其他依赖可能需要新版 Python
   - Other dependencies may require newer Python versions
   
3. **安全风险 / Security Risk** (中 / MEDIUM)
   - 旧版本可能包含已修复的安全漏洞
   - Older versions may contain security vulnerabilities that have been fixed

---

## 7. 替代方案 / Alternative Solutions

如果确实需要解决与 PyAV 12.0.0 相关的问题：
If there are issues with PyAV 12.0.0 that need to be addressed:

### 方案 A：保持当前版本 / Option A: Keep Current Version
- ✅ **推荐 / RECOMMENDED**
- 继续使用 PyAV ^12
- Continue using PyAV ^12
- 确保测试覆盖所有支持的 Python 版本
- Ensure test coverage for all supported Python versions

### 方案 B：升级到最新版本 / Option B: Upgrade to Latest
- ✅ 考虑升级到 PyAV 最新稳定版
- ✅ Consider upgrading to the latest stable PyAV version
- 获得最新功能和安全修复
- Get latest features and security fixes

### 方案 C：有条件降级 / Option C: Conditional Downgrade
- ⚠️ 仅在有明确技术原因时
- ⚠️ Only if there's a clear technical reason
- 步骤：
  1. 修改 `pyproject.toml`: `python = ">=3.9,<3.11"`, `av = "^10"`
  2. 更新 CI/CD 配置移除 Python 3.11/3.12
  3. 在所有环境测试
  4. 更新文档说明版本限制

---

## 8. 最终建议 / Final Recommendation

### 不建议降级的理由 / Reasons NOT to Downgrade

1. ❌ **无技术收益 / No technical benefits**
   - PyAV 10.0.0 没有任何优于 12.0.0 的功能
   - PyAV 10.0.0 has no advantages over 12.0.0

2. ❌ **严重的兼容性损失 / Severe compatibility loss**
   - 失去 Python 3.11 和 3.12 支持
   - Loss of Python 3.11 and 3.12 support

3. ❌ **违背技术发展趋势 / Against technology trends**
   - Python 和 PyAV 都在向前发展
   - Both Python and PyAV are moving forward

4. ❌ **高维护成本 / High maintenance cost**
   - 需要持续维护旧版本兼容性
   - Need to continuously maintain old version compatibility

5. ❌ **用户体验下降 / Degraded user experience**
   - 强制用户使用旧版 Python
   - Forces users to use older Python versions

### 推荐行动 / Recommended Action

**保持当前配置 / Keep Current Configuration**:
```toml
[tool.poetry.dependencies]
python = ">=3.9,<3.13"
av = "^12"
```

如有特殊需求，请提供具体技术原因以便重新评估。
If there are specific requirements, please provide technical reasons for re-evaluation.

---

## 9. 联系信息 / Contact Information

如对本评估有任何疑问，请提出 issue 或联系维护者。
For any questions about this evaluation, please open an issue or contact the maintainers.

**评估日期 / Evaluation Date**: 2025-10-13  
**PyAV 当前版本 / Current PyAV Version**: 12.0.0  
**项目版本 / Project Version**: 0.4.7
