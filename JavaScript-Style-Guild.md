# JavaScript Style Guide

## 1 引言

## 2 檔名

    2.1 檔案命名 英文小寫、底線 \_ 或 dash \-

    2.2 檔案編碼 UTF-8

    2.3 特殊字元

        2.3.1 空白字元
            用一般空白 ASCII (0x20) 縮排不要使用 Tab 縮排

    2.3.2 特殊跳脫序列

    2.3.3 非 ASCII 字元

## 3 檔案結構
    3.1 版權資訊

    3.2 @fileoverview JSDoc
        自行斟酌內容及使時機，但結構以JSDoc 為主 (參考7.2)。
        Ref: [https://jsdoc.app/](https://jsdoc.app/ https://jsdoc.app/)

    3.3 goog.module statement

    3.4 ES 模組

        3.4.1 Imports
            Import 語法是不能換行的，所以長度不受80字元長度限制

            3.4.1.1 Import 路徑
                例如: 
```javascript
                import './sideeffects.js';
                import * as goog from '../closure/goog/goog.js';
                import * as parent from '../parent.js';
                import {name} from './sibling.js';
```

                3.4.1.1.1 import 時的副檔名
                    Import 時必須包含副檔名
                    例如:
```javascript
                    import '../directory/file.js';
```

            3.4.1.2 同個檔案 import 多次
                不要同個檔案 import 多次
                例如不要這樣做:
```javascript
                import {short} from './long/path/to/a/file.js';
                import {aLongNameThatBreaksAlignment} from './long/path/to/a/file.js';
```

            3.4.1.3 import 檔案的命名
                3.4.1.3.1 import 模組時的命名
                    使用 lowerCamelCase 的命名方式，通常與 import 的檔案同名
                    例如:
```javascript
                    import * as fileOne from '../file-one.js';
                    import * as fileTwo from '../file_two.js';
                    import * as fileThree from '../filethree.js';
                    import * as libString from './lib/string.js';
                    import * as math from './math/math.js';
                    import * as vectorMath from './vector/math.js';
```

                3.4.1.3.2 一般預設 import 的命名方式
                    預設與檔案同名，也依循 6.2 識別規則來命名
                    例如:
```javascript
                    import MyClass from '../my-class.js';  // my-class.js 裡面只有 MyClass 這個類別
                    import myFunction from '../my_function.js'; // my_function.js 裡面只有 function(s)
                    import SOME_CONSTANT from '../someconstant.js'; //someconstant.js 裡面只有常數
```
                3.4.1.3.3 模組裡的類別或函式import 的命名
                    即 ( import {name} ) 這樣的 import ，其名應該保持與檔名相同，避免改名
                    如 ( import {SomeThing as SomeOtherThing} )
                    例如:
```javascript
                    import * as bigAnimals from './biganimals.js';
                    import * as domesticatedAnimals from './domesticatedanimals.js';

                    new bigAnimals.Cat();
                    new domesticatedAnimals.Cat();
```
                    如果仍要改名，則使用其檔名和路徑結合此類別名來改
                    例如:
```javascript
                    import {Cat as BigCat} from './biganimals.js';
                    import {Cat as DomesticatedCat} from './domesticatedanimals.js';

                    new BigCat();
                    new DomesticatedCat();
```
            3.4.2 Exports

                3.4.2.1 命名與 default exports
                    不要使用 default export，若使用則在 import 時還要特給他命名，易造成模組名的不一致。
                    例如:
```javascript
                    // Do not use default exports:
                    export default class Foo { ... } // BAD!
                    // Use named exports:
                    export class Foo { ... }
                    // Alternate style named exports:
                    class Foo { ... }

                    export {Foo};
```
                    Ref: export 與 export default 的差異請自行斟酌使用

                3.4.2.2 匯出靜態變數、類別或物件
                    不要為了命名空間而匯出只有靜態變數的類別，或硬把常數塞給類別
                    例如:
```javascript
                    // container.js
                    // Bad: Container is an exported class that has only static methods and fields.
                    export class Container {
                      /** @return {number} */
                      static bar() {
                        return 1;
                      }
                    }

                    /** @const {number} */
                    Container.FOO = 1;
```
                    應該這樣做
```javascript
                    /** @return {number} */
                    export function bar() {
                      return 1;
                    }

                    export const /** number */ FOO = 1;
```

                3.4.2.3 exports 的變異性 (Mutability of exports)
                    匯出的變數不得在模組初始化之外被變異。
                    (“變異”指的是對產品代碼的一個改變，這種改變會導致代碼的行爲發生變化。)
                    (即類似物件導向程式設計的”封裝”)
                    例如:
                    不好的方式，這個模組匯出後可以被在其外變更其變數內容。
```javascript
                    // Bad: both foo and mutateFoo are exported and mutated.
                    export let /** number */ foo = 0;

                    /**
                     * Mutates foo.
                     */
                    export function mutateFoo() {
                      ++foo;
                    }

                   /**
                    * @param {function(number): number} newMutateFoo
                    */
                   export function setMutateFoo(newMutateFoo) {
                   // Exported classes and functions can be mutated!
                     mutateFoo = () => {
                       foo = newMutateFoo(foo);
                     };
                   }
```
                   如果有需要變更模組裡的變數，則應該透過函式來修改它，實現JS的封裝能力
```javascript
                   // Good: Rather than export the mutable variables foo and mutateFoo directly,
                   // instead make them module scoped and export a getter for foo and a wrapper for
                   // mutateFooFunc.
                   let /** number */ foo = 0;
                   let /** function(number): number */ mutateFooFunc = foo => foo + 1;

                   /** @return {number} */
                   export function getFoo() {
                     return foo;
                   }

                   export function mutateFoo() {
                     foo = mutateFooFunc(foo);
                   }

                   /** @param {function(number): number} mutateFoo */
                   export function setMutateFoo(mutateFoo) {
                     mutateFooFunc = mutateFoo;
                   }
```

                3.4.2.4 export from
                    export from 語法因為不得換行，所以不受80字元長度的限制。
```javascript
                    export {specificName} from './other.js';
                    export * from './another.js';
```

            3.4.3 模組間的循環依賴
		雖然 ESMAScript 規格允許，但不要這樣玩 import 和 export。

            3.4.4 Interoperating with Closure

            3.4.4.1 Referencing goog

                3.4.4.2 goog.require in ES modules

                3.4.4.3 Declaring Closure Module IDs in ES modules

        3.5 goog.setTestOnly

        3.6 goog.require and goog.requireType statements

        3.7 實作程式檔
            先宣告完所有的引用，至少要再空一行後，再開始撰寫程式。
	    
4 格式
		術語: 塊狀結構 (block-like construct) 指的是類別、函式、方法或大括號區塊的程式碼
本體
4.1 大括號
	4.1.1所有的控制結構都使用大括號
		例如if, else, for, do, while … 等等
4.1.2 非空白區塊: 
	開括號前不換行
	開括號後換行
	閉括號前換行
閉括號後換行，但 else、catch、while、逗號、分號或右括號除外。
	例如:
class InnerClass {
	constructor() {}

	/** @param {number} foo */
	method(foo) {
		if (condition(foo)) {
			try {
 				// Note: this might fail.
 				something();	
			} catch (err) {
 				recover();
			}
		}
	}
}
4.1.3 空白區塊: 簡潔
	例如: 
	function doNothing() {}
4.2 區塊縮排: 
2個空白，適用於程式碼及註釋。
	4.2.1 陣列: 
區塊格式是可選的，應該需求而變化。
例如:
const a = [
	0,
	1,
	2,
];

const b =
	[0, 1, 2];
const c = [0, 1, 2];
someMethod(foo, [
	0, 1, 2,
], bar);
其它的格式也是允許的，尤其強調語意而分組時，但切勿只為了減少大陣列的
程式碼行數而為之。
4.2.3 類別
				不要在方法或類別區塊結尾加上分號，除非是個表示式。
				/** @extends {Foo<string>} */
foo.Bar = class extends Foo {
/** @override */
  method() {
    return super.method() / 2;
  }
};
				
/** @interface */
class Frobnicator {
/** @param {string} message */
  frobnicate(message) {}
}
4.2.4 函式表示式
	當匿名函式在函式中作為參數時，也是要縮排2個空格於其上層函式。
	例如:
	prefix.something.reallyLongFunctionName('whatever', (a1, a2) => {
// Indent the function body +2 relative to indentation depth
// of the 'prefix' statement one line above.
if (a1.equals(a2)) {
someOtherLongFunctionName(a1);
} else {
andNowForSomethingCompletelyDifferent(a2.parrot);
}
});

some.reallyLongFunctionCall(arg1, arg2, arg3)
.thatsWrapped()
.then((result) => {
// Indent the function body +2 relative to the indentation depth
// of the '.then()' call.
if (result) {
result.use();
}
});
4.2.5 Switch 陳述式
case 及 default 前都是縮排2空格，label後的新行接著 case 或者 default都是再
縮排2空格。
switch (animal) {
  case Animal.BANDERSNATCH:
    handleBandersnatch();
    break;

  case Animal.JABBERWOCK:
    handleJabberwock();
    break;

  default:
    throw new Error('Unknown animal');
}
4.3 陳述式
4.3.1 一行一個陳述式
	一行一個陳述式
4.3.2 使用分號
	每個陳述式結尾都要使用分號。禁止使用自動插入分號。
4.4 每行長度限制: 80 字元或不要超過一個螢幕寛，先自行斟酌
	除了以下所示以外，JavaScript 每行長度限制為80碼，超過此長度請
折行 (換行line-wrapped)，折行的位置請自行判斷。
例外:
-	程式碼中應可被點擊的長URL
-	會被 copied-and-pasted 的shell 指令
-	需要被完全複製或搜索的長字符串
4.5 折行(行包裏 Line-wrapping)
4.5.1 何時換行
		折行徧好以更高的語法層次 (higher syntactic level) 來進行。
		例如:
		首選
		currentEstimate =
calc(currentEstimate + x * currentEstimate) /
2.0;
		遜色
currentEstimate = calc(currentEstimate + x *
currentEstimate) / 2.0;
以上例而言其語法層次由上而下依序為: 等號、除號、函式呼叫、參數、常數。
運算子的包裹 (wrapped) 法如下:
1.	行中有運算子要換行時，應在該符號後換行。
2.	方法或建構子後直接接著開括號 “(”。
3.	逗號 “,” 緊接在前面的文字或符號。
提醒: 折行的目的是為了讓程式碼更加清楚，無需執著在少行數上。
4.5.2 折行的縮排至少要有4個空格
	至少有要有4個空格，除非是要符合這個折行區塊的規則。而同個層次的縮排
	則要一樣。
4.6 空白
	4.6.1 垂直空白行
		何時出現一個空白行:
1.	類別或物件裡的連續方法之間。
			例外情況: 
			類別中連續屬性間的邏輯分組可以選擇性地空一行。
2.	在方法區塊裡的屬性邏輯分組要盡量僅慎。而函式裡的起始和結尾之間不允許空白。
3.	在類別或物件的啟始或結尾的方法後是否空白是可選的,
允許多行空白，但不鼓勵。
4.6.2 水平空白
	此空白取決於所在位置，分為三類：頭 (如縮排)、尾 (被禁止) 和中間。
	單一空白只出現在以下地方:
1.	除了function及super外，用以區隔保留字 (像是 if 、 for 、 或是 catch )，以及開括號 ”(” 後。
2.	閉大括號 “}” 後用以區隔保留字 (像是 else 或是 catch )。
3.	開大括號 “{” 前，除以下二者例外:
a.	放在函式的第一個參數或陣例中的第一個元素(例如: foo({a: [{c: d}]}))。
b.	模板擴展 (例如正規表示式，有效 `ab${1 + 2}cd`，無效 `xy$ {3}z`)。
4.	任何運算符號二側。
5.	逗號及分號之後。
6.	冒號之後。
7.	陳述式後的註解用的雙斜線 (//) 二側，這裡也允許是多個空隔。
8.	區塊註解的啟始符號 (/**) 之後及其結束符號 (*/) 的二側，例如:
this.foo = /** @type {number} */ (bar)
function(/** string */ foo) {
baz(/* buzz= */ true)
4.6.3 水平對齊
	以下二者皆允許但第二個不鼓勵
{
  	    tiny: 42, // this is great
  	    longer: 435, // this too
};
	下面這個不鼓勵
{
  	    tiny:      42,  // permitted, but future edits
  	    longer: 435, // may leave it unaligned
};
4.6.4 函式參數
建議將所有參數放在與函式名同一行上，若過長需換行，則應依可讀的方式換行。換行則以一個參數為一行，縮排4毎空白；可以對齊括號，但不鼓勵，以下為例子:



//參數換行縮排4空格，適用在函式名稱很長時；參數放第二行時盡量全在這行
                           doSomething(
    				        descriptiveArgumentOne, descriptiveArgumentTwo, descriptiveArgumentThree) {
 					 // …
}
//參數名稱長時，不建議這樣用
doSomething(veryDescriptiveArgumentNumberOne, veryDescriptiveArgumentTwo,
    tableModelEventHandlerProxy, artichokeDescriptorAdapterIterator) {
  // …
}
//改這樣，一個參數一行
				doSomething(
    					veryDescriptiveArgumentNumberOne,
    					veryDescriptiveArgumentTwo,
    					tableModelEventHandlerProxy,
    					artichokeDescriptorAdapterIterator) {
  				   // …
}
4.7 分組括號
       略
4.8 註解
      詳參考第7章 JSDoc
4.8.1 區塊註解
 	 第一行和最後一行無內容
       	/*
 		  * This is
 		  * okay.
 		  */
              多行這樣可以
// And so
// is this.
		單行這樣也可以
/* This is fine, too. */
  	       但單純的內容註解不要使用 JSDoc 的方式
		/** … */
4.8.2 參數的註解
	參數名稱不能完全傳達意思時使用
	someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);
	    
