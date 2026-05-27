# 12-后端Web实战(登录认证)

> ⛳ 如果大家在自学的过程中，时间紧迫、没有方向、经常遇到难以解决的 Bug、没有亮眼的项目、缺少 AI 大模型经验，而又想快速系统化的学习、高薪就业的小伙伴儿，大家都可以 🔗 直接点我，了解一下我们系统化的课程体系。

---

在前面的课程中，我们已经实现了部门管理、员工管理的基本功能，但是大家会发现，我们并没有登录，就直接访问到了 Tlias 智能学习辅助系统的后台。这是不安全的，所以我们今天的主题就是 **登录认证**。

> ✅ **最终要实现的效果：**
> - 如果用户名密码错误，**不允许登录系统**。
> - 如果用户名和密码都正确，则登录成功，可以访问系统。

![登录失败示例 - 用户名密码错误](../JavaWebImages/auth-login-failed.png)

---

# 1. 登录功能

## 1.1 需求

![登录页面原型](../JavaWebImages/auth-login-page.png)

在登录界面中，我们可以输入用户的用户名以及密码，然后点击 "登录" 按钮就要请求服务器，服务端判断用户输入的用户名或者密码是否正确。如果正确，则返回成功结果，前端跳转至系统首页面。

## 1.2 接口描述

我们参照接口文档中的 **其他接口 -> 登录接口**

## 1.3 思路分析

- **怎么样才算登录成功了呢？**
  - 用户名和密码都输入正确，登录成功
  - 否则，登录失败
- **登录功能的本质是什么？**
  - **查询**
  - 根据用户名和密码查询员工信息

## 1.4 功能开发

> 💡 **AI 提示词：**
>
> 你是一名 java 开发工程师，现需要基于 SpringBoot+Mybatis 实现员工登录的基本功能，开发一个基本的登录接口，基本信息如下：
>
> 1. 接口请求路径 `/login`，请求方式 post
> 2. 接口请求参数有：用户名 `username`，密码 `password`，为 json 格式的数据 `{"username":"admin", "password":"123456"}`
> 3. 接口响应数据：json 格式，具体的数据格式如下：
>    ```json
>    {
>      "code": 1,
>      "msg": "success",
>      "data": {
>        "id": 1,
>        "username": "songjiang",
>        "name": "宋江",
>        "token": "..."
>      }
>    }
>    ```
> 4. 数据库表为 `emp`, 对应的实体类为 `Emp`，已存在，对应的表结构为：
>    ```sql
>    create table emp (
>      id int unsigned primary key auto_increment comment 'ID,主键',
>      username varchar(20) not null comment '用户名',
>      password varchar(32) default '123456' not null comment '密码',
>      name varchar(10) not null comment '姓名',
>      gender tinyint unsigned not null comment '性别, 1:男, 2:女',
>      phone char(11) not null comment '手机号',
>      job tinyint unsigned null comment '职位, 1:班主任,2:讲师,3:学工主管,4:教研主管,5:咨询师',
>      salary int unsigned null comment '薪资',
>      image varchar(300) null comment '头像',
>      entry_date date null comment '入职日期',
>      dept_id int unsigned null comment '关联的部门ID',
>      create_time datetime null comment '创建时间',
>      update_time datetime null comment '修改时间',
>      constraint emp_pk unique (phone),
>      constraint username unique (username)
>    ) comment '员工表';
>    ```

**1）准备实体类 `LoginInfo`，封装登录成功后，返回给前端的数据。**

```java
/**
 * 登录成功结果封装类
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class LoginInfo {
    private Integer id;        //员工ID
    private String username;   //用户名
    private String name;       //姓名
    private String token;      //令牌
}
```

**2）定义 `LoginController`**

```java
@Slf4j
@RestController
public class LoginController {

    @Autowired
    private EmpService empService;

    @PostMapping("/login")
    public Result login(@RequestBody Emp emp){
        log.info("员工来登录啦 , {}", emp);
        LoginInfo loginInfo = empService.login(emp);
        if(loginInfo != null){
            return Result.success(loginInfo);
        }
        return Result.error("用户名或密码错误~");
    }
}
```

**3）`EmpService` 接口中增加 login 登录方法**

```java
/**
 * 登录
 */
LoginInfo login(Emp emp);
```

**4）`EmpServiceImpl` 实现 login 方法**

```java
@Override
public LoginInfo login(Emp emp) {
    Emp empLogin = empMapper.getUsernameAndPassword(emp);
    if(empLogin != null){
        LoginInfo loginInfo = new LoginInfo(empLogin.getId(), empLogin.getUsername(), empLogin.getName(), null);
        return loginInfo;
    }
    return null;
}
```

**5）`EmpMapper` 增加接口方法**

```java
/**
 * 根据用户名和密码查询员工信息
 */
@Select("select * from emp where username = #{username} and password = #{password}")
Emp getUsernameAndPassword(Emp emp);
```

## 1.5 测试

功能开发完毕后，我们就可以启动服务，打开 Apifox 进行测试了。发起 POST 请求，访问：`http://localhost:8080/login`

![Apifox 测试登录接口](../JavaWebImages/auth-cookie-c1-response.png)

Apifox 测试通过了，那接下来，我们就可以结合着前端工程进行联调测试。先退出系统，进入到登录页面。在登录页面输入账户密码：

![登录页面输入账户密码](../JavaWebImages/auth-cookie-app-storage.png)

登录成功之后进入到后台管理系统页面：

![Tlias 后台管理系统首页](../JavaWebImages/auth-tlias-backend-page.png)

我们已经完成了基础登录功能的开发与测试，在我们登录成功后就可以进入到后台管理系统中进行数据的操作。

但是当我们在浏览器中新的页面上输入地址：`http://localhost:90`，发现没有登录仍然可以进入到后端管理系统页面。

> 📌 而真正的登录功能应该是：**登陆后才能访问后端系统页面，不登陆则跳转登陆页面进行登陆**。

为什么会出现这个问题？其实原因很简单，就是因为针对于我们当前所开发的部门管理、员工管理以及文件上传等相关接口来说，我们在服务器端并没有做任何的判断，没有去判断用户是否登录了。所以无论用户是否登录，都可以访问部门管理以及员工管理的相关数据。

所以我们目前所开发的登录功能，它只是徒有其表。而我们要想解决这个问题，我们就需要完成一步非常重要的操作：**登录校验**。

---

# 2. 登录校验

**什么是登录校验？**

> ✅ 所谓登录校验，指的是我们在服务器端接收到浏览器发送过来的请求之后，首先我们要对请求进行校验。先要校验一下用户登录了没有，如果用户已经登录了，就直接执行对应的业务操作就可以了；如果用户没有登录，此时就不允许他执行相关的业务操作，直接给前端响应一个错误的结果，最终跳转到登录页面，要求他登录成功之后，再来访问对应的数据。

