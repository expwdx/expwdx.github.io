---
title: 通过rebar3创建Erlang应用
---

### 参考
[https://praglowski.com/2016/05/10/your-first-erlang-application-with-rebar3](https://praglowski.com/2016/05/10/your-first-erlang-application-with-rebar3)
[https://github.com/erlang/rebar3/issues/1740](https://github.com/erlang/rebar3/issues/1740)
[https://www.rebar3.org/docs/basic-usage](https://www.rebar3.org/docs/basic-usage)

### 项目实践
#### 创建应用
根据目标，这里创建`app`类型的应用，当然也可以创建`escript、lib、release、plugin`类型的应用，这个要根据实际情况而定。关于应用类型，详情参考 [docs.basic-usage](https://www.rebar3.org/docs/basic-usage)
```
➜  projects rebar3 new app transaction_publish_service
===> Writing transaction_publish_service/src/transaction_publish_service_app.erl
===> Writing transaction_publish_service/src/transaction_publish_service_sup.erl
===> Writing transaction_publish_service/src/transaction_publish_service.app.src
===> Writing transaction_publish_service/rebar.config
===> Writing transaction_publish_service/.gitignore
===> Writing transaction_publish_service/LICENSE
===> Writing transaction_publish_service/README.md
```
#### 修改rebar.config，增加依赖
```
➜  transaction_publish_service cat rebar.config 
{erl_opts, [debug_info]}.
{deps, [{emqtt, {git, "https://github.com/emqx/emqtt", {tag, "v1.0.0"}}}]}.

{shell, [
  % {config, "config/sys.config"},
    {apps, [transaction_publish_service]}
]}.
➜  transaction_publish_service 
```
#### 编译构建
```
➜  transaction_publish_service rebar3 compile
===> Verifying dependencies...
===> Fetching emqtt (from {git,"https://github.com/emqx/emqtt",{tag,"v1.0.0"}})
===> Fetching gun v1.3.0
===> Version cached at /home/marco/.cache/rebar3/hex/hexpm/packages/gun-1.3.0.tar is up to date, reusing it
===> Fetching cowlib v2.6.0
===> Version cached at /home/marco/.cache/rebar3/hex/hexpm/packages/cowlib-2.6.0.tar is up to date, reusing it
===> Compiling cowlib
===> Compiling gun
===> Compiling emqtt
===> Compiling transaction_publish_service
➜  transaction_publish_service ls
_build  LICENSE  README.md  rebar.config  rebar.lock  src
➜  transaction_publish_service tree _build 
_build
└── default
    └── lib
        ├── cowlib
        │   ├── ebin
        │   │   ├── cow_base64url.beam
        │   │   ├── cow_cookie.beam
        │   │   ├── cow_date.beam
        │   │   ├── cow_hpack.beam
        │   │   ├── cow_http2.beam
        │   │   ├── cow_http.beam
        │   │   ├── cow_http_hd.beam
        │   │   ├── cow_http_te.beam
        │   │   ├── cow_iolists.beam
        │   │   ├── cowlib.app
        │   │   ├── cow_mimetypes.beam
        │   │   ├── cow_multipart.beam
        │   │   ├── cow_qs.beam
        │   │   ├── cow_spdy.beam
        │   │   ├── cow_sse.beam
        │   │   ├── cow_uri.beam
        │   │   └── cow_ws.beam
        │   ├── erlang.mk
        │   ├── hex_metadata.config
        │   ├── include
        │   │   ├── cow_inline.hrl
        │   │   └── cow_parse.hrl
        │   ├── LICENSE
        │   ├── Makefile
        │   ├── README.asciidoc
        │   └── src
        │       ├── cow_base64url.erl
        │       ├── cow_cookie.erl
        │       ├── cow_date.erl
        │       ├── cow_hpack.erl
        │       ├── cow_http2.erl
        │       ├── cow_http.erl
        │       ├── cow_http_hd.erl
        │       ├── cow_http_te.erl
        │       ├── cow_iolists.erl
        │       ├── cowlib.app.src
        │       ├── cow_mimetypes.erl
        │       ├── cow_mimetypes.erl.src
        │       ├── cow_multipart.erl
        │       ├── cow_qs.erl
        │       ├── cow_spdy.erl
        │       ├── cow_spdy.hrl
        │       ├── cow_sse.erl
        │       ├── cow_uri.erl
        │       └── cow_ws.erl
        ├── emqtt
        │   ├── ebin
        │   │   ├── emqtt.app
        │   │   ├── emqtt.beam
        │   │   ├── emqtt_frame.beam
        │   │   ├── emqtt_props.beam
        │   │   ├── emqtt_sock.beam
        │   │   └── emqtt_ws.beam
        │   ├── include
        │   │   └── emqtt.hrl
        │   ├── LICENSE
        │   ├── Makefile
        │   ├── pre-compile
        │   ├── README.md
        │   ├── rebar.config
        │   ├── rebar.config.script
        │   ├── src
        │   │   ├── emqtt.app.src
        │   │   ├── emqtt.erl
        │   │   ├── emqtt_frame.erl
        │   │   ├── emqtt_props.erl
        │   │   ├── emqtt_sock.erl
        │   │   └── emqtt_ws.erl
        │   ├── test
        │   │   └── emqtt_frame_SUITE.erl
        │   └── TODO
        ├── gun
        │   ├── ebin
        │   │   ├── gun.app
        │   │   ├── gun_app.beam
        │   │   ├── gun.beam
        │   │   ├── gun_content_handler.beam
        │   │   ├── gun_data_h.beam
        │   │   ├── gun_http2.beam
        │   │   ├── gun_http.beam
        │   │   ├── gun_sse_h.beam
        │   │   ├── gun_sup.beam
        │   │   ├── gun_tcp.beam
        │   │   ├── gun_tls.beam
        │   │   ├── gun_ws.beam
        │   │   └── gun_ws_h.beam
        │   ├── erlang.mk
        │   ├── hex_metadata.config
        │   ├── LICENSE
        │   ├── Makefile
        │   ├── README.asciidoc
        │   ├── rebar.config
        │   └── src
        │       ├── gun_app.erl
        │       ├── gun.app.src
        │       ├── gun_content_handler.erl
        │       ├── gun_data_h.erl
        │       ├── gun.erl
        │       ├── gun_http2.erl
        │       ├── gun_http.erl
        │       ├── gun_sse_h.erl
        │       ├── gun_sup.erl
        │       ├── gun_tcp.erl
        │       ├── gun_tls.erl
        │       ├── gun_ws.erl
        │       └── gun_ws_h.erl
        └── transaction_publish_service
            ├── ebin
            │   ├── transaction_publish_service.app
            │   ├── transaction_publish_service_app.beam
            │   └── transaction_publish_service_sup.beam
            ├── include -> ../../../../include
            ├── priv -> ../../../../priv
            └── src -> ../../../../src

17 directories, 101 files
➜  transaction_publish_service 
```
#### escript类型的应用创建
```
➜  erlang rebar3 new escript hello
===> Writing hello/src/hello.erl
===> Writing hello/src/hello.app.src
===> Writing hello/rebar.config
===> Writing hello/.gitignore
===> Writing hello/LICENSE
===> Writing hello/README.md
➜  erlang ls
emqtt  emqx  emqx-rel  erlang-learning  file_client  file_service  hello  helloworld  jaerlang2  learn-you-some-erlang  tutorails
➜  erlang cd hello
➜  hello ls
LICENSE  README.md  rebar.config  src
➜  hello cd src 
➜  src ls
hello.app.src  hello.erl
➜  src cat hello.erl 
-module(hello).

%% API exports
-export([main/1]).

%%====================================================================
%% API functions
%%====================================================================

%% escript Entry point
main(Args) ->
    io:format("Args: ~p~n", [Args]),
    erlang:halt(0).

%%====================================================================
%% Internal functions
%%====================================================================
➜  src cd .,.           
cd: no such file or directory: .,.
➜  src cd ..
➜  hello ls
LICENSE  README.md  rebar.config  src
➜  hello rebar3 escriptize
===> Verifying dependencies...
===> Compiling hello
===> Building escript...
➜  hello _build/default/bin/hello  
Args: []
➜  hello _build/default/bin/hello helloworld
Args: ["helloworld"]
➜  hello 
```

