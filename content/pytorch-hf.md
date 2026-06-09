---
title: "The Complete Modern PyTorch & Hugging Face Transformers Reference"
description: "A ground-up reference for building genuine understanding — not just syntax, but intuition."
---

## PART I: PYTORCH FOUNDATIONS

---

### Chapter 1: The Tensor — Everything Starts Here

A tensor is just a generalization of arrays to arbitrary dimensions. A scalar is a 0-dimensional tensor. A vector is 1-dimensional. A matrix is 2-dimensional. An image with height, width, and color channels is 3-dimensional. A batch of images is 4-dimensional. That's it. The word "tensor" sounds intimidating but it's just a container for numbers arranged in a shape.

The crucial thing that makes PyTorch tensors different from NumPy arrays is that they live on a computation graph. Every operation you do on a tensor can be recorded so that PyTorch can later figure out how to compute gradients. This is the entire foundation of deep learning in PyTorch.

**Creating Tensors**

```python
import torch

# From data
x = torch.tensor([1.0, 2.0, 3.0])
A = torch.tensor([[1.0, 2.0], [3.0, 4.0]])

# From factory functions
zeros = torch.zeros(3, 4)          # 3x4 matrix of zeros
ones = torch.ones(3, 4)
rand = torch.rand(3, 4)            # uniform [0, 1)
randn = torch.randn(3, 4)          # standard normal N(0,1)
arange = torch.arange(0, 10, 2)   # [0, 2, 4, 6, 8]
linspace = torch.linspace(0, 1, 5) # 5 evenly spaced points

# Like another tensor (same shape, same device)
x_like = torch.zeros_like(A)
```

**Shapes and Dimensions — The Most Important Skill**

Reading tensor shapes is the most fundamental skill in deep learning. A shape like (32, 512) means 32 rows and 512 columns — perhaps a batch of 32 sentences each represented as a 512-dimensional vector. Get comfortable parsing shapes immediately.

```python
x = torch.randn(32, 10, 512)
print(x.shape)        # torch.Size([32, 10, 512])
print(x.ndim)         # 3
print(x.shape[0])     # 32  — batch size
print(x.shape[-1])    # 512 — last dim (feature dim usually)

# Reshaping — total elements must stay the same
x = torch.arange(12)          # shape: (12,)
x = x.reshape(3, 4)           # shape: (3, 4)
x = x.reshape(3, -1)          # -1 means "figure it out": (3, 4)
x = x.view(2, 6)              # view shares memory with original (faster but has restrictions)

# Adding/removing dimensions
x = torch.randn(10, 512)
x = x.unsqueeze(0)            # (1, 10, 512) — add batch dim
x = x.squeeze(0)              # (10, 512) — remove size-1 dim
x = x.unsqueeze(-1)           # (10, 512, 1)

# Permuting dimensions — crucial for image/sequence work
img = torch.randn(3, 32, 32)          # C, H, W (PyTorch convention)
img = img.permute(1, 2, 0)           # H, W, C (matplotlib convention)
```

**Indexing and Slicing**

This follows NumPy exactly. If you know NumPy, you know this.

```python
x = torch.randn(4, 6)
print(x[0])         # first row
print(x[:, 0])      # first column
print(x[1:3, 2:5])  # rows 1-2, columns 2-4

# Boolean indexing
mask = x > 0
print(x[mask])      # all positive elements, flattened

# Fancy indexing
indices = torch.tensor([0, 2])
print(x[indices])   # rows 0 and 2
```

**Device Management — CPU vs GPU**

```python
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

x = torch.randn(100, 100).to(device)
y = torch.randn(100, 100).to(device)
z = x @ y   # runs on GPU if available

# The golden rule: all tensors in an operation must be on the same device
# Moving between devices
x_cpu = x.cpu()
x_gpu = x.to("cuda")
x_gpu2 = x.cuda()   # same thing
```

**Data Types (dtypes)**

```python
x = torch.tensor([1.0, 2.0])    # float32 by default
x = x.float()                   # float32  — most common for training
x = x.double()                  # float64  — more precision, slower
x = x.half()                    # float16  — less precision, much faster on GPU
x = x.bfloat16()                # bfloat16 — increasingly common (better range than fp16)

# Integer types for indices, labels
labels = torch.tensor([0, 1, 2]).long()   # int64 — required by CrossEntropyLoss
mask = torch.tensor([True, False]).bool()
```

---

### Chapter 2: Autograd — The Engine of Learning

Here's the core insight: deep learning is just function composition. You have a function f(x; θ) where θ are the parameters. You define a loss L. You want ∂L/∂θ so you can nudge θ to make L smaller. Doing this by hand for millions of parameters is impossible. Autograd does it automatically by recording every operation and applying the chain rule backwards.

**How It Works Conceptually**

When you do operations on tensors with `requires_grad=True`, PyTorch builds a computation graph — a DAG where each node is a tensor and each edge is an operation. When you call `.backward()` on a scalar loss, PyTorch traverses this graph in reverse, computing the gradient of the loss with respect to every leaf tensor that has `requires_grad=True`.

This process is called **backpropagation**, but really it's just the chain rule applied systematically. If L = f(g(x)), then ∂L/∂x = (∂f/∂g)(∂g/∂x). Chain rule. That's all of deep learning's math at the highest level.

```python
# Simple example
x = torch.tensor(2.0, requires_grad=True)
y = x ** 2 + 3 * x + 1   # y = x^2 + 3x + 1

y.backward()   # compute dy/dx

print(x.grad)  # dy/dx = 2x + 3 = 2(2) + 3 = 7.0
```

**The gradient accumulates** — this is crucial to understand. Every call to `.backward()` adds to existing `.grad` values rather than replacing them. This is why you must call `optimizer.zero_grad()` before each backward pass.

```python
# This is the standard training loop skeleton
for batch in dataloader:
    optimizer.zero_grad()   # MUST do this or gradients accumulate across batches
    
    output = model(batch)
    loss = criterion(output, targets)
    
    loss.backward()         # compute gradients
    optimizer.step()        # update parameters using gradients
```

**The `no_grad` context manager**

During inference, you don't need gradients. Computing them wastes memory and time. Use `torch.no_grad()` to disable the graph.

```python
model.eval()
with torch.no_grad():
    predictions = model(test_input)
    # no graph built, no gradients stored, much faster
```

**Detaching from the graph**

```python
# .detach() creates a new tensor that shares data but has no gradient connection
x = torch.randn(10, requires_grad=True)
y = x * 2
y_detached = y.detach()   # y_detached has no grad_fn, won't backprop through
```

---

### Chapter 3: nn.Module — Building Neural Networks

`torch.nn.Module` is the base class for every neural network component in PyTorch. It provides parameter management, device management, and a clean interface for building composable networks.

**The Anatomy of an nn.Module**

```python
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super().__init__()
        # Define layers here. Anything assigned as an attribute of type nn.Module
        # or nn.Parameter is automatically tracked as a parameter.
        self.linear1 = nn.Linear(784, 256)
        self.relu = nn.ReLU()
        self.linear2 = nn.Linear(256, 10)
    
    def forward(self, x):
        # Define the computation here
        x = self.linear1(x)
        x = self.relu(x)
        x = self.linear2(x)
        return x

model = SimpleNet()
```

When you call `model(x)`, PyTorch calls `model.forward(x)`. This also runs any hooks you've registered, which is why you should always call the model like a function rather than calling `.forward()` directly.

**Parameter Management**

```python
# Inspecting parameters
for name, param in model.named_parameters():
    print(name, param.shape, param.requires_grad)

# All parameters as a flat list
params = list(model.parameters())

# Moving the entire model to GPU (moves all parameters and buffers)
model = model.to(device)
model = model.cuda()   # same thing

# Counting parameters
n_params = sum(p.numel() for p in model.parameters())
n_trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total: {n_params:,} | Trainable: {n_trainable:,}")
```

**train() vs eval() mode**

Some layers behave differently during training vs. inference. BatchNorm uses running statistics during eval but batch statistics during train. Dropout is active during train but disabled during eval. Always set the right mode.

```python
model.train()   # before training loop
model.eval()    # before inference
```

**Built-in Layers You Need to Know**

```python
# Linear (fully connected)
# Projects input of size in_features to out_features
# y = xW^T + b
nn.Linear(in_features=512, out_features=256)

# Embedding — maps integer indices to dense vectors
# This is how words become vectors. The table has vocab_size rows, embedding_dim columns.
nn.Embedding(num_embeddings=vocab_size, embedding_dim=256)

# Dropout — randomly zeros out a fraction of neurons during training
# Prevents co-adaptation of features, acts as regularization
nn.Dropout(p=0.1)

# Normalization
nn.LayerNorm(normalized_shape=512)    # normalizes across the feature dimension
nn.BatchNorm1d(num_features=256)      # normalizes across the batch dimension

# Activation functions
nn.ReLU()
nn.GELU()         # common in transformers: smoother than ReLU
nn.SiLU()         # also called Swish, used in LLaMA, Mistral
nn.Tanh()

# Convolutional
nn.Conv2d(in_channels=3, out_channels=64, kernel_size=3, padding=1)
nn.MaxPool2d(kernel_size=2, stride=2)

# Sequence
nn.LSTM(input_size=256, hidden_size=512, batch_first=True)
nn.GRU(input_size=256, hidden_size=512, batch_first=True)

# Transformer
nn.MultiheadAttention(embed_dim=512, num_heads=8, batch_first=True)
nn.TransformerEncoderLayer(d_model=512, nhead=8, dim_feedforward=2048)
```

