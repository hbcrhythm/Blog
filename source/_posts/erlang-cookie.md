---
title: Erlang Cookie
date: 2017-10-22 00:42:13
tags:
---
最近在写一个cluster_demo， 遇到一个问题，我的集群中的两个节点在windows下大概是这样启动的。
		
		erl -name a@127.0.0.1 -setcookie aa
		erl -name b@127.0.0.1 -setcookie bb
因为我知道，节点间的互连要**cookie一致**，我们可以通过下面这个函数设置cookie
			
		erlang:set_cookie(Node, NodeCookie)
或许大家看过官方文档和一些博客里面写的，集群节点可以不需要全部节点同一个cookie，只需要使用erlang:set_cookie去设置你想要连接节点的cookie。
我把这句话理解点代码是这样:

在a@127.0.0.1节点上，我进行如下操作

		erlang:set_cookie('b@127.0.0.1', bb)

在b@127.0.0.1节点上，我进行如下操作
		
		erlang:set_cookie('a@127.0.0.1', aa)

我以为这样的话，那么我们就可以在a或者b上直接ping通对方，但是事实却不是这样，当我在 a@127.0.0.1 节点上执行
		
		net_adm:ping('b@127.0.0.1')
他的连接并没有成功，并且提示在b@127.0.0.1上给我的提示是
		
		** Connection attempt from disallowed node ‘a@127.0.0.1' **
我搜索了erlang的kernel的库代码，找到了2个地方会存在这个提示，

erlang kernel version: **kernel-4.1.1**

dist_util.erl 151行 
```
%% check if connecting node is allowed to connect with allow-node-scheme
is_allowed(#hs_data{other_node = Node, 
		    allowed = Allowed} = HSData) ->
    case lists:member(Node, Allowed) of
	false when Allowed =/= [] ->
	    send_status(HSData, not_allowed),
	    error_msg("** Connection attempt from "
		      "disallowed node ~w ** ~n", [Node]),
	    ?shutdown(Node);
	_ -> true
    end.
```

dist_util.erl 591行
```
%% wait for challenge response after send_challenge
recv_challenge_reply(#hs_data{socket = Socket, 
			      other_node = NodeB,
			      f_recv = FRecv}, 
		     ChallengeA, Cookie) ->
    case FRecv(Socket, 0, infinity) of
	{ok,[$r,CB3,CB2,CB1,CB0 | SumB]} when length(SumB) =:= 16 ->
	    SumA = gen_digest(ChallengeA, Cookie),
	    ChallengeB = ?u32(CB3,CB2,CB1,CB0),
	    ?trace("recv_reply: challenge=~w digest=~p\n",
		   [ChallengeB,SumB]),
	    ?trace("sum = ~p\n", [SumA]),
	    case list_to_binary(SumB) of
		SumA ->
		    ChallengeB;
		_ ->
		    error_msg("** Connection attempt from "
			      "disallowed node ~w ** ~n", [NodeB]),
		    ?shutdown(NodeB)
	    end;
	_ ->
	    ?shutdown(no_node)
    end.
```
很明显，不是151行的问题，通过设置 net_kernel:allow([]) 可以排除这个问题。
看代码可以发现 当SumB =/= SumA的时候就会返回这个提示，SumB是接收到的，SumA是我们计算出来的，

		SumA = gen_digest(ChallengeA, Cookie)
SumA是这样生成的，查看了ChallengeA的生成代码，只是进行一些算数，然后生成的一个值
```
gen_challenge() ->
    A = erlang:phash2([erlang:node()]),
    B = erlang:monotonic_time(),
    C = erlang:unique_integer(),
    {D,_}   = erlang:statistics(reductions),
    {E,_}   = erlang:statistics(runtime),
    {F,_}   = erlang:statistics(wall_clock),
    {G,H,_} = erlang:statistics(garbage_collection),
    %% A(8) B(16) C(16)
    %% D(16),E(8), F(16) G(8) H(16)
    ( ((A bsl 24) + (E bsl 16) + (G bsl 8) + F) bxor
      (B + (C bsl 16)) bxor 
      (D + (H bsl 16)) ) band 16#ffffffff.
```
打印Cookie的值，发现是aa ,正式我们在b@127.0.0.1上设置的a@127.0.0.1的cookie。
基本上能断定出问题的就是这个cookie值了。

但这里有一个情况出现，如果我们两个脚本的启动时这样的

		erl -name  a@127.0.0.1 -setcookie aa
		erl -name  b@127.0.0.1 -setcookie aa
我们可以直接net_adm:ping()成功对方。
我们查看官方文档，erlang:set_cookie()的解释

> Sets the magic cookie of Node to the atom Cookie. If Node is the local node, the function also sets the cookie of all other unknown nodes to Cookie (see Section Distributed Erlang in the Erlang Reference Manual in System Documentation).

因为我们启动的时候setcookie ，按照官方文档的解释，其实在a@127.0.0.1节点上也可以说是进行了这样操作的

		erlang:set_cookie('b@127.0.0.1'， aa)
同理，在b@127.0.0.1上就是
		
		erlang:set_cookie('a@127.0.0.1', aa)
这样我们可以发现，其实他们可以互连，是因为他们都设置对方的cookie为同一个值，我们做个尝试，在两个节点上分别执行下面2个操作
		
		erl -name a@127.0.0.1 -setcookie aa
		erlang:set_cookie('b@127.0.0.1', cc)

		erl -name b@127.0.0.1 -setcookie bb
		erlang:set_cookie('a@127.0.0.1', cc)

然后在进行net_adm:ping()发现，他们pong了。
我们可以得出结果，其实他们要互连，是需要双方将对方的cookie设置为同一个值。

我们在去翻下 Distributed Erlang章节的文档，发现其实是有这句话的，

> For a node Node1 with magic cookie Cookie to be able to connect to, or accept a connection from, another node Node2 with a different cookie DiffCookie, the function erlang:set_cookie(Node2, DiffCookie) must first be called at Node1. Distributed systems with multiple user IDs can be handled in this way.

仔细看，他只是说需要在Node1 上 erlang:set_cookie(Node2, DiffCookie) ，并没有说 Node2上也需要调用erlang:set_cookie()啊。。。。。。额，其实我一直以为是文档不详细。。。。。。