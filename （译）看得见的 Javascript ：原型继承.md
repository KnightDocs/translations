# （译）看得见的 Javascript ：原型继承

有没有想过为什么我们可以在字符串，数组或对象上使用诸如`.length`，`.split()`、`. join()`之类的内置方法？我们从来没有直接定义它们，那么它们从哪里来？因为有一个东西叫做原型继承（*prototypal inheritance*）。它非常酷，你会经常用到它！

我们经常会创建同一类型的许多个对象。假设我们有一个网站，人们可以浏览狗狗！

对于每一只狗来说，我们都需要创建一个对象来代表那只狗！我将会利用 `new` 关键字来创建一个 Dog 实例，这个实例是由一个构造函数产生的，而不是每次都写一个新的对象。

每一只狗都有一个名字、品种、颜色，以及一个 bark 函数！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--pDfw39RK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/caurw7uuk62htpldgtln.png)

```javascript
function Dog(name, breed, color) {
  this.name = name;
  this.breed = breed;
  this.color = color;
  this.bark = function () {
    return 'Woof!';
  }
}
```

当我们创建了 `Dog` 构造函数，但它不是我们创建的唯一对象，还有另一个对象自动地创建了出来，这个对象称为：原型（*prototype*）！默认情况下，该对象包含一个 constructor 属性，它指向原始的构造函数，在这个例子中指向的是 `Dog`。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--dWGIZ_zz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/9howj4i3zvlgun3svppp.gif)

在 Dog 构造函数上的 `prototype` 属性是不可枚举（non-enumerable）的，意味着当我们试着去获取对象的属性时，它是不会出现的。但它仍然在那！

好了，所以……为什么我们会有这个属性对象呢？首先，让我们创建一些 dogs 实例吧。简单起见，我将会把它们称为 `dog1`，`dog2`。`dog1` 是 Daisy，一只可爱的黑色拉布拉多（a cute black Labrador）！`dog2` 是 Jack，无所畏惧的白色杰克罗素（the fearless white Jack Russell）。

```javascript
const dog1 = new Dog(
	"Daisy",
  "Labrador",
  "black"
)

const dog2 = new Dog(
	"Jack",
  "Jack Russell",
  "white"
)
```

log `dog1` 到控制台，然后展开它的属性看看！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--cA-2FOVV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/tt4yfoz8ckmxfofv3f9v.gif)

我们能够看到我们添加的属性，像是 `name`，`breed`，`color` 和 `bark`。等等，那个 `__proto__` 属性是什么？！它也是一个不可枚举（non-enumerable）的属性。让我们展开它！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--zxO-eMV0--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/dye57pcku5cfaz0er60c.gif)

哇，它看起来像 `Dog.prototype`对象！好吧，猜猜是什么，`__proto__` 是一个指向 `Dog.prototype` 的指针。这就是原型继承的全部内容：构造函数的每一个实例都能获取到它的原型对象！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--FBGV--dx--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/t6kiav029gl2e0hv1xct.gif)

为什么这很酷？因为有时我们希望有一些属性是所有实例所共享的。举个例子，每一个实例中的`bark`方法都是一样的，那为什么每次创建一个新的实例的时候，我们都要再创建一次新的方法呢？这样每次都会消耗内存的。为了避免这样的消耗，我们可以在 `Dog.prototype`对象上添加共享的属性或方法！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--2026kdwz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/59nlnyqioosaowj09xn8.gif)

不管何时我们去获取一个实例上的属性，引擎都会首先在自身上去寻找是否存在对应的属性。然而，如果未找到该属性，那么引擎就会通过`__proto__`属性沿着原型链（**the prototype chain**）继续寻找。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--gg5KU5nB--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/fabyyjot1s78mttyzzk8.gif)

现在这只是一个步骤，但它可以包含几个步骤！如果你继续找下去，你就会注意到，当我展开显示 `Dog.prototype` 的原型对象时，并不仅仅包含一个属性。`Dog.prototype`本身就是一个对象，意味着它实际上是 `Object` 构造函数的一个实例！所以，`Dog.prototype` 也包含一个 `__proto__`属性，它指向 `Object.prototype`！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--vJ7k8Gb3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/8vk5w6loliot818f2lcd.gif)