**Sequential — Stacking Layers Cleanly**

```python
# When your network is just a sequence of operations, nn.Sequential is cleaner
model = nn.Sequential(
    nn.Linear(784, 256),
    nn.ReLU(),
    nn.Dropout(0.1),
    nn.Linear(256, 128),
    nn.ReLU(),
    nn.Linear(128, 10)
)
```

**ModuleList and ModuleDict — Dynamic Architectures**

```python
class TransformerEncoder(nn.Module):
    def __init__(self, n_layers, d_model, n_heads):
        super().__init__()
        # ModuleList is required — plain Python lists don't register submodules
        self.layers = nn.ModuleList([
            nn.TransformerEncoderLayer(d_model, n_heads)
            for _ in range(n_layers)
        ])
    
    def forward(self, x):
        for layer in self.layers:
            x = layer(x)
        return x
```

---

### Chapter 4: Loss Functions and Optimizers

**Loss Functions**

The loss function measures how wrong your model is. The choice of loss function determines what "wrong" means and shapes what the model learns to optimize.

```python
# Cross-Entropy Loss — the workhorse for classification
# Input: raw logits of shape (batch, num_classes)
# Target: integer class indices of shape (batch,) — long dtype
criterion = nn.CrossEntropyLoss()
logits = torch.randn(32, 10)    # batch=32, 10 classes
targets = torch.randint(0, 10, (32,))
loss = criterion(logits, targets)

# Internally: CrossEntropyLoss = Softmax + log + NLLLoss
# It applies log_softmax to logits then computes negative log likelihood
# This is numerically more stable than doing softmax yourself then log

# Binary Cross-Entropy — for binary or multi-label classification
# BCEWithLogitsLoss is better than BCELoss because it's numerically stable
criterion = nn.BCEWithLogitsLoss()
logits = torch.randn(32, 1)     # single output
targets = torch.randint(0, 2, (32, 1)).float()

# MSE Loss — for regression
criterion = nn.MSELoss()
preds = torch.randn(32)
targets = torch.randn(32)
loss = criterion(preds, targets)

# MAE / L1 Loss — for regression, more robust to outliers
criterion = nn.L1Loss()

# Huber Loss — combines L1 and L2, robust + smooth
criterion = nn.HuberLoss(delta=1.0)
```

**Optimizers**

The optimizer uses gradients to update parameters. Different optimizers have different strategies for how aggressively they move in the direction of the gradient.

```python
import torch.optim as optim

# SGD — vanilla stochastic gradient descent
# θ = θ - lr * grad
# Simple, works, but slow convergence
optimizer = optim.SGD(model.parameters(), lr=0.01)

# SGD with momentum — adds velocity: much faster convergence
# v = momentum * v - lr * grad
# θ = θ + v
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9, weight_decay=1e-4)

# Adam — Adaptive Moment Estimation
# The default choice for most tasks. Adapts learning rate per parameter.
# Maintains running averages of both gradients and squared gradients.
optimizer = optim.Adam(model.parameters(), lr=1e-3, betas=(0.9, 0.999), eps=1e-8)

# AdamW — Adam with proper weight decay (decoupled)
# Weight decay in Adam is wrong mathematically; AdamW fixes this.
# Use AdamW by default for transformers — it's almost always better than Adam.
optimizer = optim.AdamW(model.parameters(), lr=1e-4, weight_decay=0.01)

# Per-layer learning rates — useful for fine-tuning
optimizer = optim.AdamW([
    {"params": model.backbone.parameters(), "lr": 1e-5},   # lower lr for pretrained layers
    {"params": model.head.parameters(), "lr": 1e-3},       # higher lr for new layers
])
```

**Learning Rate Schedulers**

The learning rate is often the most impactful hyperparameter. Starting high and decaying helps convergence. Warming up from zero helps stability with large models.

```python
from torch.optim import lr_scheduler

# Step decay — multiply lr by gamma every step_size epochs
scheduler = lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

# Cosine annealing — smoothly decays to near zero
scheduler = lr_scheduler.CosineAnnealingLR(optimizer, T_max=100)

# Linear warmup + cosine decay — the transformer standard
# torch doesn't have this built in cleanly; use transformers library's version
from transformers import get_cosine_schedule_with_warmup
scheduler = get_cosine_schedule_with_warmup(
    optimizer,
    num_warmup_steps=1000,
    num_training_steps=total_steps
)

# ReduceLROnPlateau — reduce when a metric stops improving
scheduler = lr_scheduler.ReduceLROnPlateau(optimizer, patience=5, factor=0.5)

# Step the scheduler after each epoch (or each step, depending on scheduler type)
for epoch in range(epochs):
    train(...)
    scheduler.step()
    # or: scheduler.step(val_loss)  for ReduceLROnPlateau
```

---

### Chapter 5: Data Loading

PyTorch's data pipeline separates two concerns: (1) how to access a single sample (`Dataset`), and (2) how to batch and parallelize loading (`DataLoader`).

**Dataset**

```python
from torch.utils.data import Dataset, DataLoader

class MyDataset(Dataset):
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        # Return a single sample.
        # DataLoader will call this many times and batch the results.
        return self.data[idx], self.labels[idx]

dataset = MyDataset(X_train, y_train)
```

**DataLoader**

```python
loader = DataLoader(
    dataset,
    batch_size=32,
    shuffle=True,           # shuffles every epoch — critical for training
    num_workers=4,          # parallel data loading — set to number of CPU cores
    pin_memory=True,        # faster CPU->GPU transfer; use with CUDA
    drop_last=False,        # if True, drops final incomplete batch
)

for batch_data, batch_labels in loader:
    # batch_data: (32, ...) — DataLoader stacked individual samples
    batch_data = batch_data.to(device)
    batch_labels = batch_labels.to(device)
    # ... training step ...
```

**Transforms — Preprocessing and Augmentation**

```python
from torchvision import transforms

# Compose chains transforms sequentially
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224),      # random crop + resize to 224x224
    transforms.RandomHorizontalFlip(),      # flip with p=0.5
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),                  # PIL Image [0,255] -> Tensor [0.0,1.0]
    transforms.Normalize(
        mean=[0.485, 0.456, 0.406],         # ImageNet mean
        std=[0.229, 0.224, 0.225]           # ImageNet std
    )
])

val_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),             # no random crops at validation time
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])
```

---

### Chapter 6: The Complete Training Loop

```python
def train_one_epoch(model, loader, optimizer, criterion, device, scheduler=None):
    model.train()
    total_loss = 0
    correct = 0
    total = 0
    
    for batch_idx, (inputs, targets) in enumerate(loader):
        inputs, targets = inputs.to(device), targets.to(device)
        
        # 1. Zero gradients
        optimizer.zero_grad()
        
        # 2. Forward pass
        outputs = model(inputs)
        
        # 3. Compute loss
        loss = criterion(outputs, targets)
        
        # 4. Backward pass
        loss.backward()
        
        # 5. Gradient clipping (optional but common for transformers)
        torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        
        # 6. Update parameters
        optimizer.step()
        
        # 7. Step scheduler (if per-step, not per-epoch)
        if scheduler is not None:
            scheduler.step()
        
        # Tracking metrics
        total_loss += loss.item()   # .item() pulls scalar from tensor to Python float
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()
    
    avg_loss = total_loss / len(loader)
    accuracy = correct / total
    return avg_loss, accuracy


def evaluate(model, loader, criterion, device):
    model.eval()
    total_loss = 0
    correct = 0
    total = 0
    
    with torch.no_grad():
        for inputs, targets in loader:
            inputs, targets = inputs.to(device), targets.to(device)
            outputs = model(inputs)
            loss = criterion(outputs, targets)
            
            total_loss += loss.item()
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
    
    return total_loss / len(loader), correct / total


# Main training script
def train(model, train_loader, val_loader, epochs, lr=1e-3):
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = model.to(device)
    
    optimizer = optim.AdamW(model.parameters(), lr=lr, weight_decay=0.01)
    criterion = nn.CrossEntropyLoss()
    
    total_steps = epochs * len(train_loader)
    scheduler = get_cosine_schedule_with_warmup(
        optimizer,
        num_warmup_steps=total_steps // 10,
        num_training_steps=total_steps
    )
    
    best_val_acc = 0
    for epoch in range(epochs):
        train_loss, train_acc = train_one_epoch(
            model, train_loader, optimizer, criterion, device, scheduler
        )
        val_loss, val_acc = evaluate(model, val_loader, criterion, device)
        
        print(f"Epoch {epoch+1}/{epochs} | "
              f"Train Loss: {train_loss:.4f} Acc: {train_acc:.4f} | "
              f"Val Loss: {val_loss:.4f} Acc: {val_acc:.4f}")
        
        # Save best model
        if val_acc > best_val_acc:
            best_val_acc = val_acc
            torch.save(model.state_dict(), "best_model.pt")
    
    # Load best weights
    model.load_state_dict(torch.load("best_model.pt"))
    return model
```

**Saving and Loading Models**

```python
# Save only the parameters (preferred)
torch.save(model.state_dict(), "model.pt")

# Load
model = MyModel()
model.load_state_dict(torch.load("model.pt", map_location=device))

# Save entire model (fragile — tied to class definition location)
torch.save(model, "full_model.pt")

# Save checkpoint (for resuming training)
torch.save({
    "epoch": epoch,
    "model_state_dict": model.state_dict(),
    "optimizer_state_dict": optimizer.state_dict(),
    "loss": val_loss,
}, "checkpoint.pt")

# Load checkpoint
checkpoint = torch.load("checkpoint.pt")
model.load_state_dict(checkpoint["model_state_dict"])
optimizer.load_state_dict(checkpoint["optimizer_state_dict"])
start_epoch = checkpoint["epoch"] + 1
```

