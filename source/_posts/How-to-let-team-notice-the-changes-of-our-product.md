title: 探讨-如何让团队了解产品进度
date: 2015-09-04 15:56:38
author: frank
tags:
- 团队信息共享
- github webhook
categories:
- Backend
- 中文

---

对于一个人数30-50的创业团队来说，如何让每个成员及时了解产品的变化是一个有意思的话题。

首先，这个所谓的“变化”，是指小变化，不是大的新功能：

1. 用户看得见的变化：小的UI优化，Bug修复。
2. 用户体验的变化：性能的优化（更加流畅），交互的变化（更加合理）。

第一个问题，如何搜集变化：

1. 成员可以每日写工作总结。
 1. 根据工作内容抽离出“变化”。
 2. 有个成员专门去维护一个或一系列文档。
2. 可以维护一个 wiki。
  1. 每个成员可以（不， 是必须）去更新，将“变化”写入文档中去。
  2. 需要一些规范和 review。
3. 如果是团队使用 github，可以在 Pull Request 中详细描述这次PR的内容，然后利用 github 的 webhook 搜集相应PR。

第二个问题，如何通知每个成员：
1. 提醒每个成员每天去看文档。
2. 发邮件，将产品的“变化”写进邮件里面。

对于这两个问题，不同的创团队，不同的情况（需求）有不同的解决方案，我来介绍一下我们 Strikingly 的初步尝试，利用 github 搜集“变化”信息，每天发邮件提醒。

首先，我们没有打算做一个完美的日志更新系统，只是想帮助成员更加规范的描述 Pull Request，然后将特殊的 Pull Request 搜集起来，然后每天发送一个邮件给团队的每个成员。

其次，我们后端是 Rails，用 Redis 做临时存储，利用了 github 的 webhook 服务，将 Markdown 格式转化成 HTML 发送邮件。

下面简单介绍一下实现步骤：

#### Step1: 设置 Github
1. 项目的 settings 中设置 webhook
    1. 设置 Payload URL。
    2. 设置 events 类型：勾选 Pull Request。
2. 添加 Personal access token
3. 使用 gem `github_api`
```ruby
# ./config/initializers/github_api.rb
# 为了安全性，将 github 'username' 和 'token' (上个步骤设置的token) 写入环境变量
Github.configure do |c|
     c.basic_auth = "#{ENV['GITHUB_USERNAME']}:#{ENV['GITHUB_TOKEN']}"
end
``` 

#### Step2: 约定PR模板
- 在PR的描述Markdown中可以（必须）有一个模板
  
