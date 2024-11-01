

> 跨页面，跨标签，跨iframe，跨窗口通信，跨域通信的一种方法 postMessage

面试官：你知道不同页面之间如何通信数据吗。
你：......（思索片刻，postMessage，但忘了咋用）

面试官：如果从一个页面输入了一些信息后点击按钮跳转到新页面，带上旧页面的信息，该如何实现呢，有什么网络安全问题吗。
你：......（好熟悉啊。。。）

如果你也有以上困扰，不用怕，看下这篇文章。甚至微前端的原理也能了解点，之前大火的跨标签页的特效联动也是可以使用这个 api 去进行通信的。

#### 本文目录
+ 概念
+ 基本用法
+ 解决跨域通信
+ 可能遇到的网络安全问题
+ 用到 postMessage 的微前端技术
+ 结论

#### postMessage 概念
postMessage 是一种在不同窗口、iframe、标签页或者 Web Worker 之间安全地进行跨源通信的方法。通过 postMessage ，你可以将消息从一个上下文（例如，父窗口或子 iframe）发送到另一个上下文，并在接收端监听这些信息。下面是如何使用 postMessage 的详细说明，包括如何监听另一个页面发送的信息。

##### postMessage 的基本用法
postMessage 的原理：postMessage 是 HTML5 提供的一个 API,允许不同源的窗口（如 < iframe > 、不同的浏览器标签页或 Web Worker ）进行安全的跨源通信。其基本工作原理如下：
1.安全性： postMessage 允许一个窗口向另一个窗口发送消息，并可以指定接收方的源（origin），从而增加安全性。
2.消息发送：使用 window.postMessage(message, targetOrigin) 方法，发送一个消息到目标窗口。
3.消息接收：目标窗口通过监听 message 事件来接收消息，通常使用 window.addEventListener('message',callback) 方法。

###### 1.发送消息
使用 postMessage 方法可以将消息发送到特定的窗口或文档。

```js
    // 假设我们有一个指向子窗口的引用
    const childWindow = window.open('https://example.com')

    // 发送消息到子窗口
    childWindow.postMessage('Hello from parent', 'htts://example.com')
```

+ 第一个参数是要发送的消息内容，可以是字符串或对象。
+ 第二个参数是目标源，表示接收窗口的域名，用于安全性检查（只允许指定的源接收消息）。

###### 2.监听消息
在接收端（例如子窗口或另一个页面），可以使用 addEventListener 来监听来自其他窗口的消息。
```js
    // 在子窗口或目标页面中
    window.addEventListener('message', (event)=> {
        // 验证消息来源
        if(event.origin === 'https://your-parent-origin.com') {
            console.log('Receive message:', event.data);
            // 处理接收到的消息
        } else {
            console.warn('Invalid origin:', event.origin)
        }
    })
```

+ event.origin 提供发送消息的窗口的源，以确保消息的安全性。
+ event.data 包含发送的消息内容。

##### 实例代码
一下将简单展示下一个完整的实例，包括父窗口发送消息，子窗口接收消息的代码。

父窗口代码（例如 parent.html）
```html
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>Parent Window</title>
        </head>
        <body>
            <h1>Parent Window</h1>
            <button id="sendMessage">Send message to child</button>

            <script>
                const childWindow = window.open('child.html');

                document.getElementById('sendMessage').addEventListener('click',()=>{
                    childWindow.postMessage('Hello from Parent', '*')
                })
            </script>
        </body>
    </html>
```

子窗口代码（例如 child.html）
```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Child Window</title>
    </head>
    <body>
        <h1>
            Child Window
        </h1>
        <div id="message">
        </div>
        <script>
            window.addEventListener('message', (event)=>{
                // 这里可以验证消息来源
                console.log('Received message from parent:', event.data);
                document.getElementById('message').innetText = event.data;
            })
        </script>
    </body>
    </html> 
```

##### 重要注意事项
1.安全性：
+ 验证来源： 一定要检查 event.origin，确保只接受来自可信来源的消息，以避免安全问题。
+ 避免使用 '*' : 使用具体的源（而不是 *，通配符，表示所有域名都可以，无限制）来发送消息，确保消息仅发送到指定的源。
+ 限制传输的数据：仅传输必要的数据，避免传输敏感信息，如用户的个人信息或认证令牌。

