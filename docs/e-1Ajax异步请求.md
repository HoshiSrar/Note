## Ajax异步请求，前后端的数据交互

### Ajax

使用 axios 的ajax异步请求

~~~html
  <p>用户名: <span id="username"></p>
  <p>密码: <span id="password"><</p>
      
<script>
    //通过Get请求获取数据填充到页面
    function getInfo() {
        axios.get('/mvc/test').then(({data}) => {
            document.getElementById('username').innerText = data.username
            document.getElementById('password').innerText = data.password
        })
    }
    // 后端返回的是json格式
    return new User("username","password");
</script>  
~~~

~~~html
<script>
    //发送 username 和 password 数据
    function login() {
        //没有设置第三个包含了 headers 对象的参数时默认发送 json 格式的数据
        axios.post('/mvc/test', {
            username: 'test',
            password: '123456'
        }, {
            headers: {
                //头部报文设置请求为表单类型，
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        }).then(({data}) => {
            if(data.success) {
                alert('登录成功')
            } else {
                alert('登录失败')
            }
        })
    }
    // 后台接收方式
    public String hello(String username,String password){
        ......
    }
</script>
~~~

JSON 格式的数据可以自动被前端转换成对象，JavaScript 可以使用调用对象的方式来获取数据。

------

### 后台接收参数

> Spring默认接收表单类型的数据，可以不添加 @RequestBody 注解直接使用使用类接收参数，当发送的是 JSON 类型时需要使用实体类+ @RequestBody 注解来接收数据，Spring 会对比 JSON 中的 Key 值和类中字段自动映射成 java 对象。

#### 1.JSON格式

**发送数据**

~~~json
{
    "name": "杰哥",
 	"age": 18
}
~~~

**后台处理**

~~~ java
@PutMapping("/userInfo")
//参数在请求体中，用RequestBody标识，用实体类来接受
public ResponseResult updateUserInfo(@RequestBody User user)
~~~

#### 2.Restful风格

**发送数据**

~~~http
/xxx/updateViewCount/1
~~~

**后台处理**

~~~java
@PutMapping("/updateViewCount/{id}")//  /{}表示后面是占位符
public ResponseResult updateViewCount(@PathVariable("id") Long id){
//用@PathVariable获取id的值
~~~

#### 3. Query格式（参数在URL中）

**发送数据**

~~~http
/commentList?pageNum=1&pageSize=10&articleId=1
~~~

**后台处理**

~~~java
 @GetMapping("/commentList")
//也可以在参数前使用@RequestParam("pageNum")标记路径中的 pageNum 参数
public ResponseResult commentList(Long articleId,Integer pageNum,Integer pageSize){
}    
~~~

