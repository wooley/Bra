## 生存游戏（BRA）PHP 网页版

这是一个基于 PHP 与 MySQL 的经典网页多人对战 **生存游戏**（Battle Royale 类型）。  
玩家在地图上探索、搜集道具、战斗与躲避禁区，直到只剩下最后的生还者为止。

本项目是对早期网络游戏程序的源码整理与托管，方便自建服务器、二次开发或代码学习使用。

> 原作版权信息见根目录 `readme.htm` 中的链接与说明。

---

## 功能概览

- **多人在线生存对战**：玩家注册 / 登录后进入同一局游戏，在地图上移动、遭遇、战斗。
- **游戏轮回机制**：一局结束后会自动进入下一局，新一轮玩家重新加入战斗。
- **禁区系统**：
  - 游戏定时开放新禁区（危险区域），在模板 `index.htm` 中可看到禁区数量、间隔等信息。
  - 支持“自动逃避禁区”功能，在进入连斗后会自动关闭。
- **战况统计**：
  - 当前局游戏已进行时间、最高伤害玩家与伤害值。
  - 上一局胜利模式与优胜者信息。
  - 当前激活人数、生存人数、死亡总数等数据。
- **物品与战斗系统**（参考 `include/game/*.func.php`）：
  - 物品拾取、合成、使用等逻辑。
  - 战斗结算、状态变化、特殊事件等。
- **后台管理**（`include/admin/`）：
  - 游戏参数配置、NPC/道具/地图等数据管理。
  - 游戏轮次控制、公告与系统信息管理等。

---

## 目录结构简介

仅列出主要与开发 / 部署相关的目录与文件：

- `index.php`：网站入口，加载公共初始化逻辑后渲染首页模板。
- `config.inc.php`：**数据库与全局配置**文件（安装时需要手动修改）。
- `include/`
  - `common.inc.php`：公共引导文件，负责：
    - 关闭错误输出（`error_reporting(0)`）、初始化常量与路径。
    - 载入 `global.func.php`、`config.inc.php`。
    - 初始化数据库连接、读取 `gamedata/system.php` 与 `gamedata/gameinfo.php`。
    - 根据时间与玩家数量推进游戏状态（开局、连斗、结束等）。
  - `game/*.func.php`：游戏内核心逻辑函数，如属性、战斗、物品、事件、队伍等。
  - `admin/`：后台管理功能相关 PHP 文件。
  - 若干 JS 文件：`game.js`、`common.js`、`json.js` 等。
- `gamedata/`
  - `system.php`：系统变量与运行时状态初始化。
  - `gameinfo.php`：游戏配置、当前局参数等（由系统读写）。
  - `bak/`、`cache/`、`shopitem/` 等：缓存及游戏数据文件。
- `templates/default/`
  - `index.htm`：首页模板，展示当前局状态、上一局结果以及登录 / 进入游戏入口。
  - 其他以功能区分的模板：`game.htm`、`battle.htm`、`map.htm`、`item*.htm`、`team.htm` 等。
- `install/`
  - `bra.sql`：**数据库建表与初始化数据脚本**。
  - `gameinfo.php`：安装期或默认游戏信息配置。
  - 多语言文件：简体 / 繁体中文语言包。
- 其他入口脚本：
  - `game.php`：游戏主逻辑入口（玩家指令的处理）。
  - `login.php`：登录逻辑。
  - `map.php`、`news.php`、`help.php`、`valid.php` 等：对应各功能页面。

---

## 环境要求

- **Web 服务器**：Apache / Nginx + PHP-FPM 等任一支持 PHP 的服务器。
- **PHP 版本**：源代码针对 PHP 4.x / 5.x 编写，建议使用：
  - PHP 5.6 或 7.0–7.4（更高版本可能需要额外兼容性调整）。
- **数据库**：MySQL / MariaDB。
- **扩展与设置**：
  - 需启用 `mysql` 或兼容的数据库扩展（老版本使用 `mysql_*` 接口）。
  - 需要允许写入 `gamedata/` 及部分缓存目录（文件权限）。

