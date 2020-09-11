# 基于JavaEE(JSP)的共享资料平台的设计与实现

@[toc]

## 1.前言

本套系统是利用JSP实现的一个共享资料平台，大体上的功能有用户进行功能上传，然后上传资源的图片描述和文字描述，以及所需要的积分点。管理员对用户上传的资源进行审核，未审核的资源不会在页面展示，只有当管理员审核通过才会被展示。系统中用到了伪静态网站技术、ajax技术以及文件上传和下载。如果在文件上传和下载中遇到问题的，可以查看[《Tomcat+Chrome访问本地文件》](https://blog.csdn.net/qq_42376617/article/details/108291330)。

## 2.准备工作

**开发环境**：MySQL8.0，JDK1.8，Tomcat9.0

**开发工具**：eclipse2020，Navicat15，HBuilder X，vscode。

**开发语言**：java、html、css、javascript

**第三方工具库**：hutool、fastjson、jstl、bootstrap、jQuery、font-awesome

## 3.技术难点

在本套系统，登录注册使用到了ajax技术，在登录页，采用ajax技术判断用户的用户名和密码以及验证码是否正确，并及时以消息提示框进行反馈。如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184356919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


在注册界面，使用ajax异步校验技术，当用户输入用户名的时候，会及时向servlet发送请求，查看数据库中该用户名是否存在，这样设计有利于用户的体验感。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184433611.jpg#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184433607.jpg#pic_center)


除此之外，系统中大部分功能都采用ajax技术实现，极个别功能采用传统MVC实现。所以遇到的难点问题大多集中在在ajax请求数据和处理数据上面，因为ajax的数据请求方式是异步的，所以在外面无法获得数据，在前期不熟练的时候，经常烦恼在俺不获取不到数据，而导致一系列的困难。经过后来的多次尝试与犯错，并查阅相关文献，掌握了ajax的回调函数机制，通过该机制，可以干很多事情。

## 4.系统设计

### 4.1数据库设计

数据库设计主要包含用户表、角色表、共享产品表、共享产品的文件表以及其他的一些边沿配置。在四张主要的表里面，在角色表里面设定用户注册的角色，管理员字段为admin，且默认存在，不允许被注册。之后设计了用户的角色，用户的角色分为VIP和非VIP，由于时间的原因，目前只用到了非VIP的设计，也就是说，每个登录用户都可以实现资源的下载。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184527117.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020083018452757.jpg#pic_center)



这是角色表和用户表之间的关系，用户表从属于角色表，角色规定了每个用户的身份，他在系统中所能接触到的功能。

在文件表中，利用java的UUID给文件生成一个唯一的主键id，这样所以的文件都有自己的编号，并且采用UUID，给文件重命名，这样就会解决上传文件重命名的冲突问题。并且设计一个字段保存文件的原名称，下载的时候就显示文件名，而不是UUID。在产品表中，设计两个字段，分别为uploadfile_id和user_imgs来存放用户上传关系资料的图片详情和具体的文件资源。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184552690.jpg#pic_center)


最后就是共享产品表，设计字段中，最后一个字段也就是file_status表示是否审核，只有通过审核的资料才会显示，负责就一直处于待审核状态，不会展示在前端页面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184607194.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


### 4.2用户系统设计

未登陆的用户可以直接浏览网站首页和共享文件中心，查看相关的共享资源，但不可以下载该资料。只有登陆后才可以实现资料的下载，前期设计用户有VIP和非VIP之分，但由于时间原因，未能做出此功能。

用户通过注册，除了不可以注册admin这个用户名外，其他所有的用户名都可以进行注册，但需要满足一定的规则才会允许注册。注册完成后就可以进行登录。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184631591.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184631558.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


登录完成后，进入到个人中心，可以查看自己发布的资源，以及资源的审核状态，还可以对资源进行修改和删除。在资源添加界面中，可以上传资源的预览图片，方便用户在前端页面查看到更加详细的信息。采用富文本技术，用户可以在富文本里面进行对资源的详细描述，并且可以选择资源所属的类别。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184650310.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


添加完资源的一些描述后，返回个人中心的关系资源管理界面，可以查看到刚刚添加的资源，然后点击操作按钮，选择上传文件，然后选择与此相关的资源，点击上传，整个流程就上传成功了，之后就是默默等待管理员对资源的审核了。如果一直没有审核，就一直处于未发布状态，可能管理员没有收到审核请求，这是可以尝试重新发布一次。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184756592.jpg#pic_center)


在如上界面中，点击删除按钮，就库对该资料进行删除。

### 4.3管理员系统设计

默认设置管理员为admin，且唯一，不能有多个管理员存在。管理员登录后就进入到管理员的后台管理界面，因为时间原因，后台设计的相对丑陋一些，但实现了基本的功能。

管理员在此处可以看到用户发来的审核请求，用户的发布状态默认为未审核，所以审核选择按钮为未选中状态。通过对资料的简单查看，如果符合要求，点击按钮，就可以审核成功。按钮审核的功能也是采用ajax实现的，当按钮事件改变之后，就会想servlet发送请求，更改当前资源的审核状态。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184716574.jpg#pic_center)

### 4.4下载资源

审核成功后，就可以在前端页面查看到资源的预览。默认选择的栏目为软件书籍栏目，如果上传的是其他栏目信息，手动进行点击切换即可。点击某一个资源，进入详情界面，里面会展示资源的详情，包括图片和描述，以及所需要的积分数。如果以后觉得满意，就可以点击下载，然后就可以下载该资源。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184837705.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200830184836990.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


### 4.5查询

在本次系统中，所采用的查询方式都为模糊查询，即输入的条件在资源中存在，就会显示。查询使用在共享中心、网点资源以及管理员后台管理页面中，当资源特别多的时候，就可以进行查询，快速定位到某个资源。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020083018485258.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzc2NjE3,size_16,color_FFFFFF,t_70#pic_center)


## 5.部分源码解析

### 5.1AJAX异步校验技术

#### 5.1.1 ajax技术应用在注册界面

在上面以及讲了注册界面用到ajax去异步校验用户输入的账号是否已经存在，也把效果图贴在了上面，这里就具体讲讲如何进行实现这个功能的。

在jsp页面中，有一个表单form。

```html
<form name="regForm" action="reg.let" method="post"
                data-bv-message="验证不通过"
                data-bv-feedbackicons-valid="fa fa-check"
                data-bv-feedbackicons-invalid="fa fa-times"
                data-bv-feedbackicons-validating="fa fa-refresh">
    <!-- 这里是用户名的输入框，其他的就不一一列举了，其他还用到了密码的校验 -->
    <div class="form-group row">
        <label class="col-sm-3 col-form-label text-right"><b class="text-danger">*</b>用户名</label>
        <div class="col-sm-9">
            <input name="userName" type="text" placeholder="由3到20个单词字符组成"
                   class="form-control" value="" data-bv-trigger="keyup" required
                   data-bv-notempty-message="用户名不能为空"
                   pattern="^[a-zA-Z]\w{2,19}$"
                   data-bv-regexp-message="用户名必须由字母开始的3-20个单词字符组成"
                   data-bv-different data-bv-different-field="userPassword"
                   data-bv-different-message="用户名不能和密码一样">
        </div>
    </div>
</form>
```

新建一个js文件，然后在上面的jsp文件中引入当前这个jsp即可

`````js
var regForm = $(document.forms.regForm);
//使用到了bootstrap的表单
regForm.bootstrapValidator({
  fields: {
      //对userName这个输入框进行校验
    userName: {
      validators: {
        callback: {
          message: '不允许注册管理员账户',
          callback: function(value, validator) {
            return !value.startsWith("admin");
          }
        },
          //这一步是在用户放开键盘的那一刻就发送post请求，检查用户输入的用户名是否存在
        remote: {
          type: 'POST',
          url: 'check-name.let',
          message: '用户名已经被注册了',
          delay: 1000
        }
      }
    }
  }
}).on("success.form.bv", function(e) {
  //验证成功后使用AJAX提交
  $.post(
    base+"reg.let",
    sys.form.param(regForm[0]).toString(),
    function(data){
	  sys.loading.hide();
      if(data.code==200){
        sys.toastr.success("注册成功了");
        form.bootstrapValidator('disableSubmitButtons', false) 
            .bootstrapValidator('resetForm', true); 
      }else{
        sys.toastr.error(data.message);
      }
    },"json"
  );

}).on("error.form.bv",function(){
  //校验失败
  sys.toastr.warn("校验失败，请按要求填写！");
});
`````

#### 5.1.2 ajax技术应用在登录界面

登录界面的话，就会涉及到验证码是否错误，是否已经过期，账号密码是否正确等等。

```html
<form name="loginForm" autocomplete="off" class="pt-3 pl-3 pr-3" action="login.let" method="post">
    <div class="form-group row">
        <div class="col-sm-12">
            <div class="input-group input-group-lg">
                <div class="input-group-prepend">
                    <span class="input-group-text"><i class="fa fa-user fa-fw"></i></span>
                </div>
                <input name="userName" value="admin" type="text" required="required" class="form-control" placeholder="用户名|手机号码|邮箱">
            </div>
        </div>
    </div>

    <div class="form-group row">
        <div class="col-sm-12">
            <div class="input-group input-group-lg">
                <div class="input-group-prepend">
                    <span class="input-group-text"><i class="fa fa-key fa-fw"></i></span>
                </div>
                <input value="123" type="password" name="userPwd" required="required" class="form-control" placeholder="登录密码">
            </div>
        </div>
    </div>

    <div class="form-group row">
        <div class="col-sm-7">
            <div class="input-group input-group-lg">
                <div class="input-group-prepend">
                    <span class="input-group-text"><i class="fa fa-image fa-fw"></i></span>
                </div>
                <input value="0000" name="code" type="text" required="required" class="form-control" placeholder="验证码">
            </div>
        </div>
        <div class="col-sm-5">
            <img alt="" src="code.let?w=139&h=48&len=4&v=<%=new java.util.Date().getTime() %>" onclick="this.src='code.let?w=139&h=48&len=4&t='+new Date().getTime();" style="width:100%;height:48px;cursor: pointer;" title="点击更换验证码">
        </div>
    </div>
</form>
```

```js
var loginForm = document.forms.loginForm;

loginForm.onsubmit = ()=>{
	console.info(loginForm.action);
	$.post(
		loginForm.action,
		sys.form.param(loginForm).toString(),
		function(data){
			console.info(data);
			if(data.code==200){
				sys.toastr.success("登录成功");
				window.location=data.data.roleCode+"/";//跳转到不同的角色目录中
			}else{
				sys.toastr.error(data.message);
			}
		},"json"
	);
	return false;
}
```

### 5.2验证码servlet实现

这是登录界面的随机验证码，利用servlet生成图片，发送到前端页面即可。

```java
@WebServlet("/code.let")
public class CodeSerlvet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		Random r = new Random();
		String code = ""+r.nextInt(10)+r.nextInt(10)+r.nextInt(10)+r.nextInt(10);//随机要绘制的文字
//		System.out.println(code);
		request.getSession().setAttribute(Const.CODE_NAME, code);
		
		int width = 139;
		int height = 48;
		
			
		BufferedImage img = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
		Graphics g = img.getGraphics();
		
		//填充白色背景
		g.setColor(new Color(255, 255, 255));
		g.fillRect(0, 0, width, height);
		
		
		//画干扰线(多条)
		for(int i=0;i<height/2;i++){
			g.setColor(new Color(150+r.nextInt(100),150+r.nextInt(100),150+r.nextInt(100)));//随机颜色 150-250
			g.drawLine(0, r.nextInt(height), width, r.nextInt(height));
		}
		
		//绘制文字
		int fontSize=36;
		g.setFont(new Font("微软雅黑", Font.BOLD, fontSize));
		int w = width/code.length();
		for(int i=0;i<code.length();i++){
			g.setColor(new Color(50+r.nextInt(150),50+r.nextInt(150),50+r.nextInt(150)));//随机颜色 50-200
			g.drawString(code.substring(i,i+1), i*w, height-r.nextInt(height-fontSize));
		}
		
		response.reset();
		response.setHeader("Pragma","No-cache"); 
		response.setHeader("Cache-Control","no-cache"); 
		response.setDateHeader("Expires", 0);
		response.setContentType("image/jpeg");
		ImageIO.write(img, "jpg", response.getOutputStream());
		response.flushBuffer();
	}
}
```

### 5.3上传和下载

在这个系统中，核心功能就是上传和下载，写两个servlet类来实现文件的上传和文件的下载。有关文件上传和下载遇到的问题可以查看[《Tomcat+Chrome访问本地文件》](https://blog.csdn.net/qq_42376617/article/details/108291330)。

上传功能：UploadSerlvet.java和WebFileServiceForFile.java

```java
@WebServlet("/upload.let")
@MultipartConfig(maxFileSize=-1,maxRequestSize=-1)
public class UploadSerlvet extends HttpServletSupport {
	
	private WebFileService service = new WebFileServiceForFile();

	public Object execute(HttpServletRequest request) {
		Map<String, Object> result = new HashMap<>();
		try{
            //从前端获取到文件的part
			Part file = request.getPart("file");
            //上传这个文件的时候，生成一个ID，然后取得这个id
			Long id = service.add(file).getFileId();
			result.put("ok", true);
			result.put("id", id);
		}catch(Exception ex){
			result.put("ok", false);
			result.put("msg", ex.getMessage());
		}
		return result;
	}
}

//实现文件的上传功能，这里上传的地方是D盘下面的upload文件夹下面
public class WebFileServiceForFile implements WebFileService {
	private WebFileDAO webFileDAO = new WebFileDAO();
	@Override
	public WebFile add(Part file) throws CodeException {
		String uri = "";
		Long id = DBUtil.nextId();
		uri = "D:/upload/"+new SimpleDateFormat("yyyy/MM/dd/").format(new Date());
		new File(uri).mkdirs();
		uri += Long.toString(id, 36)+"."+FileUtil.extName(file.getSubmittedFileName());
		try {
			IoUtil.copy(file.getInputStream(), new FileOutputStream(new File(uri)));
		} catch (Exception e) {
			throw new CodeException(500, "上传的文件保存失败");
		}
    }
}
```

下载功能：DownServlet.java

```java
@SuppressWarnings("serial")
@WebServlet("/down.let")
public class DownServlet extends HttpServlet {
	
	private WebFileService service = new WebFileServiceForFile();
	private Logger log = LoggerFactory.getLogger(this.getClass());

	public void doGet(HttpServletRequest request,HttpServletResponse response) throws IOException {
		Long id = 0L;
		try{
			id = Long.valueOf(request.getParameter("id"));
		}catch(Exception ex){}
		WebFile webFile = null;
		try{
			response.reset();
			webFile = service.get(id);
			String fileName = webFile.getFileName();
			fileName = new String(fileName.getBytes(),"ISO8859-1");
			response.setHeader("Content-Disposition", "attachment;filename=" + fileName + "");
			request.setAttribute("filePath", webFile.getFileUri().substring(2));
			request.setAttribute("fileName", webFile.getFileName());
			response.setContentType(webFile.getFileContentType());
			IoUtil.copy(service.get(webFile), response.getOutputStream());
		}catch(Exception ex){
			log.debug("错误",ex);
			response.sendError(404);
			return;
		}
	}
}
```



## 6.总结

在本次项目中用到了一些前端的框架，之前用传统MVC方法的时候，根本不知道这些框架，并且书写的代码也比较冗余，很多重复的，代码量还特别大，用了ajax后，代码比较少。但理解起来可能比较困难，理解的话就感觉套路都一样，也比较简单。

如有侵权或不足，请在评论处留言。

好了，这次的项目分享到此也差不多了。觉得小弟写的不错的话，可以点赞加关注一下，谢谢！有需要源码的小伙伴也可以搜索**企鹅号863772270**，编码不易，请作者喝瓶水即可。

