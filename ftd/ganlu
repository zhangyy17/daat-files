//通用的创建函数
function commonFn(aClass, aParams) {
    //临时的中转壳函数-函数外层函数调用结束后立即销毁
    function new_() {
        //调用原型中定义的构造函数
        //中转构造逻辑及构造参数
        aClass.Create.apply(this, aParams);
    }
    //中转原型对象
    new_.prototype = aClass;
    //返回最终建立的对象
    return new new_();
}
//定义的“类”，让代码长得像静态语言里面的类
var Person = {
    Create: function(name, age) {
        this.name = name;
        this.age = age;
    },
    SayHello: function() {
        console.log("Hello,I'm " + this.name);
    },
    HowOld: function() {
        console.log(this.name + ' is ' + this.age + ' years old.');
    }
};
//调用通用构造函数创建对象，通过数组形式传递参数
var bill = commonFn(Person, ['Bill', 53]);
console.log(bill);
bill.SayHello();
bill.HowOld();
console.log(new Array(81).join("*"));

var object = {
    isA: function(aType) {
        var self = this;
        while (self) {
            if (self == aType) {
                return true;
                self = self.Type;
            }
        }
        return false;
    }
};

function Class(aBaseClass, aDefineClass) {
    //创建类的临时壳函数
    function class_() {
        this.Type = aBaseClass; //为每一个类约定一个Type属性，表明继承自谁
        for (var member in aDefineClass) {
            this[member] = aDefineClass[member];
        }
    }
    class_.prototype = aBaseClass;
    return new class_();
}
//通用创建函数
function New(aClass, aParams) {
    //创建对象的临时壳函数
    function new_() {
        this.Type = aClass; //为每一个对象约定一个Type属相，表明
        if (aClass.Create) {
            aClass.Create.apply(this, aParams);
        }
    }
    new_.prototype = aClass;
    return new new_();
}
var Person = Class(
    object, {
        Create: function(name, age) { //约定所有类的构造函数都叫做create
            this.name = name;
            this.age = age;
        },
        SayHello: function() {
            console.log("Hello, I'm " + this.name);
        },
        HowOld: function() {
            console.log(this.name + " is " + this.age + " years old.");
        }
    }
);
var Employee = Class(Person, {
    Create: function(name, age, salay) {
        Person.Create.call(this, name, age);
        this.salay = salay;
    },
    ShowMeTheMoney: function() {
        console.log(this.name + " $ " + this.salay);
    }
});
var bill = New(Person, ['Bill', 53]);
console.log(bill);
bill.SayHello();
var bob = New(Employee, ["Bob", 50, 12345]);
console.log(bob);
bob.SayHello();
bob.HowOld();
bob.ShowMeTheMoney();
console.log(new Array(81).join("*"));
