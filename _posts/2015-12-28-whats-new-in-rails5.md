---
layout: post
title:  "Rails 5.0.0新特性"
date:   2015-12-28 22:51:30 -0800
categories: translate
---
翻译自： <a>http://blog.michelada.io/whats-new-in-rails-5</a>

Rails5 在四月的RailsConf中被宣布，DHH重点讲解了一些我们所期待的在新版Rails中的特性。
他也谈论到了关于建立统一的Ruby on Rails程序不是太糟糕的看法。你可以对DHH的观点持保守态度，但事实是现代的Web Applications不单单是HTML 和 CSS，并且Rails社区也已经意识到这一点，其证明就是Rails中包含了 Rails API 和 ActionCable。
 Web已经变了，同时Ruby on Rails也需要改变。So，让我们来对Rails5中的变化有一个快速的预览。

## 一. 支持Ruby2.2.2及以上版本

Rails5是Ruby on Rails的主版本，这也就暗示着Rails API和依赖不但可能会被deprecate，增加或改进已有APIs，而且也会使用Ruby语言的新特性。
后者就是为什么Rails5只支持Ruby2.2.2及以上版本。
在Ruby on Rails程序中，我们会到处的传递symbols，当我们的内存充满着没有被垃圾回收的symbols时，会增加被DOS攻击的可能。
Ruby2.2.0引进了一个改变就是symbols可以被垃圾回收。
另一个Rails5支持Ruby2.2.2的原因就是她利用了增量GC的新特性 ------ 可以减少Rails内存的占用。
Rails5 同时也开始减少对于Ruby2.0语法的使用，Keyword Arguments 和 Module#prepend就是适应新版Ruby的例子。

## 二. API Deprecation 和 剔除

看一眼Rails的commits， 你将会看到在老版Rails中标记为deprecated而此时已经剔除的代码。值得注意的删除了得APIs如下：
    
### ActionMailer

* deliver 和 deliver! 方法已经被移除
* 在email views中的 *_path helper

### ActiveRecord

* 对于 protected_attributes gem的支持
* 对于 activerecord-deprecated_finders gem的支持

### ActivePack assertions

* assert_template 和 assigns()断言被deprecated并且被移到gem rails-controller-testing

	Rails5中的清理还包含的无用的代码和测试，并且，mocha已经被Minitest替代。
	
## 三. 性能改进

通过Ruby2.2.2 Rails5 应该在性能，内存占用，GC时间上有了改进。这一点只能说是促进了Rails5的性能，而不是Rails5得到更好性能的唯一原因。
听说核心团队已经决定制作一个更好/更快的框架。减少对象的创建，冻结不变的字符串，移除不必要的依赖和优化常用操作，这一系列方法都在帮助rails5 更快。
下面就是一些关注与性能提升的commits，很值得去看一眼，学习和理解这些有助于减少对象生成或代码优化的例子，因为其中的一些原因会应用于你的项目中。

1. Beyond Ludicrous Speed
2. Speed up AcitonController::Randerer normalize_keys by ~28%
3. 10X speed improvements for AS::Dependencies.loadable_constants_for_path
4. Speed up code and avoid unnecessary MathData objects
5. Freeze string literals when not mutated

## 四. SO，Rails5中有哪些新特性？

目前我们已经看到了Rails5怎样做到的更快，但仍没有看见一些新功能和API。

### 1. or 方法 in ActiveRecord::Relation

最终，Active::Relation得到了一个#or方法，它允许我们写出如下的ActiveRecord DSL：

{% highlight ruby %}
Book.where('status = 1').or(Book.where('status=3'))
#=> select * from books where status = 1 or status = 3
#or 方法接受一个关系作为参数，它也可以接受一个由#scope得到的model。
class Book < ActiveRecord::Base
  scope :new_coming, ->{ where(status: 3) }
end
Book.where('status = 1').or(Book.new_coming)
#=> select * from books where status = 1 or status = 3
{% endhighlight %}

### 2. belongs_to 默认被要求

从现在开始，每一个Rails程序将会有一个新的配置选项config.active_record.belongs_to_required_by_default = true，它将会触发一个验证错误当你保存一个belongs_to字段没有存在的model时。

{% highlight ruby %}
config.active_record.belongs_to_required_by_default可以被设置为false，这将会保持老版Rails的验证行为，你也可以添加一个选项 optional: true，这也将使得belongs_to不被验证。
  class Book < ActiveRecord::Base
     belongs_to :author, optional: true
  end
{% endhighlight %}

### 3. ActiveRecord's attribute API

新的api给ActiveRecord添加了一个功能，它使得你可以修改默认的字段类型。想一下，你的数据库中有一个类型为decimal的字段，但是你的应用只关心整数部分，你可以让你的应用忽略掉小数部分，让数据只显示整数部分。
有了这个attribute API，你可以做到它像下面这样容易：

{% highlight ruby %}
class Book < ActiveRecord::Base
end
book.quantity #=> 12.0

class Book < ActiveRecord::Base
  attribute :quantity, :integer
end
book.quantity #=> 12
{% endhighlight %}

这里我们覆盖了从数据库模式中自动生成的属性类型，原先的decimal类型被转换为integer，但每一次model和数据库的交互仍然将它当做decimal对待。
你甚至可以定义你自己的类型，通过一个继承于ActiveRecord：：Type：：Value的类，并且实现他的#cast，#serialize，#deserialize。

### 4. has_secure_token 被加入到ActiveRecord

一个ActiveRecord模型现在可以很容易的拥有toke属性，使用token的场景主要是生成邀请码或者重置密码。

