# OAuth, OAuth2


This is a [plugin](https://github.com/iris-contrib/plugin/tree/master/oauth).

这是一个 [插件](https://github.com/iris-contrib/plugin/tree/master/oauth)。

This plugin helps you to be able to connect your clients using famous websites login APIs, it is a bridge to the [goth](https://github.com/markbates/goth).

这个插件能够帮助你使用著名网站的登录API连接你的客户，它是连接 [goth](https://github.com/markbates/goth) 的桥梁。

## 支持的网站 / Supported Providers

    Amazon
    Bitbucket
    Box
    Cloud Foundry
    Digital Ocean
    Dropbox
    Facebook
    GitHub
    Gitlab
    Google+
    Heroku
    InfluxCloud
    Instagram
    Lastfm
    Linkedin
    OneDrive
    Paypal
    SalesForce
    Slack
    Soundcloud
    Spotify
    Steam
    Stripe
    Twitch
    Twitter
    Uber
    Wepay
    Yahoo
    Yammer


## 如何使用 - 复杂 / How to use - high level



```go
    configs := oauth.Config{
      Path: "/auth", //defaults to /auth /默认路径 /auth

      GithubKey:    "YOUR_GITHUB_KEY",
      GithubSecret: "YOUR_GITHUB_SECRET",
      GithubName:   "github", // defaults to github / 默认 github

      FacebookKey:    "YOUR_FACEBOOK_KEY",
      FacebookSecret: "YOUR_FACEBOOK_KEY",
      FacebookName:   "facebook", // defaults to facebook / 默认是 facebook
      //and so on... enable as many as you want
      // 等等... 只要你想要
    }

	// create the plugin with our configs
	// 使用自己的配置创建插件
	authentication := oauth.New(configs)
	// register the plugin to iris
	// 将插件注册到 iris
	iris.Plugins.Add(authentication)

    // came from yourhost:port/configs.Path/theprovidername
    // 来自 yourhost:port/configs.Path/theprovidername
    // this is the handler inside yourhost:port/configs.Path/theprovidername/callback
    // 这是 yourhost:port/configs.Path/theprovidername/callback 内部的处理器
    // you can do redirect to the authenticated url or whatever you want to do
    // 你可以重定向到身份认证的url 或 任何你想要做的
	authentication.Success(func(ctx *iris.Context) {
		user := authentication.User(ctx) // returns the goth.User / 返回 goth.User
    })
    authentication.Fail(func(ctx *iris.Context){})

```

Example:

示例:

```go
// main.go
package main

import (
	"sort"
	"strings"

	"github.com/iris-contrib/plugin/oauth"
	"github.com/kataras/iris"
)

// register your auth via configs, providers with non-empty values will be registered to goth automatically by Iris
// 通过 configs 变量 注册你自己的 auth 路径，非空值的供应者将被 Iris 自动注册到 goth
var configs = oauth.Config{
	Path: "/auth", //defaults to /oauth 默认 /auth

	GithubKey:    "YOUR_GITHUB_KEY",
	GithubSecret: "YOUR_GITHUB_SECRET",
	GithubName:   "github", // defaults to github / 默认 github

	FacebookKey:    "YOUR_FACEBOOK_KEY",
	FacebookSecret: "YOUR_FACEBOOK_KEY",
	FacebookName:   "facebook", // defaults to facebook / 默认为 facebook
}

func init() {
	iris.Config.Sessions.Provider = "memory"
}

// ProviderIndex ...
// 供应者索引 ...
type ProviderIndex struct {
	Providers    []string
	ProvidersMap map[string]string
}

func main() {
	// create the plugin with our configs
	// 使用自己的配置创建插件
	authentication := oauth.New(configs)
	// register the plugin to iris
	// 将插件注册到 iris
	iris.Plugins.Add(authentication)

	m := make(map[string]string)
	m[configs.GithubName] = "Github" // same as authentication.Config.GithubName 同 authentication.Config.GithubName
	m[configs.FacebookName] = "Facebook"

	var keys []string
	for k := range m {
		keys = append(keys, k)
	}
	sort.Strings(keys)

	providerIndex := &ProviderIndex{Providers: keys, ProvidersMap: m}

	// set a  login success handler( you can use more than one handler)
	// 设置登录成功处理器 (你可以不只使用一个处理器)
	// if user succeed to logged in
	// 如果用户登录成功
	// client comes here from: localhost:3000/config.RouteName/lowercase_provider_name/callback 's first handler, but the  previous url is the localhost:3000/config.RouteName/lowercase_provider_name
	// 客户端来自: localhost:3000/config.RouteName/lowercase_provider_name/callback 的第一个处理器，而其先前的 url  是 localhost:3000/config.RouteName/lowercase_provider_name
	authentication.Success(func(ctx *iris.Context) {
		// if user couldn't validate then server sends StatusUnauthorized, which you can handle by:  authentication.Fail OR iris.OnError(iris.StatusUnauthorized, func(ctx *iris.Context){})
		// 如果用户验证没有通过，那么服务器会发送 StatusUnauthorized，你可以通过一下方式来处理：authentication.Fail OR iris.OnError(iris.StatusUnauthorized, func(ctx *iris.Context){})
		user := authentication.User(ctx)

		// you can get the url by the named-route 'oauth' which you can change by Config's field: RouteName
		// 你可以通过命名路由 'auth' 取得 url，当然你也改变 Config 的 RouteName 字段来改变它。
		println("came from " + authentication.URL(strings.ToLower(user.Provider)))
		ctx.Render("user.html", user)
	})

	// customize the error page using: authentication.Fail(func(ctx *iris.Context){....})
	// 自定义错误页面使用: authentication.Fail(func(ctx *iris.Context){....})

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("index.html", providerIndex)
	})

	iris.Listen(":3000")
}

```

View: 

```html
<!-- ./templates/index.html -->

{{range $key,$value:=.Providers}}
    <p><a href="{{ url "oauth" $value}}">Log in with {{index $.ProvidersMap $value}}</a></p>
{{end}}

```

```html
<!-- ./templates/user.html -->
<p>Name: {{.Name}}</p>
<p>Email: {{.Email}}</p>
<p>NickName: {{.NickName}}</p>
<p>Location: {{.Location}}</p>
<p>AvatarURL: {{.AvatarURL}} <img src="{{.AvatarURL}}"></p>
<p>Description: {{.Description}}</p>
<p>UserID: {{.UserID}}</p>
<p>AccessToken: {{.AccessToken}}</p>
<p>ExpiresAt: {{.ExpiresAt}}</p>
<p>RefreshToken: {{.RefreshToken}}</p>

```

## 如何使用 - 简单 ／ How to use - low level

Low-level is just the [iris-contrib/gothic](https://github.com/iris-contrib/gothic) which is like the original [goth](https://github.com/markbates/goth) but converted to work with Iris.

简单的方式就是使用 [iris-contrib/gothic](https://github.com/iris-contrib/gothic) 就像原始的 [goth](https://github.com/markbates/goth)，只是转换成与 Iris 一同使用。

Example:

示例:


```go
package main

import (
	"html/template"
	"os"

	"sort"

	"github.com/iris-contrib/gothic"
	"github.com/kataras/iris"
	"github.com/markbates/goth"
	"github.com/markbates/goth/providers/amazon"
	"github.com/markbates/goth/providers/bitbucket"
	"github.com/markbates/goth/providers/box"
	"github.com/markbates/goth/providers/digitalocean"
	"github.com/markbates/goth/providers/dropbox"
	"github.com/markbates/goth/providers/facebook"
	"github.com/markbates/goth/providers/github"
	"github.com/markbates/goth/providers/gitlab"
	"github.com/markbates/goth/providers/gplus"
	"github.com/markbates/goth/providers/heroku"
	"github.com/markbates/goth/providers/instagram"
	"github.com/markbates/goth/providers/lastfm"
	"github.com/markbates/goth/providers/linkedin"
	"github.com/markbates/goth/providers/onedrive"
	"github.com/markbates/goth/providers/paypal"
	"github.com/markbates/goth/providers/salesforce"
	"github.com/markbates/goth/providers/slack"
	"github.com/markbates/goth/providers/soundcloud"
	"github.com/markbates/goth/providers/spotify"
	"github.com/markbates/goth/providers/steam"
	"github.com/markbates/goth/providers/stripe"
	"github.com/markbates/goth/providers/twitch"
	"github.com/markbates/goth/providers/twitter"
	"github.com/markbates/goth/providers/uber"
	"github.com/markbates/goth/providers/wepay"
	"github.com/markbates/goth/providers/yahoo"
	"github.com/markbates/goth/providers/yammer"
)

func init() {
	iris.Config.Sessions.Provider = "memory" // or "redis" and configure the Redis Provider / 或者 "redis" 来配置 Redis Provider
}

func main() {
	goth.UseProviders(
		twitter.New(os.Getenv("TWITTER_KEY"), os.Getenv("TWITTER_SECRET"), "http://localhost:3000/auth/twitter/callback"),
		// If you'd like to use authenticate instead of authorize in Twitter provider, use this instead.
		// 如果你喜欢使用身份认证替代 Twitter 供应者的授权，可以使用下面代码替代。
		// twitter.NewAuthenticate(os.Getenv("TWITTER_KEY"), os.Getenv("TWITTER_SECRET"), "http://localhost:3000/auth/twitter/callback"),

		facebook.New(os.Getenv("FACEBOOK_KEY"), os.Getenv("FACEBOOK_SECRET"), "http://localhost:3000/auth/facebook/callback"),
		gplus.New(os.Getenv("GPLUS_KEY"), os.Getenv("GPLUS_SECRET"), "http://localhost:3000/auth/gplus/callback"),
		github.New(os.Getenv("GITHUB_KEY"), os.Getenv("GITHUB_SECRET"), "http://localhost:3000/auth/github/callback"),
		spotify.New(os.Getenv("SPOTIFY_KEY"), os.Getenv("SPOTIFY_SECRET"), "http://localhost:3000/auth/spotify/callback"),
		linkedin.New(os.Getenv("LINKEDIN_KEY"), os.Getenv("LINKEDIN_SECRET"), "http://localhost:3000/auth/linkedin/callback"),
		lastfm.New(os.Getenv("LASTFM_KEY"), os.Getenv("LASTFM_SECRET"), "http://localhost:3000/auth/lastfm/callback"),
		twitch.New(os.Getenv("TWITCH_KEY"), os.Getenv("TWITCH_SECRET"), "http://localhost:3000/auth/twitch/callback"),
		dropbox.New(os.Getenv("DROPBOX_KEY"), os.Getenv("DROPBOX_SECRET"), "http://localhost:3000/auth/dropbox/callback"),
		digitalocean.New(os.Getenv("DIGITALOCEAN_KEY"), os.Getenv("DIGITALOCEAN_SECRET"), "http://localhost:3000/auth/digitalocean/callback", "read"),
		bitbucket.New(os.Getenv("BITBUCKET_KEY"), os.Getenv("BITBUCKET_SECRET"), "http://localhost:3000/auth/bitbucket/callback"),
		instagram.New(os.Getenv("INSTAGRAM_KEY"), os.Getenv("INSTAGRAM_SECRET"), "http://localhost:3000/auth/instagram/callback"),
		box.New(os.Getenv("BOX_KEY"), os.Getenv("BOX_SECRET"), "http://localhost:3000/auth/box/callback"),
		salesforce.New(os.Getenv("SALESFORCE_KEY"), os.Getenv("SALESFORCE_SECRET"), "http://localhost:3000/auth/salesforce/callback"),
		amazon.New(os.Getenv("AMAZON_KEY"), os.Getenv("AMAZON_SECRET"), "http://localhost:3000/auth/amazon/callback"),
		yammer.New(os.Getenv("YAMMER_KEY"), os.Getenv("YAMMER_SECRET"), "http://localhost:3000/auth/yammer/callback"),
		onedrive.New(os.Getenv("ONEDRIVE_KEY"), os.Getenv("ONEDRIVE_SECRET"), "http://localhost:3000/auth/onedrive/callback"),

		//Pointed localhost.com to http://localhost:3000/auth/yahoo/callback through proxy as yahoo
		// 像 yahoo 一样通过代理将 localhost.com 指向 http://localhost:3000/auth/yahoo/callback
		// does not allow to put custom ports in redirection uri
		// 不要允许在重定向 uri 中加上自定义端口
		yahoo.New(os.Getenv("YAHOO_KEY"), os.Getenv("YAHOO_SECRET"), "http://localhost.com"),
		slack.New(os.Getenv("SLACK_KEY"), os.Getenv("SLACK_SECRET"), "http://localhost:3000/auth/slack/callback"),
		stripe.New(os.Getenv("STRIPE_KEY"), os.Getenv("STRIPE_SECRET"), "http://localhost:3000/auth/stripe/callback"),
		wepay.New(os.Getenv("WEPAY_KEY"), os.Getenv("WEPAY_SECRET"), "http://localhost:3000/auth/wepay/callback", "view_user"),
		//By default paypal production auth urls will be used, please set PAYPAL_ENV=sandbox as environment variable for testing
		//in sandbox environment
		// paypal 默认使用产品环境下的认证URLs，在测试沙盒环境下请设置 PAYPAL_ENV=sandbox 环境变量
		paypal.New(os.Getenv("PAYPAL_KEY"), os.Getenv("PAYPAL_SECRET"), "http://localhost:3000/auth/paypal/callback"),
		steam.New(os.Getenv("STEAM_KEY"), "http://localhost:3000/auth/steam/callback"),
		heroku.New(os.Getenv("HEROKU_KEY"), os.Getenv("HEROKU_SECRET"), "http://localhost:3000/auth/heroku/callback"),
		uber.New(os.Getenv("UBER_KEY"), os.Getenv("UBER_SECRET"), "http://localhost:3000/auth/uber/callback"),
		soundcloud.New(os.Getenv("SOUNDCLOUD_KEY"), os.Getenv("SOUNDCLOUD_SECRET"), "http://localhost:3000/auth/soundcloud/callback"),
		gitlab.New(os.Getenv("GITLAB_KEY"), os.Getenv("GITLAB_SECRET"), "http://localhost:3000/auth/gitlab/callback"),
	)

	m := make(map[string]string)
	m["amazon"] = "Amazon"
	m["bitbucket"] = "Bitbucket"
	m["box"] = "Box"
	m["digitalocean"] = "Digital Ocean"
	m["dropbox"] = "Dropbox"
	m["facebook"] = "Facebook"
	m["github"] = "Github"
	m["gitlab"] = "Gitlab"
	m["soundcloud"] = "SoundCloud"
	m["spotify"] = "Spotify"
	m["steam"] = "Steam"
	m["stripe"] = "Stripe"
	m["twitch"] = "Twitch"
	m["uber"] = "Uber"
	m["wepay"] = "Wepay"
	m["yahoo"] = "Yahoo"
	m["yammer"] = "Yammer"
	m["gplus"] = "Google Plus"
	m["heroku"] = "Heroku"
	m["instagram"] = "Instagram"
	m["lastfm"] = "Last FM"
	m["linkedin"] = "Linkedin"
	m["onedrive"] = "Onedrive"
	m["paypal"] = "Paypal"
	m["twitter"] = "Twitter"
	m["salesforce"] = "Salesforce"
	m["slack"] = "Slack"

	var keys []string
	for k := range m {
		keys = append(keys, k)
	}
	sort.Strings(keys)

	providerIndex := &ProviderIndex{Providers: keys, ProvidersMap: m}

	iris.Get("/auth/:provider/callback", func(ctx *iris.Context) {

		user, err := gothic.CompleteUserAuth(ctx)
		if err != nil {
			ctx.SetStatusCode(iris.StatusUnauthorized)
			ctx.Write(err.Error())
			return
		}

		t, _ := template.New("foo").Parse(userTemplate)
		ctx.ExecuteTemplate(t, user)
	})

	iris.Get("/auth/:provider", func(ctx *iris.Context) {
		err := gothic.BeginAuthHandler(ctx)
		if err != nil {
			ctx.Log(err.Error())
		}
	})

	iris.Get("/", func(ctx *iris.Context) {
		t, _ := template.New("foo").Parse(indexTemplate)
		ctx.ExecuteTemplate(t, providerIndex)
	})
	iris.Listen(":3000")
}

// ProviderIndex ...
type ProviderIndex struct {
	Providers    []string
	ProvidersMap map[string]string
}

var indexTemplate = `{{range $key,$value:=.Providers}}
    <p><a href="/auth/{{$value}}">Log in with {{index $.ProvidersMap $value}}</a></p>
{{end}}`

var userTemplate = `
<p>Name: {{.Name}}</p>
<p>Email: {{.Email}}</p>
<p>NickName: {{.NickName}}</p>
<p>Location: {{.Location}}</p>
<p>AvatarURL: {{.AvatarURL}} <img src="{{.AvatarURL}}"></p>
<p>Description: {{.Description}}</p>
<p>UserID: {{.UserID}}</p>
<p>AccessToken: {{.AccessToken}}</p>
<p>ExpiresAt: {{.ExpiresAt}}</p>
<p>RefreshToken: {{.RefreshToken}}</p>
`

```



*high level and low level, no performance differences*

*复杂 和 简单， 没有性能上的不同*