## 2.1 思路

了解完什么是登录校验之后，接下来我们分析一下登录校验大概的实现思路。

前面在讲解 HTTP 协议的时候，我们提到 HTTP 协议是 **无状态协议**。

> 📌 **所谓无状态**：指的是每一次请求都是独立的，下一次请求并不会携带上一次请求的数据。

而浏览器与服务器之间进行交互，基于 HTTP 协议也就意味着现在我们通过浏览器来访问了登陆这个接口，实现了登陆的操作，接下来我们在执行其他业务操作时，服务器也并不知道这个员工到底登陆了没有。

**那应该怎么来实现登录校验的操作呢？具体的实现思路可以分为两部分：**

1. 在员工登录成功后，**需要将用户登录成功的信息存起来**，记录用户已经登录成功的标记。
2. 在浏览器发起请求时，**需要在服务端进行统一拦截**，拦截后进行登录校验。

> ✅ **核心思路：**
>
> 想要判断员工是否已经登录，我们需要在员工登录成功之后，存储一个登录成功的标记，接下来在每一个接口方法执行之前，先做一个条件判断，判断一下这个员工到底登录了没有。如果是登录了，就可以执行正常的业务操作，如果没有登录，会直接给前端返回一个错误的信息，前端拿到这个错误信息之后会自动的跳转到登录页面。
>
> 我们程序中所开发的查询功能、删除功能、添加功能、修改功能，都需要使用以上套路进行登录校验。此时就会出现：相同代码逻辑，每个功能都需要编写，就会造成代码非常繁琐。
>
> 为了简化这块操作，我们可以使用一种技术：**统一拦截技术**。

我们要完成以上操作，会涉及到 web 开发中的两个技术：

1. **会话技术**：用户登录成功之后，在后续的每一次请求中，都可以获取到该标记。
2. **统一拦截技术**：过滤器 Filter、拦截器 Interceptor

下面我们先学习会话技术，然后再学习统一拦截技术。

## 2.2 会话技术

### 2.2.1 介绍

**什么是会话？**

- 在我们日常生活当中，会话指的就是谈话、交谈。
- 在 web 开发当中，**会话指的就是浏览器与服务器之间的一次连接**，我们就称为一次会话。

> ✅ 在用户打开浏览器第一次访问服务器的时候，这个会话就建立了，直到有任何一方断开连接，此时会话就结束了。**在一次会话当中，是可以包含多次请求和响应的**。

比如：打开了浏览器来访问 web 服务器上的资源（浏览器不能关闭、服务器不能断开）

- 第 1 次：访问的是登录的接口，完成登录操作
- 第 2 次：访问的是部门管理接口，查询所有部门数据
- 第 3 次：访问的是员工管理接口，查询员工数据

只要浏览器和服务器都没有关闭，以上 3 次请求都属于一次会话当中完成的。

> 📌 需要注意的是：会话是和浏览器关联的，**当有三个浏览器客户端和服务器建立了连接时，就会有三个会话**。同一个浏览器在未关闭之前请求了多次服务器，这多次请求是属于同一个会话。

知道了会话的概念了，接下来我们再来了解下 **会话跟踪**。

> ✅ **会话跟踪**：一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据。

**为什么要共享数据呢？**

由于 HTTP 是无状态协议，在后面请求中怎么拿到前一次请求生成的数据呢？此时就需要在一次会话的多次请求之间进行数据共享。

> 📌 **会话跟踪技术有三种：**
>
> 1. **Cookie**（客户端会话跟踪技术）：数据存储在客户端浏览器当中
> 2. **Session**（服务端会话跟踪技术）：数据存储在服务端
> 3. **令牌技术**

### 2.2.2 会话跟踪方案

上面我们介绍了什么是会话，什么是会话跟踪，并且也提到了会话跟踪 3 种常见的技术方案。接下来，我们就来对比一下这 3 种会话跟踪的技术方案。

#### 2.2.2.1 方案一：Cookie

**Cookie** 是客户端会话跟踪技术，它是存储在客户端浏览器的，我们使用 cookie 来跟踪会话，我们就可以在浏览器第一次发起请求来请求服务器的时候，我们在服务器端来设置一个 cookie。

![Cookie 流程示意 - 服务端 Set-Cookie](../JavaWebImages/auth-jwt-illustration.png)

服务器端在给客户端在响应数据的时候，会自动的将 cookie 响应给浏览器，浏览器接收到响应回来的 cookie 之后，会自动的将 cookie 的值存储在浏览器本地。接下来在后续的每一次请求当中，都会将浏览器本地所存储的 cookie 自动地携带到服务端。

我刚才在介绍流程的时候，用了 3 个 **自动**：

- 服务器会 **自动** 的将 cookie 响应给浏览器。
- 浏览器接收到响应回来的数据之后，会 **自动** 的将 cookie 存储在浏览器本地。
- 在后续的请求当中，浏览器会 **自动** 的将 cookie 携带到服务器端。

**为什么这一切都是自动化进行的？**

是因为 cookie 它是 HTTP 协议当中所支持的技术，而各大浏览器厂商都支持了这一标准。在 HTTP 协议官方给我们提供了一个响应头和请求头：

- **响应头 `Set-Cookie`**：设置 Cookie 数据的
- **请求头 `Cookie`**：携带 Cookie 数据的

**代码测试：**

```java
@Slf4j
@RestController
public class SessionController {

    // 设置 Cookie
    @GetMapping("/c1")
    public Result cookie1(HttpServletResponse response){
        response.addCookie(new Cookie("login_username","itheima")); // 设置 Cookie / 响应 Cookie
        return Result.success();
    }

    // 获取 Cookie
    @GetMapping("/c2")
    public Result cookie2(HttpServletRequest request){
        Cookie[] cookies = request.getCookies();
        for (Cookie cookie : cookies) {
            if(cookie.getName().equals("login_username")){
                System.out.println("login_username: "+cookie.getValue()); // 输出名为 login_username 的 cookie
            }
        }
        return Result.success();
    }
}
```

**A. 访问 c1 接口，设置 Cookie，`http://localhost:8080/c1`**

![响应头 Set-Cookie](../JavaWebImages/auth-cookie-c2-request.png)

我们可以看到，设置的 cookie，通过 **响应头 `Set-Cookie`** 响应给浏览器，并且浏览器会将 Cookie，存储在浏览器端。

**B. 访问 c2 接口 `http://localhost:8080/c2`，此时浏览器会自动的将 Cookie 携带到服务端，是通过 请求头 `Cookie`，携带的。**

![请求头 Cookie 携带](../JavaWebImages/auth-cookie-request-header.png)

**优缺点：**