---

## PART II: COMPUTER VISION WITH PYTORCH

---

### Chapter 7: Convolutional Neural Networks

**The Core Intuition of Convolution**

Imagine you want to detect whether a horizontal edge exists somewhere in an image. You could slide a small filter (say 3x3) across the image. At each position, you multiply the filter values with the overlapping image values and sum them up. If there's a horizontal edge at that position, the output will be high. This sliding window operation is convolution.

CNNs use many such filters, but they learn the filter values rather than hand-crafting them. Early layers learn edges, curves, and textures. Middle layers combine these into parts (eyes, wheels). Later layers combine parts into objects. This hierarchical feature learning is why CNNs work so well on images.

**Key Terminology**

The kernel (filter) is the small matrix that slides across the input. Padding adds zeros around the border so output size matches input size. Stride controls how far the filter moves each step. Larger stride means smaller output and more downsampling. Dilation inserts gaps in the kernel, expanding the receptive field without increasing parameters.

```python
# Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0)
# A 3x3 kernel with padding=1 preserves spatial dimensions
# A 3x3 kernel with stride=2 halves spatial dimensions

conv = nn.Conv2d(3, 64, kernel_size=3, padding=1, bias=False)

# Output spatial size formula:
# out = floor((in + 2*padding - kernel_size) / stride) + 1
# With in=224, padding=1, kernel=3, stride=1: (224 + 2 - 3)/1 + 1 = 224 (same)
# With in=224, padding=1, kernel=3, stride=2: (224 + 2 - 3)/2 + 1 = 112 (halved)
```

**A Classic CNN for CIFAR-10**

```python
class ConvNet(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        
        # Feature extraction: increasing channels, decreasing spatial dims
        self.features = nn.Sequential(
            # Block 1
            nn.Conv2d(3, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.Conv2d(32, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),      # 32x32 -> 16x16
            nn.Dropout2d(0.25),
            
            # Block 2
            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(2, 2),      # 16x16 -> 8x8
            nn.Dropout2d(0.25),
        )
        
        # Classifier
        self.classifier = nn.Sequential(
            nn.Flatten(),            # (B, 64, 8, 8) -> (B, 4096)
            nn.Linear(64 * 8 * 8, 512),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(512, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x
```

**BatchNorm — Why It Matters**

Batch Normalization normalizes activations across the batch dimension at each layer. This has two powerful effects. First, it smoothes the optimization landscape, allowing higher learning rates. Second, it acts as a regularizer. The intuition: without BN, if a layer's inputs shift dramatically (covariate shift), deeper layers have to constantly re-adapt. BN stabilizes this. Always put BN after linear/conv, before the activation.

**Residual Connections — The ResNet Revolution**

Training very deep networks was essentially impossible before ResNets (2015). The problem is the vanishing gradient: as you backpropagate through many layers, gradients get multiplied by small numbers over and over until they vanish and early layers stop learning.

ResNet's insight is brutally simple: instead of learning H(x), let the layer learn the residual F(x) = H(x) - x. Then H(x) = F(x) + x. The skip connection (the `+ x` part) provides a gradient highway that bypasses all the multiplicative operations, keeping gradients healthy even in 100+ layer networks.

```python
class ResidualBlock(nn.Module):
    def __init__(self, channels):
        super().__init__()
        self.conv1 = nn.Conv2d(channels, channels, 3, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(channels)
        self.relu = nn.ReLU(inplace=True)
        self.conv2 = nn.Conv2d(channels, channels, 3, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(channels)
    
    def forward(self, x):
        residual = x                    # save input
        out = self.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        out = out + residual            # add skip connection
        out = self.relu(out)
        return out
```

**Transfer Learning — The Most Practical Skill in CV**

Training CNNs from scratch requires millions of images. Transfer learning lets you use features learned on ImageNet (1.2M images, 1000 classes) and adapt them to your task. The early layers' features (edges, textures) generalize across almost any visual task. You only need to fine-tune the later, task-specific layers.

```python
import torchvision.models as models

# Load pretrained ResNet50
model = models.resnet50(pretrained=True)

# Strategy 1: Feature extraction (freeze backbone, train only the head)
# Good when your dataset is small
for param in model.parameters():
    param.requires_grad = False

# Replace the final classification head
model.fc = nn.Linear(model.fc.in_features, num_classes)
# Only model.fc parameters have requires_grad=True
optimizer = optim.Adam(model.fc.parameters(), lr=1e-3)

# Strategy 2: Full fine-tuning (unfreeze everything, use small lr)
# Good when you have more data or your domain is very different from ImageNet
for param in model.parameters():
    param.requires_grad = True
optimizer = optim.Adam(model.parameters(), lr=1e-4)

# Strategy 3: Differential learning rates (best practice)
optimizer = optim.Adam([
    {"params": model.layer1.parameters(), "lr": 1e-5},
    {"params": model.layer2.parameters(), "lr": 1e-5},
    {"params": model.layer3.parameters(), "lr": 1e-4},
    {"params": model.layer4.parameters(), "lr": 1e-4},
    {"params": model.fc.parameters(),     "lr": 1e-3},
])
```

**Available Pretrained Models in torchvision**

```python
models.resnet50(pretrained=True)
models.efficientnet_b0(pretrained=True)
models.vgg16(pretrained=True)
models.densenet121(pretrained=True)
models.mobilenet_v3_small(pretrained=True)    # efficient, mobile-friendly
models.vit_b_16(pretrained=True)              # Vision Transformer
models.swin_t(pretrained=True)                # Swin Transformer
```

---

## PART III: NLP — FROM FIRST PRINCIPLES TO TRANSFORMERS

---

### Chapter 8: From Words to Vectors — Embeddings and Representations

Before any neural network can work with text, text must become numbers. The question is: how do you represent the meaning of a word as a vector?

**The Distributional Hypothesis**

"You shall know a word by the company it keeps." — J.R. Firth, 1957. This is the philosophical foundation of all modern NLP. Words that appear in similar contexts have similar meanings. "King" and "queen" both appear near "throne", "crown", "royal". So their vectors should be similar. This simple idea, when scaled up, produces surprisingly rich semantic representations.

**Word2Vec — The Foundational Idea**

Word2Vec (2013, Google) trains a shallow neural network with a specific pretext task: given a word, predict its neighbors (Skip-Gram) or given neighbors, predict the word (CBOW). The learned hidden layer weights become the word embeddings. The task is discarded; only the embeddings are kept. The model never sees explicit labels like "these words are synonyms." It only sees which words appear near which other words in a huge corpus.

The famous result: vector("king") - vector("man") + vector("woman") ≈ vector("queen"). The geometry of the embedding space encodes semantic relationships. This emergent property arises entirely from co-occurrence patterns.

**From Word2Vec to Learned Embeddings in PyTorch**

```python
# nn.Embedding is just a lookup table
# Each integer index maps to a learned vector
vocab_size = 10000
embedding_dim = 256
embedding = nn.Embedding(vocab_size, embedding_dim)

# Usage
token_ids = torch.tensor([1, 42, 999, 7])   # sequence of 4 token IDs
vectors = embedding(token_ids)               # shape: (4, 256)
```

The embedding layer is initialized randomly and trained end-to-end with the rest of the model. In modern transformers, this is how it's done — pretrained static embeddings like Word2Vec are largely obsolete because contextual embeddings (from BERT, GPT etc.) are far richer.

**Tokenization — Splitting Text Into Tokens**

The mapping from text to integers requires a tokenizer. Modern models use subword tokenization: the vocabulary contains whole words for common words, and character pieces for rare words. "running" might tokenize to ["run", "##ning"]. This lets the model handle any word without an out-of-vocabulary problem.

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

text = "The quick brown fox jumps over the lazy dog."
tokens = tokenizer(text, return_tensors="pt")

# tokens is a dict with:
# "input_ids": integer token IDs with [CLS] and [SEP] prepended/appended
# "attention_mask": 1 for real tokens, 0 for padding
# "token_type_ids": which sentence each token belongs to (for BERT)

print(tokenizer.tokenize(text))
# ['the', 'quick', 'brown', 'fox', 'jumps', 'over', 'the', 'lazy', 'dog', '.']

