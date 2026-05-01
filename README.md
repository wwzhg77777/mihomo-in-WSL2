# mihomo-in-WSL2
在WSL2里配置mihomo

```
mkdir -p /etc/mihomo
cd /etc/mihomo
wget https://github.com/MetaCubeX/mihomo/releases/download/Prerelease-Alpha/mihomo-linux-amd64-v2-go123-alpha-df1c5e5.gz
gzip -d mihomo-linux-amd64-v2-go123-alpha-df1c5e5.gz
mv mihomo-linux-amd64-v2-go123-alpha-df1c5e5 /usr/local/bin/mihomo
chmod +x /usr/local/bin/mihomo
```

``` # mihomo  -v
Mihomo Meta alpha-df1c5e5 linux amd64 with go1.23.12 Tue Apr 28 02:26:58 UTC 2026
Use tags: with_gvisor
```

```
cd /etc/mihomo
wget https://github.com/haishanh/yacd/releases/latest/download/yacd.tar.xz
tar -xf yacd.tar.xz
mv public/* ./ui/
touch config.yaml
```

``` config.yaml
disable-ipv6: true
socks-port: 1080
allow-lan: true
bind-address: "*"
mode: rule
unified-delay: true
tcp-concurrent: true
keep-alive-idle: 600
keep-alive-interval: 60
find-process-mode: strict

log-level: warning
external-ui: /etc/mihomo/ui # yacd解压的位置
external-controller: *:9090

profile:
  store-selected: true
  store-fake-ip: true
geodata-mode: true

# Geo 数据库下载地址
# 源地址 https://github.com/MetaCubeX/meta-rules-dat
# 可以更换镜像站但不要更换其他数据库，可能导致无法启动
geox-url:
  geoip: "https://ghfast.top/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geoip.dat"
  geosite: "https://ghfast.top/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/geosite.dat"
  mmdb: "https://ghfast.top/https://github.com/MetaCubeX/meta-rules-dat/releases/download/latest/country.mmdb"

# geoip-file: /etc/mihomo/GeoIP.dat
# geosite-file: /etc/mihomo/GeoSite.dat

# 国内外 DNS 定义
chinaDNS:
  &chinaDNS [
    'system',
    'https://dns.alidns.com/dns-query',
    'https://doh.pub/dns-query',
  ]
foreignDNS:
  &foreignDNS [
    'https://1.1.1.1/dns-query#默认代理',
    'https://8.8.8.8/dns-query#默认代理',
  ]

# --- 使用mihomo提供防污染的dns服务 ---
#dns:
#  enable: false
#  listen: :1053   # 监听 1053 端口
#  enhanced-mode: fake-ip
#  fake-ip-range: 198.18.0.1/16
#  use-hosts: true
#  use-system-hosts: true
#    - '+.cn'
#    - 'rule-set:private'
#    - 'rule-set:category_ntp'
#    - 'rule-set:fakeip_filter'
#    - 'rule-set:connectivity_check'
#    - 'rule-set:cn'
#    - 'rule-set:googlefcm'
#    - 'rule-set:steam_cn'
#    - 'rule-set:epicgames'
#    - 'rule-set:nvidia_cn'
#    - 'rule-set:microsoft_cn'
#    - 'rule-set:cloudflare_cn'
#  proxy-server-nameserver:
#    - 'https://doh.pub/dns-query#DIRECT'
#    - 'https://dns.alidns.com/dns-query#DIRECT'
#  default-nameserver:
#    - '223.5.5.5'
#    - '119.29.29.29'
#  nameserver: *foreignDNS
#  nameserver-policy:
#    '*': 'system'
#    '+.arpa': 'system'
#    '+.cn': *chinaDNS
#    'rule-set:private,cn,steam_cn,epicgames,nvidia_cn,cloudflare_cn,microsoft_cn,microsoft,googlefcm,apple,spotify': *chinaDNS
#  direct-nameserver: ['system', '223.5.5.5', '119.29.29.29']
#  direct-nameserver-follow-policy: true

hosts:
  'dns.alidns.com': ['223.5.5.5', '223.6.6.6']
  'doh.pub': ['1.12.12.12', '120.53.53.53']

  # 解决谷歌商店无法下载的问题
  'services.googleapis.cn': ['services.googleapis.com']

  # 屏蔽哔哩哔哩PCDN，解决访问视频卡顿问题
  '+.mcdn.bilivideo.com': ['0.0.0.0']
  '+.mcdn.bilivideo.cn': ['0.0.0.0']

rule-providers:
  # 秋风广告拦截规则
  # https://awavenue.top
  # 由于 Anti-AD 误杀率高，本项目已在 1.11-241024 版本更换秋风广告规则
  AWAvenue-Ads:
    type: http
    behavior: domain
    format: yaml
    # path可为空(仅限clash.meta 1.15.0以上版本)
    path: ./rule_provider/AWAvenue-Ads.yaml
    url: "https://ghfast.top/https://raw.githubusercontent.com/TG-Twilight/AWAvenue-Ads-Rule/refs/heads/main/Filters/AWAvenue-Ads-Rule-Clash-Classical.yaml"
    interval: 3600

#sniffer:
#  enable: false
#  force-dns-mapping: true
#  parse-pure-ip: true
#  override-destination: false
#  sniff:
#    TLS:
#      ports: [443,8443]
#    HTTP:
#      ports: [80,8080-8880]
#      override-destination: true
#    QUIC:
#      ports: [443, 8443]
#  skip-domain:
#    ['Mijia Cloud', '+.oray.com', '+.push.apple.com', 'cloudflare-ech.com']
#  skip-dst-address: ['rule-set:telegram_ip']

ntp:
  enable: true
  write-to-system: false
  server: 'ntp.aliyun.com'
  port: 123
  interval: 60

tun:
  enable: false
  stack: system
  auto-route: true
  strict-route: true
  auto-redirect: true
  auto-detect-interface: true
  dns-hijack: ['udp://any:53', 'tcp://any:53']

# --- 代理集与规则 ---
proxy-providers:
  my_anytls_service:
    type: http
    url: "" # 替换为你的订阅链接
    path: ./proxy_providers/anytls.yaml
    interval: 86400
    health-check:
      enable: true
      url: "https://www.gstatic.com/generate_204"
      interval: 3600
    override:
      additional-prefix: "[MyNode]"
      udp: true        # 强制执行 UDP over TCP 转发
      client-fingerprint: chrome

proxy-groups:
  - name: PROXY
    type: select
    use:
      - "my_anytls_service"

rules:
  # 防止 YouTube 等使用 QUIC 导致速度不佳, 禁用 443 端口 UDP 流量（不包括国内）
  - AND,(AND,(DST-PORT,443),(NETWORK,UDP)),(NOT,((GEOSITE,cn))),REJECT
  - AND,(AND,(DST-PORT,443),(NETWORK,UDP)),(NOT,((GEOIP,CN))),REJECT

  - DOMAIN-SUFFIX,googlevideo.com,PROXY
  - DOMAIN-SUFFIX,youtube.com,PROXY
  - DOMAIN-SUFFIX,ytimg.com,PROXY
  - DOMAIN-SUFFIX,doubleclick.net,PROXY
  - DOMAIN-SUFFIX,googleadservices.com,PROXY
  - DOMAIN-KEYWORD,google,PROXY

  # 广告、劫持等直接拒绝
  - GEOSITE,category-ads-all,REJECT
  - RULE-SET,AWAvenue-Ads,REJECT

  # 私有地址直连
  - GEOSITE,private,DIRECT
  # 国内网站直连
  #- GEOSITE,cn,DIRECT
  #- GEOIP,cn,DIRECT
  # 剩余所有流量走 PROXY 组
  - MATCH,PROXY
```

``` /etc/systemd/system/mihomo.service
[Unit]
Description=Mihomo Proxy Service
After=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/bin/mihomo -d /etc/mihomo
# 日志重定向到指定文件
StandardOutput=file:/var/log/mihomo.log
StandardError=file:/var/log/mihomo.log
# 自动重启
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl enable mihomo.service
systemctl start mihomo.service

ss -tnlp | grep mihomo
LISTEN 0      1024            *:9090       0.0.0.0:*    users:(("mihomo",pid=28255,fd=6))                       
LISTEN 0      1024            *:1080       0.0.0.0:*    users:(("mihomo",pid=28255,fd=3))
```