- ✅ **优点**：HTTP 协议中支持的技术（像 Set-Cookie 响应头的解析以及 Cookie 请求头数据的携带，都是浏览器自动进行的，是无需我们手动操作的）
- ❌ **缺点：**
  - 移动端 APP（Android、IOS）中无法使用 Cookie
  - 不安全，用户可以自己禁用 Cookie
  - **Cookie 不能跨域**

**跨域介绍：**

![Cookie 跨域示意](../JavaWebImages/auth-cookie-cross-domain.png)

> 📌 **区分跨域的维度**（三个维度有任何一个维度不同，那就是跨域操作）：
> - **协议**
> - **IP / 域名**
> - **端口**

**举例：**

- `http://192.168.150.200/login.html` ----------> `https://192.168.150.200/login` 【协议不同，跨域】
- `http://192.168.150.200/login.html` ----------> `http://192.168.150.100/login` 【IP 不同，跨域】
- `http://192.168.150.200/login.html` ----------> `http://192.168.150.200:8080/login` 【端口不同，跨域】
- `http://192.168.150.200/login.html` ----------> `http://192.168.150.200/login` 【不跨域】

#### 2.2.2.2 方案二：Session

前面介绍的时候，我们提到 **Session**，它是服务器端会话跟踪技术，所以它是存储在服务器端的。而 Session 的底层其实就是基于我们刚才所介绍的 Cookie 来实现的。

- **获取 Session**

![获取 Session 流程](../JavaWebImages/auth-jwt-localstorage-view.png)

如果我们现在要基于 Session 来进行会话跟踪，浏览器在第一次请求服务器的时候，我们就可以直接在服务器当中来获取到会话对象 Session。如果是第一次请求 Session，会话对象是不存在的，这个时候服务器会自动的创建一个会话对象 Session。而每一个会话对象 Session，它都有一个 ID，我们称之为 **Session 的 ID**。

- **响应 Cookie（JSESSIONID）**

![响应 Set-Cookie: JSESSIONID](../JavaWebImages/auth-jwt-network-request-token.png)

接下来，服务器端在给浏览器响应数据的时候，它会将 Session 的 ID 通过 Cookie 响应给浏览器。其实在响应头当中增加了一个 `Set-Cookie` 响应头。这个 Set-Cookie 响应头对应的值就是 cookie，cookie 的名字是固定的 `JSESSIONID` 代表的服务器端会话对象 Session 的 ID。浏览器会自动识别这个响应头，然后自动将 Cookie 存储在浏览器本地。

- **查找 Session**

![后续请求携带 JSESSIONID Cookie](../JavaWebImages/auth-jwt-login-flow-diagram.png)

接下来，在后续的每一次请求当中，都会将 Cookie 的数据获取出来，并且携带到服务端。接下来服务器拿到 JSESSIONID 这个 Cookie 的值，也就是 Session 的 ID。拿到 ID 之后，就会从众多的 Session 当中来找到当前请求对应的会话对象 Session。

这样我们是不是就可以通过 Session 会话对象在同一次会话的多次请求之间来共享数据了？

**代码测试：**

```java
@Slf4j
@RestController
public class SessionController {

    @GetMapping("/s1")
    public Result session1(HttpSession session){
        log.info("HttpSession-s1: {}", session.hashCode());

        session.setAttribute("loginUser", "tom"); // 往 session 中存储数据
        return Result.success();
    }

    @GetMapping("/s2")
    public Result session2(HttpServletRequest request){
        HttpSession session = request.getSession();
        log.info("HttpSession-s2: {}", session.hashCode());

        Object loginUser = session.getAttribute("loginUser"); // 从 session 中获取数据
        log.info("loginUser: {}", loginUser);
        return Result.success(loginUser);
    }
}
```

**A. 访问 s1 接口，`http://localhost:8080/s1`**

请求完成之后，在响应头中，就会看到有一个 `Set-Cookie` 的响应头，里面响应回来了一个 Cookie，就是 `JSESSIONID`，这个就是服务端会话对象 Session 的 ID。

**B. 访问 s2 接口，`http://localhost:8080/s2`**

![两次 Session 请求 hashCode 相同](../JavaWebImages/auth-session-jsessionid-response.png)

接下来，在后续的每次请求时，都会将 Cookie 的值，携带到服务端，那服务端呢，接收到 Cookie 之后，会自动的根据 JSESSIONID 的值，找到对应的会话对象 Session。

两次请求，获取到的 Session 会话对象的 hashcode 是一样的，就说明是同一个会话对象。

**优缺点：**

- ✅ **优点**：Session 是存储在服务端的，安全
- ❌ **缺点：**
  - 服务器集群环境下无法直接使用 Session
  - 移动端 APP（Android、IOS）中无法使用 Cookie
  - 用户可以自己禁用 Cookie
  - Cookie 不能跨域

> 📌 **PS**：Session 底层是基于 Cookie 实现的会话跟踪，如果 Cookie 不可用，则该方案也就失效了。

**服务器集群环境为何无法使用 Session？**

![服务器集群环境下的 Session 问题](../JavaWebImages/auth-session-cluster-issue.png)

- 现在所开发的项目，最终部署的时候都是以 **集群** 的形式来进行部署，也就是同一个项目它会部署多份。
- 用户在访问的时候，会先访问一台前置的服务器，**负载均衡服务器**，它的作用就是将前端发起的请求均匀的分发给后面的服务器。

![集群负载均衡导致 Session 失效](../JavaWebImages/auth-session-load-balance.png)

假如同一个浏览器发起了 2 次请求，结果两次请求分别被分发到了不同的 Tomcat 服务器上，第二台服务器上没有这个 ID 对应的 Session，此时就出现了 **同一个浏览器发起了 2 次请求，结果获取到的不是同一个会话对象** 这种问题。

> ❌ **这就是 Session 这种会话跟踪方案它的缺点：在服务器集群环境下无法直接使用 Session。**

#### 2.2.2.3 方案三 - 令牌技术

这里我们所提到的令牌，其实它就是一个 **用户身份的标识**，看似很高大上，很神秘，其实本质就是一个字符串。

![令牌技术示意 - 登录下发令牌](../JavaWebImages/auth-token-jwt-flow.png)

如果通过令牌技术来跟踪会话，我们就可以在浏览器发起请求。在请求登录接口的时候，如果登录成功，我就可以生成一个令牌，令牌就是用户的合法身份凭证。接下来我在响应数据的时候，我就可以直接将令牌响应给前端。

接下来我们在前端程序当中接收到令牌之后，就需要将这个令牌存储起来。这个存储可以存储在 cookie 当中，也可以存储在其他的存储空间（比如：**localStorage**）当中。

![前端将令牌存储到 localStorage](../JavaWebImages/auth-token-storage-localstorage.png)

