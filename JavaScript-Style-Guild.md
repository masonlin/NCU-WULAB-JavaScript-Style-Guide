# JavaScript Style Guide

## 1 引言
    這份 JavaScript 程式撰寫風格指南是參考 Google JavaScript Style Guild 而來，主要目的是提供同學統一並學習其良好的開發風格。
    
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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ref: https://jsdoc.app/  
  
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

## 4 格式
    術語: 塊狀結構 (block-like construct) 指的是類別、函式、方法或大括號區塊的程式碼本體
    
    4.1 大括號
    
        4.1.1所有的控制結構都使用大括號
            例如if, else, for, do, while … 等等
	    
        4.1.2 非空白區塊: 
            開括號前不換行
            開括號後換行
            閉括號前換行
            閉括號後換行，但 else、catch、while、逗號、分號或右括號除外。
            例如:
```javascript
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
````

        4.1.3 空白區塊: 
            簡潔，例如:
```javascript
            function doNothing() {}
```

    4.2 區塊縮排: 
        2個空白，適用於程式碼及註釋。
	
        4.2.1 陣列: 
            區塊格式是可選的，應該需求而變化。例如:
```javascript
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
```

            其它的格式也是允許的，尤其強調語意而分組時，但切勿只為了減少大陣列的程式碼行數而為之。
	    
        4.2.3 類別
            不要在方法或類別區塊結尾加上分號，除非是個表示式。
```javascript
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
```

        4.2.4 函式表示式
            當匿名函式在函式中作為參數時，也是要縮排2個空格於其上層函式。
            例如:
```javascript
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
```		
		
        4.2.5 Switch 陳述式
            case 及 default 前都是縮排2空格，label後的新行接著 case 或者 default都是再縮排2空格。
```javascript
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
```

        4.3 陳述式
	
            4.3.1 一行一個陳述式
                一行一個陳述式  
		
            4.3.2 使用分號
                每個陳述式結尾都要使用分號。禁止使用自動插入分號。  
		
        4.4 每行長度限制: 80 字元或不要超過一個螢幕寛，先自行斟酌
            除了以下所示以外，JavaScript 每行長度限制為80碼，超過此長度請折行 (換行line-wrapped)，折行的位置請自行判斷。
            例外:  

* 程式碼中應可被點擊的長URL  

* 會被 copied-and-pasted 的shell 指令  

* 需要被完全複製或搜索的長字符串  

```javascript
```

        4.5 折行(行包裏 Line-wrapping)

            4.5.1 何時換行
                折行徧好以更高的語法層次 (higher syntactic level) 來進行。
                例如:
                首選
```javascript
                currentEstimate =
                    calc(currentEstimate + x * currentEstimate) /
		    2.0;
```
                遜色
```javascript
                currentEstimate = calc(currentEstimate + x *
                    currentEstimate) / 2.0;
