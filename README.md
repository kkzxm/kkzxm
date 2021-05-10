@[TOC](个人开发经历--我的java学习之路（学校篇）)

# 个人介绍:
## 姓名:  不在这里说明
## 联系信息: 
平台: QQ/微信/小破站/gitHub
gitHub:    [https://github.com/kkzxm](https://github.com/kkzxm)
统一昵称: **酷酷宅小明**
头像:![在这里插入图片描述](https://img-blog.csdnimg.cn/20210504150507218.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_1,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_1,color_FFFFFF,t_70#pic_center)
>代码感悟:
 **<span style="color:red;">越少的代码,意味着越少的Bug,更加意味着越少的维护成本!</span>**
 论代码行数来判定工资.......我不敢去评判，更加不敢去想象这样的场景

>提前名词解释：
模块：在本文中，模块并不是指maven中的模块，而是，
一个entity类如：student实体类 + Student持久层类 + Student业务层 + Student控制层类的一个总称，
或许这样称呼，它是不对的，但用在这篇文章，刚好合适
 

 # 个人历程
 (排序方法并非依照  ~~技术包含~~  来排序，而是在学这技术的时候，顺手做了，或者是顺手学了) 
 
 ## jdbc阶段
 在学习jdbc时，老师也对整体的项目结构做了一些优化，具体如何优化一两句话真说不清楚，总之，是用到了jdbc的元数据，以及其它的jdbcUtils等工具。

<hr>

 ### sql生成器
 上面这些都不重要，但在这个过程中，我发现每个sql语句都基本上是固定的，查询前置部分都是
 select * from **表名** where ....
 在当时就让我萌生了想要做一个东西，来简化jdbc的开发（当时，根本就没有框架的概念），我把它叫做：***sql生成器**
 原理：StringBuffer字符串一个字符一个字符地拼接（当时也不知道有xml这个东西）
 于是，在接下来的两个月的时间里，我因为需要做这个东西，把当时还没学到的反射给学了：获取类名，来替代**表名**，对于结果集的映射，也完全由反射来完成，
 总结： 该方法对单表无敌，只要调个service层的方法，其它的完全自动，不用管，但....
 在做上面这个sql语句生成器的时候，发现其实每个模块的代码都是差不多的，于是顺手做了一个代码生成器：
 <hr>
 
### 一代代码生成器
原理：**StringBuffer字符串拼接**
1. 第一个版本：
(1)  数据来源：txt文件
(2) 文件内容：（因为源代码已经丢失，可能原本在之前是这么写的）

```json
类1备注  类名1{
	 属性1备注：属性1类型 属性1标识符；
	 属性2备注：属性2类型 属性2标识符;
	 .....等等...
}，
类2备注 类名2{
	属性1备注：属性1类型 属性1标识符；
	属性2备注：属性2类型 属性2标识符；
	.....等等...
}
```
（3）附加说明：
>类备注与属性备注都是在sql生成器里用到的，好像是当要查询某个属性的值，
>不用去输这个属性的标识符，而是添加在该属生上的注解，如：

```java
public class{
	....
	@Comment("学生姓名")
	private String studentName;
	....
}
```

>@Comment（"学生姓名"）
>（通过备注可以找到属性，通过属性也可以找到备注）

比如：
```java
studentService.selectByAttr("学生姓名","酷酷宅小明");
studentService.selectByAttr("studentName","酷酷宅小明");
//上面两句执行后，返回的结果是同一条数据
```

（4）总结：
txt能容纳的信息太少，太少，如果想要容纳更多的信息，则意味着更大的解析难度，而且，更容易报错（比如：txt文档里找空格）
再后来，学习了dom4j，第一次知道了有xml这个东西，当机立断，舍弃了txt这个解析方法，但也仅仅只是舍弃了txt解析方式，其它的还是没有任何改变（比如，它只能生成实体层代码，仅此而已）

上面sql生成器和代码生成器，源码已经不在了，但思路基本就是这样，

（5）回忆：记得当时做测试时，先创建好表，然后，再通过表结构，手写txt文档，或者xml文档，调用代码生成器入口，生成好实体层代码......
<hr>

**此时，还没到真正做项目的时候，下面才是高潮**

<hr>

 ## servlet阶段
 
新学期，我们换了老师，
servlet + jsp的学习，我是自学的，
我坐在教室的最后一桌，两个显示器，一边看着小破站的教学视频，一边敲着自己的东西，
大家都知道的，
servlet需要继承于HttpServlet这个类，并重写doGet或doPost其中一个方法，
且一个servlet类只能处理一个请求，
那如何让一个servlet处理多个请求呢？
答案是：不去直接继承于HttpServlet，而是写一个BaseServlet类，用它来继承HttpServelt，且每次请求时添加一个请求：active，接下来就是反射的一顿操作（active与方法名保持一致），成功解决问题！
但我并不会满足于此！

>我的BaseServlet（项目中写的，学的时候没有人会这么教）

```java
/**
 * @Author: 酷酷宅小明
 * <p>
 */
public abstract class BaseServlet extends HttpServlet {
	//通过这个属性active,实现一个servelt处理多个请求
    protected String active;
    
    protected HttpServletRequest req;
    protected HttpServletResponse resp;
    
    //自动配置的entity
    protected Object entity;
    
    //图片(文件上传时用到的)
    protected TuPian tuPian = new TuPian();
    //管理员()
    private Manager manager;

	//这个由子类去实现,通过这个方法,可以让父类知道entity是谁,等等信息
    abstract protected void setEntityAndSVC();
    
    //region  doGet与doPost
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        try {
            setInfo(req, resp);
            getActive();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        try {
            setInfo(req, resp);
            getActive();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    //endregion

    /**
     * 每次在doGet或doPost方法执行时,
     * 一定会第一时间来执行这里,
     * 内部方法,设置编码方式以及ObjectSvc等
     */
    private void setInfo(HttpServletRequest req, HttpServletResponse resp) throws Exception {
    	//给上面的req与resp赋值
        this.req = req;
        this.resp = resp;
        //设置字符编码,每次多写一个方法,都要去设置一次字符编码,怕不是 石乐志 转世
        setCharacter();
        //设置实体对象,以及service(给反射装配做准备)
        setEntityAndSVC();
        //设置全局管理员(当时项目搞复杂了,本来用不着这个的)
        setManager();
        //这里是考虑到文件上传时
        if (formIsMultipart()) {
        	//如果是文件上传的话,通过普通方法,无法装配entity对象
            fileInput();
        } else {
        	//如果不是文件上传的话,通过普通方法取到各种需要的值
            active = req.getParameter("active");
            //设置当前操作的实体对象entity
            Map parameterMap = req.getParameterMap();
            /*
            这个这前是通过BeanUtils的jar包来做的,
            但那jar包.参数多一个都不行，直接报错...然后就放弃了,
            自己试着模仿它的原理，做了一个
            */
            setObInfo(entity, parameterMap);
        }
    }

    /**
     * 设置全局管理员
     */
    private void setManager() {
        Object manager = req.getSession().getAttribute("manager");
        if (manager != null) {
            this.manager = (Manager) manager;
        }
    }

    /**
     * 设置字符编码
     */
    private void setCharacter() throws UnsupportedEncodingException {
        req.setCharacterEncoding("utf-8");
        resp.setContentType("text/html;charset=UTF-8");
    }

    /**
     * 查看表单是否是分段提交的
     */
    private boolean formIsMultipart() {
        return ServletFileUpload.isMultipartContent(req);
    }

    /**
     * 得到反射对象的active去调用对应的方法
     */
    private void getActive() {
        try {
            Method activeMethod = this.getClass().getDeclaredMethod(active);
            activeMethod.invoke(this);
            req = null;
            resp = null;
        } catch (Exception e) {
            System.out.println(active);
            e.printStackTrace();
        }
    }

    //region 处理文件上传
	/*
	这里，当时项目用的是绝对路径，所以，在我的电脑上没问题，
	但演示时，用的是别人的电脑，图片上传成功后，，，，图片报了404，
	也因为这个原因，比赛没能拿到名次
	*/
    /**
     * 文件上传的方法
     */
    private void fileInput() throws Exception {
        FileItemFactory fileItemFactory = new DiskFileItemFactory();
        ServletFileUpload servletFileUpload = new ServletFileUpload(fileItemFactory);
        List<FileItem> fileItems = servletFileUpload.parseRequest(req);
        Map keyVal = new HashMap();
        for (FileItem fileItem : fileItems) {
            String fieldName = fileItem.getFieldName();
            String val = "";
            if (fileItem.isFormField()) {
                //普通表单
                val = fileItem.getString("UTF-8");
                if (fieldName.equals("active")) active = val;
            } else {
                //文件上传
                //原文件名
                String fileName = fileItem.getName();
                //得到后缀名
                String fileSuffix = getFileSuffix(new File(fileName));
                //当前时间字符串
                SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMddHHmmssSS");
                String format = simpleDateFormat.format(new Date());
                //当前时间与后缀名拼接(得到新文件名)
                String newFileName = format + fileSuffix;
                //将新文件名赋给图片的name属性
                tuPian.setTuPianName(newFileName);
                //给图片加上管理员外键
                tuPian.setManager(manager);
                //将图片地址等信息写入数据库
                new TuPianServiceIMP().addTuPian(tuPian);
                //文件写入写入磁盘
                fileItem.write(new File("/img/" + newFileName));
                //回查最新的一张图片Id
                int lastTuPianId = lastTuPianId().getTuPianId();
                //将最后一张图片的id交给val
                val = String.valueOf(lastTuPianId);
            }
            keyVal.put(fieldName, val);
            setObInfo(entity, keyVal);
        }
    }
    //endregion

    //endregion
    /**
     * 查看数据库中最后一张图片
     */
    private TuPian lastTuPianId() {
        TuPianService tuPianService = new TuPianServiceIMP();
        TuPian lastTuPian = tuPianService.findLastTuPian(tuPian);
        return lastTuPian;
    }

    /**
     * 该方法在用到验证码的情况下调用
     * 注:验证码的输入框,name值为 verification<br/>
     * 相同返回true,不相同返回false;
     *
     * @return 查看输入的验证码值与输入的是否相同,
     */
    protected boolean getYzmFlag() {
        HttpSession session = req.getSession();
        String yzm_key = String.valueOf(req.getSession().getAttribute("KAPTCHA_SESSION_KEY"));
        session.removeAttribute("KAPTCHA_SESSION_KEY");
        String verification = req.getParameter("verification");
        boolean equals = yzm_key.equals(verification);
        if (!equals) req.setAttribute("verificationError", "信息错误!或!重复提交表单");
        else req.removeAttribute("verificationError");
        return equals;
    }

    //region 转发与重定向

    /**
     * 转发与重定向的总方法
     * 在该方法中最后一个值,代表着选择的方向:
     * 可选值:<br/>
     * 转发前端<br/>
     * 重定向前端<br/>
     * 转发后端<br/>
     * 重定向后端<br/>
     *
     * @param pageName 文件名/资源名
     * @param val      选择的方向
     */
    protected void zfOrCdx(String val, String pageName) {
        switch (val) {
            case "转发前端":
                transmitFrontPage(pageName);
                break;
            case "转发后端":
                transmitBackPage(pageName);
                break;
            case "重定向前端":
                redirectFrontPage(pageName);
                break;
            case "重定向后端":
                redirectBackPage(pageName);
                break;
            default:
                System.out.println(val + "输入错误");
                break;
        }
    }

    //region 转发与重定向->底

    /**
     * 转发的方法
     */
    protected void transmit(String pageName) {
        try {
            System.out.println(pageName);
            req.getRequestDispatcher(pageName).forward(req, resp);
        } catch (ServletException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 重定向
     */
    protected void redirect(String pageName) {
        try {
            resp.sendRedirect(pageName);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //endregion

    //region 前端页面转发与重定向

    /**
     * 转发前端页面
     */
    protected void transmitFrontPage(String pageName) {
        transmit("pages/store/" + pageName);
    }

    /**
     * 重定向前端页面
     */
    protected void redirectFrontPage(String pageName) {
        try {
            resp.sendRedirect(getFrontPage(pageName));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //endregion

    //region 后端页面转发与重定向

    /**
     * 转发后端页面
     */
    protected void transmitBackPage(String pageName) {
        transmit("pages/admin/" + pageName);
    }

    /**
     * 重定向后端页面
     */
    protected void redirectBackPage(String pageName) {
        try {
            resp.sendRedirect(getBackPage(pageName));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //endregion

    //endregion

    //region 得到地址
    protected String getPath() {
        return String.valueOf(this.getServletContext().getAttribute("base"));
    }

    /**
     * 得到前端地址
     */
    protected String getFrontPage(String pageName) {
        return String.valueOf(this.getServletContext().getAttribute("front")) + pageName;
    }

    /**
     * 得到后端地址
     */
    protected String getBackPage(String pageName) {
        return String.valueOf(this.getServletContext().getAttribute("back")) + pageName;
    }
    //endregion
}
```
>其中一个子类

```java
package com.sdllb.Servlet.IMP;
/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-01 09:45
 */
@WebServlet("/yiLiaoYonPinType")
public class YiLiaoYonPinTypeServletIMP extends BaseServlet implements YiLiaoYonPinTypeServlet {
    private YiLiaoYongPinTypeService yiLiaoYonPinTypeService;
    private YiLiaoYonPinType yiLiaoYonPinType;

    @Override
    protected void setEntityAndSVC() {
        entity = new YiLiaoYonPinType();
        yiLiaoYonPinType = (YiLiaoYonPinType) this.entity;
        yiLiaoYonPinTypeService = new YiLiaoYongPinTypeServiceIMP();
    }

	/**
	添加
	*/
    @Override
    public void addYiLiaoYonPinType() {
        if (yiLiaoYonPinTypeService.addYiLiaoYongPinType(yiLiaoYonPinType)) {
            req.setAttribute("addYiLiaoYongPinTypeResult", true);
        } else {
            req.setAttribute("addYiLiaoYongPinTypeResult", false);
        }
        zfOrCdx("转发后端", "addYiLiaoYonPinType.jsp");
    }

	/**
	分页查询
	*/
    @Override
    public void limitYiLiaoYonPinType() {
        int thisPage = Integer.parseInt(req.getParameter("pageNo"));
        YiLiaoYonPinType_Page yiLiaoYonPinType_page = new YiLiaoYonPinType_Page(thisPage, 5);
        req.setAttribute("yiLiaoYonPinType_page", yiLiaoYonPinType_page);
        zfOrCdx("转发后端", "yiLiaoYonPinType_list.jsp");
    }

	/**
	删除
	*/
    @Override
    public void deleteYiLiaoYonPinType() {
        if (yiLiaoYonPinTypeService.removeYiLiaoYonPinType(yiLiaoYonPinType)) {
            req.setAttribute("deleteYiLiaoYonPinType", "OK");
        } else {
            req.setAttribute("deleteYiLiaoYonPinType", "NO");
        }
        zfOrCdx("转发后端", "yiLiaoYonPinType_list.jsp");
    }

	/**
	修改
	*/
    @Override
    public void updateYiLiaoYonPinType() {
        int id = Integer.parseInt(req.getParameter("Id"));
        if (yiLiaoYonPinTypeService.updateYiLiaoYongPinType(id, yiLiaoYonPinType)) {
            req.getSession().setAttribute("addYiLiaoYongPinTypeResult", true);
        } else {
            req.setAttribute("addYiLiaoYongPinTypeResult", false);
        }
        zfOrCdx("转发后端", "addYiLiaoYonPinType.jsp");
    }
	
	/**
	修改前根据Id查询
	*/
    @Override
    public void selectYiLiaoYonPinTypeById() {
        YiLiaoYonPinType yiLiaoYonPinTypeById = yiLiaoYonPinTypeService.findYiLiaoYonPinTypeById(yiLiaoYonPinType);
        req.setAttribute("yiLiaoYonPinType", yiLiaoYonPinTypeById);
        zfOrCdx("转发后端", "addYiLiaoYonPinType.jsp");
    }
}
```

> 你还别说: zfOrCdx("转发后端", "addYiLiaoYonPinType.jsp");
> 这个方法用着是真的很爽，爽到爆的那种
> （当然，也给我自己埋下一个隐患，我直到现在都不知道转发和重定向是怎么写的）
### servlet项目中，sql生成器崩溃
> 直到现在，项目结构在控制层没出任何问题，但问题并不是出现在这里
> **原本以为我的sql生成器，在此刻一定能大展身手，好好表现一现一下**
> 结果，它就像是**张四贵的马**一样，刚拉出来就给我直接人仰马翻，555555
> 之前说过一下，它单表简直是无敌的，

这次做的项目是一个医疗系统（分前台展示页面和后台管理员页面，而我的BaseServlet也是根据，这个项目设计出来的）

> 因为之前自己做的sql生成器，它只能操作单表，面对多表时，它几乎无能为力；
> 在当时，有另一种解决方案：一个查询分两次
> 如：
> 两个实体类，一个student与school，student里引用school
> 通过第一次查询出student，里面有school的Id，
> 再通过school的Id再去查询一次，并装入student对象里面
> 但我是拒绝的，虽然不是正式的项目，可我并不喜欢这种操作，
> 这样操作的话，student它的类结构就会变成这样：

**怎么看都觉得非常别扭**
```java
public class Studnet{
	private int id;
	.....//其它属性
	private int schoolId;
	private School school;
}
```
> 所以我决定让我的sql生成器升级成可以操作多表的工具。
> 但我最终失败了，彻底失败了！！！
> 在测试sql生成器时，出现了内存泄露。就像递归一样地程序陷入死循环，
> 在那一个晚自习，我想尽各种办法，想让它停止递归，但都没有任何作用。
> 打算用计数器的方式关闭，但计数器属性的值，每次都会被覆盖，等于没设计数器。

在这种尴尬的情况下，如果说推倒之前写的sql生成器，
重写正常来写jdbc代码来实现，呵，但项目答辩迫在眉捷，时间已经来不及了。
也不知道哪里来的勇气，我也不清楚当时在想些什么。
结果就是抄起小破站，打开了MyBatis的视频：

### sql生成器崩溃后的补救 --> MyBatis && Maven

>于是在第二天晚自习，打开了小破站，开始学习MyBatis
>很荣幸，一天的时间搞定知识点。
> 在MyBatis之前，并不知道 Maven，
> 只是当时视频里是这么做的，而我也跟着做，
> 直到第三天，Maven出问题了，
> 才知道还要去配阿里去镜象，啥是阿里去镜象？？？？？
> 另外，MyBatis的多表操作，也是百度来的（视频里也并没有去教（快速入门的视频，不会教太多））
><br>
>第三天晚自习成果：

```java
package com.sdllb.service.root;

public class BaseService {
	//sqlSession工厂（整个项目中有一个足以）
    private static SqlSessionFactory factory;
    private SqlSession sqlSession;

    static {
        //这一段不用解释，就拿到sqlSession工厂
        try {
            String config = "mybatis.xml";
            InputStream in = Resources.getResourceAsStream(config);
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            factory = builder.build(in);
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
	
	/*从sqlsession工厂中得到sqlSession
	（本以为有时会用到这个方法，以防万一没设成private，但一直没有）
	*/
    public SqlSession getSqlSession() {
        sqlSession = null;
        if (factory != null) {
            sqlSession = factory.openSession(true);
        }
        return sqlSession;
    }
	
	/*
	得到Dao层的接口类
	*/
    public <T> T getDaoInterface(Class<T> daoInterface) {
        SqlSession sqlSession = this.getSqlSession();
        return sqlSession.getMapper(daoInterface);
    }

	/*
	事务提交
	*/
    public void commit() {
        sqlSession.commit(true);
    }
    
	/*
	关闭sqlSession
	*/
    public void close() {
        sqlSession.close();
    }
}

```
其中一个Service
```java
package com.sdllb.service.IMP;

public class YiLiaoYongPinTypeServiceIMP implements YiLiaoYongPinTypeService {
	/*
	第一次时，使用的是继承BaseService,但还是出问题，
	具体出什么问题，已经不记得了，好像是session冲突，
	在之后，就改成了这种属性引入的方式
	*/
	//new BaseService对象
    private BaseService baseService = new BaseService();
    //通过BaseService对象，得到Dao接口，
    private YiLiaoYongPinTypeDao yiLiaoYongPinTypeDao = baseService.getDaoInterface(YiLiaoYongPinTypeDao.class);

    @Override
    public boolean addYiLiaoYongPinType(YiLiaoYonPinType yiLiaoYongPinType) {
        int i = yiLiaoYongPinTypeDao.addYiLiaoYonPinType(yiLiaoYongPinType);
        
        /*
        在这个位置，可以说，这一句是真的多余，
        如果没有这一句，的确是可以直接返回，
        （当时没有AOP的概念，只是在这次项目做完后，才知道的）
        */
        baseService.close();
        
        return i > 0;
    }

    @Override
    public boolean updateYiLiaoYongPinType(int id, YiLiaoYonPinType yiLiaoYonPinType) {
        int i = yiLiaoYongPinTypeDao.updateYiLiaoYonPinType(id, yiLiaoYonPinType);
        baseService.close();
        return i > 0;
    }

    @Override
    public List<YiLiaoYonPinType> findAllYiLiaoYongPinType() {
        List<YiLiaoYonPinType> yiLiaoYonPinTypes = yiLiaoYongPinTypeDao.selectAllYiLiaoYongPinType();
        baseService.close();
        return yiLiaoYonPinTypes;
    }

    @Override
    public List<YiLiaoYonPinType> findLimitYiLiaoYonPinType(int thisPage, int pageSize) {
        if (thisPage <= 0) thisPage = 1;
        int start = (thisPage - 1) * pageSize;
        List<YiLiaoYonPinType> yiLiaoYonPinTypes = yiLiaoYongPinTypeDao.limitYiLiaoYonPinTypeSelect(start, pageSize);
        yiLiaoYongPinTypeDao.limitYiLiaoYonPinTypeSelect(start, pageSize);
        baseService.close();
        return yiLiaoYonPinTypes;
    }

    @Override
    public int findYiLiaoYonPinTypeCount() {
        int i = yiLiaoYongPinTypeDao.selectYiLiaoYonPinTypeCount();
        baseService.close();
        return i;
    }

    @Override
    public boolean removeYiLiaoYonPinType(YiLiaoYonPinType yiLiaoYonPinType) {
        int i = yiLiaoYongPinTypeDao.deleteYiLiaoYonPinType(yiLiaoYonPinType);
        baseService.close();
        return i > 0;
    }

    @Override
    public YiLiaoYonPinType findYiLiaoYonPinTypeById(YiLiaoYonPinType yiLiaoYonPinType) {
        YiLiaoYonPinType newYiLiaoYonPinType = yiLiaoYongPinTypeDao.selectYiLiaoYonPinTypeByID(yiLiaoYonPinType.getYiLiaoYonPinTypeId());
        baseService.close();
        return newYiLiaoYonPinType;
    }
}

```

### 项目结束，整理公共代码，
（
常用的静态方法，等等，做成一个单独的项目，并用Maven打install，供其它项目共享，
至此，它成了我的，代码仓库，
我也给这个项目起了一个名字：lingDream
仅仅是因为我的名字里，有个“令”字；
或许，这名字总是感觉那么地别扭，但以我的英语水平，我也实在想不出什么更好的名字
）


 ## SSM
 在学习SSM时，倒也没什么要说的，
 只是在写了一两个Mapper层的方法后，因为这是我正式写的第二次Mapper层方法，第一次是在上一个Servlet项目中，
 此时，我就发现，每个Mapper类代码都有一个xXXInsert（）的方法，当然，现在并没有像上次项目一样，时间紧迫，来不及思考。

尝试了一下，把前缀 xXX去了，insert（）方法提到BaseMapper中，
记忆中当时可能是遇到了点问题，然后引入了泛型，来解决，

```java
package ling.evidences.dao.root;

import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * 泛型 T 由子接口去声明
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-26 14:24
 */
public interface BaseDao<T> {
    //region 查询

    /**
     * 查询所有
     */
    List<T> selectAll();

    /**
     * 根据Id查询
     */
    T selectById(T t);

    /**
     * 查询总共有多少条数据
     */
    int selectAllCount();

    /**
     * 分页查询
     */
    List<T> selectLimit(@Param("start") int start, @Param("pageSize") int pageSize);

    //endregion

    /**
     * 增加
     */
    int insert(T t);

    /**
     * 删除
     */
    int delete(T t);

    /**
     * 修改
     */
    int update(T oldS,T newS);
}
```
其中一个子类

```java
package ling.evidences.dao;

import ling.evidences.dao.root.BaseDao;
import ling.evidences.entity.School;
/**
 * BaseDao 接口 的其中一个子接口，
 * 该接口中没有定义任何的抽象方法，
 * 唯一做的事只有声明一下泛型，
 * 如果说需要加什么特殊的方法，如abc()，
 * 需要考虑 abc() 是不是可以公用的，
 * 如果是，则直接加在BaseDao 接口中，
 * 如果不是，则放在该 接口中即可（加在此处，后期service需要做一次类型转换）
 * <School>
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-25 10:47
 */
public interface SchoolDao extends BaseDao<School> {
	// 如果没有特殊情况，这里不用写任何一丁点的代码
}

```

BaseMapper写好，然后到了提BaseService，
（此时，两个决策上的失误，让我绕了一大圈）

```java
package ling.evidences.service.root;

import java.util.List;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-26 15:06
 */
public interface BaseService<T> {
    //region 查询
    /**
     * 查询所有
     */
    List<T> findAll();

    /**
     * 根据Id查询
     */
    T findById(T t);

    //endregion

    /**
     * 增加
     */
    boolean add(T t);

    /**
     * 删除
     */
    boolean remove(T t);

    /**
     * 修改
     */
    boolean modify(T oldT,T newT);
}
```
把分页相关的抽象方法,提取到另一个接口里

```java
package ling.evidences.service.root;

import ling.evidences.tool.BasePage;

import java.util.List;

/**
 * 将分页模型提取出来
 * （分页模型需要一个Service）
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-25 15:51
 */
public interface PageService<T> {
    /**
     * 查询总共有多少条记录
     */
    int findAllCount();

    /**
     * 分页查询
     */
    List<T> findLimit(int start, int pageSize);

    /**
     * 得到分页模型对象
     * 默认default 方法
     */
     default  BasePage<T> getBasePage() {
        return new BasePage<>(this);
    }
}
```
抽象的实现类,我给它起名:Transition,即桥梁的意思

```java
package ling.evidences.service.root;

import ling.evidences.dao.root.BaseDao;

import java.util.List;

public abstract class Transition<T>
        implements PageService<T>, BaseService<T> {

	/**
	 * 无论是单表查询，还是多表查询，还是什么......
	 * 那么只要它属于service业务类，
	 * 那么它必定会有会有一个dao层的对象，
	 * 因为上面的每一步的操作，都重点突出在泛型上面，
	 * 此时，泛型的根本作用则完完全全地体现了出来！
	 */
    protected BaseDao<T> baseDao;

	/**
	 *
	 * 第一个决策失误：选择**setter**注入；
	 * 第二个决策失误：知道@Service，@AutoWrite注解，但都没有去用。
	 * 第一个失误，导致Service层，没法用@AutoWrite，它始终为Null，我一直都没明白。
	 * 虽然还可以用**构造器**注入，但在我当时的想法中，实在不想在看不惯，
	 * 子类service多出来的那一小段构造器，所以**构造器**注入的方式，在还没测试就被舍弃，
	 * 现在想来，不免有些后悔
	 * 最终结果：我的BeanXml配置文件，无限增长
	 *
	 * 以下是原注解:
	 *	 	 	 
	 * 因为一直以来都强调泛型，
	 * 所以即使是设置第一Dao层的方法可以直接提取出来，
	 * 而不用担心问题（泛型不匹配，运行时是直接报错的）
	 * （除非故意不去对应，每个节点的泛型）
	 */
    public void setBaseDao(BaseDao<T> baseDao) {
        this.baseDao = baseDao;
    }

    @Override
    public List<T> findAll() {
        return baseDao.selectAll();
    }

    @Override
    public T findById(T T) {
        return baseDao.selectById(T);
    }

    @Override
    public boolean add(T T) {
        return baseDao.insert(T) > 0;
    }

    @Override
    public boolean remove(T T) {
        return baseDao.delete(T) > 0;
    }

    @Override
    public boolean modify(T oldT, T newT) {
        return baseDao.update(oldT, newT) > 0;
    }

    @Override
    public int findAllCount() {
        return baseDao.selectAllCount();
    }

    @Override
    public List<T> findLimit(int start, int pageSize) {
        return baseDao.selectLimit(start, pageSize);
    }
}
```
其中一个子类

```java
package ling.evidences.service.impl;

import ling.evidences.entity.School;
import ling.evidences.service.root.Transition;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2020-12-25 10:47
 */
public final class SchoolServiceImpl extends Transition<School> {
	//这里如果没有特殊情况，是不用再去添加代码的
	//当然，多表查询属于特殊情况（下一个代码块处说明）
}

```
最终结果: 可用,但xml配置文件无限增长,且,对于SpringMvc中的控制层,没成功提取父类

SSM学得差不多，于是就写下了下面这篇文章：
点击**文章标题**：
[SSM 面向接口开发_实际开发中这样真的可以吗?](https://blog.csdn.net/qq_43210788/article/details/111823742)
 
> 或许，
 在现在回头看，它更应该叫
 **泛型化开发**

### 写完SSM文章后，升级代码生成器
> 经过SSM的洗礼，我能感觉到，之前的代码生成器，由于只能生成Entity层，已经不再适用。

> 上次项目结束后，
> 我一个**shift**+**delete**删除这该死的sql生成器，差点坑死我。
> 但代码生成器，保留了下来，只不过它也需要做一次升级。
> 再像之前一样地一点一点地用StringBuffer来拼接，一定是不可能的。
> 毕竟，这一次摆在我面前的，并不是只有Entity一种 java文件
> 所以我需要找到一种技术，通过 
> 模板 + 数据 => 结果
> 经过努力，
> 还真让我在小破站上找到了一个。
> 而我也在这个阶段，知道了有模板引擎这东西。
> 使用的技术：freemarker + jdbc元数据
> 缺点：freemarker所要求的后缀名为 **.ftl** 
> 其它都还好，但缺点是，编辑器没给我引导提示，这我是一定能理解的，
> 但它报错是几份意思？？更有的时候一个空指针异常，就直接把错误堆满整个文件....
> 额，，，，，好在能用，也解决了之前，只能生成entity层的缺点，暂时用着吧！
> 等以后，我学了其它技术，再回来**收拾**你！！

### 我的SSM与学校的Oracle（即将放寒假）
>  我在学着SSM时，学校老师教着Oracle数据库，但我没跟着，
>  虽然说，这并不是什么值得炫耀的事，
>  但当时我的想法是，如果公司需要用到的是Oracle，
>  我一定能在可控的时间内，搞定Oracle的知识点，并立即上手，干它
>  比起Oracle，我更需要的Spring，
>  更贴近于当时的心情，我需要的是AOP，
>  只不过在SSM学习中，我的收获，比我想象中更多，


### 假期
 学完SSM，学校也迎来了放假，本打算在利用假期做个项目的，但琐事太多，一直都没有什么机会，直到开学，才有了机会，但此时，利用SSM来开发一个项目的心已经消磨贻尽。
### 收假后第一两个星期
最后一学期开学了，学校再一次地换了老师，教的是MyBatis，我听了一两节课，就没有再听。
想来，他们学完了Oracle数据库，而我用他们学Oracle的那段时间里，我学完了SSM。

虽然开学了一两个星期，但此时的我，却并没有急着去学Spring Boot，而是继续完善着我的代码生成器，以及，用SSM框架做着我的英语单词项目，
毕竟，放假在家里，我是没有足够的时间做这些的。
<hr>
 
## springBoot 阶段

 说来，也是巧合，我进入SpringBoot阶段，是因有群里朋有的聊天，
 它说，别用SSM了，用Spring Boot来做项目，很爽，
 然后，我就打开了小破站，开始学Spring Boot。
 别说，比起SSM真的更舒服。
### 《个人英语单词》项目（先用SSM做了一点点，改SpringBoot，再暂停）
英语单词项目：前期没建管理员相关的表的时候是八张表。此时的我并没有学习到Maven可以将一个项目分割成多个模块，并进行开发；
而在此时，面对一个一打开就是一长串的类，以及堆成一坨的页面文件，几乎是毫无办法，
唯一想到的办法则是，将页面的.html模板文件，（此时，已经用的**thymeleaf**模板引擎），使用文件夹分割开。

```json
所有（八张）表{
	单词与中文相关{
		单词表：
			(该项目的根本)
			
		中文表：
			(中文翻译)
			
		单词与中文关系表：
			(每个英语单词对应的不会只有一个中文，一个中文不会只对应一个英语单词)
	}
	单词类型相关{
		单词类型表：
			(名词?副词?...)
			
		单词与类型关系表：
			(每个英语单词对应的不会只有一个类型，一个类型不会只对应一个英语单词)
			
	}
	单词标签相关{
		单词标签表：
			(人们总是喜欢把一堆又一堆东西进行打标签，再分类)
			
		单词与标签关系表：
			(一个单词可以被打上不同的标签，而一个标签可不是只属于一个单词)
			
		单词标签组表：
			(打的标签太多了，给它分一下组，就像文件夹一样)
	}
}
```
>
>这样把页面用文件夹分类开解决问题了吗？其实并没有！！整个项目文件夹变得嵌套很深
>比如：需要去找中文表的添加页面，
>路径:
> /admin/page/wordAndChinese/chinese/insertChinese.html
>  
>  其中admin文件夹说的是管理员页面，
>  与page同级的是另一个文件夹，名为：adminPublic，这是一个装着被公共引用的页面**零件**

个人英语单词项目，就做到了这里，然后.....

### srpingBoot项目（一：（学校宿舍管理系统））
开学第三个星期，学校老师让我别再做，我的~~《英语单词项目》~~垃圾项目了，他给我找新的项目，让我来做。
说真的，如果他提前问我一下，我做的是什么项目，再让我别做，我会真的听话，但，这……
好吧，我承认，我最后还是把项目接了，条件是：我不再做任何作业，不再听课。

学校宿舍管理系统总共15张表
```json
所有表{
	宿舍表：
		(整个项目的根本)
		
	宿舍分区表：
		(男生一楼，女生一楼......)
		
	宿舍状态表： 
		(因为学校的特殊情况，
		在这里写死了一部分东西，
		比如：宿舍淋浴头坏没坏，
		坏了打0，没坏打1)
		
	宿舍类型表：
		(男生宿舍、男老师宿舍、女生宿舍、女老师宿舍)
		
	宿舍卫生成绩表：
		(宿舍每隔一段时间都会评分)
		
	宿舍卫生成绩标题表：
		(它的作用就像 域名与Ip地址 一样，给每一期的评分作一个标题，好记住)
		
	班级表：
		(宿舍出问题，可以迅速查出这个宿舍归属于某个班级)
		
	管理员表：
		(登录)
		
	管理员类型表：
		(权限管理：1：普通账号，0：超级管理员账号)
		
	人员表：
		(所有的老师与学生都是人,之前试过，如果直接建一个教师表，并不容易控制，或者是因为能力尚浅)

	学生表：
		(学生表中，
		第一个字段为外键，即引用人员表，
		然后，第二个键为该学生所在的班级)
		
	人员类型表：
		(1：老师，0：学生)
		
	宿舍报修表：
		(报修时间，报修内容，报修备注)
		
	报修状态表：
		(是否修好，是否已报修等等)
}
```

继续学习着SpringBoot，偶然间，看到视频弹幕上，用MyBatis Plus，可以省掉很多spl语句，这样的一句话，让我想起以前我的sql生成器，自然而然地，我就找了篇MyBatis Plus的快速入门的视频来看。

#### MyBatisPlus（V：2.2）
MyBatis Plus同样地没看一两天就开始干了，或者说是，边看边做学校的项目。

在使用在使用MyBatisPlus时，我遇到了第一个问题：
<br>

##### 处理分页
###### 循环分页，页码
我总是习惯，先去往数据库里乱敲一堆数据在里边，反正是开发环境，然后把它查出来，显示出来；
已知:

```java
Mybatis Plus中
	org.apache.ibatis.session.RowBounds类的对象作用是处理分页
	它下面有个子类:
		com.baomidou.mybatisplus.plugins.pagination.Pagination; //但我们并不用这个,
	再往它的子类:
		com.baomidou.mybatisplus.plugins.Page<T>; //其实相信到了这里,很多人都会选择用这一个
	但当我真正在做分页的时.....
	因为我用的是在Servlet时期学到的分页样式:
	每页最多显示5个页码:
	1
	1 2 
	1 2 3 
	1 2 3 4 
	1 2 3 4 5 
	2 3 4 5 6
	3 4 5 6 7
	(当存在有上一页是,显示上一页按扭,与,存在下一页按钮,则显示下一页按钮)
			【1】，2，3，4，5 [下一页]
	[上一页] 1，【2】，3，4，5 
			1，2，【3】，4，5
```
在以前的JSP代码中，这些是靠JSTL的<C:SET/>来计算赋值，
但这段代码是从老师的文档中直接粘过来的，
这里教程里的老师故意留了一个坑，即，他只是考虑了总页数小于等于10的情况：
而在总页数超过10的时候，会有Bug
```	php
	<%--页码输出的开始--%>
	 <c:choose> 
	 	<%--情况 1：如果总页码小于等于 5 的情况，页码的范围是：1-总页码--%> 
	 	<c:when test="${ requestScope.page.pageTotal <= 5 }"> 
		 	<c:set var="begin" value="1"/> 
	 		<c:set var="end"value="${requestScope.page.pageTotal}"/>
	 	 </c:when>
	 	 <%--情况 2：总页码大于 5 的情况--%>
	 	 <c:when test="${requestScope.page.pageTotal > 5}">
		 <c:choose> 
			<%--小情况 1：当前页码为前面 3 个：1，2，3 的情况，页码范围是：1-5.--%> 
		 	<c:when test="${requestScope.page.pageNo <= 3}"> 
		 		<c:set var="begin" value="1"/> 
				<c:set var="end" value="5"/>
			</c:when>
			<%--小情况 2：当前页码为最后 3 个，8，9，10，页码范围是：总页码减 4 - 总页码--%> 
			<c:when test="${requestScope.page.pageNo > requestScope.page.pageTotal-3}">
		  		<c:set var="begin" value="${requestScope.page.pageTotal-4}"/>
		   		<c:set var="end" value="${requestScope.page.pageTotal}"/> 
	   		</c:when>
	   		<%--小情况 3：4，5，6，7，页码范围是：当前页码减 2 - 当前页码加 2--%> 	
	   		<c:otherwise>
				<c:set var="begin" value="${requestScope.page.pageNo-2}"/>
		 		<c:set var="end" value="${requestScope.page.pageNo+2}"/>
		 	</c:otherwise>
		  </c:choose> 
	</c:when>
</c:choose>

<c:forEach begin="${begin}" end="${end}" var="i">
	 <c:if test="${i == requestScope.page.pageNo}"> 【${i}】 </c:if>
	  <c:if test="${i != requestScope.page.pageNo}">
	   <a href="${ requestScope.page.url }&pageNo=${i}">${i}</a> </c:if>
	    </c:forEach>
	    <%--页码输出的结束--%>
	     <%-- 如果已经 是最后一页，则不显示下一页，末页 --%> 
	     <c:if test="${requestScope.page.pageNo < requestScope.page.pageTotal}"> 
	    	 <a href="${ requestScope.page.url }&pageNo=${requestScope.page.pageNo+1}">下一页</a>
	    	 <a href="${ requestScope.page.url }&pageNo=${requestScope.page.pageTotal}">末页</a>
	    </c:if> 
	    共${ requestScope.page.pageTotal }页，${ requestScope.page.pageTotalCount }条记录 
	   
	    到第<input value="${param.pageNo}" name="pn" id="pn_input"/>页 
	    <input id="searchPageBtn" type="button" value="确定"> 
	    
	    <script type="text/javascript"> $(function () {
		     // 跳到指定的页码 $("#searchPageBtn").click(function () {
		     	 var pageNo = $("#pn_input").val();
		     	 <%--var pageTotal = ${requestScope.page.pageTotal};--%> 
		     	 <%--alert(pageTotal);--%> 
		     	 // javaScript 语言中提供了一个 location 地址栏对象 
		     	 // 它有一个属性叫 href.它可以获取浏览器地址栏中的地址 
		     	 // href 属性可读，可写
				location.href = "${pageScope.basePath}${
					 requestScope.page.url }&pageNo=" + pageNo; 
					 });
				 }); 
		</script>
```
但到了**thymeleaf**中，虽然它也有一个与<c:set/>相似的**th:with**，
但我试了很多次，结果都一样，第一次设置begin或end的值之后，
不记得当是是报错，还是没有像jsp一样，改变为最终的值，反正就是没用；

> 回头分析一下，其实我一直需要的是 **begin** 与 **end**两个属性，用来作循环输出页码，
> 既然前端的**thymeleaf**无法计算，那么就让后端来计算输出

```java
package com.lingDream.root.tool;

import com.baomidou.mybatisplus.plugins.Page;
import lombok.EqualsAndHashCode;

/**
 * 第一个版本中，是仿照上面JSP页面中的逻辑，
 * 直接来的，但后来，总页数只要超过10，
 * 会出现Bug，
 * 
 * 而这下面是最终修改过的，
 * 继承于Page<T>对象，
 * 并添加了两个属性
 * begin 和 end
 * 
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-03-15 13:27
 */
@EqualsAndHashCode(callSuper = true)
public class MyPage<T> extends Page<T> {
    private Long begin;
    private Long end;

    public Long getBegin() {
        //默认情况
        begin = super.getCurrent()-2L;

        // 后三页
        if (super.getCurrent()>=super.getPages()-2) begin = super.getPages() -4;

        // 前三页
        if (super.getCurrent()<=3) begin = 1L;

        //特殊情况 总页数为4
        if (super.getPages() == 4) begin =1L;

        return begin;
    }


    public Long getEnd() {

        //默认情况
       end = super.getCurrent()+2L;
        // 如果是前三页
        if (super.getCurrent()<=3) end = 5L;

        // 如果是后三页
        if (super.getCurrent() >= super.getPages()-2) end = super.getPages();

        // 特殊情况 总页数为4
        if (super.getPages() == 4) end = 4L;

        return end;
    }

    public MyPage(int current, int size) {
        super(current, size);
    }
}
```
> 其实，即使到这样，它还是不够完美的，因为这是每页只显示5个页码的情况，
> 但如果我需要的是，每页显示6个页码呢？7个呢？
> 有没有一种算法，让它只需要传入你每页需要显示几个页码，
> 那么它就可以自动计算出**begin**，**end**
> 或许是有的，但我没有去深究

到此，我从数据库中，利用MyBatis Plus取值，分页成功显示，
```html
<!--region 分页-->
<!--/*@thymesVar id="page" type="com.lingDream.root.tool.MyPage<T>"*/-->
<nav aria-label="Page navigation" class="col-md-offset-4 ">
    <ul class="pagination">
        <li th:if="${page.hasPrevious()}">
            <a th:href="|@{/}${limitPath}=1|">
                <span>首页</span>
            </a>
            <a th:href="|@{/}${limitPath}=${page.current - 1}|" aria-label="Previous">
                <span aria-hidden="true">&laquo;</span>
            </a>
        </li>
        <li th:each="i:${#numbers.sequence(page.begin,page.end)}">
            <span th:if="${i==page.current}">【[[${i}]]】</span>
            <span th:if="${i!=page.current}">
                <a th:href="|@{/}${limitPath}=${i}|" th:text="${i}"></a>
            </span>
        </li>
        <li th:if="${page.hasNext()}">
            <a aria-label="Next" th:href="|@{/}${limitPath}=${page.current + 1}|"><span aria-hidden="true">&raquo;</span></a>
            <a th:href="|@{/}${limitPath}=${page.pages}|"><span>末页</span></a>
        </li>
        <li>共[[${page.pages}]]页,[[${page.total}]]条记录</li>
        <li class="page_nav_item">到第<input th:value="${page.current}" name="pn" id="pn_input"/>页
            <input id="searchPageBtn" type="button" value="确定"></li>
    </ul>
</nav>
<script type="text/javascript">
    $(function () {
        // 跳到指定的页码
        $("#searchPageBtn").click(function () {
            let pageNo = $("#pn_input").val();
            location.href = "[[|@{/}${limitPath}|]]=" + pageNo.trim();
        });
    });
</script>
<!--endregion-->
```

###### 个性化定制显示数据
下一步，做宿舍表的分页展示数据，但我需要显示一下这个宿舍里面住着有多少个人。
```json
第一种方案，分两次查询，（否决！）
	即先查询到当前宿舍，再根据当前宿舍的Id，再查询出这个宿舍的人数
第二种方案，修改Mapper.xml文件（成功一半）
	{
		com.hrxy.dormitory.mapper.ManagerMapper.selectPage
	} 
	原：
		Has been loaded by XML or SqlProvider,ignoring the injection of the SQL.
	翻译：
		已经被XML或SqlProvider加载，忽略了sQL的注入。
 就这一句话：证明了这第二种方案可行，
 	但问题是 分页查询时，最后需要limit #{start}，#{pageSize}
```
到这一步，我卡了很久，很久，查阅了百度，官网都没能找到答案。
然后，看起了MyBatisPlus的源码（从BaseMapper类开始看，地毯式地查找），
遗憾的是，我并不能完全看得懂源码，所以也没能找到解决方法，
但这次阅读源码，也并不是完全一无所获，
我在源码中找到了
	 com.baomidou.mybatisplus.service.IService的一个接口，
以及它的实现类:
	com.baomidou.mybatisplus.service.impl.ServiceImpl
	
##### 意外的收获：ServiceImpl

```java
public class ServiceImpl<M extends BaseMapper<T>, T> implements IService<T> {
....
...
}
```

接着就是把分页的事情先放到一边，百度起关于这两个类的资料：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506103955718.png)
> 原来如此，原来MyBatis Plus也同样做着一件相同的事，
> 把Mapper层，Service层，公共的代码放到父类。让子类Mapper、Service写少量，或不写代码

但我真正改写我的Service代码，继承于ServiceImpl代码时：

```java
public class DormitoryService extends ServiceImpl<DormitoryMapper<Dormitory>,Dormitory>{
	/*
	这一句写下来，总觉得不舒服，我并不愿意这样写，
	我希望的效果是：
		public class DormitoryService extends ServiceImpl<Dormitory>{}
			毕竟这样是更加简洁的
	*/
}
```
原本计划是仿照MyPage<T>一样的解决思路，
使ServiceImpl再往下继承一层，消除掉 泛型**M**
但到了真正写的时候，才发现，这是不可能的！根本不可能

原本以为只是我的版本低了点，试过将MyBatis Plus升级 3.X 的版本，
但依然，没有改变，

最后，一狠心，把ServiceImpl的代码，复制下来，改名为MyService<T>，
![在这里插入图片描述](https://img-blog.csdnimg.cn/202105061101436.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_16,color_FFFFFF,t_70)
但一开始，我并没有像上面一样，使用构造函数来弄，也是同样地按照
```java
setBaseMapper(MyMapper<T> baseMapper){this.baseMapper = baseMapper}
```
这样的方法，但它一直在报错
	（
		有些记不清，当时报的是空指针异常：baseMapper为**null**，
		还是匹配到多个Bean，导致冲突，项目都没运行成功
		）
还好没放弃，我把Setter注入，换成了构造器注入，让子类继承时，必须去实现它的构造函数。
运行成功！

这时候我才明白，所谓的@Autowired，执行的时机是在构造方法之后，set注入的方式，而我选择用构造函数，让子类来定义泛型&lt;T&gt;，则避免了这个问题。

回忆起SSM阶段，如果当时，我选的就是构造函数注入，
或许，就不用绕这么一个大圈！！！

##### 回头再次解决limit参数问题
> 第一次尝试，把limit写死，成功运行，也让我找到了规律
```xml
<select id="selectPage" resultType="返回类型">
        select 需要的列 from 表名
        limit 1,2
        <!--
        虽然没真正的显法sql语句，但从页面结果反推可知：
        MyBatis Plus自动在我的limit 1,2后,又加了一次limit。
        最终语句：
        select 需要的列 from 表名
        limit 1,2 limit 参数1，参数2
        -->
</select>
```
> 然后，删了limit，成功实现**偷梁换柱**

```xml
<select id="selectPage" resultType="返回类型">
        select 需要的列 from 表名
</select>
```

至此，有了这篇文章
点击**文章标题**：
[spring boot泛型化编程(适用于spring)](https://blog.csdn.net/qq_43210788/article/details/115859347)

#### Maven分模块
Maven分模块，也是在这一时期学的，起因是因为一个同班同学和我说的，当然，也只是提示，
最终还是百度，但百度查来的，并不能直接用，而是需要自己改动一下。

[maven模块划分（个人风格，不完美，未来会改进)](https://blog.csdn.net/qq_43210788/article/details/115865671?spm=1001.2014.3001.5501)

当然，这样的模块划分，我还是不满意：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210506131425512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_16,color_FFFFFF,t_70#pic_center)
当随着模块的增加，这一块文件夹下来，肯定是又臭又长，但起码现阶段可用，也就没继续深研了！

至此，《学校宿舍管理系统》**看似**完结，源代码，**war包**，文档说明也交给了老师。
（
直到后来我做另一个项目，才发现，spring boot 打war包，在本地windows系统中，tomcat能运行，但linux中，不知为何，同样的版本，就是运行不了，但项目交给学校老师，事情过了一个多月，他们都没有开始部署
）


当然，在做这个《学校宿舍管理系统》时，
我也不会忘记，继续完善我的代码仓库

我也要回过头来，继续写我的《个人英语单词项目》了

### 继续写《个人英语单词》项目
再次来编写《个人英语单词》项目时,第一步,当然是把项目分成多个Maven模块，这样分出来确实是比之前好很多，也更方便。
当然，利用一下Maven的特性，当某个项目依赖于其它某个项目时，它会把它依赖的jar包一同导入，
所以主配置文件只需要加上我的仓库依赖即可。
#### 反向思维，共用页面
> 前置说明：（根据我的个人情况）
> 在后台管理系统中，几乎所有的Controller都会有的请求方法：
> 1：分页的请求
> （1）去到新增页面的请求
> （2）去到修改页面
> （3）添加或修改的结果页面
> <br>
> 2：个人前期情况
> （1）一个类模块的添加和修改，共用为一个页面
> （2）所有类模块的添加和修改的结果页面，共用为一个页面
>  <br>
>  3：需要的知识准备
>  （1）thymeleaf引入页面的操作方法
>  （2）往Model对象中装数据
>  （3）反射获取类名

> BaseController接口

```java
package com.lingDream.root.controller;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-04-21 10:37
 */
public interface BaseController<T> {
    @RequestMapping({"/getPage"})
    String getPage(Model model, String val, String filter, Integer thisPage);

    @RequestMapping(
        value = {"/add"},
        method = {RequestMethod.POST}
    )
    String add(HttpServletRequest request, T entity, Model model);

    @RequestMapping(
        value = {"/delById"},
        method = {RequestMethod.POST}
    )
    @ResponseBody
    String delById(HttpServletRequest request, T entity);

    @RequestMapping({"/updateFindById"})
    String updateFindById(HttpServletRequest request, T entity, Model model);

    @RequestMapping(
        value = {"/update"},
        method = {RequestMethod.POST}
    )
    String update(HttpServletRequest request, T entity, Model model);

    @RequestMapping({"/toInsertPage"})
    String toInsertPage(Model model);
}

```
>抽象类：MyController:定义构造规则，以及一些公用的方法，
> ControllerPageConfig ：定义的页面路径对象
```java
package com.lingDream.root.controller;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-04-21 10:37
 */
public abstract class MyController<T> implements BaseController<T> {
    @Autowired
    protected ControllerPageConfig controllerPageConfig;
    protected final BaseService<T> service;
    protected final String COMMENT;

    public MyController(BaseService<T> service, String COMMENT) {
        this.service = service;
        this.COMMENT = COMMENT + " →";
    }

    protected MyPage<T> getMyPage(Integer thisPage) {
        return this.service.selectPage(new MyPage(thisPage, 10), (Wrapper)null);
    }

    protected MyPage<T> getMyPage(Integer thisPage, Integer pageSize) {
        return this.service.selectPage(new MyPage(thisPage, pageSize), (Wrapper)null);
    }

    protected MyPage<T> getMyPage(Integer thisPage, Wrapper<T> wrapper) {
        return this.service.selectPage(new MyPage(thisPage, 10), wrapper);
    }
	
	/**
	 *页面中的${title}不需要最后的 " →"
	 * 要不然显示出来会很别扭
	 * <title th:text="${title}"></title>
	 */
    protected void setTitle(Model model) {
        String substring = this.COMMENT.substring(0, this.COMMENT.indexOf(" →"));
        model.addAttribute("title", substring);
    }

    protected void setRequestURL(HttpServletRequest request, Model model) {
        String requestURL = request.getRequestURL().toString();
        String[] split = requestURL.split("/");
        String classRequestMapping = split[split.length - 2];
        model.addAttribute("toPage", classRequestMapping);
    }

    protected Wrapper<T> getWrapper(String val, String filter) {
        Wrapper<T> wrapper = new EntityWrapper();
        wrapper.like(filter, val);
        return wrapper;
    }
	
	/**
	 * 分页时使用的：val与filter用作模糊查询的参数
	 */
    protected void setPageTitleAndPartAddress(Model model, String val, String filter) {
        this.setTitle(model);
        String partAddress = this.getClass().getSimpleName();
        //就是页面零件的地址，通过反射获取小写类名
        partAddress = StringUtils.lowFirstChar(partAddress.substring(0, partAddress.length() - 10));
        model.addAttribute("partAddress", partAddress);
        String limitPath = partAddress + "/getPage?";
        if (!Objects.isNull(val)) {
            limitPath = limitPath + "val=" + val + "&";
        }

        if (!Objects.isNull(filter)) {
            limitPath = limitPath + "filter=" + filter + "&";
        }

        limitPath = limitPath + "thisPage";
        model.addAttribute("limitPath", limitPath);
    }
}
```
>抽象类ControllerImpl：实现一下BaseController

```java

package com.lingDream.root.controller;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-04-21 10:37
 */
public abstract class ControllerImpl<T> extends MyController<T> {
    public ControllerImpl(BaseService<T> service, String COMMENT) {
        super(service, COMMENT);
    }

    public String getPage(Model model, String val, String filter, Integer thisPage) {
        if (thisPage == null) {
            thisPage = 1;
        }

        Wrapper<T> wrapper = this.getWrapper(val, filter);
        MyPage<T> myPage = this.getMyPage(thisPage, wrapper);
        model.addAttribute("page", myPage);
        this.setPageTitleAndPartAddress(model, val, filter);
        return this.controllerPageConfig.getListPage();
    }

    public String toInsertPage(Model model) {
	    /*判断model中存储的信息长度,
	    设置出路径(添加/修改),
	    此方法应该放在第一行
	    */
        String path = model.asMap().size() == 0 ? "add" : "update";
        model.addAttribute("path", path);
        
        this.setTitle(model);
        String partAddress = this.getClass().getSimpleName();
        partAddress = partAddress.substring(0, partAddress.length() - 10);
        model.addAttribute("partAddress", StringUtils.lowFirstChar(partAddress));
        return this.controllerPageConfig.getInsertPage();
    }

    public String add(HttpServletRequest request, T entity, Model model) {
        super.setRequestURL(request, model);
        model.addAttribute("result", this.COMMENT + "添加成功");
        if (!this.service.insert(entity)) {
            model.addAttribute("result", this.COMMENT + "添加失败");
        }

        return this.controllerPageConfig.getResultPage();
    }

    public String delById(HttpServletRequest request, T entity) {
        return this.service.deleteById((Serializable)entity) ? this.COMMENT + "删除成功" : this.COMMENT + "删除失败";
    }

    public String updateFindById(HttpServletRequest request, T entity, Model model) {
        T t = this.service.selectById((Serializable)entity);
        model.addAttribute("entity", t);
        return this.toInsertPage(model);
    }

    public String update(HttpServletRequest request, T entity, Model model) {
        super.setRequestURL(request, model);
        model.addAttribute("result", this.COMMENT + "修改成功");
        if (!this.service.updateById(entity)) {
            model.addAttribute("result", this.COMMENT + "修改失败");
        }

        return this.controllerPageConfig.getResultPage();
    }
}

```
> 真正的子类(一)
```java
package com.lingDream.llfEnglish.controller;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-04-21 10:37
 */
@Controller
@RequestMapping(value = "/chinese")
public class ChineseController extends ControllerImpl<Chinese> {
    public ChineseController(MyService<Chinese> service) {
        super(service, "中文组词");
    }
}
```
> 真正的子类(二)

```java
package com.lingDream.llfEnglish.controller;

/**
 * @Author: 酷酷宅小明
 * @CreateTime: 2021-04-21 10:37
 */

@Controller
@RequestMapping(value = "/word")
public class WordController extends ControllerImpl<Word> {

    public WordController(MyService<Word> service) {
        super(service, "英语单词");
    }

}
```

> 分页查询的页面:**list.html**
```html
<!DOCTYPE html>
<html lang="zh-CN" xmlns:th="http://www.thymeleaf.org">
<head>
    <th:block th:replace="admin/adminPublic/links.html"/>
    <title th:text="|${title}管理|"></title>
</head>
<body>
<!--region main 容器-->
<div class="container-fluid">

    <!--region 顶部导航菜单-->
    <nav class="navbar navbar-inverse navbar-fixed-top" th:include="admin/adminPublic/adminHeader.html"></nav>
    <!--endregion-->

    <hr>

    <!--region 内容区域-->

    <div class="row">

        <!--region 左侧导航栏-->
        <div class="col-md-2 sidebar sub-menu col-sm-3" th:include="admin/adminPublic/adminLeft.html"></div>
        <!--endregion-->

        <!--region 内容显示区域-->
        <div class="col-md-offset-2 col-md-10 col-sm-9 col-sm-offset-3" style="padding: 20px">

            <!--region 导航器-->
            <ol class="breadcrumb">
                <li><a th:href="@{/toIndexPage}">首页</a></li>
                <li class="active" th:text="|${title}管理|"></li>
            </ol>
            <!--endregion-->

            <!--region 模糊查询-->
            <!--<div th:if="${showPart}" class="row"
                 th:include="|/admin/${partAddress}/list/search-wrap.html|"></div>-->
            <!--endregion-->

            <br/>

            <!--region 操作菜单-->
            <ul class="breadcrumb">
                <li><a th:href="|@{/}${partAddress}/toInsertPage|" th:text="|新增${title}|"></a></li>
            </ul>
            <!--endregion-->

            <!--region 表格-->
            <!--region partAddress：就是通过反射获到到的-->            
            <table th:include="|admin/${partAddress}_list.html|" class="table table-bordered table-striped table-hover table-condensed text-center"></table>
            <!--endregion-->

            <!--region 分页模型-->
            <div th:include="|admin/adminPublic/adminLimit.html|">
            </div>
            <!--endregion-->

        </div>
        <!--endregion-->
    </div>

<!--endregion-->

   <!--region 底部信息-->
<div class="row">
    <div></div>
</div>
<!--endregion-->

</div>
<!--endregion-->
</body>
</html>

```

>增加或修改的页面：**inser.html**

```html
<!DOCTYPE html>
<html lang="zh-CH" xmlns:th="http://www.thymeleaf.org">
<head>
    <th:block th:replace="admin/adminPublic/links.html"/>
    
    <title th:text="|${'add'.equals(path)?'新增':'修改'}${title}|"></title>
</head>
<body>
<!--region main 容器-->
<div class="container-fluid">
    <!--region 顶部导航菜单-->
    <nav class="navbar navbar-inverse navbar-fixed-top" th:include="admin/adminPublic/adminHeader.html"></nav>
    <!--endregion-->

    <hr>
    <!--region 内容区域-->

    <div class="row">
        <!--region 左侧导航栏-->
        <div class="col-md-2 sidebar" th:include="admin/adminPublic/adminLeft.html"></div>
        <!--endregion-->

        <!--region 内容显示区域-->
        <div class="col-md-offset-2 col-md-10" style="padding:50px 100px 0">

            <!--region 导航器-->
            <ol class="breadcrumb">
                <li><a th:href="@{/toIndexPage}">首页</a></li>
                <li><a th:href="|@{/}${partAddress}/getPage?thisPage = 1|" th:text="|${title}管理|"></a></li>
                <li class="active" th:text="|${'add'.equals(path)?'新增':'修改'}${title}|"></li>
            </ol>
            <!--endregion-->

            <hr>

            <!--region 表单-->
            <form th:action="|@{/}${partAddress}/${path}|" method="post" class="form-horizontal" role="form">
                <th:block th:include="|admin/${partAddress}_add.html|"/>
                <div class="form-group">
                    <div class="col-md-offset-1 col-md-2">
                        <input value="提交" type="submit" class="btn btn-primary form-control">
                    </div>
                    <div class="col-md-2">
                        <input onClick="history.go(-1)" type="button" class="btn btn-info form-control" value="返回"/>
                    </div>
                </div>
            </form>
            <!--endregion-->
        </div>
        <!--endregion-->
    </div>

    <!--endregion-->

    <!--region 底部信息-->
    <div class="row">
        底
    </div>
</div>
<!--endregion-->
<!--endregion-->
</body>
</html>
```

>结果页面:addResult.html
```html
<!DOCTYPE html>
<html lang="zh-CN"  xmlns:th="http://www.thymeleaf.org">
<head>
    <links th:replace="admin/adminPublic/links.html"></links>
    <link th:href="@{/css/style.css}" type="text/css" rel="stylesheet">
    <title>结果</title>
</head>
<body>
<div class="one">
    <!-- #region Two -->
    <div class="two">
        <!-- #region 提示文字 -->
        <div class="tx">
            <p>
                <span th:text="${result}" style="font-size: 40px"></span>
            </p>
        </div>
        <!-- #endregion 提示文字-->
        <div class="wi"></div>
        <!-- #region but -->
        <div class="but_left">
            <button class="button" style="vertical-align:middle"><span><a
                    th:href="|@{/}${toPage}/toInsertPage|">返回添加页面</a> </span></button>
        </div>
        <div class="but_right">
            <button class="button" style="vertical-align:middle"><span><a th:href="|@{/}${toPage}/getPage?thisPage=1|">返回列表页</a> </span>
            </button>
        </div>
    </div>
    <!-- #endregion but-->
</div>
<!-- #endregion Two -->
</body>
</html>
```
>缩减代码：chinese_list
```html
<tr>
    <th>序号</th>
    <th>中文组词</th>
    <th>中文备注</th>
    <th>操作</th>
</tr>
<!--/*@thymesVar id="item" type="com.lingDream.llfEnglish.entity.Chinese"*/-->
<tr th:each="item:${page.records}">
    <td th:text="${itemStat.count}"></td>
    <td th:text="${item.chineseInfo}"></td>
    <td th:text="${item.chineseComment}"></td>
    <td>
        <a class="link-update"
           th:href="|@{/}${partAddress}/updateFindById?chineseId=${item.chineseId}|">修改</a>
        <a th:onclick="|delAjax({chineseId:${item.chineseId}})|" href="#" class="link-del">删除</a>
    </td>
</tr>
```
>缩减代码 chinese_add
```html
<!--region 中文组词Id-->
<div class="form-group" th:if="${entity!=null}">
    <div class="col-md-2">
        <label>中文组词Id</label>
    </div>
    <div class="col-md-5">
        <input type="hidden" name="chineseId" class="form-control"
               th:value="${entity?.chineseId}">
        <div th:text="${entity?.chineseId}"></div>
    </div>
</div>
<!--endregion-->

<!--region 中文组词信息 -->
<div class="form-group">
    <div class="col-md-2">
        <label for="chineseInfo">
            <i class="text-danger h3">*</i>中文组词信息：
        </label>
    </div>
    <div class="col-md-5">
        <input id="chineseInfo" class="form-control"
               name="chineseInfo"
               th:value="${entity?.chineseInfo}" size="50" type="text">
    </div>
</div>
<!--endregion-->

<!--region 中文组词备注-->
<div class="form-group">
    <div class="col-md-2"><label for="chineseComment">中文组词备注：</label></div>
    <div class="col-md-5">
        <textarea name="chineseComment" id="chineseComment" class="form-control" cols="30"
                  rows="10" placeholder="请输入中文组词备注"
                  th:text="${entity?.chineseComment}"></textarea>
    </div>
</div>
<!--endregion-->
```

**word_list，word_add:不想粘了**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508103234281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_16,color_FFFFFF,t_70)



> 自定义的配置文件
```yml
# 总文件
lingDream:
  controller:
    listPage: admin/index/list
    insertPage: admin/index/insert
    resultPage: admin/adminPublic/addResult
    aspect: true
  service:
    aspect:
      update: true
      insert: true
#SpringBoot 的总配置文件

#激活开发环境
#
spring:
  profiles:
    active: dev

#激活测试环境
#spring.profiles.active=test

#激活生产环境
#spring:
#  profiles:
#    active: product
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508100921413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210508100702239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjEwNzg4,size_16,color_FFFFFF,t_70)
在曾经一次和老师的交谈中，听老师说，有的公司用的是iframe来分割页面，但我现在发现了新大陆。
不能说iframe不好，只是iframe方式更简单一些，但后期的维护，在我的认为中并不是非常方便，
但我发现的这个，前期的项目搭建，确实不是一件容易的事，但后期的扩展，却非常容易。

## github & linux
github 与 linux的学习，可能最早应该追溯到，jdbc之前，但一直都是以试着玩的心态去用，
所以一直都没什么进展。遇到问题，查百度