接下来，在后续的每一次请求当中，都需要将令牌携带到服务端。携带到服务端之后，接下来我们就需要来 **校验令牌的有效性**。如果令牌是有效的，就说明用户已经执行了登录操作，如果令牌是无效的，就说明用户之前并未执行登录操作。

![后续请求携带令牌进行校验](../JavaWebImages/auth-token-subsequent-request.png)

**优缺点：**

- ✅ **优点：**
  - 支持 PC 端、移动端
  - 解决集群环境下的认证问题
  - 减轻服务器的存储压力（无需在服务器端存储）
- ❌ **缺点**：需要自己实现（包括令牌的生成、令牌的传递、令牌的校验）

> ✅ 针对于这三种方案，现在企业开发当中使用的最多的就是 **第三种令牌技术** 进行会话跟踪。

![JWT 登录认证整体流程](../JavaWebImages/auth-jwt-login-flow-diagram.png)

**JWT 令牌最典型的应用场景就是登录认证：**

1. 在浏览器发起请求来执行登录操作，此时会访问登录的接口，如果登录成功之后，我们需要生成一个 jwt 令牌，将生成的 jwt 令牌返回给前端。
2. 前端拿到 jwt 令牌之后，会将 jwt 令牌存储起来。在后续的每一次请求中都会将 jwt 令牌携带到服务端。
3. 服务端统一拦截请求之后，先来判断一下这次请求有没有把令牌带过来，如果没有带过来，直接拒绝访问，如果带过来了，还要校验一下令牌是否是有效。如果有效，就直接放行进行请求的处理。

## 2.3 JWT 令牌

前面我们介绍了基于令牌技术来实现会话追踪。这里所提到的令牌就是用户身份的标识，其本质就是一个字符串。令牌的形式有很多，我们使用的是功能强大的 **JWT 令牌**。

### 2.3.1 介绍

> ✅ **JWT 全称 JSON Web Token**（官网：https://jwt.io/），定义了一种简洁的、自包含的格式，用于在通信双方以 json 数据格式安全的传输信息。**由于数字签名的存在，这些信息是可靠的**。

- **简洁**：是指 jwt 就是一个简单的字符串。可以在请求参数或者是请求头当中直接传递。
- **自包含**：指的是 jwt 令牌，看似是一个随机的字符串，但是我们是可以根据自身的需求在 jwt 令牌中存储自定义的数据内容。如：可以直接在 jwt 令牌中存储用户的相关信息。
- 简单来讲，jwt 就是将原始的 json 数据格式进行了安全的封装，这样就可以直接基于 jwt 在通信双方安全的进行信息传输了。

![JWT 令牌组成结构（Header.Payload.Signature）](../JavaWebImages/auth-jwt-structure-diagram.png)

> 📌 **JWT 的组成**（JWT 令牌由三个部分组成，三个部分之间使用英文的点来分割）：
>
> - **第一部分：Header（头）**，记录令牌类型、签名算法等。例如：`{"alg":"HS256","type":"JWT"}`
> - **第二部分：Payload（有效载荷）**，携带一些自定义信息、默认信息等。例如：`{"id":"1","username":"Tom"}`
> - **第三部分：Signature（签名）**，防止 Token 被篡改、确保安全性。将 header、payload，并加入指定秘钥，通过指定签名算法计算而来。

签名的目的就是为了防 jwt 令牌被篡改，而正是因为 jwt 令牌最后一个部分数字签名的存在，所以整个 jwt 令牌是非常安全可靠的。**一旦 jwt 令牌当中任何一个部分、任何一个字符被篡改了，整个令牌在校验的时候都会失败**，所以它是非常安全可靠的。

**JWT 是如何将原始的 JSON 格式数据，转变为字符串的呢？**

- 其实在生成 JWT 令牌时，会对 JSON 格式的数据进行一次编码：进行 **base64** 编码
- **Base64**：是一种基于 64 个可打印的字符来表示二进制数据的编码方式。所使用的 64 个字符分别是 A 到 Z、a 到 z、0-9，一个加号，一个斜杠，加起来就是 64 个字符。还有一个等号，它是一个补位的符号。
- 需要注意的是 **Base64 是编码方式，而不是加密方式**。

### 2.3.2 生成和校验

简单介绍了 JWT 令牌以及 JWT 令牌的组成之后，接下来我们就来学习基于 Java 代码如何生成和校验 JWT 令牌。

**1）首先我们先来实现 JWT 令牌的生成。要想使用 JWT 令牌，需要先引入 JWT 的依赖：**

```xml
<!-- JWT 依赖 -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

在引入完 JWT 依赖后，就可以调用工具包中提供的 API 来完成 JWT 令牌的生成和校验。**工具类：`Jwts`**

**2）生成 JWT 代码实现：**

```java
@Test
public void testGenJwt() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", 10);
    claims.put("username", "itheima");

    String jwt = Jwts.builder().signWith(SignatureAlgorithm.HS256, "aXRjYXN0")
            .addClaims(claims)
            .setExpiration(new Date(System.currentTimeMillis() + 12 * 3600 * 1000))
            .compact();

    System.out.println(jwt);
}
```

**运行测试方法：**

```
eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiZXhwIjoxNjcyNzI5NzMwfQ.fHi0Ub8npbyt71UqLXDdLyipptLgxBUg_mSuGJtXtBk
```

输出的结果就是生成的 JWT 令牌，通过英文的点分割对三个部分进行分割，我们可以将生成的令牌复制一下，然后打开 JWT 的官网，将生成的令牌直接放在 Encoded 位置，此时就会自动的将令牌解析出来。

![JWT 官网 jwt.io 在线解析](../JavaWebImages/auth-jwt-website.png)

> 📌 **解析说明：**
> - 第一部分解析出来，看到 JSON 格式的原始数据，所使用的签名算法为 HS256。
> - 第二个部分是我们自定义的数据，之前我们自定义的数据就是 id，还有一个 exp 代表的是我们所设置的过期时间。
> - 由于前两个部分是 base64 编码，所以是可以直接解码出来。但最后一个部分并不是 base64 编码，是经过签名算法计算出来的，所以最后一个部分是不会解析的。

**3）实现了 JWT 令牌的生成，下面我们接着使用 Java 代码来校验 JWT 令牌（解析生成的令牌）：**

```java
@Test
public void testParseJwt() {
    Claims claims = Jwts.parser().setSigningKey("aXRjYXN0")
            .parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MTAsInVzZXJuYW1lIjoiaXRoZWltYSIsImV4cCI6MTcwMTkwOTAxNX0.N-MD6DmoeIIY5lB5z73UFLN9u7veppx1K5_N_jS9Yko")
            .getBody();

    System.out.println(claims);
}
```

**运行测试方法：**

```
{id=10, username=itheima, exp=1701909015}
```

![JWT 解析测试结果](../JavaWebImages/auth-jwt-generate-result.png)

令牌解析后，我们可以看到 id 和过期时间，如果在解析的过程当中没有报错，就说明解析成功了。

下面我们做一个测试：把令牌 header 中的数字 9 变为 8，运行测试方法后发现报错：

![篡改令牌后解析报错](../JavaWebImages/auth-jwt-parse-result.png)

> 📌 **结论**：篡改令牌中的任何一个字符，在对令牌进行解析时都会报错，所以 JWT 令牌是非常安全可靠的。

我们继续测试：修改生成令牌的时指定的过期时间，修改为 1 分钟。

```java
@Test
public void genJwt(){
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", 10);
    claims.put("username", "itheima");

    String jwt = Jwts.builder().signWith(SignatureAlgorithm.HS256, "aXRjYXN0")
            .addClaims(claims)
            .setExpiration(new Date(System.currentTimeMillis() + 60 * 1000)) // 有效期 60s
            .compact();
    System.out.println(jwt);
    // 输出结果：eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiZXhwIjoxNjczMDA5NzU0fQ.RcVIR65AkGiax-ID6FjW60eLFH3tPTKdoK7UtE4A1ro
}

