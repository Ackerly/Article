# 初学 ECS 架构，实现超炫的粒子碰撞动画
**ECS概念**
ECS是一种软件架构模式，常见于游戏业务场景，其主要对象分类为
- Entity 实体,ECS架构中所有的业务对象都必须拥有一个唯一的Entity实体
- Component 组件,存储着数据结构,对应着某一种业务属性,一个Entity上可以动态挂载多个Component
- System 系统,负责主要的逻辑处理,每个System负责处理特定的Component的逻辑

快速对比下OOP  

|  ECS  |  OOP  |
| ----  | ---- |
|  数据和逻辑分离  |  数据和逻辑耦合  |
|  Entity 可动态挂载多个Component  |  Object Instance 一般是固定的某个Class的示例  |
|  Entity 做唯一性区分  |  通过指针/引用/id/uuid 做唯一区分  |
|  函数通过Entity是否挂载指定的Component 做非法检验  |  编译阶段的静态类型校验通过instanceof 或 type/sort 相关字段区分  |
|  World 存储所有的Entity, 并提供 Entity/Component 检索的方法  |  未做明确要求，由开发者根据业务特性自行设计管理存储Object Instatnce的结构。理论上也可以仿照World的概念设计  |
|  System 不建议直接相互调用,通过修改/添加/删除Entity的Component,或者利用单例组件,来延迟处理  |  未做明确要求,业务逻辑可能会相互调用,或者通过事件等机制触发  |

**从0实现demo**  
在一个画布中存在若干个运动的方块,这些方块会在触碰到墙块以及相互碰撞的时候,改变运动方向,不停循环  
利用一个ECS框架快速实现demo,主要用到下面几个API  
``` 
type EntityId = number;
interface World {
    // 创建 Entity 
    createEntity():EntityId;
    // 给某个 Entity 动态挂载Component
    addEntityComponents(entity: EntityId, ...components: Component[]): World;
    // 添加某个System
    addSystem(system: System): void;
    // 查询指定Component 的 所有Entity
    view<CC extends ComponentConstructor[]>(...componentCtors: CC): Array<[EntityId, Map<ComponentConstructor, Component>]>;
}
```

**绘制静止方块**  
定义3个组件和1个系统  
- Position 位置组件
- Rect 方块大小组件
- Color 颜色组件
- RenderSystem 绘制系统

``` 
export class Position extends Component {
  constructor(public x = 0, public y = 0) {
    super();
  }
}

export class Rectangle extends Component {
  constructor(public readonly width: number, public readonly height: number) {
    super();
  }
}

export class Color extends Component {
  constructor(public color: string) {
    super();
  }
}

export class RenderingSystem extends System {
  constructor(private readonly context: CanvasRenderingContext2D) {
    super();
  }

  public update(world: World, _dt: number): void {
    this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height);

    for (const [entity, componentMap] of world.view(Position, Color, Rectangle)) {
      const { color } = componentMap.get(Color);
      const { width, height } = componentMap.get(Rectangle);
      const { x, y } = componentMap.get(Position);

      this.context.fillStyle = color;
      this.context.fillRect(x, y, width, height);
    }
  }
}
```
再执行主逻辑,创建若干Entity,并挂载Component  
``` 
const world = new World();
world.addSystem(new RenderingSystem(canvas.getContext('2d')));
// 添加若干方块
for (let i = 0; i < 100; ++i) {
  world.addEntityComponents(
    world.createEntity(),
    new Position(
      getRandom(canvas.width - padding * 2, padding * 2),
      getRandom(canvas.height - padding * 2, padding * 2),
    ),
    new Color(
      `rgba(${getRandom(255, 0)}, ${getRandom(255, 0)}, ${getRandom(
        255,
        0,
      )}, 1)`,
    ),
    new Rectangle(getRandom(20, 10), getRandom(20, 10)),
  );
}
```

**给方块增加速度**  
让方块动起来，这里新增1个组件和1个系统  
- Velocity 速度组件
- PhysicsSystem 位置更新系统

``` 
export class Velocity extends Component {
  constructor(public x = 0, public y = 0) {
    super();
  }
}

export class PhysicsSystem extends System {
  constructor() {
    super();
  }

  update(world: World, dt: number) {
    for (const [entity, componentMap] of world.view(Position, Velocity, Rectangle)) {
      // Move the position by some velocity
      const position = componentMap.get(Position);
      const velocity = componentMap.get(Velocity);

      position.x += velocity.x * dt;
      position.y += velocity.y * dt;
    }
  }
}
```
**增加碰撞包围盒**  
为了不让方块跑出画布,新增如下组件和系统
- Collision 碰撞组件
-  CollisionSystem 碰撞系统

``` 
export class CollisionSystem extends System {
    update(world: World): void {
      for (const [entity1, components1] of world.view(
        Position,
        Velocity,
        Rectangle,
      )) {
        for (const [entity2, components2] of world.view(
          Collision,
          Rectangle,
          Position,
        )) {
          // 1. 检测每个方块与碰撞方块是否碰撞
          // 2. 判断碰撞方向
          // 3. 根据碰撞方向修改方块速度
        }
      }
    }
  }
```
在主逻辑中,增加4个方块围着画布,并给予红色,形成一个包围盒  
``` 
const padding = 10;
// left
world.addEntityComponents(
  world.createEntity(),
  new Position(0, 0),
  new Velocity(0, 0),
  new Color(`rgba(${255}, ${0}, ${0}, 1)`),
  new Rectangle(padding, canvas.height),
  new Collision(),
);
// top xx
// right xx
// bottom xx
```