---

## 安装与部署

### 1. 准备数据库

1. 在 MySQL 中创建一个新的数据库（默认示例为 `wooley`，可自定义）：

   ```sql
   CREATE DATABASE bra_game DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
   ```

2. 使用 `install/bra.sql` 初始化数据：

   ```bash
   mysql -u你的用户名 -p bra_game < install/bra.sql
   ```

### 2. 配置 `config.inc.php`

打开根目录下的 `config.inc.php`，根据你的服务器环境修改以下关键字段（**务必不要直接提交真实密码到公共仓库**）：

```php
$dbhost   = 'localhost';   // 数据库服务器
$dbuser   = '你的数据库用户名';
$dbpw     = '你的数据库密码';
$dbname   = 'bra_game';    // 或你实际创建的库名

$tablepre = 'bra_';        // 表前缀，通常保持默认即可
$charset  = 'utf-8';       // 网页字符集
$dbcharset = 'utf8';       // 数据库字符集

$bbsurl   = 'http://你的论坛或站点地址';
$gameurl  = 'http://你的游戏访问地址';
$homepage = 'http://你的游戏官网首页';
$title    = '生  存  游  戏'; // 网站标题，可按需修改
```

> 安装完成并确认运行正常后，建议：
> - 将真实数据库账号密码从仓库提交历史中移除或改为示例值。
> - 关闭 `$errorinfo` 或保持为 `0`，防止泄露服务器路径等敏感信息。

### 3. 配置 Web 服务器

1. 将整个项目目录（即本仓库内容）部署到 Web 根目录或虚拟主机目录，例如：
   - Windows + Apache：`H:\bra` 对应 `http://localhost/` 或子路径。
   - Linux + Nginx：配置站点根目录指向该项目目录。
2. 确保：
   - Web 用户对 `gamedata/`、`gamedata/cache/` 等目录有写权限。
   - 默认首页指向 `index.php`。

### 4. 访问游戏

1. 在浏览器中访问你配置好的域名或地址，例如：
   - `http://localhost/`
2. 你应能看到首页显示：
   - 游戏状态、上局结果、最高伤害等信息。
   - 登录框 / 进入游戏按钮。
3. 按页面提示注册 / 登录（具体账号策略可能由导入的 SQL 决定，可在管理后台或数据库中查看／修改）。

---

## 常见问题与注意事项

- **安全性**：
  - 本项目为早期 PHP 代码风格，默认关闭错误输出、但仍可能存在传统安全问题（如 `magic_quotes`、全局变量提取等）。
  - 若对外开放，请务必加固服务器安全策略（IP 限制、HTTPS、应用防火墙等），并按需审计代码。
- **兼容性**：
  - 在较新的 PHP 版本运行时，可能出现废弃函数 / 语法警告，需要手动适配。
  - 如遇数据库连接错误，请确认字符集设置与 MySQL 版本兼容。
- **语言与编码**：
  - 默认使用 UTF-8 编码（`$charset = 'utf-8'`，`$dbcharset = 'utf8'`）。
  - `install/` 目录中包含简体与繁体中文语言文件，可按需要调整。

---

## 开发与二次扩展建议

- 阅读 `include/common.inc.php` 和 `include/game/*.func.php`，理解：
  - 游戏流程（开局 / 禁区推进 / 连斗 / 结局）的状态机逻辑。
  - 玩家指令与页面（`game.php`）之间的数据流。
- 模板层在 `templates/default/` 中，可自行美化 UI 或重写前端：
  - CSS 在 `templates/default/css.htm` 及相关模板中。
  - 可以通过修改模板文件来定制界面，而不必大幅动核心逻辑。
- 后台管理逻辑在 `include/admin/` 下，适合扩展：
  - 新道具、新地图、新事件类型等。

如果你在部署或修改过程中遇到具体问题，可以在 Issue 中描述你的环境和报错信息，便于进一步排查。

