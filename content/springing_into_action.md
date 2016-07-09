# 1 Springing into action

## 1.1 简化Java开发

Spring主要通过下面4种关键策略来简化Java开发：

* 使用POJOs最小侵入性开发
* 通过DI和面向接口编程实现松耦合
* 通过切面和约定实现声明式编程
* 通过切面和模板消除样板代码

**POJOs**

POJO（Plan Old Java Object）即普通的Java对象，许多框架需要你继承它们的类或者实现它们的接口来使用，而Spring则尽可能避免让你的代码和它的API混在一起，顶多使用Spring注解，但类其实还是POJO。

**DI**

DI（Dependency Injection）即依赖注入，稍微复杂点的应用都至少包含两个以上互相协作的类，传统上，每个对象自己负责获取它需要的对象的引用（它的依赖），这就导致了紧耦合和难测试的代码。

看下面这个Knight类：

```
package com.springinaction.knights;

public class DamselRescuingKnight implements Knight {

  private RescueDamselQuest quest;
  
  public DamselRescuingKnight() {
    this.quest = new RescueDamselQuest();
  }
  
  public void embarkOnQuest() {
    quest.embark();
  }
}
```

DamselRescuingKnight在构造器中自己创建了一个RescueDamselQuest，如果想换个quest，就不好办了，而且也难测试。

使用依赖注入，依赖由系统中的第三方在对象创建的时候注入进去，如下图所示：

![依赖注入](QQ20160709-1.png)

看下面的BraveKnight类：

```
package com.springinaction.knights;

public class BraveKnight implements Knight {
  
  private Quest quest;
  
  public BraveKnight(Quest quest) {
    this.quest = quest;
  }
  
  public void embarkOnQuest() {
    quest.embark();
  }
}
```

BraveKnight不自己创建quest，而是在构造时通过参数传进来，即构造器注入。而且Quest是一个接口，所以可以传任意的实现进来，BraveKnight不与任何特定的Quest实现耦合，这就是DI的松耦合，BraveKnight什么都不用管，就可以轻松换一种Quest实现，这样就可以使用Mock对象来测试了：

```
package com.springinaction.knights;
import static org.mockito.Mockito.*;
import org.junit.Test;

public class BraveKnightTest {

  @Test
  public void knightShouldEmbarkOnQuest() {
    Quest mockQuest = mock(Quest.class);
    BraveKnight knight = new BraveKnight(mockQuest);
    knight.embarkOnQuest();
    verify(mockQuest, times(1)).embark();
  }
}
```

Mock测试框架Mockito创建了一个Quest接口的mock实现，注入到BraveKnight实例中，在调用了embarkOnQuest()方法之后，请求Mockito来验证Quest对象的embark()方法被恰好调用了一次。