@Test
public void parseJwt(){
    Claims claims = Jwts.parser()
            .setSigningKey("aXRjYXN0")  //指定签名密钥
            .parseClaimsJws("eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiZXhwIjoxNjczMDA5NzU0fQ.RcVIR65AkGiax-ID6FjW60eLFH3tPTKdoK7UtE4A1ro")
            .getBody();

    System.out.println(claims);
}
```

![JWT 过期后解析失败](../JavaWebImages/auth-jwt-expired-result.png)

等待 1 分钟之后运行测试方法发现也报错了，说明：**JWT 令牌过期后，令牌就失效了，解析的为非法令牌**。

> 💡 **通过以上测试，我们在使用 JWT 令牌时需要注意：**
> - JWT 校验时使用的签名秘钥，必须和生成 JWT 令牌时使用的秘钥是配套的。
> - 如果 JWT 令牌解析校验时报错，则说明 **JWT 令牌被篡改 或 失效了，令牌非法**。

### 2.3.3 登录时下发令牌

JWT 令牌的生成和校验的基本操作我们已经学习完了，接下来我们就需要在案例当中通过 JWT 令牌技术来跟踪会话。具体的思路我们前面已经分析过了，主要就是两步操作：

1. **生成令牌**：在登录成功之后来生成一个 JWT 令牌，并且把这个令牌直接返回给前端
2. **校验令牌**：拦截前端请求，从请求中获取到令牌，对令牌进行解析校验

那我们首先来完成：登录成功之后生成 JWT 令牌，并且把令牌返回给前端。

**实现步骤：**

**1）引入 JWT 工具类**：在项目工程下创建 `com.itheima.util` 包，并把提供 JWT 工具类复制到该包下

```java
package com.itheima.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;
import java.util.Map;

public class JwtUtils {

    private static String signKey = "SVRIRUlNQQ==";
    private static Long expire = 43200000L;

    /**
     * 生成 JWT 令牌
     * @return
     */
    public static String generateJwt(Map<String,Object> claims){
        String jwt = Jwts.builder()
                .addClaims(claims)
                .signWith(SignatureAlgorithm.HS256, signKey)
                .setExpiration(new Date(System.currentTimeMillis() + expire))
                .compact();
        return jwt;
    }

    /**
     * 解析 JWT 令牌
     * @param jwt JWT 令牌
     * @return JWT 第二部分负载 payload 中存储的内容
     */
    public static Claims parseJWT(String jwt){
        Claims claims = Jwts.parser()
                .setSigningKey(signKey)
                .parseClaimsJws(jwt)
                .getBody();
        return claims;
    }
}
```

**2）完善 `EmpServiceImpl` 中的 login 方法逻辑，登录成功，生成 JWT 令牌并返回**

```java
@Override
public LoginInfo login(Emp emp) {
    Emp empLogin = empMapper.getUsernameAndPassword(emp);
    if(empLogin != null){
        //1. 生成 JWT 令牌
        Map<String,Object> dataMap = new HashMap<>();
        dataMap.put("id", empLogin.getId());
        dataMap.put("username", empLogin.getUsername());

        String jwt = JwtUtils.generateJwt(dataMap);
        LoginInfo loginInfo = new LoginInfo(empLogin.getId(), empLogin.getUsername(), empLogin.getName(), jwt);
        return loginInfo;
    }
    return null;
}
```

**重启服务，打开 Apifox 测试登录接口：**

![Apifox 登录接口返回 token](../JavaWebImages/auth-jwt-login-apifox-test.png)

**打开浏览器完成前后端联调操作**：利用开发者工具，抓取一下网络请求

![前后端联调 - 网络请求中的 token](../JavaWebImages/auth-jwt-frontend-network-trace.png)

登录请求完成后，可以看到 JWT 令牌已经响应给了前端，此时前端就会将 JWT 令牌存储在浏览器本地。

**服务器响应的 JWT 令牌存储在本地浏览器哪里了呢？**

> 📌 在当前案例中，JWT 令牌存储在浏览器的本地存储空间 **localStorage** 中了。`localStorage` 是浏览器的本地存储，在移动端也是支持的。

![JWT 令牌存储在 localStorage 中](../JavaWebImages/auth-jwt-tlias-token-bar.png)

我们在发起一个查询部门数据的请求，此时我们可以看到在请求头中包含一个 token（JWT 令牌），后续的每一次请求当中，都会将这个令牌携带到服务端。

![后续请求头中携带 token](../JavaWebImages/auth-jwt-tlias-header.png)

![请求详情查看 token 头](../JavaWebImages/auth-jwt-dev-tools-request.png)

## 2.4 过滤器 Filter

刚才通过浏览器的开发者工具，我们可以看到在后续的请求当中，都会在请求头中携带 JWT 令牌到服务端，而服务端需要统一拦截所有的请求，从而判断是否携带的有合法的 JWT 令牌。

那怎么样来统一拦截到所有的请求校验令牌的有效性呢？这里我们会学习两种解决方案：

1. **Filter 过滤器**
2. **Interceptor 拦截器**

我们首先来学习过滤器 Filter。

### 2.4.1 Filter 快速入门

**什么是 Filter？**

- Filter 表示过滤器，是 JavaWeb 三大组件（**Servlet、Filter、Listener**）之一。
- 过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能
  - 使用了过滤器之后，要想访问 web 服务器上的资源，**必须先经过滤器**，过滤器处理完毕之后，才可以访问对应的资源。
- 过滤器一般完成一些通用的操作，比如：**登录校验、统一编码处理、敏感字符处理** 等。

![Filter 过滤器拦截示意](../JavaWebImages/auth-filter-flow-diagram.png)

下面我们通过 Filter 快速入门程序掌握过滤器的基本使用操作：

- **第 1 步**，定义过滤器：定义一个类，实现 `Filter` 接口，并重写其所有方法。
- **第 2 步**，配置过滤器：Filter 类上加 `@WebFilter` 注解，配置拦截资源的路径。引导类上加 `@ServletComponentScan` 开启 Servlet 组件支持。

**1）定义过滤器**

```java
public class DemoFilter implements Filter {

