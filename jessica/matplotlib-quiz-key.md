# Matplotlib Interview Quiz - Answer Key

1. Matplotlib is a plotting library for Python. Its primary use is creating static, animated, and interactive visualizations in Python.

2. You can import Matplotlib using:
   ```python
   import matplotlib.pyplot as plt
   ```

3. `plt.show()` displays the plot in a new window or inline in a Jupyter notebook, while `plt.savefig()` saves the plot to a file.

4. A simple line plot can be created using:
   ```python
   plt.plot(x, y)
   plt.show()
   ```

5. `fig, ax = plt.subplots()` creates a new figure and a set of subplots. It returns a tuple containing the figure and axes object(s).

6. You can add a title to a plot using:
   ```python
   plt.title("Your Title Here")
   ```

7. `plt.plot()` is used for line plots, while `plt.scatter()` is used for scatter plots (individual points).

8. You can customize the color and style of a line using:
   ```python
   plt.plot(x, y, color='red', linestyle='--', linewidth=2)
   ```

9. `plt.legend()` adds a legend to the plot. You can use it by adding labels to your plots:
   ```python
   plt.plot(x, y, label='Line 1')
   plt.legend()
   ```

10. You can create a histogram using:
    ```python
    plt.hist(data, bins=10)
    ```

11. `plt.subplot()` is used to create a single subplot, while `plt.subplots()` can create a grid of subplots and returns figure and axes objects.

12. You can add text annotations using:
    ```python
    plt.text(x, y, "Your annotation")
    ```

13. `plt.tight_layout()` automatically adjusts subplot parameters to give specified padding between subplots.

14. You can create a bar plot using:
    ```python
    plt.bar(x, height)
    ```

15. `plt.plot()` uses the current axes (or creates one if it doesn't exist), while `ax.plot()` uses a specific axes object.

16. You can customize tick marks and labels using:
    ```python
    plt.xticks([0, 1, 2], ['A', 'B', 'C'], rotation=45)
    plt.yticks(np.arange(0, 1, 0.2))
    ```

17. `plt.colorbar()` adds a colorbar to the plot. It's often used with plots that use color to represent a variable, like heatmaps or scatter plots with colored points.

18. To create a 3D plot:
    ```python
    from mpl_toolkits.mplot3d import Axes3D
    fig = plt.figure()
    ax = fig.add_subplot(111, projection='3d')
    ax.plot(xs, ys, zs)
    ```

19. `plt.imshow()` is used to display data as an image (good for matrices), while `plt.pcolormesh()` is used for pseudocolor plots of 2D arrays (better for data on grids).

20. You can save a high-resolution figure using:
    ```python
    plt.savefig('filename.png', dpi=300, bbox_inches='tight')
    ```
