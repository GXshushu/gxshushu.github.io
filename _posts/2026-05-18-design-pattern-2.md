---
title: 设计模式 - 策略模式
date: 2026-05-18 15:43:00 +0800
categories: [设计模式]
tags: [设计模式]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true

---

# 策略模式
## 定义
策略模式通过封装一组算法，并将每个算法封装起来，使它们可以相互替换，且算法的变化不会影响使用算法的客户。

## 结构
策略模式包含以下几个主要角色：

- 抽象策略（Strategy）类：定义了所有支持的算法的公共接口。
- 具体策略（Concrete Strategy）类：实现了抽象策略定义的接口，提供具体的算法实现。
- 环境（Context）类：持有一个策略类的引用，用于与策略模式的算法交互。
  
## 案例
```java
// 策略接口  
public interface Strategy {  
    public int doOperation(int num1, int num2);  
}  
  
// 具体策略A：加法  
public class OperationAdd implements Strategy {  
    @Override  
    public int doOperation(int num1, int num2) {  
        return num1 + num2;  
    }  
}  
  
// 具体策略B：减法  
public class OperationSubtract implements Strategy {  
    @Override  
    public int doOperation(int num1, int num2) {  
        return num1 - num2;  
    }  
}  
  
// 上下文类  
public class Context {  
    private Strategy strategy;  
      
    public Context(Strategy strategy) {  
        this.strategy = strategy;  
    }  
      
    public int executeStrategy(int num1, int num2) {  
        return strategy.doOperation(num1, num2);  
    }  
}  
  
// 客户端代码  
public class StrategyPatternDemo {  
    public static void main(String[] args) {  
        Context context = new Context(new OperationAdd()); // 使用加法策略  
        System.out.println("10 + 5 = " + context.executeStrategy(10, 5));  
          
        context = new Context(new OperationSubtract()); // 切换为减法策略  
        System.out.println("10 - 5 = " + context.executeStrategy(10, 5));  
    }  
}
```

## 应用场景
策略模式在以下几种情况下特别有用：

- 需要动态地在几种算法中选择一种时，可将每个算法封装到策略类中。
- 一个类定义了多种行为，并且这些行为在这个类的操作中以多个条件语句的形式出现，可将每个条件分支移入它们各自的策略类中以代替这些条件语句。
- 系统中各算法彼此完全独立，且要求对客户隐藏具体算法的实现细节时。
- 系统要求使用算法的客户不应该知道其操作的数据时，可使用策略模式来隐藏与算法相关的数据结构。
- 多个类只区别在表现行为不同，可以使用策略模式，在运行时动态选择具体要执行的行为。
- 通过使用策略模式，你可以使代码更加模块化，易于扩展和维护。
