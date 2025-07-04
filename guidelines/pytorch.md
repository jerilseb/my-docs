## PyTorch

- Use datasets from huggingface whenever possible
- Use tqdm for showing progress
- Use `dtype=torch.bfloat16` for mixed precison training

For example,

```python
from torch.cuda.amp import autocast

device = torch.device("cuda")
model = SimpleModel().to(device)
optimizer = torch.optim.Adam(model.parameters())

for epoch in range(10):
    for batch_idx, (data, target) in enumerate(train_loader):
        data, target = data.to(device), target.to(device)
        
        optimizer.zero_grad()
        
        with autocast(device_type='cuda', dtype=torch.bfloat16):
            output = model(data)
            loss = nn.functional.cross_entropy(output, target)
        
        loss.backward()
        optimizer.step()
```