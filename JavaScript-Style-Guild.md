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
        Ref: https://jsdoc.app/
	
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
如果有需要變更模組裡的變數，則應該透過函式來修改它，實現JS的封裝能力
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
3.4.2.4 export from
	export from 語法因為不得換行，所以不受80字元長度的限制。
	export {specificName} from './other.js';
export * from './another.js';
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