    // 初始化方法, web 服务器启动, 创建 Filter 实例时调用, 只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init ...");
    }

    // 拦截到请求时,调用该方法,可以调用多次
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        System.out.println("拦截到了请求...");
    }

    // 销毁方法, web 服务器关闭时调用, 只调用一次
    public void destroy() {
        System.out.println("destroy ... ");
    }
}
```

> ✅ **三个方法说明：**
> - **`init` 方法**：过滤器的初始化方法。在 web 服务器启动的时候会自动的创建 Filter 过滤器对象，在创建过滤器对象的时候会自动调用 init 初始化方法，**这个方法只会被调用一次**。
> - **`doFilter` 方法**：这个方法是在每一次拦截到请求之后都会被调用，所以这个方法是会被调用多次的，**每拦截到一次请求就会调用一次 doFilter() 方法**。
> - **`destroy` 方法**：是销毁的方法。当我们关闭服务器的时候，它会自动的调用销毁方法 destroy，而这个销毁方法也只会被调用一次。

**2）配置过滤器**

在定义完 Filter 之后，Filter 其实并不会生效，还需要完成 Filter 的配置，Filter 的配置非常简单，只需要在 Filter 类上添加一个注解：`@WebFilter`，并指定属性 `urlPatterns`，通过这个属性指定过滤器要拦截哪些请求。

```java
@WebFilter(urlPatterns = "/*") //配置过滤器要拦截的请求路径（ /* 表示拦截浏览器的所有请求 ）
public class DemoFilter implements Filter {
    // 初始化方法, web 服务器启动, 创建 Filter 实例时调用, 只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init ...");
    }

    // 拦截到请求时,调用该方法,可以调用多次
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        System.out.println("拦截到了请求...");
    }

    // 销毁方法, web 服务器关闭时调用, 只调用一次
    public void destroy() {
        System.out.println("destroy ... ");
    }
}
```

当我们在 Filter 类上面加了 `@WebFilter` 注解之后，接下来我们还需要在启动类上面加上一个注解 `@ServletComponentScan`，通过这个 `@ServletComponentScan` 注解来开启 SpringBoot 项目对于 Servlet 组件的支持。

```java
@ServletComponentScan // 开启对 Servlet 组件的支持
@SpringBootApplication
public class TliasManagementApplication {
    public static void main(String[] args) {
        SpringApplication.run(TliasManagementApplication.class, args);
    }
}
```

重新启动服务，打开浏览器，执行部门管理的请求，可以看到控制台输出了过滤器中的内容：

![过滤器拦截请求控制台输出](../JavaWebImages/auth-filter-console-output.png)

> 📌 **注意事项**：在过滤器 Filter 中，**如果不执行放行操作，将无法访问后面的资源**。放行操作：`chain.doFilter(request, response);`

### 2.4.2 登录校验过滤器

#### 2.4.2.1 分析

过滤器 Filter 的快速入门以及使用细节我们已经介绍完了，接下来最后一步，我们需要使用过滤器 Filter 来完成案例当中的登录校验功能。

![登录校验过滤器流程](../JavaWebImages/auth-filter-login-check-flow.png)

大概清楚了在 Filter 过滤器的实现步骤了，那在正式开发登录校验过滤器之前，我们思考两个问题：

1. **所有的请求，拦截到了之后，都需要校验令牌吗？**
   - ✅ 答案：**登录请求例外**
2. **拦截到请求后，什么情况下才可以放行，执行业务操作？**
   - ✅ 答案：有令牌，且令牌校验通过（合法）；否则都返回未登录错误结果

#### 2.4.2.2 具体流程

我们要完成登录校验，主要是利用 Filter 过滤器实现，而 Filter 过滤器的流程步骤：

![Filter 过滤器具体流程图](../JavaWebImages/auth-filter-illustration-flow.png)

基于上面的业务流程，我们分析出具体的操作步骤：

1. 获取请求 url
2. 判断请求 url 中是否包含 login，如果包含，说明是登录操作，**放行**
3. 获取请求头中的令牌（token）
4. 判断令牌是否存在，如果不存在，**响应 401**
5. 解析 token，如果解析失败，**响应 401**
6. **放行**

#### 2.4.2.3 代码实现

在 `com.itheima.filter` 包下创建 `TokenFilter`，具体代码如下：

```java
package com.itheima.filter;

import com.itheima.utils.JwtUtils;
import jakarta.servlet.*;
import jakarta.servlet.annotation.WebFilter;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpStatus;
import org.springframework.util.StringUtils;
import java.io.IOException;

/**
 * 令牌校验过滤器
 */
@Slf4j
@WebFilter(urlPatterns = "/*")
public class TokenFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        //1. 获取请求 url。
        String url = request.getRequestURL().toString();

        //2. 判断请求 url 中是否包含 login，如果包含，说明是登录操作，放行。
        if(url.contains("login")){ //登录请求
            log.info("登录请求 , 直接放行");
            chain.doFilter(request, response);
            return;
        }

        //3. 获取请求头中的令牌（token）。
        String jwt = request.getHeader("token");

        //4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
        if(!StringUtils.hasLength(jwt)){ //jwt 为空
            log.info("获取到 jwt 令牌为空, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED);
            return;
        }

        //5. 解析 token，如果解析失败，返回错误结果（未登录）。
        try {
            JwtUtils.parseJWT(jwt);
        } catch (Exception e) {
            e.printStackTrace();
            log.info("解析令牌失败, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED);
            return;
        }

        //6. 放行。
        log.info("令牌合法, 放行");
        chain.doFilter(request , response);
    }
}
```

登录校验的过滤器我们编写完成了，接下来我们就可以重新启动服务来做一个测试：

- **测试 1**：未登录是否可以访问部门管理页面

首先关闭浏览器，重新打开浏览器，在地址栏中输入：`http://localhost:90`

由于用户没有登录，登录校验过滤器返回错误信息，前端页面根据返回的错误信息结果，自动跳转到登录页面了。

![未登录被过滤器拦截跳转登录页](../JavaWebImages/auth-filter-login-redirect-page.png)

- **测试 2**：先进行登录操作，再访问部门管理页面

登录校验成功之后，可以正常访问相关业务操作页面

![登录后正常访问后台](../JavaWebImages/auth-filter-login-success-page.png)

### 2.4.3 Filter 详解

Filter 过滤器的快速入门程序我们已经完成了，接下来我们就要详细的介绍一下过滤器 Filter 在使用中的一些细节。主要介绍以下 3 个方面的细节：

