---
theme: geist
highlighter: shiki
css: unocss
colorSchema: 'light'
---

# 单元测试——理论篇

写代码虽然是一项重实践的手艺，但是大多数人忽视了理论知识所带来的影响

<div class="pt-24 text-right">
  主站研发二中心\前端技术小组\交易小组--罗超
</div>

---

# 背景
常见的疑问：

-  📝 **怎么去评判一个单元测试的用例是否有效可靠？**
-  🧑‍💻 写单元测试到底有什么意义？
-  🛠 单元测试定位是什么？

---


# 内容大纲

分享内容：

  - **定义**
  - **意义**
  - **适用场景**
  - **拆分工作单元**
  - **优秀单测的原则**
  - **集成测试**
  - **测试驱动开发（Test-Driven Development）**

---

# 定义

一个单元测试是一段代码，**这段代码调用一个工作单元，并检验该工作单元的一个具体的结果。如果关于这个结果的最终假设是错误的，单元测试就失败了**。一个单元测试的范围可以小到一个方法，大到多个类。

```js
// sum.js
export default function sum(a, b) {
  return a + b;
}

// sum.spec.js
const sum from './sum'

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

---

# 意义

<Note>测试越多越好是一个伪命题，无效的测试不如没有测试。</Note>

单测的意义：


- **保证模块的旧功能**

- **更放心大胆地进行模块重构**

- **帮助开发者快速地了解模块的功能**

- **提高代码质量**

---
layout: split
---

### 金字塔模型

<img src='/images/test.png' />

---
layout: split
---

### 现实中的模型

<img src='/images/test1.png' />

---
layout: center
---

<h3>有限的资源、有限的时间，我们必须做选择。</h3>

---

# 适合的场景

- **基础 SDK**

- **组件库**

- **模式库（utils）**

- **React 自定义 hooks**

- **more...**

---

# 易测的特点

- **有稳定的输入输出的模块**

- **组件，得益官方或社区提供的一些辅助测试方法**

---

# 不适合的场景

- **与 UI 强相关、迭代频繁的功能**

- **脚手架工具**

- **more...**

--- 

# 不易测的特点
- **投入成本过高，收益不大**

- **强依赖外部模块，难以 mock**

---
layout: full
---
# 单元划分

- **util**
- **component**
- **hook**
- **class**
- **more...**

<div class="bottom" style="bottom: 4rem;"><Note>工作单元和测试用例的区别？</Note></div>

---

# 优秀单测的原则

<div v-click="1">

- **自动的（Automatic）**
所有的测试用例应该能够自动运行，不需要通过交互地进行。
</div>

<div v-click="2">

- **可重复执行的（Repeatable）**
测试用例可以在 CI 中或者任意时间重复执行，在代码没有变动的情况下，每次测试的结果都是稳定的。
</div>

<div v-click="3">

- **独立的（Independent）**
各个测试用例之间应该没有任何依赖关系，测试用例之间随意调换执行顺序，始终不影响测试结果。
</div>
---

<div>

- **快速的（Fast）**
单元测试运行速度是快速的，就算是大型项目，运行的时间应该是分钟级别。
</div>

<div v-click="1">

- **可自我验证的（Self Validating）**
单元测试应该是可以通过简单的命令做到自执行，不需要人工验证。
</div>

<div v-click="2">

- **及时的（Timely）**
在新增了 feature 和修复了问题后，测试用例应该及时更新。对于无效的测试，应该及时清理。测试代码应该看做源码的一部分，也需要及时重构和更新。
</div>
---

<div>

- **隔离的（Isolated）**
测试用例之间应该做到互相不干扰，全局状态的修改或者 DOM 树的调整，如有必要，需要做一些状态重置。
</div>

<div v-click="1">

- **可读的（Readable）**
好的测试用例应该是可读的，测试用例的描述应该做到简明扼要。单个用例代码应该避免出现大量的代码和逻辑，例如出现 if、for等语句。

</div>

<div v-click="2">

- **易维护的（Maintainable）**
测试代码也是代码，所以可维护性也很重要。**将一些通用的测试初始化逻辑，通过函数进行封装**，一定程度上提高测试代码的可维护性。
</div>

---
layout: split
---

<div>
  <h3>一个例子</h3>
  <Note class="bottom">该例子有几个问题？</Note>
</div>

```ts
test('撤销', done => {
  let count = 0
  const editor = createEditor(document, 'div1', '', {
    onchange: function () {
      count++
      // 由 editor.cmd.do 触发的 onchange
      if (count == 1) {
        const undo = getMenuInstance(editor, Undo)
        undo.clickHandler()
        if (editor.isCompatibleMode) {
          // 兼容模式
          expect(editor.$textElem.html()).toEqual('<p><br></p>')
        } else {
          // 标准模式
          expect(editor.history.size).toEqual([0, 1])
        }
      }
      done()
    },
    compatibleMode: function () {
      return Math.round(Math.random()) == 1
    },
  })
  mockCmdFn(document)
  editor.cmd.do('insertHTML', '<span>123</span>')
})
```
---
layout: split
---
# 不好的用例
```js
it("sync cacheData", () => {
  const mockFn = (value: any) => value;
  const mockCacheData = cacheable(mockFn);
  // * 同步测试 -----------------------
  // * 首次缓存
  expect(mockCacheData(1)).toBe(1);
  // * 相同参数缓存
  expect(mockCacheData(1)).toBe(1);
  // * 不同参数类型刷新缓存
  expect(mockCacheData("1")).toBe("1");
  // * 新参数为函数类型刷新缓存
  const mockFn1 = () => {};
  const mockFn2 = () => {};
  expect(mockCacheData(mockFn1)).toBe(mockFn1);
  expect(mockCacheData(mockFn2)).toBe(mockFn2);
  // * 新参数为数组类型刷新缓存
  const mockArr1 = [1, 2];
  const mockArr2 = [1, 2, 3];
  const mockArr3 = [1, 2, 4];
  expect(mockCacheData(mockArr1)).toBe(mockArr1);
  expect(mockCacheData(mockArr2)).toBe(mockArr2);
  expect(mockCacheData(mockArr3)).toBe(mockArr3);
  // * null刷新缓存
  // 省略一些代码
});
```

---

## 问题

- **关注点太多**

- **可读性差**

---

```js
test('LogAnalyzer isValid invalid fileName', () => {
  const logan = new LogAnalyzer()
  const res = logan.isValid('12345')
  expect(res).toBeFalsy()
})

