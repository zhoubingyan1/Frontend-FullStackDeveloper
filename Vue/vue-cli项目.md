vue init webpack projectname
* 如果是在cookie里设置token的话，可以在其他的页面写
```shell
	updated() {
		//刷新当前页面的时候重新调用新的cookie
		this.AdminToken = this.getCookie("token");
	}
```

#axios使用
/*
	npm install axios --save
	在main.js中
	import Vue from 'vue'
	import axios from 'axios'

	//防止重复提交ajax
	let pending = []
	let CancelToken = axios.CancelToken
	let removePending = (config, f) => {
		let flagUrl = config.url + '&' + config.method
		pending.indexOf(flagUrl) !== -1 ?
			f ?
				f() :
				pending.splice(pending.indexOf(flagUrl), 1)
			:
			f ?
				pending.push(flagUrl) :
				pending = []
	}

	//添加请求拦截器 loading
	axios.interceptors.request.use(function (config) {
		//当请求的方式是post时候，调用防止重复提交ajax
		if (config.method === 'post') {
			config.cancelToken = new CancelToken((c) => {
				removePending(config, c)
			})
		}
		//添加时间
	  if (config.url.indexOf("?")>=0){
	    config.url = config.url + "&t=" + Date.parse( new Date() )
	  } else {
	    config.url = config.url + "?t=" + Date.parse( new Date() )
	  }

		//请求中
	  Indicator.open({
	    text: '加载中...',
	    spinnerType: 'fading-circle'
	  })
	  return config
	}),function (error) {
	  Indicator.close()
	  return Promise.reject(error)
	}
	axios.interceptors.response.use(function (config) {
		//加载完成
	  Indicator.close()
		removePending(config.config)
	  return config
	}),function (error) {
	  return Promise.reject(error)
	}

	

	import Vue from 'vue'
	import Router from 'vue-router'
	import axios from 'axios'
	import router from '@/router/index'
	import { LoadingBar,Message,Notice,Modal } from 'iview'

	Vue.use(Router)

	new Vue({router})

	LoadingBar.config({
	  color: '#f900f2',
	  failedColor: '#f0ad4e',
	  height: 2.3
	})

	Message.config({
	  top: 80,
	  duration: 3
	})
	console.log(sessionStorage.getItem('token'));
	const arg = axios.create({
	  timeout: 6000,
	  // headers: {
	  //   'token': sessionStorage.getItem('token')?sessionStorage.getItem('token'):''
	  // }
	})
	//网络监测
	if (!navigator.onLine){
	  Notice.destroy()
	  Notice.error({
	    title: '提示',
	    desc: '无网络连接'
	  })
	}
	const EventUtil = {
	  addHandler: function (element, type, handler) {
	    if (element.addEventListener) {
	      element.addEventListener(type, handler, false)
	    } else if (element.attachEvent) {
	      element.attachEvent("on" + type, handler)
	    } else {
	      element["on" + type] = handler
	    }
	  }
	}
	EventUtil.addHandler(window, "online", ()=>{
	  Notice.destroy()
	  Notice.success({
	    title: '提示',
	    desc: '已连接到网络'
	  })
	})
	EventUtil.addHandler(window, "offline", ()=>{
	  Notice.destroy()
	  Notice.error({
	    title: '提示',
	    desc: '网络连接已断开'
	  })
	})

	//添加请求拦截器 loading
	arg.interceptors.request.use(config=>{
	  config.headers.token = sessionStorage.getItem('token')
	  LoadingBar.start()
	  return config
	},error=> {
	  LoadingBar.error()
	  Message.error('请求失败')
	  return Promise.reject(error)
	})
	arg.interceptors.response.use(config=> {
	  LoadingBar.finish()
	  //401过期，402其他地方登录
	  if (Number(config.data.status) === 401 || Number(config.data.status) === 402) {
	    Modal.remove()
	    Message.error(config.data.errMsg.msg)
	    router.replace({
	      path: '/'
	    })
	    return false
	  } else {
	    // console.log(config.data)
	    // if (config.data.hasOwnProperty('info')) {
	    //   if (config.data.info.hasOwnProperty('token')) {
	    //     sessionStorage.setItem('token', config.data.info.token)
	    //   }
	    // }
	    return config
	  }
	},error=> {
	  LoadingBar.error()
	  if (!navigator.onLine){
	    Notice.destroy()
	    Notice.error({
	      title: '提示',
	      desc: '无网络连接'
	    })
	    return
	  }
	  if (error.response === undefined) {
	    Message.destroy()
	    Message.error('网络错误，请重试')
	  }
	  error.message = {
	    status: error.response.status,
	  }
	  switch (error.response.status) {
	    case 403:
	      error.message.msg = '服务器拒绝请求 '
	          break
	    case 404:
	      error.message.msg = '请求地址不存在'
	      break
	    case 500:
	      error.message.msg = '内部服务器错误 '
	          break
	    case 502:
	      error.message.msg = '错误的网关 '
	      break
	    case 503:
	      error.message.msg = '服务当前不可用 '
	      break
	    case 504:
	      error.message.msg = '服务器超时 '
	      break
	  }
	  Message.destroy()
	  Message.error(error.message.msg)
	  return Promise.reject(error.message)
	})

	Vue.prototype.$http = arg
*/

