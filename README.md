# NFD
No Fraud / Node Forward Bot

一个基于cloudflare worker的telegram 消息转发bot，集成了反欺诈功能

## 特点
- 基于cloudflare worker搭建，能够实现以下效果
    - 搭建成本低，一个js文件即可完成搭建
    - 不需要额外的域名，利用worker自带域名即可
    - 基于worker kv实现永久数据储存
    - 稳定，全球cdn转发
- 接入反欺诈系统，当聊天对象有诈骗历史时，自动发出提醒
- 支持屏蔽用户，避免被骚扰

## 搭建方法
1. 从[@BotFather](https://t.me/BotFather)获取token，并且可以发送`/setjoingroups`来禁止此Bot被添加到群组
2. 从[uuidgenerator](https://www.uuidgenerator.net/)获取一个随机uuid作为secret
3. 从[@username_to_id_bot](https://t.me/username_to_id_bot)获取你的用户id
4. 登录[cloudflare](https://workers.cloudflare.com/)，创建一个worker
5. 在 Cloudflare Dashboard 中创建 D1 数据库
   - 进入 Workers & Pages -> D1
   - 点击 "Create database" 创建新数据库，命名为 `nfd`
   - 创建完成后，点击数据库进入详情页
   - 在 "Quick queries" 中执行以下 SQL 语句创建必要的表：
   ```sql
   CREATE TABLE message_maps (
       message_id INTEGER PRIMARY KEY,
       chat_id TEXT NOT NULL
   );

   CREATE TABLE user_blocks (
       chat_id TEXT PRIMARY KEY,
       is_blocked BOOLEAN NOT NULL
   );

   CREATE TABLE last_messages (
       chat_id TEXT PRIMARY KEY,
       last_message_time INTEGER NOT NULL
   );
   ```
6. 配置 worker
   - 增加一个`ENV_BOT_TOKEN`变量，数值为从步骤1中获得的token
   - 增加一个`ENV_BOT_SECRET`变量，数值为从步骤2中获得的secret
   - 增加一个`ENV_ADMIN_UID`变量，数值为从步骤3中获得的用户id
   - 在 Settings -> D1 database bindings 中，添加一个绑定：
     - Variable name: `DB`
     - Database: 选择刚才创建的 `nfd` 数据库
7. 点击`Quick Edit`，复制[这个文件](./worker.js)到编辑器中
8. 通过打开`https://xxx.workers.dev/registerWebhook`来注册websoket

## 使用方法
- 当其他用户给bot发消息，会被转发到bot创建者
- 用户回复普通文字给转发的消息时，会回复到原消息发送者
- 用户回复`/block`, `/unblock`, `/checkblock`等命令会执行相关指令，**不会**回复到原消息发送者

## 欺诈数据源
- 文件[fraud.db](./fraud.db)为欺诈数据，格式为每行一个uid
- 可以通过pr扩展本数据，也可以通过提issue方式补充
- 提供额外欺诈信息时，需要提供一定的消息出处

## Thanks
- [telegram-bot-cloudflare](https://github.com/cvzi/telegram-bot-cloudflare)