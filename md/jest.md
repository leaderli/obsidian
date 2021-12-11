
针对 [[vue3]] [[vite]][[typescript]]的前端测试框架


```shell
npm i jest -D

# eslint 支持
npm i eslint-plugin-jest -D

# jest 运行在node环境，不支持es6语法，需要配置babel
npm i babel-jest @babel/core @babel/preset-env -D

# typescript 支持
npm i ts-babel  ts-jest types/jest -D

# vue3 支持
npm i  @vue/test-utils@next vue3-jest -D 
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
    transform: {  
        '\\.ts$': 'ts-jest',  
        '^.+\\.vue$': 'vue3-jest'  
 },  
    moduleFileExtensions: ['ts', 'js', 'json'],  
  
    moduleNameMapper: {  
        '^@/(.*)$': '<rootDir>/src/$1' //用于解决alias  
 },  
    roots: ['<rootDir>/src'] //测试文件路径  
};
```

`jest.setup.js`

```js
```