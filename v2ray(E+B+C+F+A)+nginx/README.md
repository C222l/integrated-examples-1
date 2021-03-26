介绍：

v2ray（Xray） 前置（监听443端口），vless-tcp-tls 以 h2 或 http/1.1 自适应协商连接，分流 ws（WebSocket）连接，非 v2ray（Xray） 的 http/1.1 连接直接回落给 nginx，而余下的 h2 连接回落给 trojan-tcp，trojan-tcp 处理后再回落给 nginx。其应用如下：

1、vless-tcp-tls（tls由自己提供。）

2、vless-ws-tls（tls由vless-tcp-tls提供及处理，不需配置；另可改成或添加其它ws类应用，参考反向代理ws类的单一示例。）

3、SS- v2ray-plugin-tls（tls由vless-tcp-tls提供及处理，不需配置。）

4、trojan-tcp-tls（tls由vless-tcp-tls提供及处理，不需配置。）

5、vless-kcp-seed（可改成vmess-kcp-seed，或添加它。）

注意：

1、v2ray v4.31.0 版本及以后才支持 trojan 协议。

2、nginx 支持 h2c server，但不支持 http/1.1 server 与 h2c server 共用一个端口或一个进程（Unix Domain Socket 应用），故 vless+tcp 或 trojan+tcp 的端口回落或进程回落必须分开，分别对应 http/1.1 与 h2 回落。而 trojan+tcp 目前不支持 http/1.1 与 h2 的端口回落或进程回落分离 ，故 trojan+tcp 连接只能二选一。此次 vless+tcp 应用默认采用 h2 连接及回落，故套娃 trojan+tcp 只能仅采用 h2 连接及回落。

3、nginx 预编译程序包可能不带支持 PROXY protocol 协议的模块。如要使用此项协议应用，需加 http_realip_module（必须加） 及 stream_realip_module（可选加） 两模块构建自定义模板，再进行源代码编译和安装。另编译时选取源代码版本建议不要低于1.13.11。

4、此方法采用的是套娃方式实现共用443端口，支持 vless+tcp 与 trojan+tcp 完美共存，且仅需要一个域名及普通证书即可搞定，但 trojan+tcp 不支持 xtls 应用。

5、配置1：端口转发、端口回落\分流，没有启用 PROXY protocol。配置2：进程转发、进程回落\分流，没有启用 PROXY protocol。配置3：进程转发、进程回落\分流，启用了 PROXY protocol。