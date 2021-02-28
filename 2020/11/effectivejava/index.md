# EffectiveJava


# Effective Java

## 对于所有对象都通用的方法

### 覆盖equals时遵循的约定

满足条件：

* 类的每个实例本质上都是相同的
* 类没有必要提供逻辑相等的测试功能：例java.util.regx.Pattern
* 父类覆盖了equals方法，父类的行为对子类也是合适的

**覆盖equals的约定**

* 自发性（reflexive）
* 对称性（symmetric）
* 传递性（transitive）
* 一致性（consistent）
* 对于 `x.equals(null)` 必须返回false


