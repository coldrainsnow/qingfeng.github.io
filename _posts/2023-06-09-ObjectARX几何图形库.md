---
layout: post
title: "ObjectARX几何图形库"
date: 2023-06-09
excerpt: "简单了解下AcGe"
tags: [cpp]
comments: true

---

* toc
{:toc}


# 1.引言

这几天在做CAD二次开发，涉及到几何类，略有不解，后又发现ObjectARX开发指南，所以翻译下官方的

AcGe库提供很多用于表示2D和3D几何图形的类

# 2.AcGe库概述

AcGe 库包括一组广泛的类，用于表示常用的几何图形，例如点、线、曲线和曲面。它为几何图形提供了可供任何 Autodesk 应用程序使用的通用表示形式。该库是纯数学的;虽然它的类不直接处理数据库或图形，但它的许多类被 AcDb 和 AcGi 库使用。

AcGe 库提供简单和复杂的几何类。简单的线性代数类包括点、向量、矩阵、2D 和 3D 线性实体类以及平面实体类。复杂类包括曲线类（如样条图元）和曲面类（如 NURBS 曲面）。

类层次结构为 2D 和 3D 几何图形提供单独的类。这通过清楚地区分 2D 参数空间几何体和 3D 建模空间几何体来简化编程。由于这种区别，您不能在同一操作中无意中混合使用 2D 和 3D 实体。

该库包括许多基本类型，如 、 和 ，它们具有公共数据成员，可实现快速高效的访问。这些简单的类通常由其他库以及派生自 和 的 AcGe 类使用。AcGePoint3d，AcGeVector3d，AcGeMatrix3d，AcGeEntity2d，AcGeEntity3d

为派生自 和 的所有类提供了运行时类型检查。每个类都提供一个返回对象的类的函数和一个返回对象是否属于特定类（或从该类派生的类）的函数。AcGeEntity2d，AcGeEntity3d，type()，isKindOf()

如果两个图元的类型相同且表示相同的点集，则认为它们相等。仅当曲线和曲面的参数化相同时，它们才被视为相等。

AcGe 库的类层次结构如下所示：

![img](https://help.autodesk.com/cloudhelp/2023/CHS/OARXMAC-DevGuide/images/GUID-FAC81CA1-6875-4A76-9DED-4696F43C547C.png)

# 3.参数化几何

分为曲线Curves和表面Surfaces，由于我只用到了Curves，所以只说这个

## 3.1曲线Curves

AcGe 库中的曲线和曲面是参数化的。曲线是使用带有一个参数（如 f（u））的赋值器函数将实线的区间映射到 2D 或 3D 建模空间的结果。同样，曲面是使用基于两个参数（例如 f（， ））的赋值器函数从 2D 域到 3D 建模空间的映射。每个 2D 和 3D 曲线类都有一个返回参数间隔的函数。此函数有两种形式：第一种返回间隔;第二个返回曲线的间隔以及起点和终点。uvgetInterval()

**注意：**

如果间隔在任一方向上不受限制，则起点和终点没有意义。

### 3.1.1特性

曲线具有以下特征：

- 取向
- 周期性
- 关闭
- 平面度
- 长度

曲线的方向由其参数增加的方向决定。可以使用 or 函数反转曲线的方向。AcGeCurve2d::reverseParam()AcGeCurve3d::reverseParam()

有些曲线是周期性的，这意味着它们在一定间隔后重复。例如，圆的周期是 2pi。使用以下函数确定曲线是否为周期性曲线：

```
Adesk::Boolean
AcGeCurve2d::isPeriodic(double& period) const;
 
Adesk::Boolean
AcGeCurve3d::isPeriodic(double& period) const;
```

闭合曲线具有相同的起点和终点。曲线可以是闭合的，也可以是开放的。使用以下函数确定曲线是否闭合：

```
Adesk::Boolean
AcGeCurve2d::isClosed(
    const AcGeTol& tol = AcGeContext::gTol) const;
 
Adesk::Boolean
AcGeCurve3d::isClosed(
    const AcGeTol& tol = AcGeContext::gTol) const;
```

3D 曲线可以是平面曲线（意味着其所有点都位于同一平面中）或非平面曲线。使用此函数可确定 3D 曲线是否为平面曲线：

```
Adesk::Boolean
AcGeCurve3d::isPlanar(
    AcGePlane& plane, 
    const AcGeTol& tol = AcGeContext::gTol) const;
```

给定两个参数值，您可以使用以下函数获取这两个值之间的曲线长度：

```
double
AcGeCurve2d::length(
    double fromParam, 
    double toParam,
    double tol = AcGeContext::gTol.equalPoint()) const;
 
double
AcGeCurve3d::length(
    double fromParam, 
    double toParam,
    double tol = AcGeContext::gTol.equalPoint()) const;
```

可以使用 and 函数获取与给定参数值对应的模型空间点。如果应用程序经常执行评估，您可能会发现 和 类更有效（请参阅[特殊评估类](https://help.autodesk.com/view/OARXMAC/2023/CHS/?guid=GUID-4B1FA25F-05CF-4BDB-98DA-C85149B735D5#GUID-4B1FA25F-05CF-4BDB-98DA-C85149B735D5__WS4B0506698C46277A1908CA1105A303E554-7FF7)）。用于评估点的曲线函数如下：AcGeCurve2d::evalPoint()AcGeCurve3d::evalPoint()AcGePointOnCurve3dAcGePointOnCurve2d

```
AcGePoint2d
AcGeCurve2d::evalPoint(
    double param) const;
 
AcGePoint2d
AcGeCurve2d::evalPoint(
    double param, 
    int numDeriv,
    AcGeVector2dArray& derivArray) const;
 
AcGePoint3d
AcGeCurve3d::evalPoint(
    double param) const;
 
AcGePoint3d
AcGeCurve3d::evalPoint(
    double param, 
    int numDeriv,
    AcGeVector3dArray& derivArray) const;
```

# 4.参考

[AutoCAD 2023 for Mac Developer and ObjectARX 帮助 | Using the Geometry Library | Autodesk](https://help.autodesk.com/view/OARXMAC/2023/CHS/?guid=GUID-BCD4C0B3-CCAA-4C10-9ABC-394CF6D52CA5)