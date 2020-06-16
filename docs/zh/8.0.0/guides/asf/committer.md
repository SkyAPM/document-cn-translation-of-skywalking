# Apache SkyWalking 代码提交者
SkyWalking 项目管理委员会(PMC)负责评估候选人的贡献.

在SkyWalking中，就像许多Apache项目一样，我们处理的贡献包括但不限于代码贡献.
如写博客，引导新用户，发表演讲，以各种方式推动项目，都被视为重要的贡献.

## 代码提交者
### 新的提交者提名
在SkyWalking中，**新的提交者提名**只能由现有PMC成员正式启动. 
新的贡献者可以联系任何现有的PMC成员，如果他/她认为他/她是合格的. 再与PMC成员交谈，如果一些成员同意，他们可以启动流程.

建议执行以下步骤，并且只能由现有的PMC成员启动.
1. 将 `[DISCUSS] Promote xxx as new committer` 邮件发送到 `private@skywalking.a.o`. 为了帮助PMC成员支持您的提案，请列出候选人的重要贡献.
2. 讨论持续3天以上，但不超过一周， 除非有明确的反对或关注.
3. 当PMC似乎同意这个提议后，将 `[VOTE] Promote xxx as new committer` 邮件发送到 `private@skywalking.a.o`.
4. 投票持续3天以上, 但不超过一周. 如果有 3 个 +1 票，则将结果视为 `一致通过`   +1 票 > -1 票
5. 将 `[RESULT][VOTE] Promote xxx as new committer` 邮件发送到 `private@skywalking.a.o`, 并列出投票的细节，包括选民.

### 邀请新提交者
PMC成员负责向新提交者发出邀请，并指导新提交者建立ASF env.

您应该按照以下的模板将邮件发送给新的提交者
```
To: JoeBloggs@foo.net
Cc: private@skywalking.apache.org
Subject: Invitation to become SkyWalking committer: Joe Bloggs

Hello [invitee name],

The SkyWalking Project Management Committee] (PMC) 
hereby offers you committer privileges to the project . These privileges are
offered on the understanding that you'll use them
reasonably and with common sense. We like to work on trust
rather than unnecessary constraints.

Being a committer enables you to more easily make 
changes without needing to go through the patch 
submission process. 

Being a committer does not require you to 
participate any more than you already do. It does 
tend to make one even more committed.  You will 
probably find that you spend more time here.

Of course, you can decline and instead remain as a 
contributor, participating as you do now.

A. This personal invitation is a chance for you to 
accept or decline in private.  Either way, please 
let us know in reply to the [private@skywalking.apache.org] 
address only.

B. If you accept, the next step is to register an iCLA:
    1. Details of the iCLA and the forms are found 
    through this link: http://www.apache.org/licenses/#clas

    2. Instructions for its completion and return to 
    the Secretary of the ASF are found at
    http://www.apache.org/licenses/#submitting

    3. When you transmit the completed iCLA, request 
    to notify the Apache SkyWalking and choose a 
    unique Apache id. Look to see if your preferred 
    id is already taken at 
    http://people.apache.org/committer-index.html     
    This will allow the Secretary to notify the PMC 
    when your iCLA has been recorded.

When recording of your iCLA is noticed, you will 
receive a follow-up message with the next steps for 
establishing you as a committer.
```

### 邀请验收流程
新提交人应将邮件回复至 `private@skywalking.apache.org` (选择 `回复所有人`)，并明确表示愿意接受邀请。
然后，此邀请将被视为已被项目PMC接受。当然，新的提交者可以直接说不，然后拒绝邀请.

