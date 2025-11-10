# 图形变换系统 - 程序文档

## 目录
1. [程序概述](#程序概述)
2. [设计思路](#设计思路)
3. [类结构设计](#类结构设计)
4. [关键功能实现](#关键功能实现)
5. [关键代码解析](#关键代码解析)
6. [使用说明](#使用说明)

---

## 程序概述

本程序是一个基于EGE图形库的2D图形变换系统，实现了多种几何图形（点、线段、圆、矩形、三角形）的旋转、镜像、缩放、移动等变换操作。程序采用面向对象设计，使用继承和多态实现代码复用和扩展性。

### 主要特性
- **多种图形类型**：点、线段、圆、矩形、三角形
- **变换操作**：旋转、镜像、缩放、移动
- **可视化演示**：静态演示和交互式演示
- **虚线显示**：镜像时显示虚线作为镜像轴

---

## 设计思路

### 1. 面向对象设计

采用经典的面向对象设计模式：
- **抽象基类**：`Shape` 作为所有图形的基类，定义统一的接口
- **具体实现类**：`PointShape`、`LineSegment`、`Circle`、`Rect`、`Triangle` 继承自 `Shape`
- **辅助类**：`Point` 类封装点的坐标和基本变换操作

### 2. 变换操作的数学原理

#### 旋转（Rotate）
- **原理**：使用旋转矩阵进行坐标变换
- **公式**：
  ```
  x' = (x - cx) * cos(θ) - (y - cy) * sin(θ) + cx
  y' = (x - cx) * sin(θ) + (y - cy) * cos(θ) + cy
  ```
  其中 `(cx, cy)` 是旋转中心，`θ` 是旋转角度（弧度）

#### 镜像（Mirror）
- **水平镜像**：关于水平轴 `y = axis` 镜像
  ```
  y' = 2 * axis - y
  ```
- **垂直镜像**：关于垂直轴 `x = axis` 镜像
  ```
  x' = 2 * axis - x
  ```

#### 缩放（Scale）
- **原理**：相对于缩放中心进行缩放
  ```
  x' = (x - cx) * factor + cx
  y' = (y - cy) * factor + cy
  ```

#### 移动（Move）
- **原理**：简单的坐标平移
  ```
  x' = x + dx
  y' = y + dy
  ```

### 3. 程序架构

```
程序架构
├── 核心类库
│   ├── Shape (抽象基类)
│   ├── Point (点类)
│   ├── PointShape (点图形)
│   ├── LineSegment (线段)
│   ├── Circle (圆)
│   ├── Rect (矩形)
│   └── Triangle (三角形)
├── 演示程序
│   ├── main.cpp (基础演示)
│   ├── transform_demo.cpp (静态变换演示)
│   └── transform_interactive.cpp (交互式演示)
└── 编译脚本
    └── compile.bat
```

---

## 类结构设计

### 1. Shape 抽象基类

```cpp
class Shape {
protected:
    static int objectCount;  // 对象计数器

public:
    Shape() { ++objectCount; }
    virtual ~Shape() { --objectCount; }
    
    // 纯虚函数 - 所有派生类必须实现
    virtual double area() const = 0;
    virtual double perimeter() const = 0;
    virtual void rotate(double angle) = 0;
    virtual void mirror(bool horizontal = true) = 0;
    virtual void scale(double factor) = 0;
    virtual void move(double dx, double dy) = 0;
    virtual void draw(ege::PIMAGE img, int color) const = 0;
    
    static int getObjectCount() { return objectCount; }
};
```

**设计要点**：
- 使用纯虚函数定义统一接口
- 静态成员变量统计对象数量
- 虚析构函数确保正确的内存管理

### 2. Point 类

```cpp
class Point {
private:
    double x, y;

public:
    Point(double x = 0.0, double y = 0.0) : x(x), y(y) {}
    
    // Getter和Setter
    double getX() const { return x; }
    double getY() const { return y; }
    void setX(double x) { this->x = x; }
    void setY(double y) { this->y = y; }
    
    // 变换方法
    void rotate(double angle, double centerX = 0, double centerY = 0);
    void mirror(bool horizontal = true, double axis = 0);
    void scale(double factor, double centerX = 0, double centerY = 0);
    void move(double dx, double dy);
};
```

**设计要点**：
- 封装点的坐标和基本变换操作
- 所有变换方法都支持指定变换中心
- 为其他图形类提供基础变换能力

### 3. 具体图形类

所有图形类都继承自 `Shape`，实现以下功能：
- **面积和周长计算**：根据图形特性实现
- **变换操作**：调用 `Point` 类的变换方法
- **绘制功能**：使用EGE库绘制图形

---

## 关键功能实现

### 1. 旋转功能实现

#### Point 类的旋转实现

```cpp
void Point::rotate(double angle, double centerX, double centerY) {
    // 平移到中心点
    double dx = x - centerX;
    double dy = y - centerY;
    
    // 转换为弧度
    double rad = angle * M_PI / 180.0;
    double cosA = cos(rad);
    double sinA = sin(rad);
    
    // 旋转
    double newX = dx * cosA - dy * sinA;
    double newY = dx * sinA + dy * cosA;
    
    // 平移回去
    x = newX + centerX;
    y = newY + centerY;
}
```

**实现要点**：
1. 先平移到旋转中心
2. 应用旋转矩阵
3. 平移回原位置

#### Triangle 类的旋转实现

```cpp
void Triangle::rotate(double angle) {
    Point center = getCenter();  // 获取重心
    vertex1.rotate(angle, center.getX(), center.getY());
    vertex2.rotate(angle, center.getX(), center.getY());
    vertex3.rotate(angle, center.getX(), center.getY());
}
```

**实现要点**：
- 获取三角形的重心作为旋转中心
- 对每个顶点应用旋转变换

### 2. 镜像功能实现

#### Point 类的镜像实现

```cpp
void Point::mirror(bool horizontal, double axis) {
    if (horizontal) {
        // 关于水平轴镜像（y = axis）
        y = 2 * axis - y;
    } else {
        // 关于垂直轴镜像（x = axis）
        x = 2 * axis - x;
    }
}
```

#### Rect 类的镜像实现（关键：处理四个顶点）

```cpp
void Rect::mirror(bool horizontal) {
    Point center = getCenter();
    // 获取所有四个顶点
    Point tr = getTopRight();
    Point bl = getBottomLeft();
    
    // 镜像所有四个顶点（创建副本以避免修改成员变量）
    Point tl = topLeft;
    Point br = bottomRight;
    if (horizontal) {
        tl.mirror(true, center.getY());
        tr.mirror(true, center.getY());
        br.mirror(true, center.getY());
        bl.mirror(true, center.getY());
    } else {
        tl.mirror(false, center.getX());
        tr.mirror(false, center.getX());
        br.mirror(false, center.getX());
        bl.mirror(false, center.getX());
    }
    
    // 重新计算topLeft和bottomRight
    double minX = std::min({tl.getX(), tr.getX(), br.getX(), bl.getX()});
    double maxX = std::max({tl.getX(), tr.getX(), br.getX(), bl.getX()});
    double minY = std::min({tl.getY(), tr.getY(), br.getY(), bl.getY()});
    double maxY = std::max({tl.getY(), tr.getY(), br.getY(), bl.getY()});
    
    // 更新topLeft和bottomRight
    topLeft = Point(minX, minY);
    bottomRight = Point(maxX, maxY);
}
```

**实现要点**：
- 矩形只存储两个对角点，但镜像需要处理所有四个顶点
- 镜像后重新计算边界框，更新存储的对角点

### 3. 缩放功能实现

#### Point 类的缩放实现

```cpp
void Point::scale(double factor, double centerX, double centerY) {
    // 平移到中心点
    double dx = x - centerX;
    double dy = y - centerY;
    
    // 缩放
    dx *= factor;
    dy *= factor;
    
    // 平移回去
    x = dx + centerX;
    y = dy + centerY;
}
```

### 4. 虚线绘制功能

```cpp
void drawDashedLine(ege::PIMAGE img, int x1, int y1, int x2, int y2, 
                    int color, int dashLength = 15, int gapLength = 8) {
    setcolor(color, img);
    
    double dx = x2 - x1;
    double dy = y2 - y1;
    double length = sqrt(dx * dx + dy * dy);
    
    if (length < 1) return;
    
    double unitX = dx / length;  // 单位方向向量
    double unitY = dy / length;
    
    double currentX = x1;
    double currentY = y1;
    double traveled = 0;
    bool drawing = true;  // 是否正在绘制线段
    
    while (traveled < length) {
        double segmentLength = drawing ? dashLength : gapLength;
        if (traveled + segmentLength > length) {
            segmentLength = length - traveled;
        }
        
        if (drawing) {
            double endX = currentX + unitX * segmentLength;
            double endY = currentY + unitY * segmentLength;
            // 绘制更粗的虚线
            for (int i = -1; i <= 1; i++) {
                line((int)currentX, (int)(currentY + i), 
                     (int)endX, (int)(endY + i), img);
            }
            currentX = endX;
            currentY = endY;
        } else {
            currentX += unitX * segmentLength;
            currentY += unitY * segmentLength;
        }
        
        traveled += segmentLength;
        drawing = !drawing;  // 切换绘制/空白状态
    }
}
```

**实现要点**：
- 使用单位方向向量沿路径移动
- 交替绘制线段和空白，形成虚线效果
- 使用多线绘制增强可见性

---

## 关键代码解析

### 1. 多态性的应用

```cpp
// Shape.h
class Shape {
public:
    virtual void rotate(double angle) = 0;
    virtual void draw(ege::PIMAGE img, int color) const = 0;
};

// 使用多态
Shape* shapes[] = {
    new Triangle(...),
    new Rect(...),
    new Circle(...)
};

// 统一调用，但执行不同的实现
for (int i = 0; i < 3; i++) {
    shapes[i]->rotate(45);  // 每个图形用自己的旋转方法
    shapes[i]->draw(img, RED);  // 每个图形用自己的绘制方法
}
```

**解析**：
- 基类指针可以指向任何派生类对象
- 运行时根据实际对象类型调用对应的方法
- 实现了"一个接口，多种实现"

### 2. 对象计数器的实现

```cpp
// Shape.h
class Shape {
protected:
    static int objectCount;  // 静态成员变量

public:
    Shape() { ++objectCount; }
    virtual ~Shape() { --objectCount; }
    static int getObjectCount() { return objectCount; }
};

// Shape.cpp
int Shape::objectCount = 0;  // 静态成员初始化
```

**解析**：
- `static` 成员变量属于类，不属于对象实例
- 所有 `Shape` 对象共享同一个计数器
- 构造函数中递增，析构函数中递减
- 可以随时查询当前存在的对象数量

### 3. 矩形旋转的边界框重计算

```cpp
void Rect::rotate(double angle) {
    Point center = getCenter();
    // 获取所有四个顶点
    Point tr = getTopRight();
    Point bl = getBottomLeft();
    
    // 旋转所有四个顶点（创建副本以避免修改成员变量）
    Point tl = topLeft;
    Point br = bottomRight;
    tl.rotate(angle, center.getX(), center.getY());
    tr.rotate(angle, center.getX(), center.getY());
    br.rotate(angle, center.getX(), center.getY());
    bl.rotate(angle, center.getX(), center.getY());
    
    // 重新计算topLeft和bottomRight（找到旋转后的最小和最大边界框）
    double minX = std::min({tl.getX(), tr.getX(), br.getX(), bl.getX()});
    double maxX = std::max({tl.getX(), tr.getX(), br.getX(), bl.getX()});
    double minY = std::min({tl.getY(), tr.getY(), br.getY(), bl.getY()});
    double maxY = std::max({tl.getY(), tr.getY(), br.getY(), bl.getY()});
    
    // 更新topLeft和bottomRight
    topLeft = Point(minX, minY);
    bottomRight = Point(maxX, maxY);
}
```

**解析**：
- **问题**：矩形只存储两个对角点，但旋转后需要处理四个顶点
- **解决**：获取所有四个顶点，分别旋转，然后重新计算边界框
- **关键**：使用 `std::min` 和 `std::max` 找到旋转后的最小外接矩形
- **注意**：旋转后的矩形不再是轴对齐的，但为了保持数据结构，用边界框表示

### 4. 交互式程序的主循环

```cpp
bool running = true;
while (running) {
    // 处理键盘输入
    if (kbhit()) {
        int key = getch();
        
        if (key == 27) {  // ESC
            running = false;
        } else if (key == 'r' || key == 'R') {
            // 旋转
            triTransformed->rotate(15);
            rectTransformed->rotate(15);
            circleTransformed->rotate(15);
        } else if (key == 'm' || key == 'M') {
            // 镜像
            triTransformed->mirror(false);
            rectTransformed->mirror(false);
            circleTransformed->mirror(false);
        }
        // ... 其他按键处理
    }
    
    // 绘制场景
    drawScene(img, triOriginal, rectOriginal, circleOriginal,
              triTransformed, rectTransformed, circleTransformed,
              mirrorAxisX, mirrorAxisY, showHorizontalMirror, showVerticalMirror);
    
    putimage(0, 0, img);
    delay_fps(60);  // 60 FPS
}
```

**解析**：
- **事件驱动**：使用 `kbhit()` 检测键盘输入
- **实时更新**：每次循环都重新绘制场景
- **帧率控制**：使用 `delay_fps(60)` 控制帧率
- **状态管理**：通过布尔变量控制显示模式

### 5. 镜像位置的精确计算

```cpp
// 计算镜像后的位置：关于镜像轴镜像
// 镜像后的x坐标 = 2 * mirrorAxisX - originalX
double triNewX = 2 * mirrorAxisX - triCenter.getX();
double rectNewX = 2 * mirrorAxisX - rectCenter.getX();
double circleNewX = 2 * mirrorAxisX - circleCenter.getX();

// 移动图形到镜像位置
triMirrored->move(triNewX - triCenter.getX(), 0);
rectMirrored->move(rectNewX - rectCenter.getX(), 0);
circleMirrored->move(circleNewX - circleCenter.getX(), 0);

// 关于垂直轴镜像（false表示垂直轴）
triMirrored->mirror(false);
rectMirrored->mirror(false);
circleMirrored->mirror(false);
```

**解析**：
- **两步操作**：先移动到镜像位置，再进行镜像变换
- **数学原理**：关于轴 `x = a` 镜像，点 `(x, y)` 变为 `(2a - x, y)`
- **实现方式**：
  1. 计算镜像后的中心位置
  2. 移动图形到镜像位置
  3. 关于自己的中心进行镜像（完成最终镜像）

---

## 使用说明

### 编译程序

```bash
# 编译所有程序
compile.bat

# 编译特定程序
compile.bat main          # 编译基础演示
compile.bat transform     # 编译静态变换演示
compile.bat interactive   # 编译交互式演示
```

### 运行程序

#### 1. main.exe - 基础演示
- 展示所有图形类型
- 演示基本变换操作
- 按任意键继续

#### 2. transform_demo.exe - 静态变换演示
- **第一部分**：旋转演示（45度）
- **第二部分**：水平镜像演示（带虚线）
- **第三部分**：垂直镜像演示（带虚线）
- 按任意键切换场景

#### 3. transform_interactive.exe - 交互式演示
**控制键**：
- `1` - 显示旋转演示
- `2` - 显示水平镜像演示（带虚线）
- `3` - 显示垂直镜像演示（带虚线）
- `R` - 旋转15度
- `M` - 镜像
- `S` - 放大1.1倍
- `X` - 缩小0.9倍
- `方向键` - 移动图形
- `ESC` - 退出

### 程序结构

```
ege/
├── Shape.h/cpp          # 抽象基类
├── Point.h/cpp          # 点类
├── PointShape.h/cpp     # 点图形类
├── LineSegment.h/cpp    # 线段类
├── Circle.h/cpp         # 圆类
├── Rectangle.h/cpp      # 矩形类
├── Triangle.h/cpp       # 三角形类
├── main.cpp             # 基础演示程序
├── transform_demo.cpp   # 静态变换演示
├── transform_interactive.cpp  # 交互式演示
└── compile.bat          # 编译脚本
```

---

## 总结

本程序展示了面向对象设计在图形处理中的应用，主要特点：

1. **良好的封装**：每个类负责自己的数据和操作
2. **多态性**：统一的接口，不同的实现
3. **代码复用**：通过继承减少重复代码
4. **可扩展性**：易于添加新的图形类型
5. **数学基础**：所有变换操作都有明确的数学原理

程序不仅实现了基本的图形变换功能，还提供了直观的可视化演示，帮助理解各种变换操作的原理和效果。

