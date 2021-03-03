很多同学说AgileConfig的UI实在是太丑了。我想想也是的，本来这个项目是我自己使用的，一开始甚至连UI都没有，全靠手动在数据库里修改数据。后来加上了UI也是使用了老掉牙的bootstrap3做为基础样式。前台框架也是使用了angularjs，同样是老掉牙的东西。过年期间终于下决心翻新AgileConfig的前端UI。最后选择的前端UI框架为AntDesign Pro + React。至于为啥选Ant-Design Pro是因为他好看，而且流行，选择React是因为VUE跟Angular我都略知一二，干脆趁此机会学一学React为何物，为何这么流行。   
登录的认证方案为JWT，其实本人对JWT不太感冒（请看这里《[我们真的需要jwt吗？](https://www.cnblogs.com/kklldog/p/should-we-need-jwt-always.html)》），无奈大家都喜欢，那我也只能随大流。   
其实基于ant-design pro的界面我已经翻的差不多了，因为它支持mock数据，所以我一行后台代码都没修改，已经把界面快些完了。从现在开始要真正的跟后端代码进行联调了。那么我们先从登录开始吧。先看看后端asp.net core方面会如何进行修改。
## 修改ASP.NET Core后端代码
```
  "JwtSetting": {
    "SecurityKey": "xxxxxxxxxxxx", // 密钥
    "Issuer": "agileconfig.admin", // 颁发者
    "Audience": "agileconfig.admin", // 接收者
    "ExpireSeconds": 20 // 过期时间 s
  }
```
在appsettings.json文件添加jwt相关配置。
```
  public class JwtSetting
    {
        static JwtSetting()
        {
            Instance = new JwtSetting();
            Instance.Audience = Global.Config["JwtSetting:Audience"];
            Instance.SecurityKey = Global.Config["JwtSetting:SecurityKey"];
            Instance.Issuer = Global.Config["JwtSetting:Issuer"];
            Instance.ExpireSeconds = int.Parse(Global.Config["JwtSetting:ExpireSeconds"]);
        }

        public string SecurityKey { get; set; }

        public string Issuer { get; set; }

        public string Audience { get; set; }

        public int ExpireSeconds { get; set; }

        public static JwtSetting Instance
        {
            get;
        }
    }
```
定义一个JwtSetting类，用来读取配置。
```
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMemoryCache();
            services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
                      .AddJwtBearer(options =>
                      {
                          options.TokenValidationParameters = new TokenValidationParameters
                          {
                              ValidIssuer = JwtSetting.Instance.Issuer,
                              ValidAudience = JwtSetting.Instance.Audience,
                              IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(JwtSetting.Instance.SecurityKey)),
                          };
                      });
            services.AddCors();
            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_3_0).AddRazorRuntimeCompilation();
            services.AddFreeSqlDbContext();
            services.AddBusinessServices();
            services.AddAntiforgery(o => o.SuppressXFrameOptionsHeader = true);
        }
```
修改Startup文件的ConfigureServices方法，修改认证Scheme为JwtBearerDefaults.AuthenticationScheme，在AddJwtBearer方法内配置jwt相关配置信息。因为前后端分离项目所以有可能api跟ui部署在不同的域名下，所以开启Core。
```
     // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IServiceProvider serviceProvider)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseMiddleware<ExceptionHandlerMiddleware>();
            }
            app.UseCors(op=> {
                op.AllowAnyOrigin();
                op.AllowAnyMethod();
                op.AllowAnyHeader();
            });
            app.UseWebSockets(new WebSocketOptions()
            {
                KeepAliveInterval = TimeSpan.FromSeconds(60),
                ReceiveBufferSize = 2 * 1024
            });
            app.UseMiddleware<WebsocketHandlerMiddleware>();
            app.UseStaticFiles();
            app.UseRouting();
            app.UseAuthentication();
            app.UseAuthorization();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapDefaultControllerRoute();
            });
        }
```
修改Startup的Configure方法，配置Cors为Any。
```
    public class JWT
    {
        public static string GetToken()
        {
            //创建用户身份标识，可按需要添加更多信息
            var claims = new Claim[]
            {
    new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
    new Claim("id", "admin", ClaimValueTypes.String), // 用户id
    new Claim("name", "admin"), // 用户名
    new Claim("admin", true.ToString() ,ClaimValueTypes.Boolean) // 是否是管理员
            };
            var key = Encoding.UTF8.GetBytes(JwtSetting.Instance.SecurityKey);
            //创建令牌
            var token = new JwtSecurityToken(
              issuer: JwtSetting.Instance.Issuer,
              audience: JwtSetting.Instance.Audience,
              signingCredentials: new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature),
              claims: claims,
              notBefore: DateTime.Now,
              expires: DateTime.Now.AddSeconds(JwtSetting.Instance.ExpireSeconds)
            );

            string jwtToken = new JwtSecurityTokenHandler().WriteToken(token);

            return jwtToken;
        }
    }
```
添加一个JWT静态类用来生成jwt的token。因为agileconfig的用户只有admin一个所以这里用户名，ID都直接写死。
```
 [HttpPost("admin/jwt/login")]
        public async Task<IActionResult> Login4AntdPro([FromBody] LoginVM model)
        {
            string password = model.password;
            if (string.IsNullOrEmpty(password))
            {
                return Json(new
                {
                    status = "error",
                    message = "密码不能为空"
                });
            }

            var result = await _settingService.ValidateAdminPassword(password);
            if (result)
            {

                var jwt = JWT.GetToken();

                return Json(new { 
                    status="ok",
                    token=jwt,
                    type= "Bearer",
                    currentAuthority = "admin"
                });
            }

            return Json(new
            {
                status = "error",
                message = "密码错误"
            });
        }
```
新增一个Action方法做为登录的入口。在这里验证完密码后生成token，并且返回到前端。   
到这里.net core这边后端代码改动的差不多了。主要是添加jwt相关的东西，这些内容网上已经写了很多了，不在赘述。   
下面开始修改前端代码。
## 修改AntDesign Pro的代码
AntDesign Pro已经为我们生成好了登录页面，登录的逻辑等，但是原来的登录是假的，也不支持jwt token做为登录凭证，下面我们要修改多个文件来完善这个登录。
```
export function setToken(token:string): void {
  localStorage.setItem('token', token);
}

export function getToken(): string {
  var tk = localStorage.getItem('token');
  if (tk) {
    return tk as string;
  }

  return '';
}

```
在utils/authority.ts文件内新增2个方法，用来存储跟获取token。我们的jwt token存储在localStorage里。
```

/** 配置request请求时的默认参数 */
const request = extend({
  prefix: 'http://localhost:5000',
  errorHandler, // 默认错误处理
  credentials: 'same-origin', // 默认请求是否带上cookie,
  headers: {
    Authorization: 'Bearer '+getToken(),
  },
});
```
修改utils/request.ts文件，在extend方法内添加jwt认证的头部Authorization为我们的token。    
设置prefix为http://localhost:5000这是我们的后端api的服务地址，真正生产的时候会替换为正式地址。   
设置credentials为same-origin。
```
export async function accountLogin(params: LoginParamsType) {
  return request('/admin/jwt/login', {
    method: 'POST',
    data: params,
  });
}

```
在services/login.ts文件内新增发起登录请求的方法。
```
 effects: {
    *login({ payload }, { call, put }) {
      const response = yield call(accountLogin, payload);
      yield put({
        type: 'changeLoginStatus',
        payload: response,
      });
      // Login successfully
      if (response.status === 'ok') {
        const urlParams = new URL(window.location.href);
        const params = getPageQuery();
        message.success('🎉 🎉 🎉  登录成功！');
        let { redirect } = params as { redirect: string };
        if (redirect) {
          console.log('redirect url ' , redirect);
          const redirectUrlParams = new URL(redirect);
          if (redirectUrlParams.origin === urlParams.origin) {
            redirect = redirect.substr(urlParams.origin.length);
            if (redirect.match(/^\/.*#/)) {
              redirect = redirect.substr(redirect.indexOf('#') + 1);
            }
          } else {
            window.location.href = '/';
            return;
          }
        }
        history.replace(redirect || '/');
      }
    },


     reducers: {
    changeLoginStatus(state, { payload }) {
      setAuthority(payload.currentAuthority);
      setToken(payload.token)
      return {
        ...state,
        status: payload.status,
        type: payload.type,
      };
    },
  },
```
修改models/login.ts文件，修改effects的login方法，在内部替换原来的fakeAccountLogin为accountLogin。同时修改reducers内部的changeLoginStatus方法，添加setToken的代码，这有修改后登录成功后token就会被存储起来。
```

  effects: {
    *fetch(_, { call, put }) {
      const response = yield call(queryUsers);
      yield put({
        type: 'save',
        payload: response,
      });
    },
    *fetchCurrent(_, { call, put }) {
      const response = {
        name: '管理员',
        userid: 'admin'
      };
      yield put({
        type: 'saveCurrentUser',
        payload: response,
      });
    },
  },
```
修改models/user.ts文件，修改effects的fetchCurrent方法为直接返回response。本来fetchCurrent是会去后台拉当前用户信息的，因为agileconfig的用户就admin一个，所以我直接写死了。
![](https://ftp.bmp.ovh/imgs/2021/03/e2cc17be36ece546.gif)
让我们试一下登录吧：）   
源码在这：[https://github.com/kklldog/AgileConfig/tree/react_ui](https://github.com/kklldog/AgileConfig/tree/react_ui) 🌟🌟🌟

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)