#Rounter使用
/*
	方法1:
		import Vue from 'vue'
		import Router from 'vue-router'
		Vue.use(Router)

		export default new Router({
			routes:[
				{
					path: '/',
      				name: 'login',
      				component:resolve=>require(['../pages/login,resolve)
				}
			]
		})
		

	方法2：
		import Vue from 'vue'
		import Router from 'vue-router'
		Vue.use(Router)
		
		const Login = ()=> import('@/pages/login/login')
		const Home = ()=> import('@/pages/home')
		const Personal = () => import('@/pages/personinfo/personal')

		export default new Router({
			routers:[
				{
			      path: '/',
			      name: 'login',
			      component: Login
			    },
			    {
			      path: '/home',
			      name: 'home',
			      component: Home,
			      children:[
			        {
			          // 个人中心
			          path:"/personal",
			          name:"personal",
			          component: Personal
			        }
			      ]
			    }
			]
		})


	方法3：
		import Vue from 'vue'
		import Router from 'vue-router'
		Vue.use(Router)

		import Login from '@/pages/login/login'
		import Home from '@/pages/home'
		import Personal from '@/pages/personinfo/personal'

		export default new Router({
			routers:[
				{
			      path: '/',
			      name: 'login',
			      component: Login
			    },
			    {
			      path: '/home',
			      name: 'home',
			      component: Home,
			      children:[
			        {
			          // 个人中心
			          path:"/personal",
			          name:"personal",
			          component: Personal
			        }
			      ]
			    }
			]
		})
*/

#配置反向代理
/*
	
	找到config文件中的index.js
	module.exports = {
	  	dev: {
			proxyTable: {
		      '/': {
		        target: 'http://scrm-dev.gome.max-digital.cn/',
		        changeOrigin: true,
		        pathRewrite: {
		          '^/': ''
		        }
		      }
		    }
	    }
    }
	
*/

#vuex使用
/*
	新建store.js
	import Vue from 'vue'
	import Vuex from 'Vuex'

	Vue.use(Vuex)
	export default new Vuex.Store({
		state:{
			//存储状态。也就是变量;
		},
		getters:{
			//派生状态。也就是set、get中的get，有两个可选参数：state、getters分别可以获取state中的变量和其他的getters。外部调用方式：store.getters.personInfo()。就和vue的computed差不多；
		},
		mutations:{
			//提交状态修改。也就是set、get中的set，这是vuex中唯一修改state的方式，但不支持异步操作。第一个参数默认是state。外部调用方式：store.commit('setAge', 18)。和vue中的methods类似。一条重要的原则就是要记住 mutation 必须是同步函数。
		},
		actions:{
			//和mutations类似。不过actions支持异步操作。第一个参数默认是和store具有相同参数属性的对象。外部调用方式：store.dispatch('nameAsyn')。Action 提交的是 mutation，而不是直接变更状态。Action 可以包含任意异步操作
		},

	})

	在main.js
	import store from './store/store'
	new Vue({
	  el: '#app',
	  router,
	  store,
	  components: { App },
	  template: '<App/>'
	})
*/