# Tokenize a batch with padding
batch = ["Hello world", "This is a longer sentence that needs padding."]
tokens = tokenizer(batch, padding=True, truncation=True, max_length=64, return_tensors="pt")
# padding adds [PAD] tokens to make all sequences the same length
# attention_mask tells the model which tokens are padding (0) vs real (1)
```

---

### Chapter 9: Recurrent Neural Networks — Processing Sequences

Before transformers, sequences were processed with RNNs. Understanding them builds the intuition for why transformers are better.

**The Core Idea**

A standard neural network processes inputs independently. But language is sequential — the meaning of "bank" depends on whether you're talking about a river or money. RNNs maintain a hidden state that acts as memory, updated at each step.

At timestep t: h_t = tanh(W_h * h_{t-1} + W_x * x_t + b)

The hidden state h_t is computed from the previous hidden state h_{t-1} and the current input x_t. After processing the entire sequence, h_T (the final hidden state) supposedly encodes everything the model knows about the sequence.

```python
class SimpleRNN(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.rnn = nn.RNN(embed_dim, hidden_dim, batch_first=True)
        self.fc = nn.Linear(hidden_dim, num_classes)
    
    def forward(self, x):
        # x: (batch, seq_len) token IDs
        embedded = self.embedding(x)           # (batch, seq_len, embed_dim)
        output, hidden = self.rnn(embedded)    # output: (batch, seq_len, hidden_dim)
                                               # hidden: (1, batch, hidden_dim)
        # Use final hidden state for classification
        return self.fc(hidden.squeeze(0))
```

**The Vanishing Gradient Problem**

The fundamental flaw: when backpropagating through time, gradients flow through the tanh operations at every step. tanh squashes values to [-1, 1], so its derivative is always less than 1. Multiplied across 100 timesteps, the gradient vanishes to near zero. The RNN forgets anything more than ~10-20 steps back. For long documents, this is catastrophic.

**LSTM — The Engineering Solution**

Long Short-Term Memory (1997, Hochreiter & Schmidhuber) adds a dedicated memory cell c_t with three gating mechanisms.

The forget gate decides what to erase from memory: f_t = σ(W_f * [h_{t-1}, x_t] + b_f). Values near 0 forget, values near 1 remember.

The input gate decides what new information to write: i_t = σ(W_i * [h_{t-1}, x_t] + b_i). The candidate values are ĉ_t = tanh(W_c * [h_{t-1}, x_t] + b_c).

The cell state is updated: c_t = f_t ⊙ c_{t-1} + i_t ⊙ ĉ_t. (⊙ is element-wise multiplication)

The output gate controls what gets exposed from the cell: o_t = σ(W_o * [h_{t-1}, x_t] + b_o). h_t = o_t ⊙ tanh(c_t).

The key insight: the cell state c_t flows through time with only element-wise multiplication and addition, no repeated matrix multiplications or squashing. This is the gradient highway — gradients can flow through c_t essentially unchanged across hundreds of steps.

```python
class LSTMClassifier(nn.Module):
    def __init__(self, vocab_size, embed_dim, hidden_dim, num_classes,
                 num_layers=2, dropout=0.3):
        super().__init__()
        self.embedding = nn.Embedding(vocab_size, embed_dim)
        self.lstm = nn.LSTM(
            embed_dim, hidden_dim,
            num_layers=num_layers,
            batch_first=True,
            dropout=dropout,        # dropout between LSTM layers
            bidirectional=True      # process sequence in both directions
        )
        # *2 because bidirectional concatenates forward and backward
        self.fc = nn.Linear(hidden_dim * 2, num_classes)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x):
        embedded = self.dropout(self.embedding(x))
        output, (hidden, cell) = self.lstm(embedded)
        # output: (batch, seq_len, hidden*2) — all timestep outputs
        # hidden: (num_layers*2, batch, hidden) — final hidden states
        
        # Concatenate final forward and backward hidden states
        hidden = torch.cat([hidden[-2], hidden[-1]], dim=1)
        hidden = self.dropout(hidden)
        return self.fc(hidden)
```

**GRU — A Simpler Alternative**

GRU (2014) achieves similar performance to LSTM with fewer parameters. It merges the forget and input gates into a single update gate, and combines cell and hidden state. Often use GRU when you want something lighter; use LSTM when you want maximum capacity.

```python
self.gru = nn.GRU(embed_dim, hidden_dim, batch_first=True, bidirectional=True)
```

**Sequence-to-Sequence with Attention — The Direct Ancestor of Transformers**

Machine translation showed the limits of fixed-size hidden states. You can't compress an entire long sentence into one vector and then produce a translation of arbitrary length. The 2014 attention mechanism (Bahdanau et al.) solved this: instead of using only the final encoder state, the decoder at each step can look at all encoder hidden states and choose which ones to focus on.

This is the conceptual leap: attention allows the decoder to directly access any encoder position. The question "which encoder positions are relevant for this decoding step?" is answered by computing a similarity score between the decoder's current state and each encoder state, normalizing with softmax to get weights, then computing a weighted average of encoder states. This weighted average is the context vector for this step.

This attention mechanism, generalized and parallelized, becomes the transformer's self-attention.

---

### Chapter 10: The Transformer — The Architecture That Changed Everything

**The Fundamental Intuition**

The transformer (2017, "Attention Is All You Need") discards recurrence entirely and relies solely on attention. Every token in the sequence looks at every other token and decides how much to attend to it. This solves the vanishing gradient problem fundamentally: the distance from token 1 to token 100 is always 1 attention step, not 100 LSTM steps.

The cost: attention is O(n²) in sequence length. The benefit: full parallelism (no sequential dependency) and direct long-range connections.

**Self-Attention — The Core Operation**

The idea: for each token, compute a new representation by taking a weighted average of all token representations, where the weights are based on relevance.

How? Each token produces three vectors via learned linear projections:
- Query (Q): "What am I looking for?"
- Key (K): "What do I offer?"
- Value (V): "What do I contribute if you attend to me?"

Attention(Q, K, V) = softmax(QK^T / √d_k) * V

The QK^T step computes a dot-product similarity between each query and all keys — this gives the raw attention scores (an n×n matrix). Dividing by √d_k stabilizes gradients (prevents very small gradients from softmax when d_k is large). Softmax converts scores to weights (positive, summing to 1). Multiplying by V produces the output: each position gets a weighted average of all values.

```python
import math

def self_attention(Q, K, V, mask=None):
    # Q, K, V: (batch, seq_len, d_k)
    d_k = Q.shape[-1]
    
    # Compute similarity scores
    scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(d_k)
    # scores: (batch, seq_len, seq_len)
    
    # Apply mask (e.g. causal mask for language modeling)
    if mask is not None:
        scores = scores.masked_fill(mask == 0, float('-inf'))
    
    # Normalize
    weights = torch.softmax(scores, dim=-1)
    # weights: (batch, seq_len, seq_len) — each row sums to 1
    
    # Weighted sum of values
    output = torch.matmul(weights, V)
    # output: (batch, seq_len, d_k)
    
    return output, weights
```

**Multi-Head Attention — Looking From Multiple Perspectives**

Instead of one attention computation, run h parallel attention heads with lower-dimensional Q/K/V (d_k = d_model / h), then concatenate and project back. Different heads can specialize: one might attend to syntactic relationships, another to semantic ones, another to recent context. They operate in different subspaces of the representation.

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, n_heads):
        super().__init__()
        assert d_model % n_heads == 0
        self.d_model = d_model
        self.n_heads = n_heads
        self.d_k = d_model // n_heads
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
    
    def forward(self, x, mask=None):
        batch, seq_len, _ = x.shape
        
        # Project and split into heads
        Q = self.W_q(x).view(batch, seq_len, self.n_heads, self.d_k).transpose(1, 2)
        K = self.W_k(x).view(batch, seq_len, self.n_heads, self.d_k).transpose(1, 2)
        V = self.W_v(x).view(batch, seq_len, self.n_heads, self.d_k).transpose(1, 2)
        # Q, K, V: (batch, n_heads, seq_len, d_k)
        
        # Attention
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        if mask is not None:
            scores = scores.masked_fill(mask == 0, float('-inf'))
        weights = torch.softmax(scores, dim=-1)
        attn_output = torch.matmul(weights, V)
        # attn_output: (batch, n_heads, seq_len, d_k)
        
        # Concatenate heads and project
        attn_output = attn_output.transpose(1, 2).contiguous().view(batch, seq_len, self.d_model)
        return self.W_o(attn_output)
```

**Positional Encoding — Injecting Order**

Attention is permutation-invariant: "cat sat on mat" and "mat on sat cat" produce the same attention scores if you don't encode position. To tell the model about token order, add a positional encoding to each token embedding.

The original transformer used fixed sinusoidal encodings. Modern models use learned positional embeddings or (more recently) Rotary Position Embeddings (RoPE) which encode relative position directly in the attention computation.

```python
class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(dropout)
        
        # Precompute sinusoidal encodings
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len).unsqueeze(1).float()
        div_term = torch.exp(torch.arange(0, d_model, 2).float() *
                             -(math.log(10000.0) / d_model))
        
        pe[:, 0::2] = torch.sin(position * div_term)   # even dims: sin
        pe[:, 1::2] = torch.cos(position * div_term)   # odd dims: cos
        
        pe = pe.unsqueeze(0)   # (1, max_len, d_model)
        self.register_buffer("pe", pe)   # not a parameter but moves with model.to(device)
    
    def forward(self, x):
        # x: (batch, seq_len, d_model)
        x = x + self.pe[:, :x.shape[1]]
        return self.dropout(x)
```

**The Feed-Forward Network (FFN)**

Each transformer layer has an FFN applied independently to each position after attention. It's two linear layers with a nonlinearity. It expands the dimension to 4× d_model and projects back. The FFN is thought to store factual knowledge in its weights (this is where much of the "memory" in an LLM resides).

```python
class FeedForward(nn.Module):
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model),
            nn.Dropout(dropout)
        )
    
    def forward(self, x):
        return self.net(x)
```

**Complete Transformer Encoder Layer**

```python
class TransformerEncoderLayer(nn.Module):
    def __init__(self, d_model=512, n_heads=8, d_ff=2048, dropout=0.1):
        super().__init__()
        self.attn = MultiHeadAttention(d_model, n_heads)
        self.ff = FeedForward(d_model, d_ff, dropout)
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, mask=None):
        # Pre-norm variant (used in most modern transformers)
        # Original paper used post-norm; pre-norm trains more stably
        
        # Self-attention with residual
        x = x + self.dropout(self.attn(self.norm1(x), mask))
        
        # FFN with residual
        x = x + self.dropout(self.ff(self.norm2(x)))
        
        return x
```

