Apache SkyWalking 发布指南
--------------------
本文档指导每个提交者以 Apache 的方式来发布 SkyWalking, 
并且还帮助提交者检查发布事项以及其他成员参与投票等.

## 设置开发环境
按照 [Apache maven 部署环境文档](http://www.apache.org/dev/publishing-maven-artifacts.html#dev-env)
设置gpg工具和加密密码

使用以下代码块作为模板并将其放入 `~/.m2/settings.xml` 文件中

```
<settings>
...
  <servers>
    <!-- To publish a snapshot of some part of Maven -->
    <server>
      <id>apache.snapshots.https</id>
      <username> <!-- YOUR APACHE LDAP USERNAME --> </username>
      <password> <!-- YOUR APACHE LDAP PASSWORD (encrypted) --> </password>
    </server>
    <!-- To stage a release of some part of Maven -->
    <server>
      <id>apache.releases.https</id>
      <username> <!-- YOUR APACHE LDAP USERNAME --> </username>
      <password> <!-- YOUR APACHE LDAP PASSWORD (encrypted) --> </password>
    </server>
   ...
  </servers>
</settings>
```

## 添加GPG公钥
1. 将你的GPG公钥添加到 [SkyWalking GPG KEYS](https://dist.apache.org/repos/dist/release/skywalking/KEYS) 文件中,
只有当你是一个提交者时才能修改, 使用你的 Apache id 和密码登录这个 svn, 并更新文件. **不要覆盖现有文件.**
2. 将你的GPG公钥上传到公共GPG站点.  如 [MIT 的站点](http://pgp.mit.edu:11371/). 
这个网站应该在 Apache maven staging repository 的检查列表中.

## 测试你的配置
此步骤仅用于测试, 如果您的环境设置正确, 则不需要每次都检查. 
```
./mvnw clean install -Pall (this will build artifacts, sources and sign)
```

## 准备发布
```
./mvnw release:clean
./mvnw release:prepare -DautoVersionSubmodules=true -Pall
```

- 将版本号设置为 x.y.z, 打上形如 **v**x.y.z 的 tag (版本 tag 必须以 **v** 开头, 您将在下一步中知道为什么必须这样.)

**你可以在发布之前做 GPG 签名. 如果你需要输入密码以进行签名的话, 由于 maven 没有给你输入的机会, 这会导致发布失败。
对任意文件运行 `gpg --sign xxx` 进行签名, 这可以记住密码一段时间, 应该可以在这段时间内发布完成.**


## 发布阶段
```
./mvnw release:perform -DskipTests -Pall
```

- 该版本将自动插入临时存储库中.

## 构建并签署源代码包
```shell
export RELEASE_VERSION=x.y.z (example: RELEASE_VERSION=5.0.0-alpha)
cd tools/releasing
sh create_source_release.sh
```

**注意**, `create_source_release.sh` 只适合 MacOS. 欢迎任何人贡献 Windows 和 Linux .

这个脚本应该做以下事情
1. 使用 `v` + `RELEASE_VERSION` 作为标记来克隆代码.
2. 完成 `git submodule init/update` .
3. 排除目标源 tar 中的所有不必要的文件, 例如 `.git`, `.github`, `.gitmodules`. 有关详细信息, 请参阅脚本.
4. 执行 `gpg` 和 `shasum 512`.

此时你应该可以在 `tools/releasing` 文件夹中找到 `apache-skywalking-apm-x.y.z-src.tgz` 和 .asc, .sha512 结尾的文件

## 在 Apache Nexus Staging 存储库中查找和下载分发
1. 使用 ApacheId 登录 `https://repository.apache.org/`.
2. 跳转到 `https://repository.apache.org/#stagingRepositories`.
3. 搜索 `skywalking` 并找到您的暂存存储库.
4. 关闭存储库并等待所有检查通过. 在此步骤中, 将检查您的 GPG KEYS. 如果你以前没有这样做过, 请参考[设置 GPG 文档](#add-your-gpg-public-key).
5. 跳转到 `{REPO_URL}/org/apache/skywalking/apache-skywalking-apm-/x.y.z`
6. 下载 `.tar.gz` 和 `.zip` 且有 .asc 和 .sha1 结尾的文件


## 上传到 Apache svn

1. 使用 ApacheId 登录 `https://dist.apache.org/repos/dist/dev/skywalking/`.
2. 创建文件夹, 按发行版本和阶段命名, 例如: x.y.z
3. 将源代码包上传到包含 .asc, .sha512 的文件夹
  * 包名: apache-skywalking-x.y.z-src.tar.gz
  * 有关详细信息, 请参见"构建和签署源代码包"一节 
4. 将分发包上传到包含 .asc, .sha512 的文件夹
  * 包名: apache-skywalking-bin-x.y.z.tar.gz, apache-skywalking-bin-x.y.z.zip
  * 有关详细信息, 请参见“在Apache Nexus Staging存储库中查找和下载分发”一节
  * 创建 .sha512 包: `shasum -a 512 file > file.sha512`    

## 发表内部公告

在开发邮件列表中发送公告邮件.

```
Mail title: [ANNOUNCE] SkyWalking x.y.z test build available

Mail content:
The test build of x.y.z is available.

We welcome any comments you may have, and will take all feedback into
account if a quality vote is called for this build.

Release notes:

 * https://github.com/apache/skywalking/blob/master/CHANGES.md

Release Candidate:

 * https://dist.apache.org/repos/dist/dev/skywalking/xxxx
 * sha512 checksums
   - sha512xxxxyyyzzz apache-skywalking-apm-x.x.x-src.tgz
   - sha512xxxxyyyzzz apache-skywalking-apm-bin-x.x.x.tar.gz
   - sha512xxxxyyyzzz apache-skywalking-apm-bin-x.x.x.zip

Maven 2 staging repository:

 * https://repository.apache.org/content/repositories/xxxx/org/apache/skywalking/

Release Tag :

 * (Git Tag) x.y.z

Release CommitID :

 * https://github.com/apache/skywalking/tree/(Git Commit ID)
 * Git submodule
   * skywalking-ui: https://github.com/apache/skywalking-rocketbot-ui/tree/(Git Commit ID)
   * apm-protocol/apm-network/src/main/proto: https://github.com/apache/skywalking-data-collect-protocol/tree/(Git Commit ID)
   * oap-server/server-query-plugin/query-graphql-plugin/src/main/resources/query-protocol https://github.com/apache/skywalking-query-protocol/tree/(Git Commit ID)

Keys to verify the Release Candidate :

 * https://dist.apache.org/repos/dist/release/skywalking/KEYS

Guide to build the release from source :

 * https://github.com/apache/skywalking/blob/x.y.z/docs/en/guides/How-to-build.md

A vote regarding the quality of this test build will be initiated
within the next couple of days.
```

## 为测试反馈等待至少 48 小时

任何 PMC, 提交者或贡献者都可以测试发布的功能并提供反馈.
基于此, PMC 将决定是否开始投票.

## 在 dev 群组中投票

在 `dev@skywalking.apache.org` 中发起投票

```
Mail title: [VOTE] Release Apache SkyWalking version x.y.z

Mail content:
Hi All,
This is a call for vote to release Apache SkyWalking version x.y.z.

Release notes:

 * https://github.com/apache/skywalking/blob/x.y.z/CHANGES.md

Release Candidate:

 * https://dist.apache.org/repos/dist/dev/skywalking/xxxx
 * sha512 checksums
   - sha512xxxxyyyzzz apache-skywalking-apm-x.x.x-src.tgz
   - sha512xxxxyyyzzz apache-skywalking-apm-bin-x.x.x.tar.gz
   - sha512xxxxyyyzzz apache-skywalking-apm-bin-x.x.x.zip

Maven 2 staging repository:

 * https://repository.apache.org/content/repositories/xxxx/org/apache/skywalking/

Release Tag :

 * (Git Tag) x.y.z

Release CommitID :

 * https://github.com/apache/skywalking/tree/(Git Commit ID)
 * Git submodule
   * skywalking-ui: https://github.com/apache/skywalking-rocketbot-ui/tree/(Git Commit ID)
   * apm-protocol/apm-network/src/main/proto: https://github.com/apache/skywalking-data-collect-protocol/tree/(Git Commit ID)
   * oap-server/server-query-plugin/query-graphql-plugin/src/main/resources/query-protocol https://github.com/apache/skywalking-query-protocol/tree/(Git Commit ID)

Keys to verify the Release Candidate :

 * https://dist.apache.org/repos/dist/release/skywalking/KEYS

Guide to build the release from source :

 * https://github.com/apache/skywalking/blob/x.y.z/docs/en/guides/How-to-build.md

Voting will start now (xxxx date) and will remain open for at least 72 hours, Request all PMC members to give their vote.
[ ] +1 Release this package.
[ ] +0 No opinion.
[ ] -1 Do not release this package because....
```

## 投票检查

所有 PMC 成员和提交者都应在投票 +1 之前检查这些.

1. 功能测试.
2. staging repository 中的所有工件都附带 .asc, .md5, *sha1 文件发布
3. 源代码和分发包 (apache-skywalking-x.y.z-src.tar.gz, apache-skywalking-bin-x.y.z.tar.gz, apache-skywalking-bin-x.y.z.zip)
都应该在 `https://dist.apache.org/repos/dist/dev/skywalking/x.y.z` 且包含 .asc, .sha512
4. `LICENSE` 和 `NOTICE` 文件在源代码和分发包中.
5. 检查 `shasum -c apache-skywalking-apm-x.y.z-src.tgz.sha512`
6. 遵循[文档](https://github.com/apache/skywalking/blob/master/docs/en/guides/How-to-build.md#build-from-apache-source-code-release)从源码包 (apache-skywalking-x.y.z-src.tar.gz) 构建分发包.
7. Apache RAT 检查. 运行`./mvnw apache-rat:check`. (源代码中没有二进制文件)
8. 需要有免责声明


投票结果应遵循这些.
1. PMC 投票是 +1 绑定, 所有其他投票是 +1 没有约束力.
2. 在 72 小时内, 你得到至少 3 个 (+1 绑定), 并且 +1 比 -1 更多. 投票通过.

## 发版

1. 将源代码 tar 包和分发包移动到 `https://dist.apache.org/repos/dist/release/skywalking/`.
```
> export SVN_EDITOR=vim
> svn mv https://dist.apache.org/repos/dist/dev/skywalking/x.y.z https://dist.apache.org/repos/dist/release/skywalking
....
enter your apache password
....

```

2. 在 nexus staging repo 中发布.
3. 公共下载源和分发 tar/zip 位于  `http://www.apache.org/dyn/closer.cgi/skywalking/x.y.z/xxx`.
我们只发布 Apache 镜像路径作为发布信息.
4. 公共 asc 和 sha512 位于  位于 `https://www.apache.org/dist/skywalking/x.y.z/xxx`
5. 公共密钥指向  `https://www.apache.org/dist/skywalking/KEYS`
6. 更新最新下载地址. http://skywalking.apache.org/downloads/ . 增加一个新的下载源、描述、distribution、 sha加密、文档链接等. 
按照(3)-(6)的规则可以找到链接.
7. 在网站首页和活动页面添加发布活动。发布带有变更日志或关键特性的公开版本.
8. 使用Apache账号将完成的内容邮件发送到 `dev@skywalking.apache.org`, `announce@apache.org`.
```
Mail title: [ANNOUNCE] Apache SkyWalking x.y.z released

Mail content:
Hi all,

Apache SkyWalking Team is glad to announce the first release of Apache SkyWalking x.y.z.

SkyWalking: APM (application performance monitor) tool for distributed systems,
especially designed for microservices, cloud native and container-based (Docker, Kubernetes, Mesos) architectures.

This release contains a number of new features, bug fixes and improvements compared to
version a.b.c(last release). The notable changes since x.y.z include:

(Highlight key changes)
1. ...
2. ...
3. ...

Please refer to the change log for the complete list of changes:
https://github.com/apache/skywalking/blob/vx.y.z/CHANGES.md

Apache SkyWalking website:
http://skywalking.apache.org/

Downloads:
http://skywalking.apache.org/downloads/

Twitter:
https://twitter.com/ASFSkyWalking

SkyWalking Resources:
- GitHub: https://github.com/apache/skywalking
- Issue: https://github.com/apache/skywalking/issues
- Mailing list: dev@skywalkiing.apache.org


- Apache SkyWalking Team
```
