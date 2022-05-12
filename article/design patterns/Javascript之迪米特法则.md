# Javascript之迪米特法则
迪米特法则(LoD)或者最少知道原则是一个软件开发设计准则，尤其是在面向对象编程领域。一般来说，迪米特法则是松散耦合的特殊情况。这个准则是1987年底由 Ian Holland 在波士顿东北大学提出来的。  
它是基于三个基本规则：  
- 每个单元对于其他的单元只能拥有有限的知识：只是与当前单元紧密联系的单元  
- 每个单元只能和它的朋友交谈：不能和陌生单元交谈  
- 只和自己直接的朋友交谈

在这种上下文中，单元是一个特定的编码抽象。它可以是一个函数，一个模块，或者一个类。而这里的对话意味着与该抽象概念的相结合。  
下面的例子中，我们将有三个单元：游客、出租车司机和钱包。  
假设我们有两个角色，Bob(游客)和Sam(出租车司机)。Bob是一个游客，他不知道如何去x地。在这种情况下，我们有一个tourist类，显然是针对Bob的，我们也有一个针对Sam的TaxiDriver类。  
``` 
class Tourist {}
class TaxiDriver {}
```
假设Bob有一个钱包，会把钱放在里面。因此我们需要向Tourist类添加一个wallet属性。
``` 
class Tourist {
  constructor() {
    this.wallet = new Wallet();
  }
}
class Wallet {
  constructor() {
    this.budget = 0;
  }
  addMoney(amount) {
    this.budget += amount;
  }
  takeMoney(amount) {
    this.budget -= amount;
  }
```
显然，Bob需要向Sam支付一笔钱作为车费，所以Sam和Bob之间的互动可能如下所示  
``` 
class TaxiDriver {
  /* ... */
  takeFee(totalFee, customer) {
    customer.wallet.takeMoney(totalFee);
  }
  /* ... */
}
```
这似乎没有问题，但让我们考虑一个现实生活中的类比。上面的代码告诉我们，Sam从Bob的包里拿了钱包，然后拿走他要收取的车费。当Sam完成支付流程后，他将钱包放回Bob的包里。  
根据迪米特法则来分析一下。Bob的钱包对Sam来说是陌生的。所以Sam应该和Bob谈谈，而不是他的钱包。  
``` 
class TaxiDriver {
  /* ... */
  takeFee(totalFee, customer) {
    customer.requestPayment(totalFee);
  }
  /* ... */
}
class Tourist {
  /* ... */
  requestPayment(amount) {
    this.wallet.takeMoney(amount);
  }
  /* ... */
}
```
现在看来，这似乎更加合理了。Sam直接与Bob交谈。反过来，Bob会和他的Wallet实例通信，获取所需的金额，然后交给Sam。  
迪米特法则的目的是让类之间解耦，降低耦合度。只有这样，类的可复用性才能提高。  
但是迪米特法则也有弊端，它会产生大量的中转类或跳转类，导致系统的复杂度提高。  
所以我们不要太死板的遵守这个迪米特法则，在系统设计的时候，在弱耦合和结构清晰之间反复权衡。尽量保证系统结构清晰，又能做到低耦合。  

参考:
[Javascript之迪米特法则](https://mp.weixin.qq.com/s/fhSMz8BEIyjFGVnJtcVrKg)