```
                以上例而言其語法層次由上而下依序為: 等號、除號、函式呼叫、參數、常數。
                運算子的包裹 (wrapped) 法如下:

* 行中有運算子要換行時，應在該符號後換行。

* 方法或建構子後直接接著開括號 “(”。

* 逗號 “,” 緊接在前面的文字或符號。  

&nbsp;

                提醒: 折行的目的是為了讓程式碼更加清楚，無需執著在少行數上。
		
            4.5.2 折行的縮排至少要有4個空格
                至少有要有4個空格，除非是要符合這個折行區塊的規則。而同個層次的縮排則要一樣。  
		
        4.6 空白

            4.6.1 垂直空白行
                何時出現一個空白行:  

* 類別或物件裡的連續方法之間。
   例外情況: 
   類別中連續屬性間的邏輯分組可以選擇性地空一行。
   
* 在方法區塊裡的屬性邏輯分組要盡量僅慎。而函式裡的起始和結尾之間不允許空白。
* 在類別或物件的啟始或結尾的方法後是否空白是可選的,允許多行空白，但不鼓勵。  

&nbsp;

            4.6.2 水平空白
                此空白取決於所在位置，分為三類：頭 (如縮排)、尾 (被禁止) 和中間。
                單一空白只出現在以下地方:  

1. 除了function及super外，用以區隔保留字 (像是 if 、 for 、 或是 catch )，以及開括號 ”(” 後。

2. 閉大括號 “}” 後用以區隔保留字 (像是 else 或是 catch )。

3. 開大括號 “{” 前，除以下二者例外:  

    a. 放在函式的第一個參數或陣例中的第一個元素(例如: foo({a: [{c: d}]}))。  
    
    b. 模板擴展 (例如正規表示式，有效 `ab${1 + 2}cd`，無效 `xy$ {3}z`)。  
    
4. 任何運算符號二側。

5. 逗號及分號之後。

6. 冒號之後。

7. 陳述式後的註解用的雙斜線 (//) 二側，這裡也允許是多個空隔。

8. 區塊註解的啟始符號 (/**) 之後及其結束符號 (*/) 的二側，例如:

```javascript
                this.foo = /** @type {number} */ (bar)
                function(/** string */ foo) {
                baz(/* buzz= */ true)
```

            4.6.3 水平對齊
                以下二者皆允許但第二個不鼓勵  
```javascript
                {
                  tiny: 42, // this is great
                  longer: 435, // this too
                };
```
                下面這個不鼓勵
```javascript
                {
                  tiny:   42,  // permitted, but future edits
                  longer: 435, // may leave it unaligned
                };
```

            4.6.4 函式參數
                建議將所有參數放在與函式名同一行上，若過長需換行，則應依可讀的方式換行。換行則以一個參數為一行，縮排4毎空白；
                可以對齊括號，但不鼓勵，以下為例子:
```javascript
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
```	
        4.7 分組括號 略
	
        4.8 註解
            詳參考第7章 JSDoc
	    
            4.8.1 區塊註解
                第一行和最後一行無內容
```javascript		
                /*
                 * This is
                 * okay.
                 */
```
                多行這樣可以
```javascript
                // And so
                // is this.
```
                單行這樣也可以
```javascript
                /* This is fine, too. */
```		
                但單純的內容註解不要使用 JSDoc 的方式
```javascript
                /** … */
```

            4.8.2 參數的註解
                參數名稱不能完全傳達意思時使用
```javascript
                someFunction(obviousParam, true /* shouldRender */, 'hello' /* name */);	    
```

## 5 語言特徵
    Javascript 有一些語法曖昧不明或危險，這章在說明哪些可用或不可用。
    
    5.1 區域變數的宣告
    
        5.1.1 使用 const 和 let
            使用 const 和 let，不要再使用 var
	    
            5.1.2 一次宣告一個變數
                不要宣告成這樣let a = 1, b = 2;

            5.1.3 略

            5.1.3 略

        5.2 陣列所賦之值 (Array literals)

            5.2.1 行尾使用逗號 略

            5.2.2 不要使用變數作為 Array 建構子的引數
```javascript
            const a1 = new Array(x1, x2, x3);
            const a2 = new Array(x1, x2);
            const a3 = new Array(x1);
            const a4 = new Array();
```
            上例中 a3 中的 x1 若為正整數則沒問題，但若為正整數以外的其它數值則為產生 exception。若是其它值，則會成為單一元素的陣。
            明確地宣告長度是比較合適的作法，如 new Array(length)。
	    
            5.2.3 非數值屬性 略

            5.2.4 解構 略

            5.2.5 擴散運算子 (Spread operator)
```javascript
            [...foo]   // 優於 Array.prototype.slice.call(foo)
            [...foo, ...bar]   // 優於 foo.concat(bar)
```
        5.3 物件所賦之值 (Object literals)

            5.3.1 略

            5.3.2 略

            5.3.3 勿混合使用有引號和無引號的 key
                不允許:
```javascript
                {
                  width: 42, // struct-style unquoted key
                  'maxWidth': 43, // dict-style quoted key
                }
```
            5.3.4 動態賦予的屬性名稱 (Computed property names)
                例如 {['key' + foo()]: 42} 這樣建立 key 或屬性名稱是允許的

            5.3.5方法速記 (Method shorthand) 略

            5.3.6屬性速記 (Shorthand properties) 略

        5.4 類別

            5.4.1 Getters and Setters
                不要使用 Javascript 的 Getter 及 Setter，用一般的方法實作即可。因為在編譯上存在一些問題。

        5.5  函式
            5.5.1 略

            5.5.2 略

            5.5.3 箭頭函式 (Arrow functions)
                簡潔的函式寫法，且自動帶入 this，省去 fn.bind(this) 這種額外宣告的麻煩。
	
            5.5.4 Generators 略

            5.5.5 參數 略

            5.5.6 泛型 略

            5.5.7 擴散運算子
                如同 5.2.5，也可以如下使用
```javascript
                function myFunction(...elements) {}
                myFunction(...array, ...iterable, ...generator());
```
        5.6 字串所賦之值 (String literals)

            5.6.1 使用單引號
                一般字串值使用單引號，而非雙引號。

            5.6.2 模板所賦之值 (Template literals)
                (`${文字內容}`) 應用於多行的內容，如果是多行內容時，則不用遵守縮排的規範。
                例如:
```javascript
                function arithmetic(a, b) {
                  return `Here is a table of arithmetic operations:
${a} + ${b} = ${a + b}
${a} - ${b} = ${a - b}
${a} * ${b} = ${a * b}
${a} / ${b} = ${a / b}`;
                }
```
            5.6.2 非連續行 (No line continuations)
                即一行字串折成多行，不要使用連續行符號 (line continuations, \)，雖ES5 允許使用，但它可能會引起一些不明顯的錯誤。
                不要用:
```javascript
                const longString = 'This is a very long string that far exceeds the 80 \
                    column limit. It unfortunately contains long stretches of spaces due \
                    to how the continued lines are indented.';
```
                請使用 (+) 代替:
```javascript
                const longString = 'This is a very long string that far exceeds the 80 ' +
                    'column limit. It does not contain long stretches of spaces since ' +
                    'the concatenated strings are cleaner.';
```
        5.7 數字所賦之值 (Number literals)
            包含 10 進制、16進制 (0x)、8進制 (0o) 及 2 進制 (0b)。

        5.8 控制結構 (Control structures)

            5.8.1 For loops 略

            5.8.2 Exceptions
                發生例外時請丟出 new Error 物件或其字物件，勿只丟出一個字串或其它非相干物件。
                可在系統 Error 資訊不足時自定 Error 物件。
                遇到 Error 應立即丟出，而非傳遞它。

                5.8.2.1 空白的 catch 區塊
                    有時 catch 發生時我們會什麼都不做，這時可加上一些說明
```javascript
                    try {
                      return handleNumericResponse(response);
                    } catch (ok) {
                      // it's not numeric; that's fine, just continue
                    }
                    return handleTextResponse(response);
```
            5.8.2 Switch 語句 略

	5.9 this 略

        5.10 等於
            一般使用 ( === / !==)，以下為例外，例如預期值為 null 或 undefined 時:
```javascript
            if (someObjectOrPrimitive == null) {
              // Checking for null catches both null and undefined for objects and
              // primitives, but does not catch other falsy values like 0 or the empty string.
}
```
        5.11 不允許的功能
            不要用 with
            不要用 eval
            不要自動插入分號
            一些非標準或已被棄用的功能

## 6 命名

        6.1 通用的標示規則
            不要怕變數名稱太長，不要發表可能會被誤會或無法理解的縮寫。
```javascript
            errorCount          // 無縮寫
            dnsConnectionIndex  // DNS 大家都知道
            referrerUrl         // URL 同上
            customerId          // Id 你不知道是什麼嗎?
```
            不允許以下命名法
```javascript
            n                   // 無意義.
            nErr                // 不清楚是什麼的縮寫詞
            nCompConns          // 縮到不知其意
            wgcConnections      // 只有你的團隊知道這是什麼
            pcReader            // pc 可是以很多意思
            cstmrId             // 刪除單字中的字母
            kSecondsPerDay      // 不要使用匈牙利命名法，例 bBusy、nTimes
```

        6.2 命名規則

            6.2.1 Package 的命名
                lowerCamelCase 例如 my.exampleCode.deepSpace

            6.2.2 類別的命名
                UpperCamelCase 的方式。
                其命名方式通常是使用名詞或名詞詞組，
                例如: Request, ImmutableList, 或 VisibilityMode。
                而介面 interface 則有時使用形容詞詞組，例如 Readable。

            6.2.3 方法的命名
                lowerCamelCase 通常使用動詞或動詞詞組，例如: sendMessage。

            6.2.4 列舉的命名
                像類別命名一樣使用UpperCamelCase，並使用單數名詞，而裡面個別的項目則用全大寫字母，單字之間使用底線分隔表示，例如: CONSTANT_CASE。

            6.2.5 常數的命名
                使用全大寫如CONSTANT_CASE 方式表示，單字間使用底線分隔，而且是在module-local (top-level) 時使用全大寫表示，方法或函式裡的常數仍是使用 lowerCamelCase 來表示。
 
             6.2.6 一般變數的命名
                開頭小寫的駝峰式 lowerCamelCase，由名詞和名詞詞組組成，例如: computedValues。

            6.2.7 參數的命名
                開頭小寫的駝峰式 lowerCamelCase，建構函式也是如此。
                注意參數名稱不應只用一個字元來表示。
                例外: 有些第三方框架的參數名可能使用一個字元表示，例如: $。

            6.2.8區域變數的命名
                開頭小寫的駝峰式 lowerCamelCase，且在函式範圍裡的常數也是使用 lowerCamelCase 方式表示 (module-local 才使用全大寫)。

## 7 JSDoc
    JSDoc 可用於類別、欄位、方法上。

        7.1 一般形式
            一般 JSDoc 區塊長像下面這樣
```javascript
            /**
             * Multiple lines of JSDoc text are written here,
             *  wrapped normally.
             * @param {number} arg A number to do something to.
             */
```

        7.2 Markdown 略

        7.3 JSDoc 的標籤
            所有的標籤都應於行首。可參考 JSDoc Block Tags 或 JSDoc Tags。

        7.4 換行
            換行空4格。如下例:
```javascript
            /**
             * Illustrates line wrapping for long param/return descriptions.
             * @param {string} foo This is a param with a description too long to fit in
             *     one line.
             * @return {number} This returns something that has a description too long to
             *     fit in one line.
             */
            exports.method = function(foo) {
              return 5;
            };
```

        7.5 Top或文件級的註解
            通常是用來說明版權、作者資訊等。其中也包含整個人件是怎麼組成，或者是提供給不熟悉內容的讀者了解程式碼是在寫什麼。換行不縮排。例如:
```javascript
            /**
             * @fileoverview Description of file, its uses and information
             * about its dependencies.
             * @package
             */
```

        7.6 類別的註解
            類別、介面或模組應該要有註解說明其用及其參數、實現的介面、封裝狀況或其它適當的標籤。類別要充份說明如何使用它。如下例:
```javascript
            /**
             * A fancier event target that does cool things.
             * @implements {Iterable<string>}
             */
            class MyFancyTarget extends EventTarget {
              /**
               * @param {string} arg1 An argument that makes this more interesting.
               * @param {!Array<number>} arg2 List of numbers to be processed.
               */
              constructor(arg1, arg2) {
                // ...
              }
            };

            /**
             * Records are also helpful.
             * @extends {Iterator<TYPE>}
             * @record
             * @template TYPE
             */
            class Listable {
              /** @return {TYPE} The next item in line to be returned. */
              next() {}
            }
```

        7.7 列舉或型別定義的註解
            所有的列舉及型別定義都應要有適當的註解及 JSDoc 的標籤 (@typedef 或 @enum) 於行首。如下例:
```javascript
            /**
             * A useful type union, which is reused often.
             * @typedef {!Bandersnatch|!BandersnatchType}
             */
            let CoolUnionType;

            /**
             * Types of bandersnatches.
             * @enum {string}
             */
            const BandersnatchType = {
              /** This kind is really frumious. */
              FRUMIOUS: 'frumious',
              /** The less-frumious kind. */
              MANXOME: 'manxome',
            };
```

        7.8 方法和函式的註解
            如果方法是繼承而來的，請加上 @override 註解。若方法的JSDoc 或其簽名已足以表示其內意思，則其方法、
            參數或回傳的描述是可以省略的。
            若方法是繼承superclass而來的，則在方法上要注明 @override。方法繼承後也會把Superclass 的JSDoc 標籤一併繼承下來，
            固除非有額外需描述的內容，否則相同的內容不需再描述。
            例:
```javascript
            /** A class that does something. */
            class SomeClass extends SomeBaseClass {
              /**
               * Operates on an instance of MyClass and returns something.
               * @param {!MyClass} obj An object that for some reason needs detailed
               *     explanation that spans multiple lines.
               * @param {!OtherClass} obviousOtherClass
               * @return {boolean} Whether something occurred.
               */
              someMethod(obj, obviousOtherClass) { ... }

              /** @override */
              overriddenMethod(param) { ... }
            }

            /**
             * Demonstrates how top-level functions follow the same rules.  This one
             * makes an array.
             * @param {TYPE} arg
             * @return {!Array<TYPE>}
             * @template TYPE
             */
            function makeArray(arg) { ... }
```