最后，对于所有的内置方法来自哪里这个问题，我们有了答案：它们在原型链上！

比如，`.toString()`方法。它是定义在`dog1`实例身上的么？嗯，不是……那它是定义在`dog1.__proto__`指向的原型对象 `Dog.prototype` 上的吗？也不是！那是定义在`Dog.prototype.__proto__`指向的`Object.prototype`上的吗？是的！

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--16IwaVkk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/fpt5nndkbq5kau0nqeqj.gif)

现在，我们用了构造函数（`function Dog() {...}`），它仍然是有效的 Javascript。然而，ES6 实际上引入了更加简单的语法，通过 class 来创建构造函数。

> Class 只是构造函数的语法糖（**syntactical sugar**）。

我们用 `class` 关键字来写 class。一个 class 就是一个构造函数，和 ES5 写的那个一样！我们想要添加到原型上的属性，可以被定义在 class 本身。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--3PePIjz5--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/qnbqubcipqjl5pb3i8ds.gif)

关于 class 的，另一个好处是我们可以轻松地扩展（**extend** ）其他类。

比如说，我们想要展示几只相同品种的狗，吉娃娃。简单起见，我只给 Dog 类传一个 `name` 属性。但是这些吉娃娃实例也可以做特定的事情，它们能小声地 bark，而不是说 `Woof!`。

在一个扩展的class中，我们能够通过 `super` 关键字来获取父级类的构造器。如果想让父级类中参数包含在内，我们就必须将那些参数传到`super`中，这里只有一个参数`name`。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--Fitn1c9K--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/tx25dar3duqo0z2bpfam.png)

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }
  bark () {
    return 'Woof!'
  }
}

class Chihuahua extends Dog {
  constructor(name) {
    super(name);
  }
  smallBark() {
    return 'Small woof!';
  }
}

const myPet = new Chihuahua('Max');
```

`myPet`既能获取到 `Chihuahua.prototype`，又能获取到 `Dog.prototype`，还有 `Object.prototype`，因为 `Dog.prototype` 也是一个对象。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--WOeqUeM3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/qija16dju8t5j1ksy0ps.gif)

因为 `Chihuahua.prototype` 有 `smallBark` 函数，`Dog.prototype`有 `bark`函数，所以，我们在`myPet`上既可以获取 `smallBark`，也可以获取 `bark`！

现在你可以想象，原型链不会永远持续下去。最终，有一个对象的原型等于`null`：`Object.prototype`对象的`__proto__`就指向了`null`！如果我们去获取一个哪都找不到的属性（在本身或是沿着原型链都找不到），那就会返回 `undefined`。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s---EseK2fk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/1905zxijp45soy0jzle2.gif)

------

尽管我在这儿解释了关于构造函数以及类的一切，但是另一种添加对象属性的方式是用 `Object.create`方法。用这个方法，我们可以创建一个新的对象，也可以额外去指定我们想要的属性！

我们这样做，传一个存在的对象作为`Object.create`方法的参数。这个对象就是我们创建的对象实例的原型！

```javascript
const person = {
  name: 'Lydia',
  age: 21
}
const me = Object.create(person);
```

我们 log 一下我们刚刚创建的对象。

![img](https://res.cloudinary.com/practicaldev/image/fetch/s--9sWtvaRG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://thepracticaldev.s3.amazonaws.com/i/6zzt8zpy85gtitxmpwi9.gif)

我们没有添加任何属性到 `me` 实例上， 它只是简单地包含了一个不可枚举的 `__proto__`属性！这个属性指向一个对象，这个对象由我们定义，它是构造出来的实例的原型：也就是`person`对象，它有一个 `name` 和 `age`属性。因为`person`是一个对象，所以`person.__proto__`的值就是 `Object.prototype`。

**2021.4.20 Translated by Knight**