#util.js
/*
	//获取cookie、
	export function getCookie(name) {
	  var arr, reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)");
	  if (arr = document.cookie.match(reg))
	      return (arr[2]);
	  else
	      return null;
	}

	//设置cookie
	export function setCookie(c_name, value, expiredays) {
	  var exdate = new Date();
	  exdate.setDate(exdate.getDate() + expiredays);
	  document.cookie = c_name + "=" + escape(value) + ((expiredays == null) ? "" : ";expires=" + exdate.toGMTString());
	};

	//删除cookie
	export function delCookie(name){
	  var exp = new Date();
	  exp.setTime(exp.getTime() - 1);
	  var cval = getCookie(name);
	  if (cval != null)
	      document.cookie = name + "=" + cval + ";expires=" + exp.toGMTString();
	};

	//复制内容到剪切板
	//id: 要复制的内容的标签id，例如'#copy-content'
	export function copyContent(id){
	  let range;
	  if (document.selection) {
	　　//IE
	    range = document.body.createTextRange();
	    range.moveToElementText(document.getElementById('selectable'));
	    range.select();
	  }else if (window.getSelection) {
	    range = document.createRange();
	    let content = document.querySelector(id);
	    range.selectNode(content);
	    let selection = window.getSelection();
	    if(selection.rangeCount > 0) {
	      selection.removeAllRanges();
	    }
	    selection.addRange(range);
	  }
	  document.execCommand('copy');
	};

	/** 转换时间戳为指定格式
	 * lzg 2018-04-27
	 * @param {Number} timestamp 时间戳：10位就*1000, 13位或标准时间字符串保留，不是就默认转当前时间
	 * @param {String} type 格式类型：支持8种格式。默认格式 年月日时分秒
	 * @param {String} joinStr 连字符："-", "/"等等。 默认 "-" 连接年月日
	 */
	export function getFormatDate(timestamp, type, joinStr) {
	  function addZero(n) {
	    return Number(n) < 10 ? "0" + n : n;
	  }
	  if((timestamp).toString().length == 10) {
	    date = new Date(timestamp * 1000);
	  }else if((timestamp).toString().length >= 13){
	    date = new Date(timestamp);
	  }else {
	    date = new Date();
	  }
	  if(!type) {
	    type = "ymdhms";
	  }
	  if(!joinStr) {
	    joinStr = "-";
	  }

	  var date;
	  var time;
	  
	  var Y = date.getFullYear();
	  var M = addZero(date.getMonth() + 1);
	  var D = addZero(date.getDate());
	  var h = addZero(date.getHours());
	  var m = addZero(date.getMinutes());
	  var s = addZero(date.getSeconds());

	  time = Y + joinStr + M + joinStr + D + ' ' + h + ':' + m + ':'+ s;

	  //年月日
	  if(type == "ymd") {
	    time = Y + joinStr + M + joinStr + D;
	  }
	  //年月
	  if(type == "ym") {
	    time = Y + joinStr + M;
	  }
	  //月日
	  if(type == "md") {
	    time = M + joinStr + D;
	  }
	  //时分秒
	  if(type == "hms") {
	    time = h + ':' + m + ':'+ s;
	  }
	  //时分
	  if(type == "hm") {
	    time = h + ':' + m;
	  }
	  //年月日 时分秒
	  if(type == "ymdhms") {
	    time = Y + joinStr + M + joinStr + D + ' ' + h + ':' + m + ':'+ s;
	  }
	  //年月日 时分
	  if(type == "ymdhm") {
	    time = Y + joinStr + M + joinStr + D + ' ' + h + ':' + m;
	  }
	  //月日 时分
	  if(type == "mdhm") {
	    time = M + joinStr + D + ' ' + h + ':' + m;
	  }

	  return time;
	}
*/


