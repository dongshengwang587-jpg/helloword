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

# 声明合并

声明合并在不直接改变源码的情况下，对源码的接口等做调整。

例子如下：

// sdk.ts
export interface AppMessage {

    type: string;

}

declare module "@earendil-works/pi-ai" {

    interface AppMessage {

        progress: number;

        timestamp: number;

    }

}

TypeScript自动认为：

interface AppMessage {

    type: string;

    progress: number;

    timestamp: number;

}

SDK无需修改。

