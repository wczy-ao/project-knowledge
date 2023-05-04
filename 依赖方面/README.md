# 安装依赖问题汇总

### 依赖问题

1. 安装`antdv`报错

   ```
   官网的会报错，下面不会
   npm i --save ant-design-vue@next
   ```

2. 清除依赖

   ```
   yarn cache clean
   ```

3. node-sass

   ```
   npm rebuild node-sass
   ```

4. error：报一些axios之类的错误

   ```
   yarn config set "strict-ssl" false -g
   ```

   

### npm问题

1. 淘宝源

   ```
   // npm设置新淘宝源
   npm config set registry https://registry.npmmirror.com
   // npm设置回本源
   npm config set registry https://registry.npmjs.org
   ```

   

