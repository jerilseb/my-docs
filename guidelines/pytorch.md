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
    progress_bar = tqdm(train_dataloader, desc=f"Epoch [{epoch}/{num_epochs}]", leave=False)
    for batch_idx, (inputs, targets) in enumerate(progress_bar, 1):
        inputs, targets = inputs.to(device), targets.to(device)
        
        optimizer.zero_grad()
        
        with autocast(device_type='cuda', dtype=torch.bfloat16):
            output = model(inputs)
            loss = nn.functional.cross_entropy(output, targets)
        
        loss.backward()
        optimizer.step()
```

DO NOT use `GradScaler` since we are using `dtype=torch.bfloat16`