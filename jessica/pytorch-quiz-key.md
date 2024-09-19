# PyTorch Interview Quiz - Answer Key

1. PyTorch is an open-source machine learning library. Unlike TensorFlow, PyTorch uses dynamic computational graphs, which allows for more flexible model architectures.

2. You can create a tensor in PyTorch using:
   ```python
   x = torch.tensor([1, 2, 3])
   ```

3. `torch.Tensor` is a class, while `torch.tensor` is a function. `torch.Tensor` creates a tensor with uninitialized data, while `torch.tensor` infers the data type from the input.

4. Matrix multiplication can be performed using:
   ```python
   result = torch.matmul(matrix1, matrix2)
   # or
   result = matrix1 @ matrix2
   ```

5. Autograd is PyTorch's automatic differentiation engine. It computes gradients automatically. You can enable it by setting `requires_grad=True` on tensors.

6. You can define a neural network by creating a class that inherits from `nn.Module`:
   ```python
   class Net(nn.Module):
       def __init__(self):
           super(Net, self).__init__()
           self.fc1 = nn.Linear(784, 64)
           self.fc2 = nn.Linear(64, 10)

       def forward(self, x):
           x = torch.relu(self.fc1(x))
           return self.fc2(x)
   ```

7. `torch.nn.Module` is the base class for all neural network modules in PyTorch. It provides a way to organize layers and encapsulate parameters.

8. You can implement a custom loss function by subclassing `nn.Module`:
   ```python
   class MyLoss(nn.Module):
       def forward(self, output, target):
           return custom_loss_calculation(output, target)
   ```

9. `model.eval()` sets the model to evaluation mode, which affects layers like dropout. `model.train()` sets the model to training mode.

10. You can save and load a PyTorch model using:
    ```python
    # Save
    torch.save(model.state_dict(), 'model.pth')
    # Load
    model.load_state_dict(torch.load('model.pth'))
    ```

11. `optimizer.zero_grad()` resets the gradients of all parameters to zero. This is necessary because gradients accumulate by default.

12. You can use GPUs in PyTorch by moving your model and data to the GPU:
    ```python
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model.to(device)
    data = data.to(device)
    ```

13. `torch.no_grad()` temporarily disables gradient computation, while `requires_grad=False` permanently disables gradient computation for a tensor.

14. Transfer learning can be implemented by loading a pre-trained model, freezing its layers, and adding new layers:
    ```python
    pretrained_model = models.resnet18(pretrained=True)
    for param in pretrained_model.parameters():
        param.requires_grad = False
    num_ftrs = pretrained_model.fc.in_features
    pretrained_model.fc = nn.Linear(num_ftrs, num_classes)
    ```

15. `DataLoader` is used to load data in batches, shuffle it, and use multiple workers for data loading.

16. Variable-length sequences can be handled using padding and pack_padded_sequence:
    ```python
    packed_sequence = nn.utils.rnn.pack_padded_sequence(padded_input, lengths, batch_first=True)
    ```

17. `nn.Module` is used to define neural network modules, while `nn.functional` provides functional interfaces for neural network layers.

18. You can implement a custom dataset by subclassing `torch.utils.data.Dataset`:
    ```python
    class MyDataset(torch.utils.data.Dataset):
        def __init__(self, data):
            self.data = data

        def __len__(self):
            return len(self.data)

        def __getitem__(self, idx):
            return self.data[idx]