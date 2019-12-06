---
title: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory Node内存溢出解决方案
date: 2019-1-14
tag: node
---

 ## 项目背景
redskull2: 0.0.49版本
 ## 触发条件
修改js代码热加载时，报下面的错误自动退出
```
<--- Last few GCs --->

[43697:0x103800600]   342449 ms: Mark-sweep 1365.6 (1403.2) -> 1365.4 (1402.7) MB, 344.7 / 0.0 ms  (average mu = 0.781, current mu = 0.000) last resort GC in old space requested
[43697:0x103800600]   342774 ms: Mark-sweep 1365.4 (1402.7) -> 1365.4 (1402.7) MB, 325.1 / 0.0 ms  (average mu = 0.644, current mu = 0.000) last resort GC in old space requested


<--- JS stacktrace --->

==== JS stack trace =========================================

    0: ExitFrame [pc: 0x1303985041bd]
Security context: 0x21a3cba1e589 <JSObject>
    1: byteLength(aka byteLength) [0x21a307393d21] [buffer.js:530] [bytecode=0x21a3093b5109 offset=204](this=0x21a3865022e1 <undefined>,string=0x21a34030b0c9 <Very long string[8949653]>,encoding=0x21a3cba2f7b9 <String[4]: utf8>)
    2: arguments adaptor frame: 3->2
    3: fromString(aka fromString) [0x21a3073aa799] [buffer.js:341] [bytecode=0x21a3093af471 of...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: node::Abort() [/usr/local/bin/node]
 2: node::OnFatalError(char const*, char const*) [/usr/local/bin/node]
 3: v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, bool) [/usr/local/bin/node]
 4: v8::internal::Heap::FatalProcessOutOfMemory(char const*) [/usr/local/bin/node]
 5: v8::internal::Heap::AllocateRawWithRetry(int, v8::internal::AllocationSpace, v8::internal::AllocationAlignment) [/usr/local/bin/node]
 6: v8::internal::Factory::NewRawTwoByteString(int, v8::internal::PretenureFlag) [/usr/local/bin/node]
 7: v8::internal::String::SlowFlatten(v8::internal::Handle<v8::internal::ConsString>, v8::internal::PretenureFlag) [/usr/local/bin/node]
 8: v8::String::Utf8Length() const [/usr/local/bin/node]
 9: node::Buffer::(anonymous namespace)::ByteLengthUtf8(v8::FunctionCallbackInfo<v8::Value> const&) [/usr/local/bin/node]
10: v8::internal::FunctionCallbackArguments::Call(v8::internal::CallHandlerInfo*) [/usr/local/bin/node]
11: v8::internal::MaybeHandle<v8::internal::Object> v8::internal::(anonymous namespace)::HandleApiCallHelper<false>(v8::internal::Isolate*, v8::internal::Handle<v8::internal::HeapObject>, v8::internal::Handle<v8::internal::HeapObject>, v8::internal::Handle<v8::internal::FunctionTemplateInfo>, v8::internal::Handle<v8::internal::Object>, v8::internal::BuiltinArguments) [/usr/local/bin/node]
12: v8::internal::Builtin_Impl_HandleApiCall(v8::internal::BuiltinArguments, v8::internal::Isolate*) [/usr/local/bin/node]
13: 0x1303985041bd
14: 0x130398513429
[1]    43696 abort      npm start
```

 ## 出现的原因
Node中通过javascript使用内存时只能使用部分内存（64位系统下约为1.4 GB，32位系统下约为0.7 GB），redskull2编译时占用的资源如果超出了限制，就会出现刚刚那个问题

查看任务管理器，发现每次redskull2热加载时内存稳步提升(
![image](https://user-images.githubusercontent.com/18004081/51100318-72700c80-1810-11e9-929b-abd13300e047.png)

 ## 解决方案
- 升级redskull..., 新版本的依赖包不会有这个问题
- 常规webpack/node项目可以传递 **--max-old-space-size** 或 **--max-new-space-size**来调整内存大小的使用限制，但是rdskull2 本身没有监听这两个命令.... pass
- npm插件 increase-memory-limit  试了redskull2下依旧无效
- 设置内存环境变量：export NODE_OPTIONS=--max_old_space_size=4096  暴力点可以放到npm start命令前面