{% highlight ruby %}
class Invite < ActiveRecord::Base
  has_secure_token :invitation_code
end
invite = Invite.new
invite.save
invite.invitation_code
invite.regenerate_invitation_code
{% endhighlight %}

SecureRandom被用于生成24个字节的token。你可以使用model generator创建需要的迁移。它允许我们设置字段类型为token。

{% highlight bash %}
$ rails g migration add_token_to_invites token:token
{% endhighlight %}

这个迁移为我们生成了一个unique，索引。

### 5. Mysql ActiveRecord 适配器开始支持JSON类型

如果你的Rails程序使用MySQL5.7.8作为数据库，那么你将获得一个原生的JSON类型。从Rails5开始你可以在ActiveRecord中使用这个原生类型。

### 6. 在controllers之外渲染模板

Rails5允许你在controllers之外渲染模板或内联代码。这个特性对于ActiveJob和新的ActionCable（这将会之后讨论）非常重要。
ActionController：：Renderer实现了这个功能，并且可以在我们的ApplicationController类中使用。让我们通过一些例子看它是如何工作的。

{% highlight ruby %}
# render inline code
ApplicationController.render inline: '<%= "Hello Rails" %>' # => "Hello Rails"
# render a template
ApplicationController.render 'sample/index' # => Rendered sample/index.html.erb within layouts/application (0.0ms)
# render an action
SampleController.render :index # => Rendered sample/index.html.erb within layouts/application (0.0ms)
# render a file
ApplicationController.render file: ::Rails.root.join('app', 'views', 'sample', 'index.html.erb') # =>   Rendered sample/index.html.erb within layouts/application (0.8ms)
{% endhighlight %}

这很好，但是可以传递变量到我们的模板吗？

{% highlight ruby %}
# Pass assigns
ApplicationController.render assigns: { rails: 'Rails' }, inline: '<%= "Hello #{@rails}" %>' # => "Hello Rails"
# Pass locals
ApplicationController.render locals: { hello: 'Hello' }, assigns: { rails: 'Rails' }, inline: '<%= "#{hello} #{@rails}" %>' # => "Hello Rails"
{% endhighlight %}
	
### 7. 更好的Minitest 测试系统

自从test_unit从Rails中移除后我就一直在使用Minitest。我非常喜欢Minitest的简单，但却有Rspec的所有功能，Minitest很大程度上满足了我对于测试的所有需求。但在过去的Rails中Minitest有一个问题-------就是它的测试系统看起来太基本了。然而在Rails5中这已经不是一个问题了，Rails5 改进了它。
现在当你执行 bin/rails test -h，你将会得到一些非常有用的帮助信息。

* 运行满足匹配模式的测试
* 通过指定文件或行号来测试相应的测试
* 执行特定的测试文件或测试目录

{% highlight bash %}
$ bin/rails test -h
  minitest options:
    -h, --help                       Display this help.
    -s, --seed SEED                  Sets random seed. Also via env. Eg: SEED=n rake
    -v, --verbose                    Verbose. Show progress processing files.
    -n, --name PATTERN               Filter run on /regexp/ or string.
	Known extensions: pride, rails
    -p, --pride                      Pride. Show your testing pride!

    Usage: bin/rails test [options] [files or directories]
        You can run a single test by appending a line number to a filename:
       	bin/rails test test/models/user_test.rb:27
       	You can run multiple files and directories at the same time:
       	bin/rails test test/controllers test/integration/login_test.rb
       	Rails options:
           -e, --environment ENV            Run tests in the ENV environment
           -b, --backtrace                  Show the complete backtrace
{% endhighlight %}

### 8. Turbolinks 3

Turbolinks在Rails4中被加入，这是一个让人喜爱或讨厌的功能。Rails5中升级了它--------支持HTML5定制的数据类型，我们期望它能够有更好的渲染速度。
新版中，最显著的改变就是Partial Replacement功能的加入，从浏览器端，我们将能够告诉Turbolinks哪些内容需要改变/替换。
Turbolinks将会寻找HTML5 自定义属性 data-turbolinks-permanent 和 data-turbolinks-temporary 来决定DOM的替换策略。在客户端我们可以使用Tubrolinks.visit or Turbolinks.replace 来触发一个替换。visit和replace的不同在于前者。。。。。

### 9. Rails API

Rails API是通过Rails构建API的另一个方案，Rails API是一个独立的gem，它有自己的构建API程序的策略。它的目的就是通过移除一些不必要的中间件保留对于API应用必须的中间件来创建快速的Rails API程序。
从Rails5 开始， Rails API被集成到其中，没有必要再去包含额外的gems。创建API程序仅需要在rails命令后添加一个 --api选项。

{% highlight bash %}
$ rails new app --api
{% endhighlight %}

新的应用将包含ActiveModelSerializers，在Gemfiles中移除了JQuery和Turbolinks gems。并且config/application.rb有了config.api_only = true选项，application_controller.rb不再检查CSRF，并且继承于ActionController::API。

{% highlight ruby %}
# application.rb file
module MicheladaApi
  class Application < Rails::Application
    config.api_only = true
  end
end
# application_controller.rb file
class ApplicationController < ActionController::API
end
{% endhighlight %}

### 10. ActionCable

ActionCable是Rails5中为了实时通信而新添加的功能。

总结：

这不是关于Rails5 新特性完整的列表，仅仅是我们期望的新功能的预览。
毫无疑问，现在是准备升级你项目的时候了，仅仅是为了性能提升也是值得的。如果你使用的是Rails4.x 那么升级几乎是无痛的。
