# [TypeScript] TSConfig Reference

> https://www.typescriptlang.org/tsconfig



## TSConfig Introduction

- 디렉토리 내에 위치한 TSConfig 파일은 해당 디렉토리가 TS 또는 JS 프로젝트의 루터 폴더임을 나타낸다.
- TSConfig 파일은 `tsconfig.json` 또는 `jsconfig.json`이라는 이름으로 작성될 수 있다.
  - 둘 다 같은 config variable 집합을 가지고 있다. 



### 탑레벨 옵션들의 종류

- "compilerOptions"
- "files"
- "extends"
- "include"
- "exclude"
- "references"



## Files - `"files" ` 옵션

- 프로그램 내에 포함시킬 파일들의 리스트를 명시한다.

- 명시한 파일들 중 하나라도 찾을 수 없다면 에러가 발생한다.

- example

  ```json
  {
    "compilerOptions": {},
    "files": [
      "core.ts",
      "sys.ts",
      "types.ts",
      "scanner.ts",
      "parser.ts",
      "utilities.ts",
      "binder.ts",
      "checker.ts",
      "tsc.ts"
    ]
  }
  ```

- 이 옵션은 사용할 파일의 개수가 적어서 많은 파일들을 참조하기 위해 glob을 사용할 필요가 없을 때 유용하다.

- 많은 파일을 사용해야 하면, `"include"` 옵션을 사용하자.



**Default**: `false`

**Related**: `"include"`, `"exclude"`



## Extends - `"extends"` 옵션

- extends 옵션은 상속받을 다른 configuration file을 지정할 때 사용된다.
- base file의 설정 값들이 먼저 로딩된 다음에, inheriting 설정 파일의 값들로 override 된다.



## Include - `"include"` 옵션

- 프로그램에 포함시킬 파일들을 파일명이나 pattern으로 배열 내에 기술한다.

- 이 파일명들은 `tsconfig.json` 파일이 위치한 디렉토리(루트 디렉토리)를 기준으로 resolved 된다.

- ex)

  ```json
  {
    "include": ["src/**/*", "tests/**/*"]
    // src 폴더와 tests 폴더 하위의 모든 파일들을 포함시켜라.
  }
  ```

  위와 같이 작성하면

  ```
  .
  ├── scripts                ⨯
  │   ├── lint.ts            ⨯
  │   ├── update_deps.ts     ⨯
  │   └── utils.ts           ⨯
  ├── src                    ✓
  │   ├── client             ✓
  │   │    ├── index.ts      ✓
  │   │    └── utils.ts      ✓
  │   ├── server             ✓
  │   │    └── index.ts      ✓
  ├── tests                  ✓
  │   ├── app.test.ts        ✓
  │   ├── utils.ts           ✓
  │   └── tests.d.ts         ✓
  ├── package.json
  ├── tsconfig.json
  └── yarn.lock
  ```

  위와 같은 파일들이 포함된다.

- 참고로 `"include"`와 `"exclude"` 옵션들은 와일드카드 문자들을 지원해 glob pattern들을 만들 수 있게 해준다.

  - `*`: 0개 이상의 문자 (excluding directory separators)
  - `?`: 문자 1개 (excluding directory separators)
  - `**/`: any directory nested to any level

- 만약에 glob pattern에 파일 확장자명을 기술하지 않는 경우, `.ts`, `.tsx`, `.d.ts`같이 지원되는 파일명만 포함된다. (`allowJs` 옵션이 true로 설정되면 `.js`, `.jsx` 까지 포함)

   

**Default:** 

- `"files"` 옵션이 명시된 경우: `[]`
- 그렇지 않은 경우: `**`

**Related**: `"files"`, `"exclude"`



## References - `"references"` 옵션

Project references are a way to structure your TypeScript programs into smaller pieces. Using Project References can greatly improve build and editor interaction times, enforce logical separation between components, and organize your code in new and improved ways.

You can read more about how references works in the [Project References](https://www.typescriptlang.org/docs/handbook/project-references.html) section of the handbook

- Default:

  `false`



## Compiler Options - `"compilerOptions"` 옵션



### `"modules"` 옵션

- 프로그램의 module 시스템을 설정한다.
- [Modules](https://www.typescriptlang.org/docs/handbook/modules.html) 레퍼런스 페이지를 참고하자.
- `"module"` 옵션을 바꾸는 것은 `"moduleResolution"` 옵션에도 영향을 미친다.

**Default:**

- "target" 옵션이 `"es3"` 또는 `"es5"`이면 `"commonjs"`
- 그렇지 않은 경우 `"es6/es2015"`

**Allowed**:

- "none"
- "commonjs"
- "amd"
- "umd"
- "system"
- "es6/es2015"
- "es2020"
- "es2022"
- "esnext"
- "node12"
- "nodenext"

**Related**: `"moduleResolution"`

- example

  ```json
  {
  	"compilerOptions": {
      "module": "commonjs",
      "target": "es5"
      ...
    }
  }
  ```

  



### `"moduleResolution"` 옵션

- module resolution 전략을 명시하는 옵션이다.
  - `"node"`: node.js의 CommonJS 구현
  - `"node12"` 또는 `"nodenext"`: Node.js의 ECMAScript Module Support [from TypeScript 4.5 onwards](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/)
  - `"classic"`: 모던 코드에서는 사용할 필요가 없는 옵션이다.



**Default:**

- `Classic` if [`module`](https://www.typescriptlang.org/tsconfig#module) is `AMD`, `UMD`, `System` or `ES6`/`ES2015`,
- Matches if [`module`](https://www.typescriptlang.org/tsconfig#module) is `node12` or `nodenext`, `Node` otherwise.

**Allowed:**

- `classic`
- `node`

**Related:**

- [`module`](https://www.typescriptlang.org/tsconfig#module)