![pr_template.png](http://upload-images.jianshu.io/upload_images/17174-db041acc9c2e8fab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示：如果这个pr会影响产品的用户体验，就需要勾选`contains user facing changes`

#### Step3: 记录部署时间
```ruby
class DeployLogger
  DEPLOY_KEY = 'AWESOME_REPO_LAST_DEPLOY_KEY'

  # 根据实际情况使用默认时间
  def self.deploy_at
    $redis.get(BOBCAT_DEPLOY_KEY) || '2015-08-10T02:27:38Z'
  end

  def self.deploy_at= time
    $redis.set BOBCAT_DEPLOY_KEY, time
    $redis.expire BOBCAT_DEPLOY_KEY, 60.days
  end
end
```
当然这里需要做一些访问认证

#### Step4: 处理 github webhook

1. 获取 PR 的 Number
  ```ruby
    # request_json 是 webhook request body
    def api_params_from request_json
       request_json['pull_request']['number']
    end
  ```

2. 因为我们并不能相信 webhook，所以我们应该基于 webhook 的信息主动通过 github API 获取数据。
  ```ruby
    # 访问 github API
    github_response = Github.new.pull_requests.get(PROP_OWNER, PROP_NAME, pr_num)
  ```

3. 筛选PR: merged 到 BASE_BRANCH 且含有 USER_FACING_LINE
  ```ruby
    USER_FACING_LINE = '- [x] **contains user facing changes'
    def user_facing_changes_pr? response
       response.merged &&
       response.base.ref == BASE_BRANCH &&
       response.body.body.include?(USER_FACING_LINE)
    end
  ```

4. 整合信息
  ```ruby
    # 数据结构
    result = {
        matched: false,
        pr_description: {
          key_word: 'GithubPR_User_Facing_Changes',
          merged_at: '',
          content: nil
        }
      }

    result[:pr_description][:merged_at] = pr_merged_time_form github_response
    result[:pr_description][:content] = pr_description_from github_response

    def pr_merged_time_form github_response
       github_response.body.merged_at
    end
    
    # 需要 PR 的 Number, Title, Developer
    def pr_description_from github_response
       body = github_response.body
       pr_developer_name = pr_developer_name_for(body.user.login)
       "#Num#{body.number}: #{body.title}\n###Developer: #{pr_developer_name}\n#{body.body}"
    end

    # github_response 只提供了 login name，我们需要 username
    # git_api 并没有提供直接获取用户名的接口，只能手动获取
    def pr_developer_name_for login
       pr_developer_name = MultiJson.load(RestClient.get("https://api.github.com/users/#{login}").body)['name']
       pr_developer_name ? pr_developer_name : login
    end
  ```

#### Step5: 存储到 Redis
构建PrRequest Model
```ruby
lass PrDescription
  LAST_SEND_EMAIL_KEY = 'LAST_SEND_EMAIL_KEY'

  # 使用 redis ordered set
  # merged_at 时间秒数作为 score
  def self.create columns = {}
    key_word = columns[:key_word]
    merged_at = columns[:merged_at]
    content = columns[:content]
    $redis.zadd key_word, Time.parse(merged_at).to_i, content
    $redis.expire key_word, 2.days
  end

  def self.count_of key_word
    $redis.zcount key_word, '-inf', '+inf'
  end

 # 根据情况做适当调整
  def self.last_send_email_at
    $redis.get(LAST_SEND_EMAIL_KEY) || '2015-08-10T02:27:38Z'
  end

 # 根据情况做适当调整
  def self.last_send_email_at= time
    $redis.set LAST_SEND_EMAIL_KEY, time
    $redis.expire LAST_SEND_EMAIL_KEY, 5.days
  end

  # 获取当前部署好的PR
  # 范围：最近的一次发邮件的时间到最近的一次部署时间
  def self.descriptions_need_to_send_for key_word
    from_time_score = Time.parse(self.last_send_email_at).to_i + 0.01
    to_time_score = Time.parse(DeployLogger.deploy_at).to_i
    $redis.zrangebyscore key_word, from_time_score, to_time_score
  end
end
```

####Step6: 准备邮件
```ruby
# 在每个 PrDescription 之间要用"\n"隔开，注意是双引号，需要转义，不能加空格，会影响 Markdown 的转化
markdown_string = PrDescription.descriptions_need_to_send_for(USER_FACING_CHANGES_KEY).join("\n")

if markdown_string.present?
  # 使用gem maruku 将Markdown
  # 有些小问题：
  #   1. 需要将 " 转换成 '
  #   2. 去除 '\n'
  @content = Maruku.new(markdown_string).to_html.gsub(/\"/,"'").gsub(/\n/, '').html_safe
  mail subject: "User Facing Changes #{Date.today}", from: PRODUCT_EMAIL, to: TEAM_EMAIL
  # 设置最后一次发邮件的时间
  PrDescription.last_send_email_at = DeployLogger.deploy_at
end
```

#### Step7: 循环发邮件
```ruby
# 用到 gem：sidekiq 和 sidetiq
class SendToSupportTeamChangesEmail
  include Sidekiq::Worker
  include Sidetiq::Schedulable

  sidekiq_options :queue => :default, :backtrace => true, :retry => 2

  # everyday 8:30 AM 
  # 请注意时区，可能需要转换
  recurrence { daily.hour_of_day(8).minute_of_hour(30) }

  def perform
    InternalMailer.user_facing_change_email.deliver
  end
end
```
最后，如果今天有`PR` merged 到 develop 分支并且还被部署到 production 环境，我们明天一早会受到一封邮件：
![pr_email.png](http://upload-images.jianshu.io/upload_images/17174-e5c3a2d58b4480d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这方面的探索还在继续，思路和代码还有可多地方可以提升，请各位多多指教。

王涛
Backend Engineer @ Strikingly.com
   
