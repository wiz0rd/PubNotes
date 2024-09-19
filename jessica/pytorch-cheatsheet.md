# PyTorch Cheat Sheet

## Importing PyTorch
```python
import torch
import torch.nn as nn
import torch.optim as optim
```

## Creating Tensors
```python
x = torch.tensor([1, 2, 3])
y = torch.zeros(3, 3)
z = torch.rand(3, 3)
```

## Basic Operations
```python
a = torch.tensor([1, 2, 3])
b = torch.tensor([4, 5, 6])
c = a + b  # Element-wise addition
d = torch.matmul(a, b)  # Matrix multiplication
```

## Neural Network Modules
```python
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(784, 64)
        self.fc2 = nn.Linear(64, 10)

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        x = self.fc2(x)
        return x

model = Net()
```

## Loss and Optimizer
```python
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)
```

## Training Loop
```python
for epoch in range(num_epochs):
    for data, target in dataloader:
        optimizer.zero_grad()
        output = model(data)
        loss = criterion(output, target)
        loss.backward()
        optimizer.step()
```

## Saving and Loading Models
```python
# Save
torch.save(model.state_dict(), 'model.pth')

# Load
model = Net()
model.load_state_dict(torch.load('model.pth'))
```

## GPU Usage
```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model.to(device)
data = data.to(device)
```

## Data Loading
```python
from torch.utils.data import DataLoader, TensorDataset

dataset = TensorDataset(x_train, y_train)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)
```

For more detailed information, refer to the [official PyTorch documentation](https://pytorch.org/docs/stable/index.html).
