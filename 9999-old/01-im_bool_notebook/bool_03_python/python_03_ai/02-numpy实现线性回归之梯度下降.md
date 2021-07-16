[toc]

## 1,  线性回归之梯度下降步骤

### 1.1, 计算损失函数

### 1,2, 计算梯度

### 1.3, 根据梯度更新w, b

### 1.4, 循环执行上面三步，逐步优化



## 2, numpy实现线性回归之梯度下降

```python
# Author: 不二
# Date  : 12/3/2019-8:16 PM
# desc  : 线性回归问题(使用numpy直接实现的线性回归)

import numpy as np

# 1，计算损失函数
# y = wx + b
def compute_error_for_line_given_points(b, w, points):
    totalError = 0
    for i in range(0, len(points)):
        x = points[i, 0]
        y = points[i, 1]
        # compute mean-squared-error, 计算均方差误差
        totalError += (y - (w*x + b)) ** 2
    # average loss for each point
    return totalError/float(len(points))

# 2, 计算梯度 3, 根据梯度更新w，b
# 遍历一次数据集，也就是更新一次w, b
def step_gredient(b_current, w_current, points, learningRate):
    b_gradient = 0
    w_gradient = 0
    N = float(len(points))
    for i in range(0, len(points)):
        x = points[i, 0]
        y = points[i, 1]
        # 损失函数对于b的斜率：grad_b = 2(wx+b-y)
        # 下面除以N表示平均，以免太大
        b_gradient += (2/N) * ((w_current * x + b_current) - y)
        w_gradient += (2/N) * x * ((w_current * x + b_current) - y)
    # update w', b'
    new_b = b_current - (learningRate * b_gradient)
    new_w = w_current - (learningRate * w_gradient)
    return [new_b, new_w]

# 4，循环执行，逐渐优化
def gradient_descent_runner(points, starting_b, starting_w, learnint_rate, num_iterations):
    b = starting_b
    w = starting_w
    for i in range(num_iterations):
        b, w = step_gredient(b, w, np.array(points), learnint_rate)
    return [b, w]

def run(num_iterations):
    points = np.genfromtxt("./data/data.csv", delimiter=",")
    learning_rate = 0.0001
    initial_b = 0
    initial_w = 0
    num_iterations = num_iterations
    print("开始前：b = {0}, w = {1}, error = {2}".format(initial_b, initial_w, compute_error_for_line_given_points(initial_b, initial_w, points)))
    [b, w] = gradient_descent_runner(points, initial_b, initial_w, learning_rate, num_iterations)
    print("开始前：b = {0}, w = {1}, error = {2}".format(b, w, compute_error_for_line_given_points(b, w, points)))

if __name__ == '__main__':
    run(1000)
```