1. **过滤器的执行流程**
2. **过滤器的拦截路径配置**
3. **过滤器链**

#### 2.4.3.1 执行流程

首先我们先来看下过滤器的执行流程：

![Filter 执行流程示意](../JavaWebImages/auth-filter-execution-flow.png)

> 📌 过滤器当中我们拦截到了请求之后，如果希望继续访问后面的 web 资源，就要执行放行操作，放行就是调用 `FilterChain` 对象当中的 `doFilter()` 方法。
> - 在调用 `doFilter()` 这个方法之前所编写的代码属于 **放行之前的逻辑**。
> - 在放行后访问完 web 资源之后还会回到过滤器当中，回到过滤器之后如有需求还可以执行 **放行之后的逻辑**，放行之后的逻辑我们写在 `doFilter()` 这行代码之后。

**测试代码：**

```java
@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {

    @Override //初始化方法, 只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init 初始化方法执行了");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        System.out.println("DemoFilter   放行前逻辑.....");

        //放行请求
        filterChain.doFilter(servletRequest,servletResponse);

        System.out.println("DemoFilter   放行后逻辑.....");
    }

    @Override //销毁方法, 只调用一次
    public void destroy() {
        System.out.println("destroy 销毁方法执行了");
    }
}
```

![Filter 放行前后逻辑测试](../JavaWebImages/auth-filter-test-console.png)

#### 2.4.3.2 拦截路径

执行流程我们搞清楚之后，接下来再来介绍一下过滤器的拦截路径，Filter 可以根据需求，配置不同的拦截资源路径：

![Filter 拦截路径配置](../JavaWebImages/auth-filter-tlias-after-login.png)

下面我们来测试 "拦截具体路径"：

```java
//拦截 /login 具体路径
@WebFilter(urlPatterns = "/login")
public class DemoFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("DemoFilter   放行前逻辑.....");

        //放行请求
        filterChain.doFilter(servletRequest,servletResponse);

        System.out.println("DemoFilter   放行后逻辑.....");
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        Filter.super.init(filterConfig);
    }

    @Override
    public void destroy() {
        Filter.super.destroy();
    }
}
```

#### 2.4.3.3 过滤器链

最后我们再来介绍下 **过滤器链**，什么是过滤器链呢？所谓过滤器链指的是 **在一个 web 应用程序当中，可以配置多个过滤器，多个过滤器就形成了一个过滤器链**。

比如：在我们 web 服务器当中，定义了两个过滤器，这两个过滤器就形成了一个过滤器链。

而这个链上的过滤器在执行的时候会一个一个的执行，会先执行第一个 Filter，放行之后再来执行第二个 Filter，如果执行到了最后一个过滤器放行之后，才会访问对应的 web 资源。

访问完 web 资源之后，按照我们刚才所介绍的过滤器的执行流程，还会回到过滤器当中来执行过滤器放行后的逻辑，而在执行放行后的逻辑的时候，**顺序是反着的**。

先要执行过滤器 2 放行之后的逻辑，再来执行过滤器 1 放行之后的逻辑，最后再给浏览器响应数据。

> ✅ **过滤器链上过滤器的执行顺序**：注解配置的 Filter，优先级是按照 **过滤器类名（字符串）的自然排序**。比如：
> - `AbcFilter`
> - `DemoFilter`
>
> 这两个过滤器来说，`AbcFilter` 会先执行，`DemoFilter` 会后执行。

## 2.5 拦截器 Interceptor

### 2.5.1 快速入门

**什么是拦截器？**

- 是一种 **动态拦截方法调用的机制**，类似于过滤器。
- 拦截器是 **Spring 框架** 中提供的，用来动态拦截控制器方法的执行。
- 拦截器的作用：拦截请求，在指定方法调用前后，根据业务需要执行预先设定的代码。

在拦截器当中，我们通常也是做一些通用性的操作，比如：我们可以通过拦截器来拦截前端发起的请求，将登录校验的逻辑全部编写在拦截器当中。在校验的过程当中，如发现用户登录了（携带 JWT 令牌且是合法令牌），就可以直接放行，去访问 spring 当中的资源。如果校验时发现并没有登录或是非法令牌，就可以直接给前端响应未登录的错误信息。

下面我们通过快速入门程序，来学习下拦截器的基本使用。拦截器的使用步骤和过滤器类似，也分为两步：

1. **定义拦截器**
2. **注册配置拦截器**

**1）自定义拦截器**

实现 `HandlerInterceptor` 接口，并重写其所有方法

```java
//自定义拦截器
@Component
public class DemoInterceptor implements HandlerInterceptor {

    //目标资源方法执行前执行。 返回 true：放行   返回 false：不放行
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle .... ");

        return true; //true 表示放行
    }

    //目标资源方法执行后执行
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle ... ");
    }

    //视图渲染完毕后执行，最后执行
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion .... ");
    }
}
```

> 📌 **注意：**
> - **`preHandle` 方法**：目标资源方法执行前执行。返回 `true`：放行；返回 `false`：不放行
> - **`postHandle` 方法**：目标资源方法执行后执行
> - **`afterCompletion` 方法**：视图渲染完毕后执行，最后执行

**2）注册配置拦截器**

在 `com.itheima` 下创建一个包，然后创建一个配置类 `WebConfig`，实现 `WebMvcConfigurer` 接口，并重写 `addInterceptors` 方法

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    //自定义的拦截器对象
    @Autowired
    private DemoInterceptor demoInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册自定义拦截器对象
        registry.addInterceptor(demoInterceptor).addPathPatterns("/**"); //设置拦截器拦截的请求路径（ /** 表示拦截所有请求）
    }
}
```

重新启动 SpringBoot 服务，打开 Apifox 测试：

![Apifox 测试拦截器拦截](../JavaWebImages/auth-interceptor-apifox-test.png)

可以看到控制台输出的日志：

![拦截器三个方法控制台输出](../JavaWebImages/auth-interceptor-console-output.png)

接下来我们再来做一个测试：**将拦截器中返回值改为 `false`**

使用 Apifox，再次点击 send 发送请求后，没有响应数据，说明请求被拦截了没有放行

![拦截器返回 false 后请求被拦截](../JavaWebImages/auth-interceptor-return-false-test.png)

### 2.5.2 令牌校验 Interceptor

讲解完了拦截器的基本操作之后，接下来我们需要完成最后一步操作：**通过拦截器来完成案例当中的登录校验功能**。

登录校验的业务逻辑以及操作步骤我们前面已经分析过了，和登录校验 Filter 过滤器当中的逻辑是完全一致的。现在我们只需要把这个技术方案由原来的过滤器换成拦截器 interceptor 就可以了。

**1）`TokenInterceptor`**

在 `com.itheima.interceptor` 包下创建 `TokenInterceptor`

```java
@Slf4j
@Component
public class TokenInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //1. 获取请求 url。
        String url = request.getRequestURL().toString();

        //2. 判断请求 url 中是否包含 login，如果包含，说明是登录操作，放行。
        if(url.contains("login")){ //登录请求
            log.info("登录请求 , 直接放行");
            return true;
        }

        //3. 获取请求头中的令牌（token）。
        String jwt = request.getHeader("token");

        //4. 判断令牌是否存在，如果不存在，返回错误结果（未登录）。
        if(!StringUtils.hasLength(jwt)){ //jwt 为空
            log.info("获取到 jwt 令牌为空, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED);
            return false;
        }

        //5. 解析 token，如果解析失败，返回错误结果（未登录）。
        try {
            JwtUtils.parseJWT(jwt);
        } catch (Exception e) {
            e.printStackTrace();
            log.info("解析令牌失败, 返回错误结果");
            response.setStatus(HttpStatus.SC_UNAUTHORIZED);
            return false;
        }

        //6. 放行。
        log.info("令牌合法, 放行");
        return true;
    }
}
```

**2）配置拦截器**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    //拦截器对象
    @Autowired
    private TokenInterceptor tokenInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册自定义拦截器对象
        registry.addInterceptor(tokenInterceptor).addPathPatterns("/**");
    }
}
```

