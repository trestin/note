# skynet笔记

#### 全局变量

| 变量名 | 类型                  | 模块            | 初始化             | 备注           |
| ------ | --------------------- | --------------- | ------------------ | -------------- |
| Q      | struct                | global_queue    | skynet_mq_init     | 初始化消息队列 |
| M      | struct modules        | skynet_module.c | skynet_module_init | 加载so         |
| H      | struct handle_storage | skynet_handle.c | skynet_handle_init | 保存服务handle |
|        |                       |                 |                    |                |

------

#### skynet_module 加载流程

代码文件：skynet_module.c，对应so

_try_open() -> dlopen() 加载so

open_sym() -> dlsym() 导入 create/init/release/signal 函数

#### skynet_context 加载流程

代码文件：skynet_server.c，对应服务

ctx = skynet_context_new()，创建

| 成员变量 | 初始化函数                         | 说明                       |
| -------- | ---------------------------------- | -------------------------- |
| mod      | skynet_module_query()              |                            |
| instance | skynet_module_instance_create(mod) | 调用module导入的create函数 |
| handle   | skynet_handle_register(ctx)        | 为服务注册handle           |
| queue    | skynet_mq_create(handle)           | 消息队列，通过handle关联   |
|          |                                    |                            |
| cb/cb_ud | skynet_callball(ctx, cb_ud, cb)    | skynet.start->c.callback   |

skynet_module_instance_init(ctx) ，调用 module 导入的 init 函数，至此服务加载完成。

#### 示例：snlua加载流程

代码文件：service_snlua.c

main() -> skynet_start() -> bootstrap() -> skynet_context_new()

ctx -> snlua_create();  ctx->snlua_init() -> load_file("./lualib/loader.lua") -> loadfile(bootstrap)

skynet.name(name, handle) 为handle注册名字，由 H 管理

bootstrap 启动 skynet 服务 launcher/cdummy/cmaster/cslave/datacenterd/service_mgr，以及业务服务 main，然后退出

------

#### 服务启动方式

1. skynet.launch("snlua", "script") -> core.command("LAUNCH", param) -> skynet_context_new()  -- 由 core 启动
2. skynet.call(".launcher", "lua", "LAUNCH", "script")  -- 由 launcher 服务通过`1`启动，并管理
3. skynet.newservice("script")  -- 封装方式`2`
4. skyneet.call(".service", "lua", "LAUNCH", global, ...) -- 由 service_mgr 服务通过`1`启动，并管理
5. skynet.uniqueservice(global, ...) -- 封装方式`4`，启动一个唯一服务

#### 服务退出步骤

skynet.exit() -> skynet.send(".launcher", "lua", "REMOVE", skynet.self(), false) -- launcher 服务移除服务

core.command("EXIT") -> skynet_server.c:handle_eixt(ctx, handle) -> skynet_handle_retire(handle) -- 删除 handle 和名字

skynet_context_release(ctx) -> delete_context(ctx) -- 删除 skynet_context

#### Lua 服务启动流程

skynet.start()

core.callback(skynet.dispatch_message) -> skynet_callback(ctx, gL, _cb) -- 注册消息处理函数，通过`_cb`调用lua函数

skynet.init_service() -> skynet_require.init_all() -- ???

skynet.send(".launch", "lua", "LAUNCHOK"/"ERROR") -- 通知`launcher` 服务启动结果

------

#### 发送消息

skynet.call(addr, typename, ...) / skynet.send(addr, typename, ...)

lua-skynet.c:lsend()

skynet_server.c:skynet_sendname(name) -- 由 name 获取 handle，调用 skynet_send(handle)

skynet_context_push(handle, msg)，由 handle 获取 ctx，调用 skynet_mq_push(ctx, msg) -- 将消息加入消息队列

#### 处理消息

skynet_start.c:thread_worker() -> skynet_context_message_dispatch(sm, q, weight)

ctx = q->handle->ctx -- 获取 skynet_context

msg = skynet_mq_pop(q) -- 获取消息

dispatch_message(ctx, msg) -> ctx->cb(msg) -- 分派消息，cb 即 skynet.start 设置的回调函数

------

#### skynet.send 同步调用

TODO

#### skynet.call 异步调用

TODO