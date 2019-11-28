
# 代理
为其他对象提供一种代理用以控制为这个对象的访问；代理模式通用类图如下
```mermaid
classDiagram
class Subject{
    + request()
}
class Proxy{ }
class RelSubject { }
RelSubject<--Proxy
Subject<|--RelSubject::implements
Subject<|--Proxy:implements
```
# 普通代理
只能访问代理角色而不能访问真实角色


# 强制代理

# 动态代理


    学习至《设计模式之禅第二版》-- 秦小波