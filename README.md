# XDUCraft Invite

Minecraft 白名单邀请码自助领取与私聊运维管理插件。绑定单一核心群聊，群成员在群内申请时核查资格并私聊下发邀请码，管理员与超级用户通过私聊进行初始化与发码管理。

如果需要部署包含全部插件的完整机器人，请阅读[整套机器人的安装说明](https://github.com/FlyingPig278/XDUCraft-bot)。

## 功能与行为

插件仅服务于一个绑定的群聊。新入群成员在群内申请时，机器人核验其入群时间与成员资格，并通过私聊下发邀请码。

初始化时会自动记录当前群内的已有成员。初始化前入群的历史成员无法自助领取，需由管理员手动发码，防止老成员重复刷取。每位成员仅可成功自助领取一次。

## 安装与加载

在现有 NoneBot 2.4.x 项目环境中，先安装 Core 核心包，再安装本插件：

```bash
pip install "nonebot-plugin-xducraft-core[onebot] @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-core.git@v0.3.0"
pip install "nonebot-plugin-xducraft-invite @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-invite.git@v0.1.0"
```

在已有项目的 `pyproject.toml` 中的 `plugins` 列表中追加模块，不要覆盖已有配置：

```toml
[tool.nonebot]
plugins = [
    # ... 已有插件 ...
    "xducraft_config",
    "xducraft_invite",
]
```

## 部署参数与群绑定

本插件包含以下启动期参数，修改后必须重启 NoneBot 进程生效：

- `invite_group_id`（环境变量 `INVITE_GROUP_ID`）：绑定的群聊群号。初始化后该参数永久锁定，不可迁移到其他群。
- `invite_api_url`（环境变量 `INVITE_API_URL`）：邀请码生成后端接口，必须为 HTTPS 地址，默认值为 `https://www.xducraft.cn/api/invitation-codes/generate`。
- `invite_api_secret`（环境变量 `INVITE_API_SECRET`）：后端接口鉴权密钥。在 WebUI 管理面板中保存后不会回显已填写的密钥内容。

以上参数可先在 `.env` 中声明基准值，也可以在 WebUI 管理面板中修改，重启进程后生效。

## 启用与前置初始化步骤

插件全局开关与自助功能默认关闭。自助领取生效需按顺序完成以下配置：

首先确认已配置 `invite_group_id` 与 `invite_api_secret`，且 `invite_api_url` 为合法 HTTPS 地址，并重启 NoneBot 进程加载这些参数。

接着开启插件全局总开关。全局开关是所有邀请码指令（包括群内申请和私聊管理）生效的前提。请在 WebUI 管理面板的插件页面开启邀请码插件，或者参考[不使用网页时启用插件](https://github.com/FlyingPig278/nonebot-plugin-xducraft-core#不使用网页时启用插件)在代码中单次启用。

然后由超级用户私聊机器人发送 `/invite init` 完成初始化。机器人会拉取绑定群当前所有成员建立历史记录，并自动开启该群的自助领取功能。在未完成初始化前，自助领取功能无法开启。

初始化完成后自助领取已处于开启状态。后续若需临时暂停或重新恢复，管理员可在私聊发送 `/invite off` 或 `/invite on`；若安装了 `nonebot-plugin-xducraft-features` 插件，也可以在群内发送 `/功能 off 邀请码自助领取` 或 `/功能 on 邀请码自助领取`。

## 指令说明

### 群内申请指令

必须在绑定的群聊中发送，群成员均可调用：

- `/邀请码` 或 `@机器人 邀请码`：申请白名单邀请码。申请指令不能附加额外参数。机器人核验资格后，通过私聊向申请人发送邀请码，并在群内回复发送回执。

### 私聊管理指令

管理指令必须在与机器人的私聊中发送，群内发送不予受理：

- `/invite init`：仅限 SUPERUSER。读取当前绑定群成员列表建立快照，确立初始化状态并自动开启自助发码。
- `/invite status`：群主、管理员或 SUPERUSER 可用。查看当前绑定群、初始化状态、历史成员数、已领取数与发码统计。
- `/invite on` 或 `/invite off`：群主、管理员或 SUPERUSER 可用。开启或关闭成员自助领取功能。未初始化时无法开启。
- `/invite <QQ号>`：群主、管理员或 SUPERUSER 可用。为指定群成员生成并发放邀请码，直接在私聊中将邀请码返回给执行操作的管理员。若该成员已领取过则拒绝。
- `/invite force <QQ号>`：群主、管理员或 SUPERUSER 可用。强制为已领取过邀请码的群成员重复生成并发放邀请码。

## 业务数据存储与自定义路径

群成员快照、发放记录与领取状态保存在独立业务数据文件中（位于数据目录下的 `plugins/invite/`）。

若需要指定外部数据目录，必须在启动前通过操作系统环境变量设置 `XDUCRAFT_DATA_DIR`（绝对路径），不能仅写在 `.env` 中。

## 本地开发与测试

```bash
pip install "nonebot-plugin-xducraft-core[onebot] @ git+https://github.com/FlyingPig278/nonebot-plugin-xducraft-core.git@v0.3.0"
pip install -e ".[dev]"
pytest
python -m build
```

## 许可证

本项目采用 [MIT License](LICENSE) 授权开源。
