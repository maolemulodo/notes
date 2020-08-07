The following tutorial for setting ESlint and Prettier on VSCode

## Extenstions will installed on the VSCode

1. Prettier
   ![prettier extension](../assets/images/prettier-extension.png?raw=true)

2. ESlint
   ![eslint extension](../assets/images/eslint-extension.png?raw=true)

## Setting for editor of VSCode

1. Hold key Command + ,
2. Click setting symbol at the top right conner
   ![Open setting file](../assets/images/setting-file.png?raw=true)
3. Addding these following configuration

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true
}
```

## Packages will installed

```yarn
yarn add -D eslint-config-prettier eslint-plugin-prettier eslint-plugin-react husky lint-staged prettier
```

## Setting for the ESlint

Please refer at the [.eslintrc](../assets/configs/.eslintrc) file

## Setting for the prettier

Please refer at the [.prettierrc](../assets/configs/.prettierrc) file

## Note for anyone doesn't use the VSCode

We add `husky` package for automation format with `prettier --write`.
I mean that the source code will formatted automatically before the source code is commited to the repository

```json
"husky": {
   "hooks": {
     "pre-commit": "lint-staged"
   }
},
"lint-staged": {
   "src/**/*.{js, jsx, json, css}": [
     "prettier --write",
     "git add"
   ]
},
```
