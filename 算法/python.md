求欧式距离

```python
import torch
x = torch.randn(3,5)
y = torch.randn(3,5)
dist = torch.stack([torch.norm(x - i,dim=1) for i in y])
```

