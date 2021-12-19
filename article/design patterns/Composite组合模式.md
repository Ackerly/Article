# Composite（组合模式）  
Composite（组合模式）属于结构型模式，是一种统一管理树形结构的抽象方式。将对象组合成树形结构以表示 “部分 - 整体” 的层次结构。Composite 使得用户对单个对象和组合对象的使用具有一致性。  

## 例子
**公司组织关系树**  
公司组织关系可能分为部门与人，其中人属于部门，有的人有下属，有的人没有下属。如果统一将部门、人抽象为组织节点，就可以方便的统计某个部门下有多少人、财务数据等等，而不用关心当前节点是部门还是人。  
**操作系统的文件夹与文件**  
操作系统的文件夹与文件也是典型的树状结构，为了方便递归出文件夹内文件数量或者文件总大小，最好设计的时候就将文件夹与文件抽象为文件，这样每个节点都拥有相同的方法添加、删除、查找子元素，而不需要关心当前节点是文件夹或是文件。  
**搭建平台的组件与容器**  
容器与组件的关系很小，用户常常认为容器也是一种组件，但搭建平台实现时，容器与组件稍有不同，不同之处在于容器可以嵌套子元素，而组件不可以。如果因此搭建平台就将组件分为容器与组件，会导致 API 割裂为两套，不利于组件开发者维护与用户理解，比较好的设计思路是将组件与容器统一看成组件，组件只是一种没有子元素的特殊容器，这样组件与容器就可以拥有相同的 API，统一理解与操作了
## 解释
组合是指多个对象虽然有一定差异，但共同组合成了一个树形结构，那么对象之间就一定存在 “部分 - 整体” 的关系，组合模式要求抽象一个对象 Component 作为统一操作模型，叶子结点与非叶子结点都实现了所有功能，即便是没有子元素的叶子结点，为了强调透明性，还是具备比如 getChildren 方法，只不过永远都返回 null。

![image](./../../assets/images/design%20patterns/Composite%20combination.png)  

Component 是组合中对象声明接口，一般会实现所有公共类的所有接口，还要提供一个接口管理其子组件。Leaf 表示叶子结点，没有子结点，相应的 Composite 就是有子结点的节点。  
组合模式就是将树状结构中所有节点统一抽象了，不需要关心叶子结点与非叶子结点的差异，而可以通过组合模式的抽象屏蔽掉这些差异，统一处理。  
``` 
// 统一的抽象
class Component {
  // 添加子元素
  public add() {}
  // 获取名称
  public getName() {}
  // 获取子元素
  public getChildren() {}
}

// 非叶子结点
class Composite extends Component {
  public add(component: Component) {
    this.children.push(component)
  }

  public getName() {
    return this.name
  }

  public getChildren() {
    return this.children
  }
}

// 叶子结点
class Leaf extends Component {
  public add(component: Component) {
    throw Error('叶子结点无法添加元素')
  }

  public getName() {
    return this.name
  }

  public getChildren() {
    return null
  }
}
```
## 缺点
组合模式是针对树状结构这个特定场景的统一抽象方案，对降低系统复杂度有很重要的意义

参考:  
[Composite（组合模式）](https://github.com/ascoders/weekly/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/174.%E7%B2%BE%E8%AF%BB%E3%80%8A%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20Composite%20%E7%BB%84%E5%90%88%E6%A8%A1%E5%BC%8F%E3%80%8B.md)
