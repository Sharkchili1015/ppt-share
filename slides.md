---
# replace "./" with "purplin"
# when you copy this example.md file over
# to your own slidev environment and install
# purplin as a module
theme: purplin
colorSchema: 'light'
---

# 浅试前端单元测试

省时还是耗时？

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---
layout: intro
---

## 内容目录

<br />
<br />

- 如何开始？
- 单元测试在`工时逻辑`中的应用

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---
layout: intro
---

## 下载相关依赖

<br />
<br />

<div class="grid grid-cols-2 gap-x-4">
<div>

我们本次分享主要使用[Vitest](https://vitest.dev/)这个测试框架来进行分享

在前端单元测试中其实主要分为两个部分的测试

- 逻辑部分
- dom部分

逻辑部分：在相关依赖中`@vue/test-utils`是 `Vue.js` 官方提供的一个用于编写单元测试和集成测试的工具库，可以帮助开发者更方便地测试 Vue.js 组件。
<br>
<br>
Dom部分：`happy-dom`主要是一个基于 JavaScript 的虚拟 DOM 库，可以帮助开发者更方便地测试和模拟 DOM 操作。

</div>
<div>

### 前期准备

```ts
# devDependencies

  "devDependencies": {
    "@vue/test-utils": "^2.3.2",
    "@vitejs/plugin-vue": "^4.1.0",
    "@vitest/coverage-istanbul": "^0.30.1",
    "happy-dom": "^9.8.4",
    "typescript": "^4.9.3",
    "vite": "^4.2.0",
    "vitest": "^0.30.1",
    "vue-tsc": "^1.2.0"
  }
```

</div>
</div>

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---
layout: quote
position: left
---

# TodoList组件（Dom）
<div class="grid grid-cols-2 gap-x-2">
<div class="flex items-center justify-center">

在todolist组件中我们实现了`添加`、`删除`、`完成`,`command命令式添加`的基础功能，具体功能稍后请看演示

</div>
<div class="overflow-y-auto" style="max-height:350px;min-wight">
```ts
<template>
    <h1>ToDo App</h1>
    <div class="form">
      <label>New ToDo </label>
      <input v-model="newTodo" id="newTodo" name="newTodo" autocomplete="off" />
      <button id="newBtn" @click="addTodo()">Add ToDo</button>
      <button id="removeAllBtn" @click="store.removeAll">Remove All</button>
    </div>
    <h2>ToDo List</h2>
    <ul class="todo-ul">
      <li v-for="(todo, index) in todoList" :key="index">
        <span :class="{ done: todo.done }" @click="store.doneTodo(index)">{{
          todo.content
        }}</span>
        <button id="removeBtn" @click="store.removeTodo(index)">Remove</button>
      </li>
    </ul>
    <h4 id="empty" v-if="todoList.length === 0">Empty list.</h4>
  </template>
<script lang="ts" setup>
import { ref } from "vue";
import { useTodotore } from "../store/modules/todo";

const store = useTodotore();
const { todoList } = store;

const newTodo = ref<String>("");
function addTodo() {
  store.addTodo(newTodo.value as string);
    newTodo.value = "";
  }
</script>
```
</div>
</div>
<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---
layout: quote
position: left
---

# TodoListStore（逻辑）

<div class="grid grid-cols-2 gap-x-2">
<div class="flex items-center justify-center p-4">

TodoList组件所依赖的所有数据都在store中管理，具体涉及到的逻辑操作都在store中进行这样更有利于编写测试代码（并不一定都需要在store中管理，可以直接在组件中对数据进行管理，一样可以编写测试代码）,具体逻辑看演示。

</div>
<div class="overflow-y-auto" style="max-height:350px;min-wight">

```ts
import { defineStore } from "pinia";
import { ref, watch } from "vue";
type Todo = {
  done: boolean;
  content: string;
};
export const useTodotore = defineStore("todo", () => {
  const count = ref(0);
  const add = () => {
    count.value++;
  };
  const defaultData = [
    {
      done: false,
      content: "Write a blog post",
    },
  ];
  const isRealClear = ref<Boolean>(false)
  const todoList = ref<Todo[]>([]);
  watch(
    () => todoList.value,
    () => {
      if (todoList.value.length === 0 && !isRealClear.value) {
        todoList.value.push(...defaultData);
      }
    },{
      immediate:true
    }
  );
  const addTodo = (todo: string) => {
    const isTop = todo.includes("top:");
    const isDone = todo.includes("done:");
    const addContent = todo.replace("top:", "").replace("done:", "");
    if (isTop) {
      todoList.value.unshift({
        done: isDone,
        content: addContent,
      });
      return;
    }
    todoList.value.push({
      done: isDone,
      content: addContent,
    });
  };
  const removeTodo = (index: number) => {
    todoList.value.splice(index, 1);
    return todoList.value[index]
  };
  const doneTodo = (index:number) =>{
    todoList.value[index].done = !todoList.value[index].done
  }

  const removeAll = () =>{
    isRealClear.value = true
    todoList.value = []
  }
  return {
    count,
    add,
    addTodo,
    removeTodo,
    todoList,
    doneTodo,
    removeAll
  };
});

```
</div>
</div>
<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---
layout: image-x
image: '../assets/vitest-plugin.jpg'
---

# 对单个测试用例进行测试

在vitest插件中，可以像vscode的`debugger`功能一样，选择运行某个测试case，不需要跑命令

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---

# 单元测试在`工时逻辑`中的应用

<br>

## 工时的逻辑
- `工时`的输出有两种情况，null的时候是`-`，0的时候是0。
- 精度在小数点后一位
- 最大值>=999
- 不能有负数
<br>

## 工时交互（dom）

- 不允许用户输入两位精度的小数或者复数，当有这种情况出现的时候值保持在符合要求的那个数字
- 当出现不允许的情况出现红色报错提示

  


<br>

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---

# ！！无法适合编写单元测试的代码

<div class="overflow-y-auto" style="max-height:350px;min-wight">

```ts
    const currentUpdateEstimateLabour = useDebounceFn(() => {
      //第一件事：对比预估工时与保留的值
      if (
        !isEqual(currentLabourInput.value, stagingInputValue.value) &&
        !loading.value
      ) {
        //第二件事：判断预估工时是否合法
        const isNull = isEmpty(currentLabourInput.value);
        const remainingLabour =
          new Decimal(currentLabourInput.value || 0)
            .sub(currentWorkActualLabour.value || 0)
            .toNumber();
        const id = Number(issue.value?.id);

        const data = {
          estimateLabour: isNull
            ? null
            : new Decimal(currentLabourInput.value || 0).toNumber(),
          //父任务剩余工时：修改的预估工时-已消耗工时(actualLabour)
          remainingLabour: remainingLabour,
          //总预估工时 = currentLabourInput.value + 所有子任务的预估工时
          totalEstimateLabour: isNull
            ? null
            : new Decimal(allEstimateLabour.value || 0).toNumber(),
        };
        //第三件事：判断预估工时是否不等于复数
        remainingLabour < 0 ? delete data.remainingLabour : null;
        //第四件事：发送请求
        store.updateValue(data, id).then((res) => {
          if (!res.success) {
            //第五：请求结束后的回调
            currentLabourInput.value = stagingInputValue.value;
          } else {
            noError();
            labourEdit.value = null;
          }
        });
      } else {
        noError();
        labourEdit.value = null;
      }
    }, 500);
```
如上所示确实不适合编写测试代码，因为一个函数确实太耦合了，一个函数做了至少五件事，无数个判断边缘条件。
<br>

`个人认为：能编写出单元测试的代码的可读性以及健壮性会远高于无法编写测试的代码`

<br>
例如我上面的这个代码，说实话现在看来真的就是屎山

</div>


<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---

# 改写预估工时的逻辑

<div class="overflow-y-auto" style="max-height:350px;min-wight">

```ts
    // 1、判断是否能合法进入预估工时代码块
    const isEnableToUpdateEstimateLabour = () => {
      return (
        !isEqual(currentLabourInput.value, stagingInputValue.value) &&
        !loading.value
      );
    };
    // 2、判断预估工时的值是否是null或者''
    const isNullValue = (value: any): boolean => {
      return isEmpty(value);
    };
    // 3、计算剩余工时的值（预估工时 - 已用工时）
    const calculateRemainingLabour = (
      estimateLabour: StringOrNull,
      actualLabour: StringOrNull,
    ): number => {
      const remainingLabour = new Decimal(estimateLabour || 0)
        .sub(actualLabour || 0)
        .toNumber();
      return remainingLabour < 0 ? 0 : remainingLabour;
    };
    //4、计算总的预估工时（子任务总预估工时+当前登记的预估工时）
    const calculateTotalEstimateLabour = (
      estimateLabour: StringOrNull,
      allSubtaskEstimateLabour: StringOrNull,
    ): number | null => {
      if (isNullValue(estimateLabour)) {
        return null;
      }
      const totalEstimateLabour = new Decimal(estimateLabour || 0)
        .add(allSubtaskEstimateLabour || 0)
        .toNumber();
      return totalEstimateLabour;
    };
    //5、将每个函数结合起来，转换成data参数
    const coverEstimateLabour = () => {
      const isNull = isNullValue(currentLabourInput.value);
      const remainingLabour = calculateRemainingLabour(
        currentLabourInput.value!,
        currentWorkActualLabour.value,
      );
      const estimateLabour = isNull
        ? null
        : new Decimal(currentLabourInput.value || 0).toNumber();
      const totalEstimateLabour = calculateTotalEstimateLabour(
        currentLabourInput.value!,
        allEstimateLabour.value,
      );
      const data = {
        estimateLabour,
        remainingLabour,
        totalEstimateLabour,
      };
      return data;
    };
    //6、请求失败后回调
    const afterNotSuccess = () => {
      noError();
      labourEdit.value = null;
      return labourEdit.value;
    };
    //7、请求成功后回调
    const afterSuccess = () =>{
      currentLabourInput.value = stagingInputValue.value;
      return currentLabourInput.value;
    }
    //父任务更新预估工时
    const currentUpdateEstimateLabour = useDebounceFn(async () => {
      if (isEnableToUpdateEstimateLabour()) {
        const params = coverEstimateLabour();
        const result = await store.updateValue(params, Number(issue.value?.id));
        result.success
          ? afterNotSuccess()
          : afterSuccess();
      } else {
        afterNotSuccess();
      }
    }, 500);
```
经过改写之后的代码从一个函数拆分成了七个函数，不仅颗粒度变低了，代码的可读性也变得更好<br>
重要的是代码的可测试性大大提高，七个函数基本上每一个函数只复杂做一件事<br>
在编写测试代码的时候，只需要观察每个函数的输入和输出是否是按预期执行的即可（涉及到dom的操作先不考虑）<br>
在执行单元测试的时候，如果哪个环节出错了很快就能发现问题
<br>

# 小结
 - 尽可能降低代码的`耦合性`，每个函数`各司其职`

<br>
优化完了之后是不是看着爽多了~

</div>


<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>

---

# 编写预估工时对应的单元测试

<div class="overflow-y-auto" style="max-height:350px;min-wight">

```ts
import {
  isEnableToUpdateEstimateLabour,
  isNullValue,
  calculateRemainingLabour,
  calculateTotalEstimateLabour,
  coverEstimateLabour,
} from "../utils";

import { describe, expect, it, beforeEach } from "vitest";

describe("isEnableToUpdateEstimateLabour函数测试，进入预估工时函数之前的前置条件函数", () => {
  it("当两个值相同的时候返回false", () => {
    expect(isEnableToUpdateEstimateLabour(1, 1)).toBe(false);
  });
  it("当两个值不同的时候返回true", () => {
    expect(isEnableToUpdateEstimateLabour(1, 2)).toBe(true);
  });
});

describe("isNullValue函数测试", () => {
  it("输入的值为null或者为''的时候返回true", () => {
    expect(isNullValue('')).toBe(true);
  });

  it("输入的值不为null或''的时候返回false", () => {
    // navieUI输入框默认是string类型
    expect(isNullValue('1')).toBe(false);
  });
});

 // 3、计算剩余工时的值（预估工时 - 已用工时）
describe("calculateRemainingLabour函数测试", () => {
  it("当剩余工时小于0时，应该返回0", () => {
    expect(calculateRemainingLabour(0,1)).toBe(0);
  });

  it("当预估工时为null时，默认为0", () => {
    expect(calculateRemainingLabour(null,1)).toBe(0);
  });

  it("当剩余工时为>0时，正常输出", () => {
    expect(calculateRemainingLabour(10,1)).toBe(9);
  });
  it("当预估工时为1，剩余工时为1，输出为0", () => {
    expect(calculateRemainingLabour(1,1)).toBe(0);
  });
});

describe("calculateTotalEstimateLabour函数测试", () => {
  it("当剩余工时小于0时，应该返回0", () => {
    expect(calculateTotalEstimateLabour('1','1')).toBe(2);
  });
});
//4、计算总的预估工时（子任务总预估工时+当前登记的预估工时）
describe("coverEstimateLabour函数测试", () => {
  it("在最后的汇总函数中对所有的函数再重复执行一遍", () => {
    console.log('coverEstimateLabour(1,1,1)',coverEstimateLabour(1,1,1))
    expect(coverEstimateLabour('1','1','1')).toContain({
      estimateLabour: isNullValue('1') ? null : 1,
      remainingLabour:calculateRemainingLabour(1,1),
      totalEstimateLabour:calculateTotalEstimateLabour('1','1')
    });
  });
});

```
</div>
<br><br>
如上述的单元测试代码，我们基本将预估工时涉及到的所有的函数都进行一遍测试，可以看`coverage`中的测试覆盖率，在utils文件下的测试覆盖率已经达到100%
<br>
<br>

# 总结

## 以上就是非常简单的前端单元测试的一个demo，我对于单元测试也不太熟悉，但是从我了解到的单元测试内容来说，我认为还是有很多好处的

- 对于一些公共工具类函数（不会因为需求的变动而改变），单元测试我认为是非常需要的
- 对于一些交互很复杂，并且不够底层的组件，不太适合使用单元测试，因为其实编写单元测试也是需要时间成本的，如果一些组件很复杂单元测试代码会很多，如果需求变动可能整个测试代码就没用了
- 单元测试代码可以让我们的代码更加易懂，因为单元测试`it("在最后的汇总函数中对所有的函数再重复执行一遍")`,其实可以充当`代码文档`的存在


<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>
---
layout: center
class: "text-center"
---

# Thank You！

<br>

## 有什么说的不对的请大家尽情喷我~

[Documentations](https://sli.dev) / [GitHub Repo](https://github.com/Sharkchili1015/vitest-share)

<BarBottom  title="Liga技术分享">
  <Item text="Sharkchili1015">
    <carbon:logo-github />
  </Item>
  <Item text="ligai.cn/">
    <carbon:link />
  </Item>
</BarBottom>