#vue权限管理(以国美和koc为例)
	* 国美:登录页(login)、首页(indexpage)、忘记密码(resetPassword)、账号详情(accountdetail)、个人信息(personal)
	>方法一:
		1.登录后根据这个用户获取author,存在 sessionStorage.setItem('auth_dis',JSON.stringify(res.data.author)),限制页面上小按钮点击跳转的权限
		author:[
			{author_id: 1, author_name: "消息推送"},
			{author_id: 2, author_name: "欢迎语设置"},
			{author_id: 3, author_name: "菜单设置"},
			{author_id: 4, author_name: "用户列表"},
			{author_id: 5, author_name: "会员信息管理"},
			{author_id: 6, author_name: "用户等级分组管理"},
			{author_id: 7, author_name: "修改会员积分"},
			{author_id: 8, author_name: "自定义便签"},
			{author_id: 9, author_name: "查看用户画像"},
			{author_id: 10, author_name: "用户数据分析"},
			{author_id: 11, author_name: "活动分析"},
			{author_id: 12, author_name: "帐号管理"},
			{author_id: 13, author_name: "角色管理"},
			{author_id: 14, author_name: "部门管理"},
			{author_id: 15, author_name: "任务管理"},
			{author_id: 16, author_name: "门店位置"}
			]

		a.全局自定义指令 v-auth
		//directives.js
		```shell
		import Vue from 'vue'
		Vue.directive('auth',{
			inserted: function (el,v) {
				let auth_dis = JSON.parse(sessionStorage.getItem('auth_dis'))
				let is_admin = sessionStorage.getItem('is_admin')
				let sigmor = null
				if (is_admin && is_admin == 0) {
					for (let i in auth_dis) {
						if (auth_dis[i].author_id == v.value) {
							sigmor = true
							break
						} else {
							sigmor = false
						}
					}
				} else {
					sigmor = true
				}
				if (sigmor === false) {
					el.remove()
				}
			}
		})
		```
		/*
			//所有的权限
			//[
			// {"author_id":1,"author_name":"消息推送"},
			// {"author_id":2,"author_name":"欢迎语设置"},
			// {"author_id":3,"author_name":"菜单设置"},
			// {"author_id":4,"author_name":"用户列表"},
			// {"author_id":5,"author_name":"会员信息管理"},
			// {"author_id":6,"author_name":"用户等级分组管理"},
			// {"author_id":7,"author_name":"修改会员积分"},
			// {"author_id":8,"author_name":"自定义便签"},
			// {"author_id":9,"author_name":"查看用户画像"},
			// {"author_id":10,"author_name":"用户数据分析"},
			// {"author_id":11,"author_name":"活动分析"},
			// {"author_id":12,"author_name":"帐号管理"},
			// {"author_id":13,"author_name":"角色管理"},
			// {"author_id":14,"author_name":"部门管理"},
			// {"author_id":15,"author_name":"任务管理"},
			// {"author_id":16,"author_name":"门店位置"}
			// ]	
		*/

		在使用到权限的页面使用自定义指令<div v-auth="6" class="managementgroup" @click="$router.push('/managementGroup')">管理分组</div>

		2.侧边栏显示的权限 发送请求获取回来的放在sessionStorage.setItem('auth', JSON.stringify(res.data.info.auth_list))
		[ 
			{ 
				"auth_type": "个性化营销", 
				"authors": [ 
						{ "author_id": "1-1", "author_name": "消息推送" },
						{ "author_id": "1-2", "author_name": "欢迎语设置" },
						{ "author_id": "1-3", "author_name": "菜单设置" }
					],
				"auth_type_id": 1, 
				"class": "iconfont icon-xingxing" 
			}, 
			{ 
				"auth_type": "用户管理", 
				"authors": [ 
					{ "author_id": "2-4", "author_name": "用户列表" } 
				], 
				"auth_type_id": 2, 
				"class": "iconfont icon-geren" 
			}, 
			{ 
				"auth_type": "用户画像", 
				"authors": [ 
					{ "author_id": "3-8", "author_name": "自定义便签" }, 
					{ "author_id": "3-9", "author_name": "查看用户画像" } 
				], 
				"auth_type_id": 3, 
				"class": "iconfont icon-zu" 
			}, 
			{ 
				"auth_type": "数据分析", 
				"authors": [ 
					{ "author_id": "4-10", "author_name": "用户数据分析" }, 
					{ "author_id": "4-11", "author_name": "活动分析" } 
				], 
				"auth_type_id": 4, 
				"class": "iconfont icon-shuju"
			}, 
			{ 
				"auth_type": "o2o服务", 
				"authors": [ 
					{ "author_id": "7-16", "author_name": "门店位置" } 
				], 
				"auth_type_id": 7, 
				"class": "iconfont icon-box" 
			}, 
			{ 
				"auth_type": "设置", 
				"authors": [ 
					{ "author_id": "8-12", "author_name": "帐号管理" }, 
					{ "author_id": "8-13", "author_name": "角色管理" }, 
					{ "author_id": "8-14", "author_name": "部门管理" }, 
					{ "author_id": "8-15", "author_name": "任务管理" } 
				], 
				"auth_type_id": 8, 
				"class": "iconfont icon-shezhi" 
			} 
		]	
		//SideNav.vue
		```shell
			<template>
				<div id="sideNav"><!-- :active-name="activeName" :open-names="openNames"-->
					<Menu @on-select="routeTo" :active-name="activeName" ref="Menu">
						<MenuItem name="0" v-if="token">
							<Icon class="iconfont icon-home"></Icon>
							<span style="font-size: 16px">首页</span>
						</MenuItem>
						<Submenu v-if="menuList" :name="item.auth_type_id" v-for="item in menuList" :key="item.auth_type_id" :class="{activeSubMenu: openNames[0]==item.auth_type_id}">
							<template slot="title">
								<Icon :class="item.class"></Icon>
								{{item.auth_type}}
							</template>
							<MenuItem v-if="item.authors.length!==0" v-for="list in item.authors" :name="list.author_id" :key="list.author_id">{{list.author_name}}</MenuItem>
						</Submenu>
					<!-- <Submenu name="2">
							<template slot="title">
								<Icon class="iconfont icon-geren"></Icon>
								用户管理
							</template>
							<MenuItem name="2-1">用户列表</MenuItem>
						</Submenu>
						<Submenu name="3">
							<template slot="title">
								<Icon class="iconfont icon-zu"></Icon>
								用户画像
							</template>
							<MenuItem name="3-1">用户标签</MenuItem>
							<MenuItem name="3-2">用户画像</MenuItem>
						</Submenu>
						<Submenu name="4">
							<template slot="title">
								<Icon class="iconfont icon-box"></Icon>
								O2O服务
							</template>
							<MenuItem name="4-1">门店位置</MenuItem>
						</Submenu>
						<Submenu name="5">
							<template slot="title">
								<Icon class="iconfont icon-shezhi"></Icon>
								设置
							</template>
							<MenuItem name="5-1">账号管理</MenuItem>
							<MenuItem name="5-2">角色管理</MenuItem>
							<MenuItem name="5-3">部门管理</MenuItem>
						</Submenu>-->
					</Menu>
				</div>
			</template>
			<script>
				import Vue from 'vue'
				import { Menu,MenuGroup,MenuItem,Submenu,Icon } from 'iview'
				import routers from '@/router/pagePath'
				Vue.component('Menu', Menu)
				Vue.component('MenuGroup', MenuGroup)
				Vue.component('MenuItem', MenuItem)
				Vue.component('Submenu', Submenu)
				Vue.component('Icon', Icon)
				export default {
					data() {
						return{
							routes: routers.routes,
							token: sessionStorage.getItem('token'),
							/*menuList: [
								{
									id: '1', class: 'iconfont icon-xingxing', name: '个性化营销',children:[
										{id: '1-1', name: '消息推送'},
										{id: '1-2', name: '欢迎语设置'},
										{id: '1-3', name: '菜单设置'},
									]
								},
								{
									id: '2', class: 'iconfont icon-geren', name: '用户管理',children:[
										{id: '2-1', name: '用户列表'}
									]
								},
								{
									id: '3', class: 'iconfont icon-zu', name: '用户画像',children:[
										{id: '3-1', name: '用户标签'},
										{id: '3-2', name: '用户画像'},
									]
								},
								{
									id: '4', class: 'iconfont icon-shuju', name: '数据分析',children:[
										{id: '4-1', name: '用户数据分析'},
										{id: '4-2', name: '活动数据分析'},
									]
								},
								{
									id: '5', class: 'iconfont icon-box', name: 'O2O服务',children:[
										{id: '5-1', name: '门店位置'},
									]
								},
								{
									id: '6', class: 'iconfont icon-shezhi', name: '设置',children:[
										{id: '6-1', name: '账号管理'},
										{id: '6-2', name: '角色管理'},
										{id: '6-3', name: '部门管理'},
										{id: '6-4', name: '任务管理'},
									]
								}
							],*/
							menuList: null,
							activeName: '0',
							openNames: ['0'],
						}
					},
					created(){
						this.getUserInfo()
					},
					watch: {
						'$route': 'setNav'
					},
					methods: {
						routeTo(name) {
							switch (name) {
								case "0":
									this.$router.push({ name: "indexpage" })
											break
								case "1-1":
									this.$router.push({ name: "messages" })
									break
								case "1-2":
									this.$router.push({ name: "greeting" })
									break
								case "1-3":
									this.$router.push({ name: "menuList" })
									break
								case "2-4":
									this.$router.push({ name:'userList'})
											break
								case "3-8":
									this.$router.push({ name: "userLabel" })
									break
								case "3-9":
									this.$router.push({ name: "userPortrayal" })
									break
								case "4-10":
									this.$router.push({ name: "analyzeUser" })
									break
								case "4-11":
									this.$router.push({ name: "analyzeActive" })
									break
								case "7-16":
									this.$router.push({ name: "shopLocation" })
									break
								case "8-12":
									this.$router.push({ name:'manageaccount'})
											break
								case "8-13":
									this.$router.push({ name:'managerole'})
											break
								case "8-14":
									this.$router.push({ name:'managedepartment'})
											break
								case "8-15":
									this.$router.push({ name:'managetask'})
											break
								default:
									break;
							}
						},
						getUserInfo() {
							this.$http.get(this.PATH.GET_SECUR, {
								params: {
									id: sessionStorage.getItem('user_id')?sessionStorage.getItem('user_id'):''
								}
							}).then(res=>{
								console.log(res)
								if (res.data.status == 0) {
									res.data.info.auth_list.forEach( arg => {
										if (arg.auth_type_id === 1) {
											arg.class = 'iconfont icon-xingxing'
										} else if (arg.auth_type_id === 2) {
											arg.class = 'iconfont icon-geren'
										} else if (arg.auth_type_id === 3) {
											arg.class = 'iconfont icon-zu'
										} else if (arg.auth_type_id === 4) {
											arg.class = 'iconfont icon-shuju'
										} else if (arg.auth_type_id === 7) {
											arg.class = 'iconfont icon-box'
										} else if (arg.auth_type_id === 8) {
											arg.class = 'iconfont icon-shezhi'
										}
									})
									this.menuList = res.data.info.auth_list
									sessionStorage.setItem('auth', JSON.stringify(this.menuList))
									this.$nextTick(()=>{
										this.setNav()
									})
								} else {
									this.$Modal.error({
										width: 360,
										content: res.data.errMsg.msg
									})
								}
							}).catch(err => {

							})
						},
						//设置菜单高亮
						setNav() {
							let routeName = this.$route.name
							this.activeName = this.routes[routeName].id
							this.openNames = this.routes[routeName].parent
							/*this.$nextTick(function () {
								this.$refs.Menu.updateOpened()
								this.$refs.Menu.updateActiveName()
							})*/
						}
					}
				}
			</script>
			<style lang="scss">
				#sideNav{
					width: 240px;
					height: 100%;
					float: left;
					.ivu-menu-light.ivu-menu-vertical .ivu-menu-item-active:not(.ivu-menu-submenu) {
						border-right: none;
						border-left: 4px solid var(--base);
						color: var(--base);
					}
					.ivu-menu-submenu-title{
						font-size: 16px;
						color: var(--nav);
					}
					.ivu-menu-item{
						font-size: 14px;
						color: var(--font);
					}
					>.ivu-menu{
						height: 94%;
						overflow-y: auto;
						margin-top: 65px;
						padding-bottom: 20px;
						box-shadow: -2px -1px 10px 0 #ece6e6;
						position: fixed;
						transition: all .2s linear;
						&:hover{
							box-shadow: 3px 1px 20px 0 #ece6e6;
						}
						&:before{
							content: '';
							display: block;
							width: 100%;
							height: 55px;
							background-color: #fff;
						}
					}
					.ivu-menu-submenu{
						&:first-of-type{
							.ivu-icon-ios-arrow-down{
								display: none;
							}
						}
					}
					.iconfont{
						font-size: 20px;
					}
					.ivu-menu-vertical.ivu-menu-light:after{
						display: none;
					}
					.ivu-menu-light.ivu-menu-vertical .ivu-menu-item{
						border-right: none;
					}
					.activeSubMenu .ivu-menu-submenu-title {
						color: var(--base);
					}
				}
			</style>
		```

		```	shell
			export default {
				routes: { // 路由对应的菜单id和他爸
					'indexpage': {id: '0', parent: ['0']},
					'personal': {id: '0', parent: ['0']},
					'accountdetail': {id: '0', parent: ['0']},
					'messages': {id: '1-1', parent: ['1']},
					'greeting': {id: '1-2', parent: ['1']},
					'menuList': {id: '1-3', parent: ['1']},
					'userList': {id: '2-4', parent: ['2']},
					'userDetail': {id: '2-4', parent: ['2']},
					'managementGroup': {id: '2-4', parent: ['2']},
					'userLabel': {id: '3-8', parent: ['3']},
					'userPortrayal': {id: '3-9', parent: ['3']},
					'analyzeUser': {id: '4-10', parent: ['4']},
					'analyzeActive': {id: '4-11', parent: ['4']},
					'shopLocation': {id: '7-16', parent: ['7']},
					'manageaccount': {id: '8-12', parent: ['8']},
					'managerole': {id: '8-13', parent: ['8']},
					'managedepartment': {id: '8-14', parent: ['8']},
					'managetask': {id: '8-15', parent: ['8']},
				},
				Login: ()=> import('@/pages/login/login'),
				Error: ()=> import('@/pages/error'),
				ResetPassword: ()=> import('@/pages/login/resetPassword'),
				Home: ()=> import('@/pages/home'),
				Personal: () => import('@/pages/index/personal'),
				Accountdetail: () => import('@/pages/index/accountdetail'),
				IndexPage: () => import('@/pages/index/indexpage'),
				ManageAccount: () => import('@/pages/management/manageAccount'),
				ManageRole: () => import('@/pages/management/manageRole'),
				ManageDepartment: () => import('@/pages/management/manageDepartment'),
				ManageTask: () => import('@/pages/management/manageTask'),
				UserList: () => import('@/pages/usermanagement/userList'),
				ManagementGroup: () => import('@/pages/usermanagement/managementGroup'),
				UserDetail: () => import('@/pages/usermanagement/userDetail'),
				Greeting: () => import('@/pages/marketing/greeting'),
				MenuList: () => import('@/pages/marketing/menuList'),
				Messages: () => import('@/pages/marketing/messages'),
				UserLabel: () => import('@/pages/userPortrayal/userLabel'),
				UserPortrayal: () => import('@/pages/userPortrayal/userPortrayal'),
				ShopLocation: () => import('@/pages/O2Oserve/shopLocation'),
				AnalyzeUser: () => import('@/pages/dataAnalyze/analyzeUser'),
				AnalyzeActive: () => import('@/pages/dataAnalyze/analyzeActive'),
			}
		```

		//authority.js
		```shell
		import Vue from 'vue'
		import router from '../router'
		import RoutePath from '../router/pagePath'
		new Vue({router})

			class _avenger {
				_argLock() {
					let _authID = new Array()
					let forbiden = new Array()
					let _auth = JSON.parse(sessionStorage.getItem('auth'))
					if (_auth) {
						_auth.map((arg)=>{
							[...arg.authors].forEach((v)=>{
								_authID.push(v.author_id)
							})
						})
						for (let i in RoutePath.routes) {
							for (let j in _authID) {
								if (RoutePath.routes[i].id === _authID[j]) {
									forbiden.push(i)
								}
							}
						}
						forbiden.push('indexpage','accountdetail','personal','login','resetPassword')
						forbiden = Array.from(new Set(forbiden))
						return forbiden
					}
				}
			}
			router.beforeEach((to,from,next)=>{
				if (sessionStorage.getItem('auth')) {
					let $forbid = new _avenger()._argLock()
					if ($forbid.toString().indexOf(from.name) !== -1 && $forbid.toString().indexOf(to.name) !== -1) {
						next()
					}
				} else {
					if (to.name !== 'login' && to.name !== 'resetPassword' && to.name !== 'indexpage' && to.name !== 'error') {
						next({path: '/error'})
					} else {
						next()
					}
				}
		})


		window.onload = function () {
			let forbid = new _avenger()._argLock()
			let _localhost = location.href.split('#')[0]
			let _href = location.href.split('#')[1].split('/')[1]
			if (_href.indexOf('?') !== -1) {
				_href = _href.split('?')[0]
			}
			if (forbid && forbid.toString().indexOf(_href) === -1 && _href !== 'error') {
				location.href = _localhost + '#/error'
				location.reload()
			}
		}
		```

		* 宝洁KOC
	>方法二:登录后根据这个用户获取Model_list
		Model_list: [
				{
					"ID": "Social",
					"Model": "社交管理",
					"Name": "1",
					"class": "icon iconfont icon-chengyuanguanli",
					"classify": {
						"Classify_list": [
							{
								"ID": "SOCa",
								"Classify": "微博互动",
								"class": "1-1"
							},	
							{
								"ID": "SOCb",
								"Classify": "KOC微信群互动",
								"class": "1-2"
							}
						]
					}
				},
				{
					"ID": "KOC",
					"Model": "KOC管理",
					"Name": "2",
					"class": "icon iconfont icon-shejiao",
					"classify": {
						"Classify_list": [
								{
									"ID": "KOCa",
									"Classify": "KOC成员管理",
									"class": "2-1"
								},
								{
									"ID": "KOCb",
									"Classify": "积分管理",
									"class": "2-2"
								},
								{
									"ID": "KOCc",
									"Classify": "活动管理",
									"class": "2-3"
								},
								{
									"ID": "KOCd",
									"Classify": "积分核销",
									"class": "2-4"
								}
							]
					}
				},
				{
					"ID": "Report",
					"Model": "报表",
					"Name": "3",
					"class": "icon iconfont icon-icon1",
					"classify": {
						"Classify_list": [
							{
								"ID": "REPa",
								"Classify": "微博互动报表",
								"class": "3-1"
							},
							{
								"ID": "REPb",
								"Classify": "微博客服概况",
								"class": "3-2"
							},
							{
								"ID": "REPc",
								"Classify": "品牌客服概况",
								"class": "3-3"
							},
							{
								"ID": "REPd",
								"Classify": "微信群分析报表",
								"class": "3-4"
							},
							{
								"ID": "REPe",
								"Classify": "KOC信息报表",
								"class": "3-5"
							},
							{
								"ID": "REPf",
								"Classify": "活动反馈报表",
								"class": "3-6"
							},
							{
								"ID": "REPg",
								"Classify": "积分报表",
								"class": "3-7"
							}
						]
					}
				},
				{
					"ID": "Knowledge",
					"Model": "知识库",
					"Name": "4",
					"class": "konwFont icon iconfont icon-zhishiku",
					"classify": {
						"Classify_list": [
							{
								"ID": "KNOa",
								"Classify": "知识库",
								"class": "4-1"
							}
						]
					}
				},
				{
					"ID": "Management",
					"Model": "管理中心",
					"Name": "5",
					"class": "icon iconfont icon-guanlizhongxin",
					"classify": {
						"Classify_list": [
							{
								"ID": "MANa",
								"Classify": "系统用户管理",
								"class": "5-2"
							},
							{
								"ID": "MANb",
								"Classify": "微信群管理",
								"class": "5-3"
							},
							{
								"ID": "MANc",
								"Classify": "微博及品牌管理",
								"class": "5-4"
							}
						]
					}
				}
		]
		a.在side.vue
		```shell
		created() {
			let data = JSON.parse(localStorage.getItem("menuTitle"));
			// console.log(data)
			data[4].classify.Classify_list.unshift({ID: "daskjk", Classify: "个人中心", class: "5-1"})
			this.Model_list = data;
		}
		```

		b.在需要权限的页面写<div id="revisepersonal" v-if="this.$route.query.auth === 'MANc41'">或者页面上小按钮权限<router-link v-if="detailAuthority.KOCb19 === 'KOCb19'" class="headBtn" :to="{name:'pointsshopmall',query:{auth:this.detailAuthority.KOCb19}}">积分商城后台&nbsp;></router-link>
		detailAuthority:{
			KOCb15:'',
			KOCb16:'',
			KOCb17:'',
			KOCb18:'',
			KOCb19:'',
			KOCb20:''
		}

		this.$router.push({
			name: "addscore",
			path: "/addscore",
			// params: this.scoreinfo,
			query:{auth:this.detailAuthority.KOCb16,id:id}
		});

#webpack-bundle-analyzer 打包后分析
* npm run build --report


