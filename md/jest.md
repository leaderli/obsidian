[📒 Getting Started · Jest](https://jestjs.io/docs/getting-started)

针对 [[vue3]] [[vite]][[typescript]]的前端测试框架


```shell
npm i jest -D

# eslint 支持
npm i eslint-plugin-jest -D

# jest 运行在node环境，不支持es6语法，需要配置babel
npm i babel-jest @babel/core @babel/preset-env -D

# typescript 支持
npm i ts-babel  ts-jest  @types/jest -D

# vue3 支持
npm i  @vue/test-utils@next vue3-jest -D 

# pinia 支持

npm i  @pinia/testing -D
```


新增配置`babel.config.js`
```js
module.exports = {  
  
    presets: [  
        [  
            '@babel/preset-env',  
            {  
                targets: {  
                    node: true  
 }  
            }  
        ]  
    ]  
};
```

`.eslintrc.js`新增配置项

```js
module.exports = {
	env: {  
 		jest:true // 支持jest测试语法糖  
	},
	plugins: ['@typescript-eslint','jest'],
}
```


新增配置`jest.config.js`

```js
/** @type {import('ts-jest/dist/types').InitialOptionsTsJest} */  
module.exports = {  
    globals: {  
        // work around: https://github.com/kulshekhar/ts-jest/issues/748#issuecomment-423528659  
 'ts-jest': {  
            diagnostics: {  
                ignoreCodes: [151001]  
            }  
        }  
    },  
    setupFiles: ['./jest.setup.js'],  
    preset: 'ts-jest',  
    testPathIgnorePatterns: ['/node_modules/', 'dist'],  
    modulePathIgnorePatterns: ['/node_modules/', 'dist'],  
    testEnvironment: 'jsdom',  
  
    // 文件需要进行转换才能执行测试  
 transform: {  
        '\\.ts$': 'ts-jest',  
        '^.+\\.vue$': 'vue3-jest'  
 },  
    // 仅执行特定格式的测试文件  
 testMatch:[  
        '**/__tests__/**/*.spec.ts',  
    ],  
    moduleFileExtensions: ['ts', 'js', 'json'],  
  
    moduleNameMapper: {  
        '^@/(.*)$': '<rootDir>/src/$1' //用于解决alias  
 },  
    roots: ['<rootDir>/src'] //  测试文件根目录  
};
```

`jest.setup.js`

```js
/* eslint-disable @typescript-eslint/no-var-requires */  
const { createTestingPinia } = require('@pinia/testing');  
  
const { config } = require('@vue/test-utils');  
  
config.global.stubs = {};  
config.global.plugins = [createTestingPinia()]; //vue组件的全局配置文件  
  
process.addListener('unhandledRejection', (err) => console.error(err));
```


测试文件`src/components/about/__tests__/li-icon.spec.ts`

```js
import { mount } from '@vue/test-utils';  
  
import about from '../index.vue';  
  
test('displays message', () => {  
    const wrapper = mount(about, {  
  
        // 可以覆盖jest.setup.js中设置的全局的属性  
 //例如 global.plugins = [createTestingPinia()]; //vue组件的全局配置文件 });  
    expect(wrapper.text()).toContain('暖的像火炉的暖男');  
    expect(wrapper.text()).toContain('1110');  
});
```


`package.json`

```js
"scripts": {
     "test": "jest"
   },

```


执行测试

```shell
npm run test
```