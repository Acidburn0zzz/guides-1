Emberを学ぶと、`Ember.Component.extend()`や`DS.Model.extend()`といったコードを目にするでしょう。 ここでは、Emberオブジェクトモデルの他の主な機能だけでなく、この `extend()`メソッドについて学びます。

### クラスを定義する

Emberで新しい*クラス*を定義するには、[`Ember.Object`](http://emberjs.com/api/classes/Ember.Object.html)の[`extend()`](http://emberjs.com/api/classes/Ember.Object.html#method_extend)メソッドを呼び出します。

```javascript
const Person = Ember.Object.extend({
  say(thing) {
    alert(thing);
  }
});
```

これで`say()`メソッドを持つ新しい`Person`クラスが定義されます。

任意の既存クラスから*サブクラス*を作成するには、そのクラスの`extend()`メソッドを呼び出します。 例えば、Emberに組み込まれている[`Ember.Component`](http://emberjs.com/api/classes/Ember.Component.html)のサブクラスを作成したいなら、次のようにします。

```app/components/todo-item.js export default Ember.Component.extend({ classNameBindings: ['isUrgent'], isUrgent: true });

    <br />### 親クラスのメソッドをオーバーライドする
    
    サブクラスを定義する際、メソッドをオーバーライドすることができます。このとき、特別な`_super()`メソッドを呼び出すことで親クラスの実装にアクセスすることが可能です。
    
    ```javascript
    const Person = Ember.Object.extend({
      say(thing) {
        alert(`${this.get('name')} says: ${thing}`);
      }
    });
    
    const Soldier = Person.extend({
      say(thing) {
        // this will call the method in the parent class (Person#say), appending
        // the string ', sir!' to the variable `thing` passed in
        this._super(`${thing}, sir!`);
      }
    });
    
    let yehuda = Soldier.create({
      name: 'Yehuda Katz'
    });
    
    yehuda.say('Yes'); // alerts "Yehuda Katz says: Yes, sir!"
    

特定のケースでは、オーバーライドの前後で`_super()`に引数を渡します。

これによって、元のメソッドは正常に動作し続けることができます。

一般的な例に、EmberDataのシリアライザーの1つ、[`normalizeResponse()`](http://emberjs.com/api/data/classes/DS.JSONAPISerializer.html#method_normalizeResponse)フックをオーバーライドする場合があります。

このようなケースで使える便利なショートカットに、`...arguments`のように記述する「スプレッド演算子」があります。

```javascript
normalizeResponse(store, primaryModelClass, payload, id, requestType)  {
  // Customize my JSON payload for Ember-Data
  return this._super(...arguments);
}
```

上記の例では、(カスタマイズした後で) 元の引数を親クラスに戻すため、通常の操作を続行できます。

### インスタンスを作成する

クラスを定義すると、[`create()`](http://emberjs.com/api/classes/Ember.Object.html#method_create)メソッドを呼び出すことで、そのクラスの*インスタンス*を作成できます。 あなたがクラスに定義した全てのメソッド、プロパティ、計算プロパティはインスタンスに存在することになります。

```javascript
const Person = Ember.Object.extend({
  say(thing) {
    alert(`${this.get('name')} says: ${thing}`);
  }
});

let person = Person.create();

person.say('Hello'); // alerts " says: Hello"
```

インスタンスを作成する際、`create()`メソッドに省略可能なハッシュを渡すことによって、プロパティの値を初期化できます。

```javascript
const Person = Ember.Object.extend({
  helloWorld() {
    alert(`Hi, my name is ${this.get('name')}`);
  }
});

let tom = Person.create({
  name: 'Tom Dale'
});

tom.helloWorld(); // alerts "Hi, my name is Tom Dale"
```

パフォーマンス上の理由から、`create()`の呼び出し中には、インスタンスの計算済みプロパティを再定義できないこと、既存のメソッドの再定義新しいメソッドの定義をすべきでないことに注意してください。 `create()`を呼び出したときは、単純なプロパティのみを設定すべきです。 もしメソッドや計算済みプロパティを定義あるいは再定義する必要があるなら、新しいサブクラスを作成してそれをインスタンス化してください。

慣習として、クラスを保持するプロパティまたは変数はパスカルケース (パスカル記法)ですが、インスタンスはそうではありません。 たとえば、変数Personはクラスを指す一方、personはインスタンス (通常はPersonクラスのインスタンス) を指します。 Emberアプリケーションでは、これらの命名規則に従う必要があります。

### インスタンスの初期化

When a new instance is created, its [`init()`](http://emberjs.com/api/classes/Ember.Object.html#method_init) method is invoked automatically. This is the ideal place to implement setup required on new instances:

```js
const Person = Ember.Object.extend({
  init() {
    alert(`${this.get('name')}, reporting for duty!`);
  }
});

Person.create({
  name: 'Stefan Penner'
});

// alerts "Stefan Penner, reporting for duty!"
```

If you are subclassing a framework class, like `Ember.Component`, and you override the `init()` method, make sure you call `this._super(...arguments)`! If you don't, a parent class may not have an opportunity to do important setup work, and you'll see strange behavior in your application.

Arrays and objects defined directly on any `Ember.Object` are shared across all instances of that object.

```js
const Person = Ember.Object.extend({
  shoppingList: ['eggs', 'cheese']
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan and Robert both trigger their addItem.
// They both end up with: ['eggs', 'cheese', 'bacon', 'sausage']
```

この現象を避けるためには、`init()`の中でそれらの配列やオブジェクトのプロパティを初期化することを奨励します。これにより、各インスタンスは一意になります。

```js
const Person = Ember.Object.extend({
  init() {
    this.set('shoppingList', ['eggs', 'cheese']);
  }
});

Person.create({
  name: 'Stefan Penner',
  addItem() {
    this.get('shoppingList').pushObject('bacon');
  }
});

Person.create({
  name: 'Robert Jackson',
  addItem() {
    this.get('shoppingList').pushObject('sausage');
  }
});

// Stefan ['eggs', 'cheese', 'bacon']
// Robert ['eggs', 'cheese', 'sausage']
```

### オブジェクトの属性へのアクセス

When accessing the properties of an object, use the [`get()`](http://emberjs.com/api/classes/Ember.Object.html#method_get) and [`set()`](http://emberjs.com/api/classes/Ember.Object.html#method_set) accessor methods:

```js
const Person = Ember.Object.extend({
  name: 'Robert Jackson'
});

let person = Person.create();

person.get('name'); // 'Robert Jackson'
person.set('name', 'Tobias Fünke');
person.get('name'); // 'Tobias Fünke'
```

Make sure to use these accessor methods; otherwise, computed properties won't recalculate, observers won't fire, and templates won't update.