**Why Pre-Norm vs Post-Norm?** The original paper applies LayerNorm after the residual addition. Modern models (GPT-3, LLaMA etc.) apply it before (pre-norm). Pre-norm stabilizes training at large scale: gradients don't blow up because the norm is applied before rather than after the potentially large residual additions.

**Causal Masking for Language Modeling**

When training a language model (predicting next token), each position can only attend to previous positions, not future ones. A causal mask is an upper-triangular mask of -inf values.

```python
def causal_mask(seq_len):
    # Lower triangular matrix: position i can see positions 0..i
    mask = torch.tril(torch.ones(seq_len, seq_len))
    return mask   # shape: (seq_len, seq_len)
```

---

## PART IV: THE HUGGING FACE ECOSYSTEM

---

### Chapter 11: The Transformers Library — Core Concepts

The Hugging Face `transformers` library is the standard interface for working with pretrained transformer models. It standardizes a huge zoo of architectures (BERT, GPT-2, T5, LLaMA, Mistral, Falcon, etc.) under a consistent API.

**The Three-Part Structure**

Every model in the library has three components: a configuration, a tokenizer, and the model itself.

```python
from transformers import AutoConfig, AutoTokenizer, AutoModel

# Load all three from a pretrained checkpoint
config = AutoConfig.from_pretrained("bert-base-uncased")
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")
```

The `Auto` classes automatically figure out the right class for the given checkpoint. `AutoModel` gives you the base model without a head. Task-specific heads are added by `AutoModelForSequenceClassification`, `AutoModelForTokenClassification`, `AutoModelForCausalLM`, etc.

**The Configuration Object**

The config defines all hyperparameters: number of layers, hidden size, number of heads, vocab size, etc.

```python
config = AutoConfig.from_pretrained("bert-base-uncased")
print(config.hidden_size)       # 768
print(config.num_hidden_layers) # 12
print(config.num_attention_heads) # 12
print(config.vocab_size)         # 30522

# Creating a custom architecture
from transformers import BertConfig, BertModel
small_config = BertConfig(
    hidden_size=256,
    num_hidden_layers=4,
    num_attention_heads=4,
    intermediate_size=1024,
    vocab_size=30522
)
small_model = BertModel(small_config)  # randomly initialized
```

**Tokenizer Deep Dive**

```python
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# Single sentence
enc = tokenizer("Hello, how are you?", return_tensors="pt")
# enc["input_ids"]: [[101, 7592, 1010, 2129, 2024, 2017, 1029, 102]]
# 101 = [CLS], 102 = [SEP]

# Batch with padding and truncation
texts = ["Short sentence.", "This is a much longer sentence that contains more tokens."]
enc = tokenizer(
    texts,
    padding="longest",      # pad to longest in batch; or "max_length" for absolute max
    truncation=True,        # truncate to max_length
    max_length=128,
    return_tensors="pt"
)
# enc["attention_mask"] will be 0 for padding tokens

# Decoding back to text
ids = enc["input_ids"][0]
print(tokenizer.decode(ids))               # with special tokens
print(tokenizer.decode(ids, skip_special_tokens=True))  # without

# Token-level details
tokens = tokenizer.convert_ids_to_tokens(ids.tolist())
print(tokens)   # ['[CLS]', 'short', 'sentence', '.', '[SEP]', '[PAD]', ...]

# Pair encoding (for NLI, QA tasks — two sentences with separator)
enc = tokenizer("Sentence A.", "Sentence B.", return_tensors="pt")
# BERT uses: [CLS] A tokens [SEP] B tokens [SEP]
```

**The Pipeline API — Quickest Path to Results**

For rapid prototyping and inference, pipelines are the fastest interface:

```python
from transformers import pipeline

# Text classification (sentiment)
classifier = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
result = classifier("This movie was absolutely incredible!")
# [{'label': 'POSITIVE', 'score': 0.9998}]

# Named entity recognition
ner = pipeline("ner", model="dslim/bert-base-NER", aggregation_strategy="simple")
result = ner("Elon Musk founded SpaceX in 2002.")
# [{'entity_group': 'PER', 'word': 'Elon Musk', ...}, {'entity_group': 'ORG', 'word': 'SpaceX', ...}]

# Question answering
qa = pipeline("question-answering", model="deepset/roberta-base-squad2")
result = qa(question="Who founded SpaceX?", context="Elon Musk founded SpaceX in 2002 in California.")
# {'answer': 'Elon Musk', 'score': 0.99, 'start': 0, 'end': 9}

# Text generation
generator = pipeline("text-generation", model="gpt2")
result = generator("The future of AI is", max_new_tokens=50, num_return_sequences=3)

# Summarization
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
result = summarizer(long_text, max_length=130, min_length=30)

# Translation
translator = pipeline("translation_en_to_fr", model="Helsinki-NLP/opus-mt-en-fr")
result = translator("Machine learning is transforming the world.")

# Zero-shot classification (classify without any labeled data for that task!)
zeroshot = pipeline("zero-shot-classification", model="facebook/bart-large-mnli")
result = zeroshot(
    "This football match was breathtaking.",
    candidate_labels=["sports", "politics", "science"]
)
# {'labels': ['sports', 'politics', 'science'], 'scores': [0.98, 0.01, 0.01]}

# Run on GPU
classifier = pipeline("text-classification", device=0)   # device=0 for first GPU
```

---

### Chapter 12: BERT and Encoder-Only Models

**The Architecture and Intuition**

BERT (Bidirectional Encoder Representations from Transformers, 2018, Google) is a transformer encoder trained on two pretraining tasks:

1. Masked Language Modeling (MLM): Randomly mask 15% of tokens and predict the masked tokens. Because the encoder sees the entire sequence (bidirectionally), it must understand context from both directions to fill in the mask.

2. Next Sentence Prediction (NSP): Given two sentences, predict if the second follows the first. (Later research showed NSP is not very helpful; models like RoBERTa dropped it.)

The key intuition: BERT produces **contextual embeddings**. Unlike Word2Vec where "bank" always has the same vector, BERT's representation of "bank" changes based on the surrounding context — different for "river bank" and "bank account". This is possible because the self-attention mechanism allows each token to incorporate information from all other tokens.

**BERT Variants**

`bert-base-uncased`: 12 layers, 768 hidden, 12 heads, 110M parameters. Lowercase input.
`bert-large-uncased`: 24 layers, 1024 hidden, 16 heads, 340M parameters.
`distilbert-base-uncased`: 6 layers, 66M params, 97% of BERT quality, 60% faster. Use for production.
`roberta-base`: BERT but trained longer with more data, better tokenization (BPE not WordPiece), no NSP. Generally outperforms BERT.
`deberta-v3-base`: Microsoft's model with disentangled attention, often SOTA on benchmarks.
`albert-base-v2`: Parameter-efficient BERT: shares parameters across layers. Much smaller.

**Fine-tuning BERT for Classification**

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch
import torch.nn as nn
from torch.optim import AdamW
from transformers import get_linear_schedule_with_warmup

# Load pretrained model with classification head
# AutoModelForSequenceClassification adds a linear head on top of [CLS] token
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=2   # binary classification
)

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# Fine-tuning hyperparameters for BERT (from the original paper)
# lr: 2e-5 to 5e-5, batch: 16 or 32, epochs: 2-4
# These are very small because BERT's weights are already good

optimizer = AdamW(model.parameters(), lr=2e-5, weight_decay=0.01)

# Warmup is important: start with lr=0 and ramp up to avoid catastrophic forgetting
total_steps = len(train_dataloader) * num_epochs
scheduler = get_linear_schedule_with_warmup(
    optimizer,
    num_warmup_steps=total_steps // 10,
    num_training_steps=total_steps
)

# Training loop
model.train()
for batch in train_dataloader:
    input_ids = batch["input_ids"].to(device)
    attention_mask = batch["attention_mask"].to(device)
    labels = batch["labels"].to(device)
    
    outputs = model(input_ids, attention_mask=attention_mask, labels=labels)
    # When you pass labels, HuggingFace models compute loss internally
    loss = outputs.loss
    logits = outputs.logits   # raw predictions before softmax
    
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    optimizer.step()
    scheduler.step()
    optimizer.zero_grad()
```

**Named Entity Recognition with BERT**

```python
# For token classification (NER, POS tagging), we need per-token predictions
model = AutoModelForTokenClassification.from_pretrained(
    "bert-base-cased",   # cased for NER — capitalization matters
    num_labels=len(label_list)   # e.g. B-PER, I-PER, B-ORG, O, etc.
)

# The model outputs logits of shape (batch, seq_len, num_labels)
# Note: because BERT uses subword tokenization, you need to align
# subword tokens with word-level labels
# Typically: give the first subword the label, mark others as -100 (ignored by loss)
```

**Extractive Question Answering**

```python
from transformers import AutoModelForQuestionAnswering

model = AutoModelForQuestionAnswering.from_pretrained("bert-base-uncased")

# For QA, tokenizer encodes question and context together:
# [CLS] question tokens [SEP] context tokens [SEP]

enc = tokenizer(question, context, return_tensors="pt", truncation=True, max_length=384)
outputs = model(**enc)

# Model outputs start and end logits over context positions
start_logits = outputs.start_logits    # (batch, seq_len)
end_logits = outputs.end_logits        # (batch, seq_len)

