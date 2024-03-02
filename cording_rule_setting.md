# Add ESLint & Prettier & husky

## Setting

### 1.Install command

```
$ cd {your_project}
$ npm init
$ npm install --save-dev typescript
$ npm install --save-dev eslint eslint-config-prettier prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin husky lint-staged

$ touch tsconfig.eslint.json
$ touch .eslintrc.js
$ touch .prettierrc
```

### 2.Set package.json

Add scripts

```
{
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "lint": "eslint --fix 'src/**/*.{js,ts}'",
    "format": "prettier --write",
    "lint-fix": "eslint --fix './src/**/*.{js,ts}' && prettier --write './src/**/*.{js,ts}'"
  },
}
```

Add husky & lint-staged

```
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,ts,tsx}": [
      "npm run lint-fix"
    ]
  }
}
```

### 3.Write

- husky & eslint & prettier demo
  https://github.com/MusashiSaito/ts-husky-demo
- eslint-demo
  https://github.com/MusashiSaito/ts-eslint-demo

- prettier-demo
  https://github.com/MusashiSaito/ts-prettier-demo

## If husky doesn't run

＊動かない場合、一度 husky をインストールし直してください。

```
$ npm uninstall husky
$ npm install -D husky@4.3.8
```

## Docs

### Rules

- ESLint Rules Reference
  https://eslint.org/docs/latest/rules/
  https://eslint.org/docs/latest/rules/camelcase

- ESLint で TypeScript のコーディング規約チェックを自動化
  https://typescriptbook.jp/tutorials/eslint

### Invalid rules

部分的にルールを無効にするには、その行の前にコメント eslint-disable-next-line を追加します。

```
// eslint-disable-next-line camelcase
export const hello_world = "Hello World";
```
