参加或留意了最近举行的[JSConf CN 2017](https://www.zhihu.com/question/62154473/answer/200045569)的同学，想必对 Next.js 不再陌生， Next.js 的作者之一到场进行了精彩的演讲。其实在更早些时候，由 Facebook 举办的 React Conf 2017，他就到场并有近40分钟的分享。但两次分享带来的 demo 都是 hacker news。我观察 Next.js 时间较长，看着它从1.x 版本一直到了今天的 3.x，终于决定写一篇入门级的新手指导文章。而这篇文章试图通过一个全新的例子，来让大家了解 Next.js 到底是如何与 React 配合，达到服务端渲染的。

“React universal” 是社区上形容基于 React 构建 web 应用，并采用“服务端渲染”方式的一个词语。也许很多人对 “isomorphic” 这个单词更加熟悉，其实这两个词语想要表达的概念类似。今天这篇文章显然不是讨论这两个词语的，**我们要尝试使用最新版 [Next.js](https://github.com/zeit/next.js)，构件一个简单的服务端渲染 React 应用。**最终项目地址可以[点击这里查看。](https://github.com/HOUCe/server-side-cache-with-express)

## 为何要开发 Universal 应用？

React app 实现了虚拟 DOM，来实现对真实 DOM 的抽象。这样的设计迅速引领了前端开发浪潮。但是 “Every great thing comes with a price”，虚拟 DOM 同样带来了一些弊端，比如在前后端分离的开发模式下，SEO就成了问题；同样首屏加载时间变长，各种 loading 消磨人的耐心。就像下面截图所展现的那样：

![页面](http://upload-images.jianshu.io/upload_images/4363003-0caa6abef06e4c3a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![查看网页源码](http://upload-images.jianshu.io/upload_images/4363003-fec93cb7b90b4938.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 使用 Next.js 实现 Universal

Universal 应用架构可以简单粗暴先而片面的理解成应用将在客户端和服务端共同完成渲染。这样取代了完全由客户端渲染（前后端分离方式）模式。在 React 场景下，我们可以使用 React 自身的 renderToString 完成服务端初次渲染。但是如果我们每次手动来完成这些过程，手动实现服务端繁琐配置，难免令人头大心烦。

Next.js 的出现，就是为你解决这种恼人的问题。我们先来认识一下它的几个原则和思想：

- 不需要除 Next 之外，多余的配置和安装（比如 webpack，babel）；
- 使用 [Glamor](https://github.com/threepointone/glamor) 处理样式；
- 自动编译和打包；
- 热更新；
- 方便的静态资源管理；
- 成熟灵活的路由配置，包括路由级别 prefetching；


## Demo：英超联赛积分榜

其实关于更多的 Next.js 设计理念我不想再赘述了，读者都可以在其官网找到丰富的内容。下面，我将使用 Football Data API 来简单开发一个基于 Next.js 的应用，这个应用将展现英超联赛的实时积分榜。同时包含了简单的路由开发和页面跳转。

### 小试牛刀

相信所有的开发者都厌恶超长时间的安装和各种依赖、插件配置。不要担心，Next.js 作为一个独立的 npm package 最大限度的替你完成了很多耗时且无趣的工作。我们首先需要进行安装：

    # Start a new project
    npm init
    # Install Next.js
    npm install next --save
    
安装结束后，我们就可以开启脚本：

    "scripts": {
       "start": "next"
     },
     
Next 安装的同时，也会安装 React，所以无需自己费心。接下来所需要做的很简单，就是在根目录下创建一个 pages 文件夹，并在其下新建一个 index.js 文件：

    // ./pages/index.js

    // Import React
    import React from 'react'
    
    // Export an anonymous arrow function
    // which returns the template
    export default () => (
      <h1>This is just so easy!</h1>
    )

好了，现在就可以直接看到结果：

    # Start your app
    npm start
    


![页面](http://upload-images.jianshu.io/upload_images/4363003-b1afa6031e077f67.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



验证一下它来自服务端渲染：



![查看网页源码](http://upload-images.jianshu.io/upload_images/4363003-e1310faf126ca61e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





就是这么简单，清新。如果我们自己手段实现这一切的话，除了 NodeJS 的种种繁琐不说，webpack 配置，node_modules 依赖，babel插件等等就够折腾半天的了。


## 添加 Page Head

在 ./pages/index.js 文件内，我们可以添加页面 head 标签、meta 信息、样式资源等等：
    
    // ./pages/index.js
    import React from 'react'
    // Import the Head Component
    import Head from 'next/head'
    
    export default () => (
      <div>
        <Head>
            <title>League Table</title>
            <meta name="viewport" content="initial-scale=1.0, width=device-width" />
            <link rel="stylesheet" href="https://unpkg.com/purecss@0.6.1/build/pure-min.css" />
        </Head>
        <h1>This is just so easy!</h1>
      </div>
    )

这个 head 当然不是指真实的 DOM，千万别忘了 React 虚拟 DOM 的概念。其实这是 Next 提供的 Head 组件，不过最终一定还是被渲染成为真实的 head 标签。


## 发送 Ajax 请求

Next 还提供了 getInitialProps 方法，这个方法支持异步选项，并且是服务端/客户端同构的。我们可以使用 async/await 方式，处理异步请求。请看下面的示例：

    import React from 'react'
    import Head from 'next/head'
    import axios from 'axios';
    
    export default class extends React.Component {
        // Async operation with getInitialProps
        static async getInitialProps () {
            // res is assigned the response once the axios
            // async get is completed
            const res = await axios.get('http://api.football-data.org/v1/competitions/426/leagueTable');
            // Return properties
            return {data: res.data}
          }
     }
     
我们使用了 axios 类库来发送 HTTP 请求。网络请求是异步的，因此我们需要在未来某个合适的时候（请求结果返回时）接收数据。这里使用先进的 async/await，以同步的方式处理，从而避免了回调嵌套和 promises 链。

我们将异步获得的数据返回，它将自动挂载在 props 上（注意 getInitialProps 方法名，顾名思义），render 方法里便可以通过 this.props.data 获取：

    import React from 'react'
    import Head from 'next/head'
    import axios from 'axios';
    
    export default class extends React.Component {
      static async getInitialProps () {
        const res = await axios.get('http://api.football-data.org/v1/competitions/426/leagueTable');
        return {data: res.data}
      }
      render () {
        return (
          <div>
            <Head>
                ......
            </Head>
            <div className="pure-g">
                <div className="pure-u-1-3"></div>
                <div className="pure-u-1-3">
                  <h1>Barclays Premier League</h1>
                  <table className="pure-table">
                    <thead>
                      <tr>
                        ......
                      </tr>
                    </thead>
                    <tbody>
                    {this.props.data.standing.map((standing, i) => {
                      const oddOrNot = i % 2 == 1 ? "pure-table-odd" : "";
                      return (
                          <tr key={i} className={oddOrNot}>
                            <td>{standing.position}</td>
                            <td><img className="pure-img logo" src={standing.crestURI}/></td>
                            <td>{standing.points}</td>
                            <td>{standing.goals}</td>
                            <td>{standing.wins}</td>
                            <td>{standing.draws}</td>
                            <td>{standing.losses}</td>
                          </tr>
                        );
                    })}
                    </tbody>
                  </table>
                </div>
                <div className="pure-u-1-3"></div>
            </div>
          </div>
        );
      }
    }

这样，再访问我们的页面，就有了：


![页面](http://upload-images.jianshu.io/upload_images/4363003-da9e671c438902f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 路由和页面跳转

也许你已经有所感知：我们已经有了最基本的一个路由。Next 不需要任何额外的路由配置信息，你只需要在 pages 文件夹下新建文件，每一个文件都将是一个独立的页面。

让我们来新建一个 team 页面吧！新建 ./pages/details.js 文件：

    // ./pages/details.js
    import React from 'react'
    export default () => (
      <p>Coming soon. . .!</p>
    )

我们使用 Next 已经准备好的组件 <Link> 来进行页面跳转：

    // ./pages/details.js
    import React from 'react'
    
    // Import Link from next
    import Link from 'next/link'
    
    export default () => (
      <div>
          <p>Coming soon. . .!</p>
          <Link href="/"><a>Go Home</a></Link>
      </div>
    )

这个页面不能总是 “Coming soon. . .!” 的信息，我们来进行完善以展示更多内容，通过页面 URL 的 query id 变量，我们来请求并展现当前相应队伍的信息：

    import React from 'react'
    import Head from 'next/head'
    import Link from 'next/link'
    import axios from 'axios';
    
    export default class extends React.Component {
        static async getInitialProps ({query}) {
            // Get id from query
            const id = query.id;
            if(!process.browser) {
                // Still on the server so make a request
                const res = await axios.get('http://api.football-data.org/v1/competitions/426/leagueTable')
                return {
                    data: res.data,
                    // Filter and return data based on query
                    standing: res.data.standing.filter(s => s.position == id)
                }
            } else {
                // Not on the server just navigating so use
                // the cache
                const bplData = JSON.parse(sessionStorage.getItem('bpl'));
                // Filter and return data based on query
                return {standing: bplData.standing.filter(s => s.position == id)}
            }
        }
    
        componentDidMount () {
            // Cache data in localStorage if
            // not already cached
            if(!sessionStorage.getItem('bpl')) sessionStorage.setItem('bpl', JSON.stringify(this.props.data))
        }
    
        // . . . render method truncated
     }


这个页面根据 query 变量，动态展现出球队信息。具体来看，getInitialProps 方法获取 URL query id，根据 id 筛选出（filter 方法）展示信息。有意思的是，因为一直球队的信息比较稳定，所以在客户端使用了 sessionStorage 进行存储。

完整的 render 方法：

    // . . . truncated

    export default class extends React.Component {
        // . . . truncated
        render() {
    
            const detailStyle = {
                ul: {
                    marginTop: '100px'
                }
            }
    
            return  (
                 <div>
                    <Head>
                        <title>League Table</title>
                        <meta name="viewport" content="initial-scale=1.0, width=device-width" />
                        <link rel="stylesheet" href="https://unpkg.com/purecss@0.6.1/build/pure-min.css" />
                    </Head>
    
                    <div className="pure-g">
                        <div className="pure-u-8-24"></div>
                        <div className="pure-u-4-24">
                            <h2>{this.props.standing[0].teamName}</h2>
                            <img src={this.props.standing[0].crestURI} className="pure-img"/>
                            <h3>Points: {this.props.standing[0].points}</h3>
                        </div>
                        <div className="pure-u-12-24">
                            <ul style={detailStyle.ul}>
                                <li><strong>Goals</strong>: {this.props.standing[0].goals}</li>
                                <li><strong>Wins</strong>: {this.props.standing[0].wins}</li>
                                <li><strong>Losses</strong>: {this.props.standing[0].losses}</li>
                                <li><strong>Draws</strong>: {this.props.standing[0].draws}</li>
                                <li><strong>Goals Against</strong>: {this.props.standing[0].goalsAgainst}</li>
                                <li><strong>Goal Difference</strong>: {this.props.standing[0].goalDifference}</li>
                                <li><strong>Played</strong>: {this.props.standing[0].playedGames}</li>
                            </ul>
                            <Link href="/">Home</Link>
                        </div>
                    </div>
                 </div>
                )
        }
    }
    
注意下面截图中，同一页面不同 query 值，分别展示了冠军🏆切尔西和曼联的信息。


![切尔西](http://upload-images.jianshu.io/upload_images/4363003-463c88cdb1e175ed.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![曼联](http://upload-images.jianshu.io/upload_images/4363003-14352128450bb0fb.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





别忘了我们的主页（排行榜页面）index.js 中，也要使用相应的 sessionStorage 逻辑。同时，在 render 方法里加入一条链接到详情页的 <Link>：

    <td><Link href={`/details?id=${standing.position}`}>More...</Link></td>
    

## 错误页面

在 Next 中，我们同样可以通过  error.js 文件定义错误页面。在 ./pages 下新建 error.js：

    // ./pages/_error.js
    import React from 'react'
    
    export default class Error extends React.Component {
      static getInitialProps ({ res, xhr }) {
        const statusCode = res ? res.statusCode : (xhr ? xhr.status : null)
        return { statusCode }
      }
    
      render () {
        return (
          <p>{
            this.props.statusCode
            ? `An error ${this.props.statusCode} occurred on server`
            : 'An error occurred on client'
          }</p>
        )
      }
    }

当传统情况下页面404时，得到：


![404页面](http://upload-images.jianshu.io/upload_images/4363003-811c8bca1607cca0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在我们设置 _ error.js 之后，便有：


![自定义错误页面](http://upload-images.jianshu.io/upload_images/4363003-af29722f2a62216e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




## 总结

这篇文章实现了一个简易 demo，只是介绍了**最基本**的 Next.JS 搭建 React 同构应用的基本步骤。

想想你是否厌烦了 webpack 恼人的配置？是否对于 Babel 各种插件云里雾里？
**使用 Next.js，简单、清新而又设计良好。这也是它在推出短短时间以来，便迅速走红的原因之一。**

除此之外，Next 还有非常多的功能，非常多的先进理念可以应用。
- 比如 <Link> 搭配 prefetch，预先请求资源；
- 再如动态加载组件（Next.js 支持 TC39 dynamic import proposal），从而减少首次 bundle size；
- 虽然它替我们封装好了 Webpack、Babel 等工具，但是我们又能 customizing，根据需要自定义。

最后，对于这些本文章没有演示到的功能是否有些手痒？感兴趣的读者可以关注本文 demo 的[Github项目地址]()，自己手动尝试起来吧～



本文意译了[Chris Nwamba的：React Universal with Next.js: Server-side React 一文，](https://scotch.io/tutorials/react-universal-with-next-js-server-side-react)并对原文进行了升级，兼容了最新的 Next 设计。

我的其他关于 React 文章：
- [做出Uber移动网页版还不够 极致性能打造才见真章](http://www.jianshu.com/p/49029b49f2b4)
- [解析Twitter前端架构 学习复杂场景数据设计](http://www.jianshu.com/p/7a56ac1de2a8)
- [React Conf 2017 干货总结1: React + ES next = ♥](http://www.jianshu.com/p/83c86dd0802d)
- [React+Redux打造“NEWS EARLY”单页应用 一个项目理解最前沿技术栈真谛](http://www.jianshu.com/p/cde3cf7e2760)
- [一个react+redux工程实例](http://www.jianshu.com/p/8e28be0e7ab1)





Happy Coding!

PS: 
作者[Github仓库](https://github.com/HOUCe) 和 [知乎问答链接](https://www.zhihu.com/people/lucas-hc/answers)
欢迎各种形式交流。