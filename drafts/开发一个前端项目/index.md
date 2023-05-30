# 前端项目脚手架


## 技术栈
* pnpm
* vite
* React
* TypeScript
* Tailwind CSS

## 详情
1. 创建项目
```shell
# 使用 create-react-app 创建一个 typescript 格式的项目
pnpm create vite [project-name] --template react-ts

cd [project-name] 
pnpm install
pnpm run dev
```
项目的目录结构和页面展示如下图所示

2. 修改相对路径
在 tsconfig.json 中的 compilerOptions 下添加**"baseUrl": "./src",** ，tsconfig.json 为
```shell
{
  "compilerOptions": {
    "baseUrl": "./src", # 添加的路径
    "target": "es5",
    # ... 
}   
```
3. 添加代码格式化工具
参考[prettier官方文档](https://prettier.io/docs/en/install.html)
```shell
# 下载依赖
yarn add --dev --exact prettier
# 创建配置
echo {}> .prettierrc.json
# 创建 ignore
touch .prettierignore
```
4. 添加 precommit-hook
参考[precommit](https://prettier.io/docs/en/precommit.html)
```shell
# 添加 precommit 依赖
npx mrm@2 lint-staged
```
5. lint
```shell
# 安装 eslint-config-prettier 依赖
yarn add  eslint-config-prettier -D
# package.json 的 eslintConfig.extends 中添加
 "prettier"
```
6. 规范 commit
参考[commitlint README](https://github.com/conventional-changelog/commitlint)
```shell
# 安装依赖
yarn add @commitlint/{config-conventional,cli}
yarn add @commitlint/config-conventional @commitlint/cli

# 创建commitlint.config.js
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
# 在package.json中添加
"commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
```

7. 引入 tailwind
```shell
pnpm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
# 创建 tailwind.config.js 文件
touch tailwind.config.js
```
并在 tailwind.config.js 中写入
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

## mock 方案
1. 写死数据

2. mock.js 

3. 接口管理工具
yapi，swagger，rap
4. json-server


## 常见问题的解决方法
1. Cannot find module Please verify that the package.json has a valid "main" entry nuxtjs error on npm run dev
```
rm -rf node_modules
pnpm install
```


## 相关链接
[vite 中文文档](https://cn.vitejs.dev/guide/)
[pnpm 中文网](https://www.pnpm.cn/)