start = start_logits.argmax()
end = end_logits.argmax() + 1
answer = tokenizer.decode(enc["input_ids"][0][start:end])
```

---

### Chapter 13: GPT and Decoder-Only Models

**The Architecture and Intuition**

GPT (Generative Pretrained Transformer, OpenAI) uses a causal (decoder-only) transformer. Unlike BERT's bidirectional attention, GPT uses a causal mask: each token can only attend to itself and previous tokens. This makes it autoregressive — it generates one token at a time, each conditioned on all previous tokens.

Pretraining task: next token prediction on massive text corpora. Given all previous tokens, predict the next one. This single task, run at scale on billions of parameters and tokens, produces models with remarkable emergent capabilities.

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
model = GPT2LMHeadModel.from_pretrained("gpt2")

# Text generation — greedy decoding
input_ids = tokenizer.encode("The history of artificial intelligence", return_tensors="pt")
output = model.generate(input_ids, max_new_tokens=100)
print(tokenizer.decode(output[0], skip_special_tokens=True))

# Beam search — keeps k candidates at each step
output = model.generate(
    input_ids,
    max_new_tokens=100,
    num_beams=5,            # beam width
    early_stopping=True
)

# Sampling — stochastic generation with temperature
output = model.generate(
    input_ids,
    max_new_tokens=100,
    do_sample=True,
    temperature=0.7,        # lower = more focused, higher = more random
    top_k=50,               # only sample from top 50 tokens
    top_p=0.92,             # nucleus sampling: sample from smallest set with cumulative p >= 0.92
)

# Repetition penalty
output = model.generate(input_ids, repetition_penalty=1.3)
```

**Temperature, Top-k, and Nucleus Sampling — The Intuition**

Temperature divides the logits before softmax. Low temperature (0.3) makes the distribution sharper — the highest-probability token becomes even more dominant, leading to repetitive but coherent output. High temperature (1.5) flattens the distribution, creating more varied but potentially incoherent output.

Top-k sampling restricts the sampling pool to the k most likely tokens, preventing sampling from the very long tail of improbable tokens.

Nucleus (top-p) sampling is more adaptive: it takes the smallest set of tokens whose cumulative probability exceeds p. If the model is confident, this is a small set (1-2 tokens). If the model is uncertain, it's a larger set. This adapts to the model's own uncertainty.

**Computing Perplexity — Evaluating Language Models**

Perplexity = exp(average negative log likelihood). A model with perplexity 50 means it's as uncertain as uniformly choosing among 50 words at each step. Lower is better.

```python
def compute_perplexity(model, tokenizer, text, device="cpu"):
    model.eval()
    encodings = tokenizer(text, return_tensors="pt")
    input_ids = encodings.input_ids.to(device)
    
    with torch.no_grad():
        outputs = model(input_ids, labels=input_ids)
    
    # Loss is the average negative log likelihood
    return torch.exp(outputs.loss).item()
```

---

### Chapter 14: Seq2Seq Models — T5, BART, and Encoder-Decoder Architecture

**The Encoder-Decoder Architecture**

For tasks that require transformation from one sequence to another — translation, summarization, question answering — the full encoder-decoder transformer is natural. The encoder processes the input into rich contextual representations. The decoder generates the output autoregressively, using cross-attention to look at encoder outputs at each generation step.

Cross-attention is like self-attention, but queries come from the decoder and keys/values come from the encoder. Each decoder position can look at any encoder position. This is the direct descendant of the seq2seq attention mechanism.

**T5 — Text-to-Text Transfer Transformer**

T5 (Google, 2019) reframes every NLP task as text-to-text. Classification, translation, summarization, QA — all become "given this input text, produce this output text." This unification is powerful because the same model and training procedure works for everything.

```python
from transformers import T5ForConditionalGeneration, T5Tokenizer

tokenizer = T5Tokenizer.from_pretrained("t5-base")
model = T5ForConditionalGeneration.from_pretrained("t5-base")

# T5 uses task prefixes to condition behavior
# Summarization
input_text = "summarize: " + long_article
input_ids = tokenizer(input_text, return_tensors="pt", max_length=512, truncation=True).input_ids
outputs = model.generate(input_ids, max_length=150, num_beams=4, early_stopping=True)
summary = tokenizer.decode(outputs[0], skip_special_tokens=True)

# Translation
input_text = "translate English to French: " + "The weather is beautiful today."
input_ids = tokenizer(input_text, return_tensors="pt").input_ids
outputs = model.generate(input_ids, max_length=100)
translation = tokenizer.decode(outputs[0], skip_special_tokens=True)
```

**BART — Denoising Autoencoder**

BART (Facebook/Meta, 2019) is pretrained by corrupting text (token masking, deletion, infilling, sentence permutation) and training the model to reconstruct the original. This makes it excellent at generation tasks, especially summarization.

```python
from transformers import BartForConditionalGeneration, BartTokenizer

tokenizer = BartTokenizer.from_pretrained("facebook/bart-large-cnn")
model = BartForConditionalGeneration.from_pretrained("facebook/bart-large-cnn")

inputs = tokenizer(article, return_tensors="pt", max_length=1024, truncation=True)
summary_ids = model.generate(
    inputs["input_ids"],
    max_length=200,
    min_length=50,
    length_penalty=2.0,    # > 1 penalizes short outputs, encourages longer summaries
    num_beams=4
)
summary = tokenizer.decode(summary_ids[0], skip_special_tokens=True)
```

---

### Chapter 15: Hugging Face Datasets — Loading and Processing Data

The `datasets` library is a separate HF library for loading and manipulating datasets. It stores data in Apache Arrow format (columnar, memory-mapped) which means you can work with datasets far larger than RAM.

**Loading Datasets**

```python
from datasets import load_dataset

# Load from the HF Hub (thousands of public datasets)
dataset = load_dataset("imdb")
# Returns a DatasetDict with "train" and "test" splits

print(dataset)
# DatasetDict({
#     train: Dataset({features: ['text', 'label'], num_rows: 25000})
#     test:  Dataset({features: ['text', 'label'], num_rows: 25000})
# })

# Access splits
train_data = dataset["train"]
print(train_data[0])        # first example: {'text': '...', 'label': 1}
print(train_data[:5])       # first 5 examples as a dict of lists

# Load specific configurations or splits
dataset = load_dataset("glue", "sst2", split="train")
dataset = load_dataset("squad", split="train[:5000]")   # first 5000 examples

# Load from local files
dataset = load_dataset("csv", data_files={"train": "train.csv", "test": "test.csv"})
dataset = load_dataset("json", data_files="data.jsonl")
dataset = load_dataset("text", data_files="corpus.txt")

# Load from pandas
import pandas as pd
from datasets import Dataset
df = pd.read_csv("data.csv")
dataset = Dataset.from_pandas(df)
```

**Processing Datasets**

```python
# map() applies a function to every example
def tokenize(example):
    return tokenizer(
        example["text"],
        padding="max_length",
        truncation=True,
        max_length=512
    )

# batched=True processes many examples at once (much faster)
tokenized = dataset.map(tokenize, batched=True, batch_size=1000)

# You can also map in batch mode to leverage vectorized operations
def tokenize_batch(batch):
    return tokenizer(batch["text"], padding="max_length", truncation=True, max_length=512)

tokenized = dataset["train"].map(tokenize_batch, batched=True)

# filter() keeps examples that satisfy a condition
short_dataset = dataset["train"].filter(lambda x: len(x["text"].split()) < 100)

# remove_columns() drops columns you don't need
tokenized = tokenized.remove_columns(["text"])

# rename_column()
dataset = dataset.rename_column("label", "labels")

# Set the format for PyTorch — makes __getitem__ return tensors
tokenized.set_format("torch", columns=["input_ids", "attention_mask", "label"])

# Now use directly with DataLoader
from torch.utils.data import DataLoader
loader = DataLoader(tokenized, batch_size=32, shuffle=True)
```

**Saving and Loading Processed Datasets**

```python
# Save to disk (Arrow format — fast loading)
tokenized.save_to_disk("processed_data/")

# Load back
from datasets import load_from_disk
dataset = load_from_disk("processed_data/")

# Push to Hub (share with the world)
dataset.push_to_hub("your-username/your-dataset-name")
```

---

### Chapter 16: The Trainer API — Production-Grade Fine-Tuning

The `Trainer` class handles the training loop, evaluation, checkpointing, mixed precision, gradient accumulation, distributed training, and logging — so you don't have to implement all of this yourself.

```python
from transformers import Trainer, TrainingArguments
import numpy as np
from datasets import load_metric   # or evaluate library

# Define training configuration
training_args = TrainingArguments(
    output_dir="./results",
    
    # Training dynamics
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=64,
    gradient_accumulation_steps=2,   # effective batch size = 16 * 2 * num_gpus
    
    # Learning rate
    learning_rate=2e-5,
    weight_decay=0.01,
    warmup_ratio=0.1,
    lr_scheduler_type="cosine",
    
    # Mixed precision — HUGE speed boost on modern GPUs
    fp16=True,        # float16, use on V100/A100
    # bf16=True,      # bfloat16, prefer on A100/newer hardware
    
    # Evaluation and saving
    eval_strategy="epoch",   # "steps" or "epoch"
    save_strategy="epoch",
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    greater_is_better=True,
    
    # Logging
    logging_dir="./logs",
    logging_steps=100,
    report_to="wandb",   # "none", "wandb", "tensorboard"
    
    # Gradient clipping
    max_grad_norm=1.0,
    
    # Reproducibility
    seed=42,
    data_seed=42,
)

# Compute metrics function
from evaluate import load as load_metric

accuracy_metric = load_metric("accuracy")
f1_metric = load_metric("f1")

def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    acc = accuracy_metric.compute(predictions=predictions, references=labels)
    f1 = f1_metric.compute(predictions=predictions, references=labels, average="weighted")
    return {**acc, **f1}

# Create Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_val,
    tokenizer=tokenizer,
    compute_metrics=compute_metrics,
)

# Train
trainer.train()

# Evaluate
metrics = trainer.evaluate()

# Save
trainer.save_model("./best_model")
tokenizer.save_pretrained("./best_model")
```

