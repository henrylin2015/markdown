<link href="markdown.css" rel="stylesheet">
# yii2 学习笔记

## 修改默认控制器
> 'defaultRoute' => 'welcome' ,//(在config配置文件中$config下添加)

## 一些常用的文件类库

> yii\Helpers\Html;//这个类库 提供一些html格式文件输出

> yii\widgets\ActiveForm;//这个类库 提供了form表单的提交等


## 控制器生命周期

处理一个请求时，应用主体 会根据请求路由创建一个控制器，控制器经过以下生命周期来完成请求：

* 在控制器创建和配置后，yii\base\Controller::init() 方法会被调用。

* 控制器根据请求操作ID创建一个操作对象:

1 如果操作ID没有指定，会使用yii\base\Controller::defaultAction默认操作ID；

2 如果在yii\base\Controller::actions()找到操作ID，会创建一个独立操作；

3 如果操作ID对应操作方法，会创建一个内联操作；

4 否则会抛出yii\base\InvalidRouteException异常。

* 控制器按顺序调用应用主体、模块（如果控制器属于模块）、控制器的 beforeAction() 方法；

1 如果任意一个调用返回false，后面未调用的beforeAction()会跳过并且操作执行会被取消； action execution will be cancelled.

2 默认情况下每个 beforeAction() 方法会触发一个 beforeAction 事件，在事件中你可以追加事件处理操作；

* 控制器执行操作:

1 请求数据解析和填入到操作参数；

* 控制器按顺序调用控制器、模块（如果控制器属于模块）、应用主体的 afterAction() 方法；

1 默认情况下每个 afterAction() 方法会触发一个 afterAction 事件，在事件中你可以追加事件处理操作；

* 应用主体获取操作结果并赋值给响应.

## 收发邮件
Yii 支持组成和发送电子邮件。然而，该框架提供的只有内容组成功能和基本接口。实际的邮件发送机制可以通过扩展提供， 因为不同的项目可能需要不同的实现方式，它通常取决于外部服务和库。

大多数情况下你可以使用 yii2-swiftmailer 官方扩展。

### 配置

件组件配置取决于你所使用的扩展。一般来说你的应用程序配置应如下：

`
 return [
    //....
    'components' => [
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
        ],
    ],
];
`

### 基本用法

一旦 “mailer” 组件被配置，可以使用下面的代码来发送邮件：

`
Yii::$app->mailer->compose()
    ->setFrom('from@domain.com')
    ->setTo('to@domain.com')
    ->setSubject('Message subject')
    ->setTextBody('Plain text content')
    ->setHtmlBody('<b>HTML content</b>')
    ->send();
`

### 撰写邮件内容

Yii 允许通过特殊的视图文件来撰写实际的邮件内容。默认情况下，这些文件应该位于 “@app/mail” 路径。

一个邮件视图内容的例子：
> <?php
> use yii\helpers\Html;

> use yii\helpers\Url;

> /* @var $this \yii\web\View view component instance */

> /* @var $message \yii\mail\BaseMessage instance of newly created mail message */

> ?>

> <h2>This message allows you to visit our site home page by one click</h2>

> <?= Html::a('Go to home page', Url::home('http')) ?>


为了通过视图文件撰写正文可传递视图名称到 compose() 方法中：


`
Yii::$app->mailer->compose('home-link') // 渲染一个视图作为邮件内容
    ->setFrom('from@domain.com')
    ->setTo('to@domain.com')
    ->setSubject('Message subject')
    ->send();
`

### yii2 发送邮件（最全配置文件）

1 在配置文件main-local.php components=>[]里面配置

`
"mailer" => [
    "class" => "yii\swiftmailer\Mailer",
    "useFileTransport" =>false,//这句一定有，false发送邮件，true只是生成邮件在runtime文件夹下，不发邮件
    "transport" => [
    "class" => "Swift_SmtpTransport",
    "host" => "smtp.163.com",  //每种邮箱的host配置不一样
    "username" => "15618380091@163.com",
    "password" => "*******",
    "port" => "25",
    "encryption" => "tls",
    ],
    "messageConfig"=>[
    "charset"=>"UTF-8",
    "from"=>["15618380091@163.com"=>"admin"]
    ],
],
`

2 controller控制器中 代码：

`
$mail= Yii::$app->mailer->compose();
$mail->setTo('***********@qq.com');
$mail->setSubject("邮件测试");
//$mail->setTextBody('zheshisha ');   //发布纯文字文本
$mail->setHtmlBody("<br>问我我我我我");    //发布可以带html标签的文本
if($mail->send())
    echo "success";
else
    echo "failse";
die(); 
`
ok，这样就可以发送邮件了

如需加载模板 把$mail= Yii::$app->mailer->compose(); 

修改成 $mail= Yii::$app->mailer->compose('xiaoma',['aa'=>222]); 

注:aa是想xiaoma.php里面传递的参数。

3 邮件模板 xiaoma.php里面的代码 ：

`
<?php
use yii\helpers\Html;
/* @var $this yii\web\View */
/* @var $user common\models\User */
$resetLink = Yii::$app->urlManager->createAbsoluteUrl(['site/reset-password', 'token' => $aa]);
?>
< a href="#" ><?php echo $resetLink ?></a>
`

3 加载模板的邮件代码：

`
$mail= Yii::$app->mailer->compose('xiaoma',['aa'=>222]);
       $mail->setTo('1401619705@qq.com');
       $mail->setSubject("邮件测试");
       $mail->setTextBody('zheshisha ');
       if($mail->send())
           echo "success";
       else
           echo "failse";
     die();
`

通过以上配置就ok了


## yii2高级模板中自定义类库

* 我定义的一般在common这个目录下创建一个文件夹，如：

> common\libs;

> common\libs\Tools.php;

* 在controller中调用刚刚创建的这个类库，如：

> use common\libs\Tools;  //这个是命名空间的调用

> Tools::test();  //控制器中的调用，这个是静态方法


## yii2 修改布局(layout)

* 方案1:控制器内成员变量

> public $layout = false; //不使用布局

> public $layout = "main"; //设置使用的布局文件

* 方案2：控制器成员方法内

> $this->layout = false; //不使用布局

> $this->layout = "main"; //设置使用的布局文件

* 方案3：视图中选择布局

> $this->context->layout = false; //不使用布局

> $this->context->layout = 'main'; //设置使用的布局文件
