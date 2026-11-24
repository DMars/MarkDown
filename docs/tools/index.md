# 工具链

沉淀自动化脚本、版本管理与效率工具的实践记录。

## Python 自动化

Python 适合做管线自动化与批处理。常用场景：

- 批量重命名 / 整理文件
- 解析与生成配置文件
- DCC 软件的脚本扩展（如 Maya、3ds Max 的 Python API）

```python
from pathlib import Path

def collect_files(root: Path, suffix: str):
    """递归收集指定后缀的文件。"""
    return sorted(root.rglob(f"*{suffix}"))

for f in collect_files(Path("."), ".py"):
    print(f.name)
```

## Git 工作流

- 主分支 `main` 保持稳定，功能在分支开发后合并
- 提交信息简洁明确，说明「改了什么」和「为什么」
- 敏感信息（Token、密码）绝不入库

## 待补充

- [ ] 添加具体工具的使用笔记
- [ ] 整理常用命令速查表
- [ ] 记录环境配置步骤
