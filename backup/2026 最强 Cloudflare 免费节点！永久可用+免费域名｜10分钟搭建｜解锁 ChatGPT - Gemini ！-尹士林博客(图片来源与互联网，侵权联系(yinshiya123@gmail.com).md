如果你在找一种真正“长期可用”的免费节点方案，那么2026年，这套基于 Cloudflare 的搭建方式，可能是目前最值得尝试的一种选择。

它最大的特点只有三个：免费、稳定、速度极快！不需要服务器、不需要额外成本，甚至连域名都可以免费获取；整个搭建过程从零开始，大约10分钟就能完成。同时，由于依托 Cloudflare 本身的网络基础，这套方案在稳定性和可用性方面，也远远优于传统的临时节点或不稳定方案。

相比过去那些复杂、容易失效、需要频繁维护的方式，这种方案更像是一种“轻量级长期解决方案”：一次搭建，持续可用，几乎不需要额外折腾，非常适合个人用户或轻度使用场景。

接下来，我会从准备工作开始，一步一步带你完整搭建这套 Cloudflare 免费节点方案，并把过程中所有关键细节和容易踩坑的地方，全部讲清楚。

部署步骤：
 

# 第一步：
注册一个永久免费的域名，下方我提供了2种免费域名的注册链接，第二种我们之前已经介绍过了，所以这次我们就用第一种方式：
https://www.dnshe.com/

注册登入以后，即可注册5个永久免费的域名，根域名选择ccwu.cc 或者 us.ci 都是完全免费的！

<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/b69ad7a4-49f6-44ee-8817-da8f1f011e0a" />

# 第二步
注册并登入 Cloudflare 平台，然后将上方注册的免费域名托管进去

【[长按注册](https://www.cloudflare.com/)】

<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/99e958b2-3ed4-4bb9-938e-941cb6ddd8b4" />

# 第三步：
创建KV 空间，在Cloudflare后台找到：存储和数据库 – Worker KV，来创建一个KV空间以备后面对接。
<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/de4cc104-f029-4c33-836e-485d96d9de88" />

# 第四步：
创建Pages,在Cloudflare后台找到 Workers 和 Pages

<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/5c494d93-796f-466c-a7c2-8e398738c5be" />
进入以后，选择下方第二个选项，就是创建Pages，如下图所示：

<img width="940" height="504" alt="Image" src="https://github.com/user-attachments/assets/7ea9f4ef-9e54-4721-b116-6b0b89566cb3" />
#接着需要上传由 [CMliu ](https://github.com/cmliu/edgetunnel)开源的程序，他的源码是明文的，你可以自己去[获取查看](https://github.com/cmliu/edgetunnel/blob/main/_worker.js)，目前是开源到GitHub上。或者直接下载Pages的专属安装包！
下载以后直接将附件上传到Pages，进行步骤即可
部署完成后点击 继续处理站点 后，选择 设置 > 环境变量 > 制作为生产环境定义变量 > 添加变量。 变量名称填写ADMIN，值则为你的管理员密码，后点击 保存即可。
返回 部署 选项卡，在右下角点击 创建新部署 后，重新上传 [main.zip](https://github.com/cmliu/edgetunnel/archive/refs/heads/main.zip) 文件后点击 保存并部署 即可。
绑定 KV 命名空间：

在 设置选项卡中选择 绑定 > + 添加 > KV 命名空间，然后选择一个已有的命名空间或创建一个新的命名空间进行绑定。
变量名称填写KV，然后点击 保存后重试部署即可。
给 Pages绑定 CNAME自定义域：

在 Pages控制台的 自定义域选项卡，下方点击 设置自定义域。
填入你的自定义次级域名，注意不要使用你的根域名，例如： 您分配到的域名是 EasonYin.ccwu.cc，则添加自定义域填入 EasonYin.ccwu.cc即可；
按照 CF 的要求将返回你的域名DNS服务商，添加 该自定义域 lizi的 CNAME记录 edgetunnel.pages.dev 后，点击 激活域即可。
访问后台：

访问 https://example.ccwu.cc/admin 输入管理员密码即可登录后台。

<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/d1e13199-a204-4435-8b39-9ff735c5304b" />
接着把上方的节点链接格式或者自适应订阅地址，导入到你的代理软件里，他支持 VLESS、Trojan 等主流协议，深度集成加密传输。

✨ 核心特性
 

🛡️ 协议支持：支持 VLESS、Trojan 等主流协议，深度集成加密传输。
📊 管理面板：内置可视化后台，支持实时配置修改、日志查看及流量统计。
🛠️ 部署灵活：完整适配 CF Workers 及 CF Pages (GitHub / 上传)。
🔄 订阅系统：内置自动订阅生成及混淆转换，适配主流客户端（Clash, Sing-box, Surge 等）。
⚡ 性能加速：支持自定义 ProxyIP、SOCKS5/HTTP 链式代理及优选 API，优化网络延迟。
🌐 多台适配：完美适配 Windows, Android, iOS, MacOS 及各种软路由固件。
比如我们可以使用常见的 [V2Ray](https://github.com/2dust/v2rayN)客户来进行导入代理节点，就可以愉快的使用Cloudflare永久免费的VPN节点

<img width="940" height="529" alt="Image" src="https://github.com/user-attachments/assets/b203b655-9cee-4629-80b4-54c5a8aed37b" />

当然如果你需要自定义更多的选项，比如优选订阅地址，指定ProxyIP订阅，那么可以点击顶部的：我是高手！我就是要折腾！

优选订阅地址：

`Cm.Soso.Edu.Kg`
`Sub.Cmliussss.Net`
`Owo.O00o.Ooo`
`PROXYIP`
 订阅:
`ProxyIP.US.CMLiussss.Net`
`ProxyIP.SG.CMLiussss.Net`
`ProxyIP.JP.CMLiussss.Net`
你可以玩出更多花样更多高级功能
-end
 (C)EasonYin2026 pic from Iternet
powered by AndroidGroup(TM)