**Data Collators**

Data collators handle final batching: padding sequences in a batch to the same length. Using `DataCollatorWithPadding` is more efficient than pre-padding to max_length because each batch is padded only to the length of its longest sequence.

```python
from transformers import DataCollatorWithPadding

# Tokenize WITHOUT fixed padding
def tokenize(examples):
    return tokenizer(examples["text"], truncation=True, max_length=512)
    # no padding=True here

tokenized = dataset.map(tokenize, batched=True)

# The collator pads each batch dynamically to the batch's max length
data_collator = DataCollatorWithPadding(tokenizer=tokenizer)

trainer = Trainer(
    ...
    data_collator=data_collator,
)
```

---

## PART V: MODERN TECHNIQUES

---

### Chapter 17: Parameter-Efficient Fine-Tuning (PEFT)

Fine-tuning a 7B parameter model requires storing 7B gradient values and 7B optimizer states in addition to the model itself. On a 16GB GPU this is impossible. PEFT methods adapt large models while only training a small fraction of parameters.

**LoRA — Low-Rank Adaptation**

LoRA (2021, Microsoft) is the most widely used PEFT method. The core insight: when you fine-tune a pretrained model, the change in weight matrices ΔW has a low intrinsic rank. Instead of updating the full W (d×d = d² parameters), you decompose the update as ΔW = BA where B is d×r and A is r×d, with rank r << d. You only train A and B, which together have 2dr parameters instead of d².

In the weight matrix W, instead of computing Wx, you compute Wx + BAx / √r. The pretrained W is frozen; only A and B are trained. At inference time, you can merge: W' = W + BA, so there's zero additional latency.

```python
from peft import LoraConfig, get_peft_model, TaskType

# Configure LoRA
lora_config = LoraConfig(
    task_type=TaskType.SEQ_CLS,    # or CAUSAL_LM, SEQ_2_SEQ_LM
    r=16,                          # rank — higher = more parameters, more capacity
    lora_alpha=32,                 # scaling factor: ΔW is scaled by alpha/r
    target_modules=["query", "value"],   # which modules to apply LoRA to
    lora_dropout=0.1,
    bias="none",                   # don't train biases
)

# Wrap model with LoRA
model = get_peft_model(model, lora_config)

# See how many parameters you're actually training
model.print_trainable_parameters()
# trainable params: 294,912 || all params: 109,778,688 || trainable%: 0.27

# Train exactly as normal — only LoRA parameters will be updated
# The training loop doesn't change at all

# Save only the LoRA adapter weights
model.save_pretrained("lora_adapter/")

# Load and merge later
from peft import PeftModel
base_model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased")
model = PeftModel.from_pretrained(base_model, "lora_adapter/")
merged = model.merge_and_unload()   # merges adapter into base weights
```

**Other PEFT Methods**

Prefix Tuning prepends learnable "virtual tokens" to the input at every layer. The model processes these alongside real tokens and they condition all attention computations. Only the prefix parameters are trained.

Prompt Tuning (soft prompts) is a simplified version: learnable embeddings prepended only at the input layer. Very few parameters, works well at large scales.

IA³ (Infused Adapter by Inhibiting and Amplifying Inner Activations) rescales activations by learned vectors. Even fewer parameters than LoRA, comparable quality.

```python
# Using other PEFT methods is equally simple
from peft import PromptTuningConfig, IA3Config

prompt_config = PromptTuningConfig(
    task_type=TaskType.CAUSAL_LM,
    num_virtual_tokens=20,
    tokenizer_name_or_path="gpt2"
)
```

---

### Chapter 18: Quantization — Making Models Smaller

**The Basic Idea**

Model weights are stored as float32 (4 bytes per value). A 7B model = 28 GB. Most modern consumer GPUs have 8-24 GB VRAM. Quantization represents weights in lower precision (int8, int4) reducing memory by 2-8× with modest quality loss.

```python
# 8-bit quantization via bitsandbytes
# Install: pip install bitsandbytes accelerate

from transformers import AutoModelForCausalLM, BitsAndBytesConfig

# 8-bit: ~2x memory reduction
quantization_config = BitsAndBytesConfig(load_in_8bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=quantization_config,
    device_map="auto"   # automatically distributes across available GPUs
)

# 4-bit (NF4): ~4x memory reduction — QLoRA's approach
quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.bfloat16,   # compute in bf16 despite 4-bit storage
    bnb_4bit_quant_type="nf4",              # NormalFloat4, better for normally distributed weights
    bnb_4bit_use_double_quant=True          # quantize the quantization constants too
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=quantization_config,
    device_map="auto"
)
```

**QLoRA — The Combination That Enables LLM Fine-Tuning on a Single GPU**

QLoRA (2023, Dettmers et al.) combines 4-bit quantization with LoRA. The base model is frozen in 4-bit, you train only LoRA adapters in bfloat16. This allows fine-tuning a 65B parameter model on a single 48GB GPU, or a 7B model on a 16GB GPU.

```python
from peft import prepare_model_for_kbit_training

# After loading with 4-bit quantization:
model = prepare_model_for_kbit_training(model)   # enables gradient checkpointing for quantized model
model = get_peft_model(model, lora_config)

# Training proceeds normally — only LoRA adapters are trained
```

---

### Chapter 19: Mixed Precision Training

**The Problem**

Full precision (float32) training uses 4 bytes per number. float16 uses 2 bytes. If you could train in float16, you'd use half the memory and achieve 2-3× speedup on modern GPUs. But float16 has a limited range (max ~65504) and precision issues with very small gradients (they underflow to 0).

**AMP — Automatic Mixed Precision**

PyTorch's AMP solution: keep a full-precision master copy of weights for updates, but do the forward/backward pass in float16 for speed. A gradient scaler multiplies the loss before backward to prevent underflow, then unscales before the optimizer step.

```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for batch in loader:
    optimizer.zero_grad()
    
    # Forward pass in float16
    with autocast():
        output = model(batch)
        loss = criterion(output, targets)
    
    # Backward pass with scaled gradients
    scaler.scale(loss).backward()
    
    # Unscale, check for inf/nan, then clip
    scaler.unscale_(optimizer)
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    
    # Update weights (skips update if gradients have inf/nan)
    scaler.step(optimizer)
    scaler.update()
```

The Trainer API enables this with just `fp16=True` or `bf16=True` in TrainingArguments.

---

### Chapter 20: Modern LLM Architectures and Improvements

Understanding what makes recent open-source LLMs (LLaMA, Mistral, Gemma, Phi) different from vanilla transformers builds architectural intuition.

**RoPE — Rotary Position Embeddings**

Instead of adding position encodings to token embeddings, RoPE rotates query and key vectors by an angle proportional to their position. The dot product of Q and K then naturally encodes relative position: the rotation difference between position i and j appears directly in the attention score. Advantages: better length generalization (can extrapolate beyond training length), relative rather than absolute position representation.

**SwiGLU — Upgraded Activation in FFN**

Standard FFN: FFN(x) = GELU(xW₁)W₂. SwiGLU (used in LLaMA, Mistral, Gemma): FFN(x) = (SiLU(xW₁) ⊙ xW₃)W₂. Two parallel projections, one through SiLU, element-wise multiplied together. This gating mechanism enables richer feature selection. The FFN dimension is typically 2/3 × 4 × d_model to maintain similar parameter count.

**Grouped-Query Attention (GQA)**

Standard multi-head attention (MHA): Q, K, V all have n_heads. Multi-query attention (MQA): K and V have 1 head, Q has n_heads — reduces KV cache size but hurts quality. GQA is the middle ground: K and V have g groups (e.g. 8 instead of 32), Q has full n_heads. Used in LLaMA 2 70B, Mistral 7B. Reduces KV cache memory significantly with minimal quality loss.

**RMSNorm — Simpler LayerNorm**

Standard LayerNorm: normalize by subtracting mean and dividing by std (centering + scaling). RMSNorm: only normalize by RMS (root mean square), skip the centering. Simpler, slightly faster, empirically as good.

```python
class RMSNorm(nn.Module):
    def __init__(self, dim, eps=1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(dim))
    
    def forward(self, x):
        rms = x.pow(2).mean(-1, keepdim=True).add(self.eps).sqrt()
        return x / rms * self.weight
```

**Key-Value Cache — How Autoregressive Inference Works**

When generating autoregressively, at each step you compute attention over all previous tokens. Without caching, you'd recompute K and V for all previous tokens at every step. The KV cache stores these computed K and V tensors. Each new step only computes K and V for the new token and appends to the cache. This makes generation O(n) instead of O(n²) per step.

```python
# Using KV cache with HuggingFace
model.eval()
past_key_values = None

with torch.no_grad():
    for step in range(max_steps):
        outputs = model(
            input_ids=current_token,
            past_key_values=past_key_values,
            use_cache=True
        )
        past_key_values = outputs.past_key_values   # store for next step
        next_token = outputs.logits[:, -1, :].argmax(dim=-1, keepdim=True)
        current_token = next_token
```

---

### Chapter 21: Supervised Fine-Tuning and Instruction Tuning

**The Paradigm Shift**