如果他们接受了，那么他们需要做以下事情.
1. 确保他们已经订阅了 `dev@skywalking.apache.org`.通常他们已订阅.
2. 签署ICLA(个人贡献者许可协议)，可前往 [here](http://www.apache.org/licenses/contributor-agreements.html#clas).
3. 如果你想在日常工作中为项目做出贡献，CCLA是你的不二选择.
4. 关于ICLA标志，您需要打印ICLA，手动签名，并将其扫描为pdf. 
5. 选择您的Apache ID并将其填入ICLA，可前往 [here](http://people.apache.org/committer-index.html) 找到所有现有的Apache ID
6. 将gpg签署到您的icla.pdf.
7. 将 `icla.pdf` 和 `icla.pdf.asc` 邮件发送到 `secretary@apache.org` 和 `private@skywalking.apache.org`.

然后PMC将等待 Apache 确认ICLA的文件. 新的提交者和PMC将收到如下邮件
```
Dear XXX,

This message acknowledges receipt of your ICLA, which has been filed in the Apache Software Foundation records.

Your account has been requested for you and you should receive email with next steps
within the next few days (can take up to a week).

Please refer to https://www.apache.org/foundation/how-it-works.html#developers
for more information about roles at Apache.
```

如果在某些情况下，帐户没有被请求(很少看到)，PMC成员应该联系项目V.P.
V.P.可以请求通过 [Apache Account Submission Helper Form](https://whimsy.apache.org/officers/acreq).

几天后，新的提交者将收到创建的帐户邮件，即这个标题, `Welcome to the Apache Software Foundation (ASF)!`.
此时，祝贺你!您拥有正式的Apache ID.

PMC成员应该将新的提交者添加到正式提交者列表中 [roster](https://whimsy.apache.org/roster/committee/skywalking). 

### 设置 Apache ID 和 dev env
1. 前往 [Apache Account Utility Platform](https://id.apache.org/), 初始化你的密码，设置你的个人邮箱(`Forwarding email address`) 
和 GitHub 账号(`Your GitHub Username`).
2. 如果你想使用 `xxx@apache.org` 发送邮件，你需要使用GMail，并遵循[本指南](https://reference.apache.org/committer/email).
3. Use [GitBox Account Linking Utility](https://gitbox.apache.org/setup/) to make your GitHub account to join Apache GitHub org.
注意，如果没有收到你的GitHub邀请?访问github.com/apache看看你是否有一个待定的邀请. 同样，你的GitHub MFA状态也是必须的.

如果你想让别人看到你在Apache GitHub org，你需要去 [Apache GitHub org people page](https://github.com/orgs/apache/people), 
搜索你自己，然后选择 `Organization visibility` to `Public`.

### 提交者的权利、义务和责任
SkyWalking项目在你成为提交者后不需要继续的贡献，但是我们希望你可以继续.

作为一个提交者，你可以
1. 在Apache repo中检查并将pull请求合并到主分支. 拉请求通常包含多次提交. 那些提交**必须被压缩并合并到一个带有注释**的提交**中. 对于新的提交者，我们希望您能要求一些资深提交者重新检查pull请求.
2. 在Apache repo中创建代码并将其推送到新的分支.
3. 遵循 [发布过程](../How-to-release.md) 发布新版本. 当然，您需要请求提交者团队确认现在是发布的正确时间.

PMC希望新的提交者能够参与release和release的投票，甚至仍然考虑 `+1无约束力`.
但是熟悉版本是晋升为PMC成员的关键之一.

## 项目管理委员会
项目管理委员会(PMC)成员对代码贡献没有特殊权利。
它们只是覆盖并确保项目符合Apache需求，
包括 
1. 发布有约束力的投票和许可检查
2. 新的提交者和PMC成员的认可
3. 识别品牌问题并进行品牌保护.
4. 回答ASF董事会的问题，采取必要的行动.

副主席和PMC主席是秘书，负责整理董事会报告.

通常情况下，新的PMC成员应该从提交者团队中提名。
但是，如果PMC能够同意并确信候选人已经准备好，比如他/她曾经是另一个项目的PMC成员，Apache成员或Apache官员，那么直接成为PMC成员是不被禁止的.

新PMC的投票程序也应遵循与 `[DISCUSS]`, `[VOTE]` 和 `[RESULT][VOTE]` 相同的私人邮件列表中的 [新提交者投票](#new-committer-nomination).
发送邀请前的最后一步，PMC [需要发送通知邮件到Apache](http://www.apache.org/dev/pmc.html#newpmc).
```
To: board@apache.org
Cc: private@skywalking.apache.org
Subject: [NOTICE] Jane Doe for SkyWalking PMC

SkyWalking proposes to invite Jane Doe (janedoe) to join the PMC.

(include if a vote was held) The vote result is available here: https://lists.apache.org/...
```

72小时后，如果董事会不反对(通常不会反对)，发出邀请.

提交人接受邀请后，PMC成员应该通过 [名册](https://whimsy.apache.org/roster/committee/skywalking)
将新的提交者添加到官方PMC列表中.