登录校验的拦截器编写完成后，接下来我们就可以重新启动服务来做一个测试：（**关闭登录校验 Filter 过滤器**）

- **测试 1**：未登录是否可以访问部门管理页面

首先关闭浏览器，重新打开浏览器，在地址栏中输入：`http://localhost:90`

由于用户没有登录，校验机制返回错误信息，前端页面根据返回的错误信息结果，自动跳转到登录页面了

![拦截器拦截未登录请求跳转登录页](../JavaWebImages/auth-interceptor-test-result.png)

- **测试 2**：先进行登录操作，再访问部门管理页面

登录校验成功之后，可以正常访问相关业务操作页面

![登录后拦截器放行进入系统](../JavaWebImages/auth-interceptor-success-tlias.png)

到此我们也就验证了所开发的登录校验的拦截器也是没问题的。**登录校验的过滤器和拦截器，我们只需要使用其中的一种就可以了**。

### 2.5.3 Interceptor 详解

拦截器的入门程序完成之后，接下来我们来介绍拦截器的使用细节。拦截器的使用细节我们主要介绍两个部分：

1. **拦截器的拦截路径配置**
2. **拦截器的执行流程**

### 2.5.4 拦截路径

首先我们先来看拦截器的拦截路径的配置，在注册配置拦截器的时候，我们要指定拦截器的拦截路径，通过 `addPathPatterns("要拦截路径")` 方法，就可以指定要拦截哪些资源。

在入门程序中我们配置的是 `/**`，表示拦截所有资源，而在配置拦截器时，不仅可以指定要拦截哪些资源，还可以指定不拦截哪些资源，只需要调用 `excludePathPatterns("不拦截路径")` 方法，指定哪些资源不需要拦截。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    //拦截器对象
    @Autowired
    private DemoInterceptor demoInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册自定义拦截器对象
        registry.addInterceptor(demoInterceptor)
                .addPathPatterns("/**")            //设置拦截器拦截的请求路径（ /** 表示拦截所有请求）
                .excludePathPatterns("/login");    //设置不拦截的请求路径
    }
}
```

![拦截器最终测试 - Apifox 验证](../JavaWebImages/auth-interceptor-final-test.png)

在拦截器中除了可以设置 `/**` 拦截所有资源外，还有一些常见拦截路径设置：

| 拦截路径 | 含义 | 举例 |
| --- | --- | --- |
| `/*` | 一级路径 | 能匹配 `/depts`、`/emps`、`/login`，不能匹配 `/depts/1` |
| `/**` | 任意级路径 | 能匹配 `/depts`、`/depts/1`、`/depts/1/2` |
| `/depts/*` | `/depts` 下的一级路径 | 能匹配 `/depts/1`，不能匹配 `/depts`、`/depts/1/2` |
| `/depts/**` | `/depts` 下的任意级路径 | 能匹配 `/depts`、`/depts/1`、`/depts/1/2`，不能匹配 `/emps/1` |

![拦截路径配置一览表](../JavaWebImages/auth-interceptor-paths-table.png)

![拦截器拦截路径详解](../JavaWebImages/auth-interceptor-paths-detail-table.png)

### 2.5.5 执行流程

介绍完拦截路径的配置之后，接下来我们再来介绍拦截器的执行流程。通过执行流程，大家就能够清晰的知道过滤器与拦截器的执行时机。

![拦截器与过滤器执行流程对比](../JavaWebImages/auth-interceptor-execution-flow.png)

- 当我们打开浏览器来访问部署在 web 服务器当中的 web 应用时，此时我们所定义的过滤器会拦截到这次请求。拦截到这次请求之后，它会先执行放行前的逻辑，然后再执行放行操作。而由于我们当前是基于 springboot 开发的，所以放行之后是进入到了 spring 的环境当中，也就是要来访问我们所定义的 controller 当中的接口方法。
- Tomcat 并不识别所编写的 Controller 程序，但是它识别 Servlet 程序，所以在 Spring 的 Web 环境中提供了一个非常核心的 Servlet：**`DispatcherServlet`（前端控制器）**，所有请求都会先进行到 DispatcherServlet，再将请求转给 Controller。
- 当我们定义了拦截器后，会在执行 Controller 的方法之前，请求被拦截器拦截住。执行 **`preHandle()` 方法**，这个方法执行完成后需要返回一个布尔类型的值，如果返回 `true`，就表示放行本次操作，才会继续访问 controller 中的方法；如果返回 `false`，则不会放行（controller 中的方法也不会执行）。
- 在 controller 当中的方法执行完毕之后，再回过来执行 **`postHandle()`** 这个方法以及 **`afterCompletion()`** 方法，然后再返回给 DispatcherServlet，最终再来执行过滤器当中放行后的这一部分逻辑的逻辑。执行完毕之后，最终给浏览器响应数据。

![拦截器整体执行流程图示](../JavaWebImages/auth-interceptor-overall-flow.png)

以上就是拦截器的执行流程。通过执行流程分析，大家应该已经清楚了过滤器和拦截器之间的区别，其实它们之间的区别主要是两点：

> 📌 **过滤器 Filter vs 拦截器 Interceptor**
>
> - **接口规范不同**：过滤器需要实现 `Filter` 接口，而拦截器需要实现 `HandlerInterceptor` 接口。
> - **拦截范围不同**：过滤器 Filter 会拦截 **所有的资源**，而 Interceptor 只会拦截 **Spring 环境中的资源**。
