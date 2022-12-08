### vue文件跳转问题

使用ts的时候，我们喜欢用alias进行别名定义路径，然后ctrl+左键进行跳转。使用scf进行开发，在route.ts文件引入vue文件的时候会跳转到`.d.ts`文件（在使用@作为别名引入的情况），这种情况想要跳转到具体的文件，不能用@作为引用，用相对路径引用就没问题。Dec 8, 2022 22:01

### 使用tsx文件作为开发问题

当我们使用[@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx) 进行转js开发的时候，想像react的tsx开发效果一样，不使用defineComponent和setup进行开发，当不使用vue的生命周期函数，那是没问题的。比如
```vue
export const App = () => {
	return <div>Hello Vue</div>
}
```
但是，如果在纯函数当中使用了，使用了onMounted的时候，会出现
```
onMounted is called when there is no active component instance to be associated with. Lifecycle injection APIs can only be used during execution of setup(). If you are using async setup(), make sure to register lifecycle hooks before the first await statement.
```
onMounted必须使用在setup当中，因此，纯函数是不能作为核心开发方案的，必须套在defineComponent当中或者在scf当中。

Dec 8, 2022 22:11


### 使用vite开发插件是一个很简单的过程
我想要vite在root的同级引用一个public的文件的内容，很简单。
`static-directory.ts`
```ts
import { ViteDevServer } from 'vite';
import * as path from 'path';
import * as fs from 'fs';

type Props = {
  url: string;
  path: string;
};
export const StaticDirectory = (props: Props) => {
  const { path: _path, url } = props;
  return {
    name: 'configure-server',
    configureServer(server: ViteDevServer) {
      server.middlewares.use((req, res, next) => {
        if (req.url?.includes(url)) {
          const _url = req.url;
          const filepath = path.join(_path, _url);
          if (fs.existsSync(filepath)) {
            fs.createReadStream(filepath).pipe(res);
            return;
          }
        }
        next();
      });
    },
  };
};
```

配置,引用上一级的lib的库
```js
plugins: [
    StaticDirectory({ url: '/lib', path: path.join(process.cwd(), '..') }),
  ],
```
出现原因，一个仓库出现很多个demo的内容。
Dec 8, 2022 22:11

