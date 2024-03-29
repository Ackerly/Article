# Flyweight享元模式
Flyweight（享元模式）属于结构型模式，是一种共享对象的设计模式。运用共享技术有效地支持大量细粒度的对象。

## 举例
**富文本编辑器的字母对象**  
富文本编辑器在英文环境下，其中的文本由大量字母组成，为了便于做统一的格式化、计算等处理，需要将每个字母都存储为对象，但这样存储的代价太大了。  
已知英文字母一共 26 个，所以文档中存在大量重复使用的字母，而每个字母除了位置信息外，其它信息都是相同且只读的，那么有办法降低富文本场景巨大的字母对象数量吗？
**网盘存储**  
当我们上传一部电影时，有时候几十 GB 的内容不到一秒就上传完了，这是网盘提示你，“已采用极速技术秒传”，你会不会心生疑惑，这么厉害的技术为什么不能每次都生效？  
网盘存储时，同一部电影可能都会存放在不同用户的不同文件夹中，而且电影文件又特别巨大，和富文本类似，电影文件也只有存放位置是不同的，而其余内容都特别巨大且只读，有什么办法能优化存储呢？  
**大型多人游戏**  
玩多人游戏时，为了防止外挂，一般对象的创建与计算是在服务器完成的，那如何保证一个玩家拾取物品后，另一个玩家看到的物品会消失？  
虽然在不同客户端之间，游戏对象是相互独立的，但在一局游戏中，所有玩家的对象在服务器是共享的。  
## 解释
“共享” 就是享元模式的精髓，将那些大量的，具有很多内部状态而外部状态很少的对象进行共享，就是享元模式的使用方式。  
共享技术可以理解为缓存，当一个对象创建后，再次访问相同对象时，就不再创建新的对象了，而只有在访问没有被缓存过的对象时，才创建新对象，并立即缓存起来。  
这样做可以有效支持大量细粒度的对象，在富文本例子中，无数的字母就是大量细粒度对象，在网盘存储中，电影文件就是大量细粒度对象，在大型多人游戏中，每局游戏内存在大量细粒度对象。  
这些细粒度对象都拥有相同的特征：  
- 量特别大，这个很容易理解。
- 具有大量内部状态，且不随着客户端的不同而改变。
    - 富文本的字母，不因为展示到不同语句中而发生变化，变化的只有状态；电影文件，不因为放在不同用户的文件夹中而对电影内容产生变化，变化的只有属于哪些用户，放在哪些文件夹里；多人游戏中，同一把武器对象，不因为有多个人的电脑独立运行而拥有更多的弹药，变化的只有在哪些客户端被访问
- 具有少量外部状态，甚至没有外部状态。在上面已经解释了，字母的位置、电影的位置、游戏对象的客户端都是外部状态，这些外部状态相比于其内部状态来说，大小微乎其微，且方便分离存储。

尤其是对于网盘的场景，承诺给用户 2 TB 的存储空间，这个用户看到其他人分享了 100 个电影，就点击 “下载到我的网盘”，此时虽然占用了自己 1 TB 的网盘空间，但实际上网盘运营商并没有增加 1 TB 的存储空间，实际可能增加了 1kb 的存储空间，记录了存储位置，并不占用空间的内容  
享元模式的价值，对网盘公司来说，价值巨大，对用户来说，没有价值。所以享元模式的价值体现在全局，比如对整个富文本编辑器来说，减少了巨量字母对象数量，但对于每一个字母对象而言，并没有任何优化。

![image](./../../assets/images/design%20patterns/Flyweight.png)  

- Flyweight: 共享接口，通过这个接口可以操作对象的外部状态。
- ConcreteFlyweight: 实现 Flyweight 接口的对象，这个对象是可被共享的。
- UnsharedConcreteFlyweight: 不被共享的对象，因为在享元模式中，实际上并不是所有对象都可以被共享。
- FlyweightFactory: 创建并管理 Flyweight 对象，通过其返回的 Flyweight 对象，如果已创建，则会返回之前创建的那个，没有的话才会创建一个新的。
- Client: 使用 Flyweight 的客户端。

``` 
class FlyweightFactory {
  public getFlyWeight(key) {
    if (this.flyweight[key]) {
      return this.flyweight[key]
    }

    const flyweight = new Flyweight()
    this.flyweight[key] = flyweight
    return flyweight
  }
}
```
## 缺点
享元模式的本质就是尽可能的共享对象，特别适用于存在大量细粒度对象，而这些对象内部状态特别多，外部状态较少的场景。  
对于云存储来说，享元模式是必须使用的，因为云存储的场景决定了，存在大量细粒度文件对象，而存在大量只读的文件，就非常适合共享一个对象，每个用户存储的只是引用。

原文:
[](https://github.com/ascoders/weekly/tree/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F)