2.数据类型：
+ postMessage 可以发送任意类型的数据，包括对象。如果发送对象，它会被序列化为 JSON 格式，因此需要确保数据是可以被序列化的。

3.跨域问题：
+ postMessage 是跨域通信的解决方案，可以用于跨域 iframe、不同的子域等情况。


##### 用 postMessage 解决跨域通信问题
postMessage 是 HTML5 中引入的一种机制，用于在不同源（域名、协议或端口不同）之间安全地进行通信。这种跨域通信非常有用，尤其是在涉及 iframe、窗口或 Web Worker 的应用场景中。下面是对 postMessage 跨域通信的详细说明。

1.基本概念
在浏览器的同源策略下，不同源的文档之间无法相互访问。例如 https://example.com 和 https://another.com 的页面无法直接通过 JavaScript 进行交互。postMessage 是提供了一种安全的方式来解决这个限制。

2.使用场景
postMessage 主要用于以下场景
+ iframe 通信：父页面与嵌套的 iframe 之间进行通信。
+ 窗口通信：打开新的窗口与父窗口之间的通信。
+ Web Worker 通信：主线程与 Web Worker 之间的通信。

3.使用方法
postMessage 的基本使用方法包括两部分：发送消息和接收消息。

发送消息
使用 postMessage 发送消息的基本语法（获取页面已有 iframe 的情况下）
```js
    // 在发送方窗口
    const targetWindow = document.getElementById('my-iframe').contentWindow; // 获取目标窗口
    const message = {type:'greeting',text:'Hello, iframe!'}; // 消息内容
    targetWindow.postMessage(message, 'https://example.com'); // 发送消息，指定目标源
```
+ targetWindow：目标窗口（可以是 iframe 或打开的窗口）。
+ message: 要发送的消息，可以是字符串、对象（会被序列化）等。
+ targetOrigin: 指定目标源（可以是具体的 URL 或 '*'，但使用 '*'时需要注意安全性）。

接收消息
和上面说过的一样，不再重复了。

##### 如果面试官再问
那如果场景是页面A输入了用户认证信息点击按钮打开了新标签页面，然后新标签页面要携带这些信息，然后输入新的信息去做验证呢。要怎么实现呢。

答：
方法1：使用 window.open 的 URL 参数 (不太安全，明文暴露用户信息)
一种简单的方法是将用户认证信息作为查询参数附加到新标签页的 URL 中。在这种情况下，页面 A 可以在打开新标签页时，将认证信息编码到 URL 中。

方法2：使用 locaoStorage 或 sessionStorage （但是会受到同源策略限制，也就是跨域）

方法3：使用 postMessage （派上用场）
如果需要更安全的方式，可以使用 postMessage。在这种情况下，可以先打开新标签页，然后在页面 A 发送消息到页面 B。

###### 此时面试官继续追问，那这中间遇到的网络安全问题呢以及如何预防？
安全问题，如果用的是相对安全和通用的 postMessage （其他方式其实也类似会遇到一样问题），在涉及跨域通信时，可能存在以下一些可能的风险和注意事项：
+ 1.恶意监听器：
用户可以在浏览器的开发者控制台中注入自己的 JavaScript 代码，添加一个监听器来处理来自其他窗口或 iframe 的消息。这可能导致消息泄露，尤其是如果传输的信息包含敏感数据。
+ 2.信息泄露：
如果使用 postMessage 发送的数据未经过适当的验证或过滤，攻击者可能会通过监听消息的方式窃取信息，尤其是在没有设置适当的源限制时。
+ 3.跨站脚本攻击（XSS）：
如果攻击者能够注入 JavaScript 代码到你的页面中，他们可能会利用 postMessage 的通信来执行 XSS 攻击，获取用户的敏感信息或控制用户的会话。
+ 4.消息伪造：
如果不验证接收到的消息来源，攻击者可能会发送伪造的消息。例如，一个恶意网站可能尝试利用 postMessage 伪造来自可信源的消息。
+ 5.不当的源限制：
使用 postMessage 时，如果未严格指定消息接收方的源（origin），可能会使得不可信的来源能够接收到信息。确保你总是指定接收消息的源，以降低风险。

最佳实践
为了降低 postMessage 的风险，建议遵循以下最佳实践：

