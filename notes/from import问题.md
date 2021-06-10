```python
# 含__init__是一个模块
import os
import sys

os.sys.path.append(r"c:\users")
cur_path=os.path.abspath(os.path.dirname(__file__))
os.sys.path.insert(0, cur_path+"/..")
print(sys.path)
```

