# Matplotlib Cheat Sheet

## Table of Contents
1. [Basic Plot](#basic-plot)
2. [Figure and Axes](#figure-and-axes)
3. [Line Plots](#line-plots)
4. [Scatter Plots](#scatter-plots)
5. [Bar Plots](#bar-plots)
6. [Histograms](#histograms)
7. [Subplots](#subplots)
8. [Customization](#customization)
9. [Saving Plots](#saving-plots)
10. [3D Plots](#3d-plots)

## Basic Plot
```python
import matplotlib.pyplot as plt

plt.plot(x, y)
plt.show()
```

## Figure and Axes
```python
fig, ax = plt.subplots()
ax.plot(x, y)
plt.show()
```

## Line Plots
```python
plt.plot(x, y, color='red', linestyle='--', marker='o')
plt.title("Title")
plt.xlabel("X-axis")
plt.ylabel("Y-axis")
plt.show()
```

## Scatter Plots
```python
plt.scatter(x, y, c=colors, s=sizes, alpha=0.5)
plt.colorbar()
plt.show()
```

## Bar Plots
```python
plt.bar(x, height, width=0.8, bottom=None, align='center')
plt.show()
```

## Histograms
```python
plt.hist(data, bins=10, range=None, density=False)
plt.show()
```

## Subplots
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 5))
ax1.plot(x1, y1)
ax2.plot(x2, y2)
plt.tight_layout()
plt.show()
```

## Customization
```python
plt.style.use('seaborn')
plt.grid(True)
plt.legend()
plt.xticks(rotation=45)
plt.yticks(rotation=45)
```

## Saving Plots
```python
plt.savefig('filename.png', dpi=300, bbox_inches='tight')
```

## 3D Plots
```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.plot_surface(X, Y, Z)
plt.show()
```

For more detailed information, refer to the [official Matplotlib documentation](https://matplotlib.org/stable/contents.html).