A base pretrained language model (like GPT-2 or LLaMA base) is a next-token predictor. It will complete "What is the capital of France?" by continuing plausible-sounding text, not by answering "Paris." Instruction tuning teaches the model the format: question → answer.

**SFT on Instruction Data**

Instruction tuning formats data as conversations, then fine-tunes using the standard causal LM objective but computing loss only on the assistant's responses.

```python
from trl import SFTTrainer, DataCollatorForCompletionOnlyLM

# TRL (Transformer Reinforcement Learning) library from HF
# pip install trl

def format_instruction(sample):
    return f"""### Instruction:
{sample['instruction']}

### Response:
{sample['output']}"""

# DataCollatorForCompletionOnlyLM masks the loss on the instruction,
# so the model only learns to predict the response tokens
response_template = "### Response:"
data_collator = DataCollatorForCompletionOnlyLM(
    response_template, tokenizer=tokenizer
)

trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    formatting_func=format_instruction,
    data_collator=data_collator,
    args=SFTConfig(
        output_dir="./sft_model",
        max_seq_length=2048,
        per_device_train_batch_size=4,
        gradient_accumulation_steps=4,
        fp16=True,
        num_train_epochs=3,
    ),
)

trainer.train()
```

---

### Chapter 22: Pushing to and Pulling from the Hugging Face Hub

The Hub is the central repository for models, datasets, and spaces. Knowing how to use it is essential.

```python
from huggingface_hub import HfApi, login

# Authenticate
login(token="your_hf_token")   # or set HF_TOKEN env variable

# Push model and tokenizer
model.push_to_hub("your-username/model-name")
tokenizer.push_to_hub("your-username/model-name")

# Push dataset
dataset.push_to_hub("your-username/dataset-name")

# Load anything from the Hub
model = AutoModel.from_pretrained("your-username/model-name")

# Download a specific file
from huggingface_hub import hf_hub_download
path = hf_hub_download(repo_id="bert-base-uncased", filename="config.json")

# Using accelerate for device mapping
from accelerate import Accelerator
accelerator = Accelerator(fp16=True)
model, optimizer, train_loader = accelerator.prepare(model, optimizer, train_loader)
# Works identically across CPU, single GPU, multi-GPU, TPU
```

---

### Chapter 23: Advanced Patterns and Best Practices

**Gradient Accumulation — Simulating Larger Batches**

If your GPU can only fit a batch size of 8 but you want the dynamics of batch size 32, accumulate gradients over 4 steps before each optimizer step.

```python
accumulation_steps = 4
optimizer.zero_grad()

for step, batch in enumerate(loader):
    output = model(batch)
    loss = criterion(output, targets) / accumulation_steps   # scale loss
    loss.backward()
    
    if (step + 1) % accumulation_steps == 0:
        optimizer.step()
        optimizer.zero_grad()
```

**Gradient Checkpointing — Trading Compute for Memory**

The standard forward pass stores all intermediate activations for the backward pass. For large models, this is most of the memory usage. Gradient checkpointing discards activations and recomputes them during the backward pass. Increases compute by ~33% but massively reduces memory.

```python
model.gradient_checkpointing_enable()   # HuggingFace models have this built-in
# or for custom models:
# use torch.utils.checkpoint.checkpoint() around blocks
```

**Custom Metrics and Evaluation**

```python
from evaluate import load

# The evaluate library has 100+ metrics
accuracy = load("accuracy")
f1 = load("f1")
bleu = load("bleu")
rouge = load("rouge")
bertscore = load("bertscore")

# Example: BLEU for translation
predictions = ["The cat sat on the mat."]
references = [["The cat is sitting on the mat.", "The cat sat on the mat."]]
score = bleu.compute(predictions=predictions, references=references)
```

**Model Inspection and Debugging**

```python
# Print architecture
print(model)

# Check parameter shapes
for name, param in model.named_parameters():
    print(f"{name}: {param.shape} | requires_grad: {param.requires_grad}")

# Check for NaN/Inf in gradients (common source of training instability)
for name, param in model.named_parameters():
    if param.grad is not None:
        if torch.isnan(param.grad).any() or torch.isinf(param.grad).any():
            print(f"Bad gradient in {name}")

# Hook to print intermediate activations
def print_hook(module, input, output):
    print(f"Layer: {module.__class__.__name__}, output shape: {output.shape}")

handle = model.layer1.register_forward_hook(print_hook)
model(x)
handle.remove()   # always clean up hooks
```

**Efficient Inference with torch.compile**

PyTorch 2.0+ can compile models to optimized kernels with a single line:

```python
model = torch.compile(model)   # compiles on first forward pass, then fast
# Supports most models. Falls back gracefully on unsupported ops.
```

**Writing a Clean, Reproducible Training Script**

```python
import torch
import random
import numpy as np

def set_seed(seed: int):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    # For full determinism (slower):
    # torch.backends.cudnn.deterministic = True
    # torch.backends.cudnn.benchmark = False

set_seed(42)
```

---

## PART VI: PUTTING IT ALL TOGETHER

---

### Chapter 24: End-to-End Project Walkthroughs

**Text Classification — Full Pipeline**

```python
from datasets import load_dataset
from transformers import (AutoTokenizer, AutoModelForSequenceClassification,
                          TrainingArguments, Trainer, DataCollatorWithPadding)
import numpy as np
from evaluate import load

# 1. Data
dataset = load_dataset("imdb")
tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")

def tokenize(examples):
    return tokenizer(examples["text"], truncation=True, max_length=512)

tokenized = dataset.map(tokenize, batched=True)
tokenized = tokenized.rename_column("label", "labels")
tokenized.set_format("torch", columns=["input_ids", "attention_mask", "labels"])

# 2. Model
model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=2
)

# 3. Metrics
accuracy = load("accuracy")
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    preds = np.argmax(logits, axis=-1)
    return accuracy.compute(predictions=preds, references=labels)

# 4. Training
args = TrainingArguments(
    output_dir="imdb_classifier",
    num_train_epochs=3,
    per_device_train_batch_size=32,
    per_device_eval_batch_size=64,
    learning_rate=2e-5,
    warmup_ratio=0.1,
    eval_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    fp16=True,
)

trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized["train"],
    eval_dataset=tokenized["test"],
    tokenizer=tokenizer,
    data_collator=DataCollatorWithPadding(tokenizer),
    compute_metrics=compute_metrics,
)

trainer.train()
```

**LLM Fine-Tuning with QLoRA**

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training
from trl import SFTTrainer, SFTConfig

# 1. Load model in 4-bit
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=bnb_config,
    device_map="auto",
)
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-2-7b-hf")
tokenizer.pad_token = tokenizer.eos_token

# 2. Prepare and wrap with LoRA
model = prepare_model_for_kbit_training(model)
lora_config = LoraConfig(
    r=64,
    lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(model, lora_config)

# 3. Train
trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    args=SFTConfig(
        output_dir="./llama_finetuned",
        num_train_epochs=3,
        per_device_train_batch_size=2,
        gradient_accumulation_steps=8,
        learning_rate=2e-4,
        bf16=True,
        max_seq_length=2048,
    ),
)
trainer.train()
```

---

### Chapter 25: The Mental Map — Understanding What You've Learned

The thread running through everything in this document is the same: representation learning. Every technique is about learning better ways to represent data so that patterns become easier to find.

**The Core Progression**

Tensors are the containers. Autograd makes them learnable. nn.Module gives structure. Loss functions define what "good" means. Optimizers search for good. These five things — tensors, autograd, modules, losses, optimizers — are all of PyTorch's conceptual core.

CNNs learn spatial hierarchies: pixels → edges → parts → objects. The translational invariance of convolution perfectly matches the structure of natural images.

RNNs learn temporal hierarchies: characters → words → phrases → meaning. The sequential recurrence naturally matches language's sequential structure. But the memory bottleneck limits long-range dependencies.

Transformers break the sequential constraint entirely. Every token directly attends to every other token. The O(n²) cost is worth the gain in expressivity and parallelism. Self-attention is the key operation: dynamic, input-dependent weighting of all positions.

Pretraining + fine-tuning is the dominant paradigm: train on huge unlabeled data to learn general representations, then adapt to specific tasks with small labeled datasets. BERT learns bidirectional context. GPT learns to predict the future. T5 learns to transform text into text.

PEFT makes fine-tuning affordable: instead of updating billions of parameters, update thousands. LoRA's low-rank decomposition is mathematically elegant and practically transformative.

**When to Use What**

For text classification, NER, extractive QA: BERT-family models (distilbert for speed, roberta/deberta for performance).

For text generation, instruction following, chat: GPT-family decoder models (GPT-2 small, LLaMA/Mistral for serious work).

For summarization, translation, text-to-text tasks: T5 or BART.

For image classification: ResNet for baseline, EfficientNet for efficiency, ViT for SOTA.

For fine-tuning with limited compute: always use LoRA. On a single consumer GPU, QLoRA lets you fine-tune models up to 13B parameters.

For a new task with limited data: transfer learning from ImageNet (vision) or HuggingFace pretrained models (NLP). Freeze the backbone, train only the head first, then optionally unfreeze with a very small learning rate.

**The Single Most Important Intuition**

Scale matters. Every major advance in deep learning has been enabled partly by scale: more data, more parameters, more compute. But scale alone isn't enough — the right architectural inductive biases matter. CNNs encode the bias that local spatial patterns matter and should be shared across locations. Transformers encode the bias that any position can directly influence any other. Getting the right bias for your data's structure is the fundamental design decision in building a neural network.

The rest is hyperparameter tuning, engineering, and iteration.