## 以一个简单例子对比OOP和ECS
_题目_  
假设现在要做一个动物竞技会的游戏,开发要求是有形形色色种类的动物,和形形色色的竞技项目，要求每种动物都要去参加它能参加的竞技项目并得到成绩  

_OOP 两种思路_  
常规的思路是定义好动物和项目的基类及其子类，在子类中描述了动物拥有的能力和项目要求的能力，那么下一步就是处理业务的主逻辑,这里展示两个常规思路  

**运动员注册制**  
创建一个动物的时候,将其适合的运动注册一个运动员身份，并保存运动员索引列表，比赛的时候根据运动员索引表将对应项目的运动员索引出来  
``` 
// 创建随机的动物
function createAnimals(): Animal[] {
  // xxxx
}
class CompetionList {
// 某个动物注册运动员身份
 registerSporter(animal: Animal) {}
// 注册运动项目
 registerCompetion(competion: Competion){}
 // 获取所有竞技的成绩
 run(){
   this.competions.forEach(competion=>{
       const scores = competion.sporters.map(xxx);
   })
 }
}

const competionList = new CompetionList();
// 注册竞技项目
competionList.registerCompetion(xxx)
// 注册运动员
const animals = createAnimals();
animals.forEach((a) => {
  competionList.registerSporter(a);
});
// 比赛结果
competionList.run()
```
**现场报名制**  
将所有的动物混编,举行项目比赛的时候,按照要求实时查询，而这一思路便和ECS比较像  
``` 
function createAnimals(): Animal[] {
    //xxx;
  }
  function canItRun() {}
  function canItSwim() {}
  function canItClimb() {}

  class AnimalsList {
      find(canItXX:(animal:Animal)=>boolean):Animal[]{}
  }
  class CompetionList {
    // 动物报名
     registerAnimal(animal: Animal) {}
    // 注册运动项目
     registerCompetion(competion: Competion){}
     // 获取所有竞技的成绩
     run(animals: AnimalsList){
         this.competions.forEach(competion=>{
             animals.find(competion.canItXX).map(xxx)
         })
     }
  }
```
对比下两者的优劣势  

|  运  |  运动员注册  |  现场报名  |
|  ----  |  ----  |  ----  |
| 选手固定时 | 1. 存在缓存,运动员不发生改变的情况下不会产生新的运算 | 1. 运动员不改变时，也需要每次查询消耗性能 |
| 选手变化时 | 1. 运动员/项目发生任何变化的时候,都需要更新运动员注册表 | 1. 不影响主逻辑2. 用运行时的一些性能损耗，提高主逻辑的兼容性，减少后续的开发成本 |

**ECS**  
相关示例代码  
``` 
import {
  System,
  World,
  Component,
  ComponentConstructor,
} from '@jakeklassen/ecs';

// 设计组件
class Animal extends Component {}
class RunAbility extends Component {
  constructor(public speed: number) {
    super();
  }
}
class SwimAbility extends Component {
  constructor(public speed: number) {
    super();
  }
}

class CompetionComponent extends Component {
  constructor(
    public name: string,
    public canItXX: ComponentConstructor[],
    public getScoreFunc: any,
    public scoreMap: Map<number, number>,
  ) {
    super();
  }
}
// 设计比赛的系统
class CompetionSystem extends System {
  update(world: World) {
    for (const [entityCompetion, componentsCompetion] of world.view(
      CompetionComponent,
    )) {
      const competion = componentsCompetion.get(CompetionComponent);
      for (const [entityAnimal, componentsAnimal] of world.view(
        ...competion.canItXX,
        Animal,
      )) {
        // 计算分数
        const score = competion.getScoreFunc(componentsAnimal);
        competion.scoreMap.set(entityAnimal, score);
      }
    }
  }
}

// 创建world
const world = new World();
// 添加比赛系统
world.addSystem(new CompetionSystem());
// 添加项目组件
world.addEntityComponents(
  world.createEntity(),
  new CompetionComponent(
    '百米跑',
    [RunAbility],
    (animalComponents: any) => {
      return 100 / animalComponents.get(RunAbility).speed;
    },
    new Map<number, number>(),
  ),
);
world.addEntityComponents(
  world.createEntity(),
  new CompetionComponent(
    '百米游泳',
    [SwimAbility],
    (animalComponents: any) => {
      return 100 / animalComponents.get(SwimAbility).speed;
    },
    new Map<number, number>(),
  ),
);
// 随机添加动物组件
for (let i = 0, len = 15; i < len; i++) {
  const entity = world.createEntity();
  if (Math.random() >= 0.5) {
    world.addEntityComponents(
      entity,
      new Animal(),
      new RunAbility(Math.random() * 10),
    );
  }
  if (Math.random() >= 0.5) {
    world.addEntityComponents(
      entity,
      new Animal(),
      new SwimAbility(Math.random() * 10),
    );
  }
}

// 运行跑分
world.update();
for (const [entityCompetion, componentsCompetion] of world.view(
  CompetionComponent,
)) {
  const component = componentsCompetion.get(CompetionComponent);
  console.log('%s 比赛分数如下', component.name);
  console.group();
  for (const [id, score] of component.scoreMap.entries()) {
    console.log('动物id:%d  分数:%d', id, score);
  }
  console.groupEnd();
}

```




原文:  
[前端初学 ECS 架构，实现超炫的粒子碰撞动画](https://mp.weixin.qq.com/s/zxSv8IpJtR__--pIwwb6AQ)
