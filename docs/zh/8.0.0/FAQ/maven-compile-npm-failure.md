### 问题描述： Maven编译失败，错误类似于“error: not found: python2”

当编译apm-webapp项目时，会出现以下错误。

注意关键词，如“node-sass”和“Error: not found: python2”。

```
[INFO] > node-sass@4.11.0 postinstall C:\XXX\skywalking\skywalking-ui\node_modules\node-sass
[INFO] > node scripts/build.js

[ERROR] gyp verb check python checking for Python executable "python2" in the PATH
[ERROR] gyp verb `which` failed Error: not found: python2
[ERROR] gyp verb `which` failed     at getNotFoundError (C:\XXX\skywalking\skywalking-ui\node_modules\which\which.js:13:12)
[ERROR] gyp verb `which` failed     at F (C:\XXX\skywalking\skywalking-ui\node_modules\which\which.js:68:19)
[ERROR] gyp verb `which` failed     at E (C:\XXX\skywalking\skywalking-ui\node_modules\which\which.js:80:29)
[ERROR] gyp verb `which` failed     at C:\XXX\skywalking\skywalking-ui\node_modules\which\which.js:89:16
[ERROR] gyp verb `which` failed     at C:\XXX\skywalking\skywalking-ui\node_modules\isexe\index.js:42:5
[ERROR] gyp verb `which` failed     at C:\XXX\skywalking\skywalking-ui\node_modules\isexe\windows.js:36:5
[ERROR] gyp verb `which` failed     at FSReqWrap.oncomplete (fs.js:152:21)

[ERROR] gyp verb `which` failed   code: 'ENOENT' }
[ERROR] gyp verb check python checking for Python executable "python" in the PATH
[ERROR] gyp verb `which` succeeded python C:\Users\XXX\AppData\Local\Programs\Python\Python37\python.EXE
[ERROR] gyp ERR! configure error 
[ERROR] gyp ERR! stack Error: Command failed: C:\Users\XXX\AppData\Local\Programs\Python\Python37\python.EXE -c import sys; print "%s.%s.%s" % sys.version_info[:3];
[ERROR] gyp ERR! stack   File "<string>", line 1
[ERROR] gyp ERR! stack     import sys; print "%s.%s.%s" % sys.version_info[:3];
[ERROR] gyp ERR! stack                                ^
[ERROR] gyp ERR! stack SyntaxError: invalid syntax
[ERROR] gyp ERR! stack 
[ERROR] gyp ERR! stack     at ChildProcess.exithandler (child_process.js:275:12)
[ERROR] gyp ERR! stack     at emitTwo (events.js:126:13)
[ERROR] gyp ERR! stack     at ChildProcess.emit (events.js:214:7)
[ERROR] gyp ERR! stack     at maybeClose (internal/child_process.js:925:16)
[ERROR] gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:209:5)
[ERROR] gyp ERR! System Windows_NT 10.0.17134
......
[INFO] server-starter-es7 ................................. SUCCESS [ 11.657 s]
[INFO] apm-webapp ......................................... FAILURE [ 25.857 s]
[INFO] apache-skywalking-apm .............................. SKIPPED
[INFO] apache-skywalking-apm-es7 .......................... SKIPPED
```

### 原因

该问题与skywalking本身没关系，根据[https://github.com/sass/nosass/issues/1176]如果你访问“GitHub”和“npmjs.org”请求非常慢，
那么在“npm安装”期间，一些预编译的依赖项“node-sass”二进制文件将无法下载，然后npm会尝试自己编译，npm自己编译需要依赖python2。

### 解決方案
#### 1.  建议修改 `skywalking\apm-webapp\pom.xml`  采用npm私服
默认
```
<configuration>  
 <arguments>install --registry=https://registry.npmjs.org/</arguments>  
</configuration>
```
替换后
```
<configuration>  
 <arguments>install --registry=https://registry.npm.taobao.org/ --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/</arguments>  
</configuration>
```
#### 2. 使用网络比较好的VPN通道连接