1.严格设置目标源：确保在调用 postMessage 时，准确设置接受者的源，避免使用'*',而是指定确切的域。
2.验证消息内容：在接收信息时，检查消息的结构和内容，确保它们符合预期。
3.限制接收消息的窗口：确保只有可信的窗口或 iframe 可以接收消息。
4.定期审计和测试：定期审查代码和实现，以确保没有潜在的安全漏洞。
5.用户教育：对用户进行安全教育，避免他们在控制台中随意运行脚本。

具体实现：
1.严格设置目标源：
确保在调用 postMessage 时，精确指定接收者以及发送者的源，以防止消息被来自不可信来源的窗口接收。
```js
    // 在页面 A 中发送消息
    newTab.postMessage({username,password},'https://example.com');

    // 在页面 B 中接收消息
    window.addEventListener('message', function(event){
        if(event.origin!=='https://A.com') {
            console.warn('不可信的来源，消息被忽略：', event.origin);
            return; // 丢弃来自不可信源的消息
        }
        // 处理可信消息
    })
```
2.验证消息内容：
确保接收到的消息内容符合预期格式，避免处理恶意构造的消息。
```js
    // 在页面 B 中接收消息
    window.addEventListener('message',function(event){
        if(event.origin!=='https://A.com') {
            // ...
        }

        const {username,password} = event.data;

        // 验证消息内容
        if(typeof username !== 'string' || typeof password !=='string') {
            console.warn('消息格式不正确');
            return ; // 丢弃不符合预期格式的消息
        }

        // 处理消息
    })
```
3.限制接收消息的窗口：
仅允许可信的窗口或 iframe 接收消息。可以通过配置 iframe 的 sandbox 属性，限制其行为。
```js
    // 在页面 A 中
    <iframe src="https://example.com/child.html" sandbox="allow-same-oirin allow-scripts">
```
代码解释：
sandbox属性：
+ sandbox 属性：用于启用“沙盒”模式，这可以限制在 < iframe > 中运行的内容的能力。通过使用 sandbox 属性，可以提高安全性，防止某些类型的攻击，如 XSS （跨站脚本攻击）。
+ allow-same-origin：允许 < iframe > 中的内容访问同源的资源。换句话说，加载的页面可以与父页面共享同一个源（协议，主机和端口），这使得 < iframe > 中的脚本能够访问同源的 Cookie 和其他存储。
+ allow-scripts：允许在 < iframe > 中运行 JavaScript 脚本。如果没有这个权限，加载的页面中的脚本将无法执行。
安全性考虑：
使用 sandbox 属性可以极大地增强安全性。通过适当的设置，你可以控制 < iframe > 中内容的行为，降低潜在的安全风险。
+ 如果不想 < iframe > 中的内容执行任何脚本或访问同源资源，你可以仅使用 sandbox 属性而不添加任何允许的特性。
+ 这样做将完全限制 < iframe > 的能力，无法运行脚本或访问父文档。

4.定期审计和测试：
定期审查代码和实现，确保没有潜在的安全漏洞，包括检查使用 postMessage 的地方。

实现措施：
+ 定期进行代码审查：组织定期的代码审查会议，审查所有使用 postMessage 的地方。
+ 使用自动化工具：使用静态代码分析工具检测潜在的安全漏洞。
+ 定期进行渗透测试：请专业的安全团队对应用进行渗透测试，识别可能的漏洞。

5.用户教育：
对用户进行安全教育，确保他们了解如何安全使用网站和应用程序，尤其是在控制台中运行代码的潜在风险。

实现措施：
+ 提供安全提示：在应用中添加提示，告知用户不要在控制台中运行未知代码。
+ 创建教育材料：编写关于网络安全的文章或视频，说明如何保护个人信息，特别是关于 XSS 和信息泄露的知识。
+ 组织安全培训：定期对团队成员和用户进行网络安全培训，提升他们的安全意识。

额外防御措施：
+ 使用 CSP （内容安全策略）：配置 CSP 来限制可以加载的资源和执行的脚本从而减少 XSS 攻击的风险。
+ HTTPS：确保所有数据传输都通过 HTTPS 进行，防止中间人攻击。

###### 如果面试官说，需要校验用户信息该怎么处理呢
如果直接明文传输，肯定是不安全的，所以可以使用加密工具或者和后端合作，前端加密工具太假了，还是可以破解的，不建议，所以说下和后端合作的方式，使用 JWT 的方式，短期有效令牌。