test('LogAnalyzer isValid valid fileName', () => {
  const logan = new LogAnalyzer()
  const res = logan.isValid('12345789')
  expect(res).toBeTruthy()
})
```
---

## 问题

- **重复性逻辑**

- **可维护性差**

---

```js
describe("observable", () => {
  it("", () => {
    const mockObserveList = observe();
    const mockFn1 = (a: number) => {};
    const mockFn2 = (a: number) => {};
    mockObserveList.subscribe(mockFn1);
    mockObserveList.subscribe(mockFn2);
    mockObserveList.next(2);
    mockObserveList.subscribe(mockFn2).unsubscribe();
    // ???
  });
});
```
---

## 问题

- **缺少断言**

- **可读性差**

---
layout: split
---

<div>

## 好的用例

<div class="bottom" style="bottom: 4em;font-weight: bold;" v-click>
  优点：清除潜在的副作用
</div>

</div>


```ts
describe('convert to html or text', () => {
  let container
  beforeEach(() => {
    container = document.createElement('div')
    document.body.appendChild(container)
  })
  afterEach(() => {
    document.body.removeChild(container)
  })

  it('convert to html if has selector', () => {
    const editor = createEditor({
      selector: container,
      content: [
        {type:'paragraph', children:[{text: 'hello' }]}],
    })
    expect(editor.getHtml()).toBe('<div>hello</div>')
  })

  it('convert to html if not selector', () => {
    const editor = createEditor({
        selector: container,
        content: [
          {type:'paragraph', children:[{text:'hello'}]}],
    })
    expect(editor.getHtml()).toBe('<div>hello</div>')
  })
```
---

```ts
let editor: IDomEditor
let menu: UploadImageMenu

describe('Upload image menu', () => {
  beforeEach(() => {
    editor = createEditor()
    menu = new UploadImageMenu()
  })

  test('UploadImageMenu instance title, () => {
    expect(menu.title).toBe('上传图片')
  })

  test('UploadImageMenu isActive always return false', () => {
    expect(menu.isActive(editor)).toBe(false)
  })
}) 
```

<div class="advantage" v-click>

  **优点：隔离测试对象，保证用例的独立性**

</div>

<style>
  .advantage {
    margin-top: 4em;
  }
</style>

---

```ts
function createLogan () {
	const logan = new LogAnalyzer()
  // logan.init()
  return logan
}	

test('LogAnalyzer isValid invalid fileName', () => {
	const logan = createLogan()
  const res = logan.isValid('12345')
  expect(res).toBeFalsy()
})
```

<div class="advantage" v-click>

  **优点：封装重复逻辑，提高测试的可维护性**
  
</div>

<style>
  .advantage {
    margin-top: 4em;
  }
</style>

---


# 集成测试
任何代码测试，如果它运行速度不快，结果不稳定，或者依赖被测试单元的一个或者多个依赖物（可能是一个服务，也可能是一个模块），这样的测试，一般称之为**集成测试**。

---

# 测试驱动（Test-Driven Development）

**TDD 解决的是何时编写单元测试的问题。**

---
layout: split
---

# 传统测试

<img src="/images/unit-traditional.png" style="max-width: 85%;" />

---
layout: split
---
# TDD

<img src="/images/tdd.png" style="max-width: 85%;" />

---

# TDD 的优势

- **提高代码质量，提前优化代码设计**

- **较早地发现缺陷，减少缺陷的数量**

---

# TDD 的缺点

- **时间成本高，可能导致项目延期**

- **使用不当，也可能降低代码的质量**

---
layout: center
---

# Q & A

