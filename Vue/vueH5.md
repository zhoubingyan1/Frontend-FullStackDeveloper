###vue开发微信浏览器H5

##坑1: 前端路由模式 hash history

https://www.cnblogs.com/fan-zha/p/11250549.html

在hash情况下 页面域名上 比如
背景:vue中单页面SPA项目和History模式，导致微信网页签名失败，invalid signature
ue项目中为了去掉路径中的#号，导致微信签名失败，原因分析：history模式副带的页面刷新问题和iOS、Android获取url方式不同的兼容问题，在vue-router模式为history的情况下, 由于IOS微信浏览器在验证微信jssdk签名时,需要的URL是第一次进入该应用时的URL, 并不是当前页面的URL, 所以这里需要针对IOS微信浏览器作特殊处理；
从 A页面，跳转到B页面，由于没有刷新，B调用 JSSDK的 内容，由于vue-router切换的时候 都是操作的浏览器历史记录，真实url为第一次刚进入时的url。每次路由变化时都重新请求下签名，签名的url 需要用第一次进入时的url

IOS：微信IOS版，每次切换路由，SPA的url是不会变的，发起签名请求的url参数必须是当前页面的url就是最初进入页面时的url

Android：微信安卓版，每次切换路由，SPA的url是会变的，发起签名请求的url参数必须是当前页面的url(不是最初进入页面时的)

思路

由于hash模式# 号的存在，授权后链接会被扰乱。所以 我希望在 授权前 将重定向的链接 即 redirect_uri 改为没有# 的url。然后在 项目入口处 进行 url 重置，将其改回到 丑陋的 带#链接。

即：两步操作

1、授权前 将redirect_uri 改为没有# 的url

2、项目入口处，将没有#的url改回到 带#的url，跳到指定页面。

主要代码

1.授权前，redirect_uri 重置
这里会将当前页面的原有参数信息、额外参数、codepath参数 作为 redirect_uri 的新参数。其中codepath是 用来指定 跳转页面的，通常为 授权发起页。

```shell
/**
   * 开始设置页面链接，进行code 授权重定向
   *
   * @param scope 授权作用域
   * @param path  授权所在页面 path
   * @param extraPm 额外参数，需要在 配置中 进行参数配置
   * @returns {string}
   */
  get_codeV3(scope, path, extraPm={}) {
        if (process.env.NODE_ENV === 'development' && config.isMockWeChat)
          return '0713eFVW0AmFP12kGVTW0oGDVW03eFV2'

      let homeUrl = '', searchOb = {}, searchstr = ''
      let oriUrls = window.location.href.split('?')
      let baseShareURl = window.location.origin + window.location.pathname
      homeUrl = baseShareURl
      if (oriUrls.length > 1) {
        let searchUrls = oriUrls[1].split('#')
        let searchUrlReal = searchUrls[0]
        searchOb = qs.parse(searchUrlReal)
        let {code} = searchOb
        if (code)
          delete searchOb.code
      }
      searchOb.codepath = path
      Object.assign(searchOb, extraPm)
      searchstr = `?${qs.stringify(searchOb)}`
      homeUrl += searchstr

      let redirect_uri = encodeURIComponent(homeUrl);

      let response_type = 'code';
      // 应用授权作用域，
      // snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），
      // snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且， 即使在未关注的情况下，只要用户授权，也能获取其信息 ）

      // 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
      let state = '';
      let url = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${appid}&redirect_uri=${redirect_uri}&response_type=${response_type}&scope=${scope}&state=${state}#wechat_redirect`;
      window.location.href = url;
    }
```

2、链接前置判断
在项目入口处，发现有 codepath 参数，即 将该链接视为 授权后的redirect_uri 链接。进行页面重定向，跳转到 codepath 指定页面

```shell
/**
   * 页面重定向（授权）-确保页面链接正常
   * 支持多参、无参、指定页面　　* code 在 返回的search 里
   * @returns {{needRedirect: boolean, homeUrl: string, search: {}}}
   */
  urlResetForWxCode: function () {
    //需要重定向时：codepath 参数 与 code 参数同在。

    let needRedirect = false, homeUrl = '', searchOb = {}, searchstr = ''
    let oriUrls = window.location.href.split('?')
    let baseShareURl = window.location.origin + window.location.pathname
    homeUrl = baseShareURl
    if (oriUrls.length > 1) {
      let searchUrls = oriUrls[1].split('#')
      let searchUrlReal = searchUrls[0]
      let searchObReal = {}
      searchOb = qs.parse(searchUrlReal)
      let { codepath } = searchOb
      needRedirect = codepath
      if (codepath) {
        config.platform.wxCode[codepath].forEach(item => searchObReal[item] = searchOb[item])
        if (config.platform.wxCode[codepath].length)
          searchstr = `?${qs.stringify(searchObReal)}`
        homeUrl += `#${codepath}` + (searchstr.length > 1 ? searchstr : '')
      }
    }
    return {
      needRedirect,
      homeUrl,
      search: searchOb
    }
  }
```

// 配置信息

```shell
config = {
  platform: {
    wxCode: {'/': ['hello', 'yiha'], '/spe': ['spp']}, // 微信授权-需要授权的页面： [页面用到的参数]
　　｝
｝
```