实现措施：
+ 使用 Token 进行认证：
可以使用短期有效的认证令牌（如 JWT ），并在后端进行验证。用户在登录后，后端生成一个有效期短暂的令牌（例如15分钟）（在 A 页面那里登录拿到的安全 Token），并通过 postMessage 将该令牌发送到新页面。

示例代码：
```js
    // 发送页面
    const token = 'your_jwt_token'; // 假设这是用户的 JWT
    const newWindow = window.open('https://example.com')

    newWindow.postMeesage({token},'https://example.com')

    // 接收页面
    window.addEventListener('message', function(event)=>{
        // 验证 event.origin
        if(event.origin !== 'https://A.com') {
            return ; // 丢弃不安全的消息
        }

        const {token} = event.data;

        // 在后端验证 token 
        fetch('https://your-api.example.com/verify',{
            method:'POST',
            headers:{
                'Authorization': `Bearer ${token}`,
                'Content-type': 'application/json',
            },
            body: JSON.stringify({})
        }).then(response=>{
            // 处理验证结果
        })
    })

```
代码解释：
在 fetch 里，body是使用的 JSON.stringify({})，创建了一个空的 JSON 对象 {}，表示没有要发送的具体数据。这在某些情况下是合适的，尤其是在进行身份验证时，服务器可能只需要验证头部中的令牌，而不需要额外的请求数据。

##### postMessage 在一些微前端里的运用
在微前端架构中，postMessage 是一种常用的跨域通信机制，特别适用于在不同的前端应用或框架之间进行数据传递和消息传递。以下是关于微前端中使用 postMessage 的一些框架及其相关原理的介绍。

###### 微前端概述
微前端是一种将单一应用拆分成多个独立的前端应用的方法，每个应用可以独立开发、部署和运行。这种方法类似于微服务架构，目的是提高开发效率和团队的自主性。

在微前端中的应用：
微前端架构中的应用通常在同一页面中运行多个子应用，它们可能来自不同的域或子域。postMessage 提供了一种简便的方法来实现它们之间的通信。以下是几个常用的框架以及它们如何使用 postMessage 的相关原理。

1.Single-SPA
概述：
Single-SPA 是一个流行的微前端框架，允许将多个前端框架（如React、Vue、Angular）组合在一起。
使用 postMessage 的原理：
+ 在子应用之间，可以使用 postMessage 来共享状态或事件。例如，一个子应用可以在用户登录后发送登录事件，其他子应用可以监听这个事件并更新其状态。
+ Single-SPA 自身并不强制使用 postMessage，但它允许开发者在需要的情况下实现这一功能。

2.qiankun
概述：
qiankun 是一个基于 Single-SPA 的微前端框架，提供了更为丰富的功能和简化的 API。
使用 postMessage 的原理：
+ qiiankun 支持使用 postMessage 在主应用和子应用之间传递信息。主应用可以向子应用发送数据，例如路由变化、主题切换等。
+ 具体实现中，qiankun 会在主应用加载子应用时，自动设置 postMessage 的通信逻辑，确保信息能够在它们之间流畅传递。

3.Module Federation(Webpack 5)
概述：
Webpack 5 中引入的 Module Federation 功能允许多个 Webpack 构建共享和加载模块。
使用 postMessage 的原理：
+ 虽然 Module Federation 主要通过 Webpack 进行模块共享，但在某些情况下，仍然可以使用 postMessage 来传递消息。
+ 例如，在应用间共享状态或通知更新时，开发者可以使用 postMessage 进行通信，尤其是在涉及跨域的子应用时。

总结：
在微前端架构中，postMesaage 提供了一种有效的跨域通信机制，允许不用的前端应用在运行时进行交互。各种微前端框架（如Single-SPA、qianknu 等）都可以利用这一技术来实现模块之间的消息传递和数据共享。这种方式不仅提高了应用的解耦性，还增强了灵活性和可扩展性。通过合理设计消息结构和事件监听机制，开发者可以实现复杂的应用交互逻辑。

##### 结论
使用 postMessage ，你可以安全地实现跨域通信，允许不同窗口之间互相发送和接收消息。确保始终检查消息来源，以增强安全性。如果你有其他见解或者疑问，欢迎评论留言！