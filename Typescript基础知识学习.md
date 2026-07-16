# 变量命名关键字

let(可修改) const(不可修改)是块级作用域

块级作用域：即{}之间的内容，出了大括号就不起作用。

# 函数

function add(a,b){
    "普通函数"
    return a+b;
}

const handler = async (event)=>{
      "箭头函数"
    await process(event)
}

# interface

该关键字用于描述一个对象应该具有什么结构。

例子：

interface Message {

    role:string;

    content:string;

}

const msg: Message = {

    role:"user",

    content:"你好"

};



