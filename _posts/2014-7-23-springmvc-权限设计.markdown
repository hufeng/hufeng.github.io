---
layout: post
title:  "SpringMVC-权限设计"
date:   2014-07-23 23:41:27
categories: jekyll update
---

>万丈高楼平地起。

权限管理对于管理系统而言已经是标配中的标配了吧，对于我等俗人更是不能免俗。同时就目前的项目状况而言，我们还不需要那么高大上的开源的解决方案，如Spring Security，Shiro。小伙伴一致决定我们还是从基本的功能迭代起来吧。

目标：

    1.实现权限的管理（CRUD）
    2.实现部门管理 （CRUD)
    3.实现人员的管理 （CRUD）
    4.实现部门和权限的关联管理（CRUD）
    5.实现人员和部门的关联关联 （CRUD）
    6.实现页面的权限到具体的动作而非仅仅页面级别的控制。

一眼望去，哦哦，除了6，剩下的就是分别对应我们的权限管理的5张表了。


	1.权限表
	2.部门表
	3.人员表
	4.人员和部门关联表
	5.部门权限关联表

关键实现思路

	1.权限表保存所有的系统的url，通过给不同的部门分配不同的url来控制权限。
	2.页面的操作(a标签或者button之类)，封装自定义JSP的tag，通过判断该用户所在的部门是否关联该权限做屏蔽。

难点：
		
	我们都知道Spring2.5后，Spring对REST风格的架构支持的是非常的好的，同时Restful风格的url简单，清晰.
	通常在Spring中通过@PathVariable占位符的形式来支持这种风格。如：
	@RequestMapping(value=“/user/{id}”, method=RequestMethod.GET)
	public ..findUser(@PathVariable String id) {
	}
		
	这样就会对我们的权限造成很大的影响，因为如果我们的权限表保存的是：
	id	url
	1	/user/{id}


	而用户实际访问的是/user/1，这样就造成这两个URL无法匹配，那我们的权限系统不就歇菜了嘛。那我权限就配置到/user呢？
	你确定？如果这样就放掉了太多url了的可能了吧。

解决思路:

	public class SecurityInterceptor extends HandlerInterceptorAdapter {
 		@Override
    	public boolean preHandle(HttpServletRequest request, 
    		HttpServletResponse response, 
    		Object handler) throws Exception {
 			// 控制访问
        	if (handler instanceof HandlerMethod) {
            	//获取请求的最佳匹配路径
            	String pattern = (String)request.getAttribute(
            		HandlerMapping.BEST_MATCHING_PATTERN_ATTRIBUTE);
            	//获取@PathVariable的值
            	Object attribute = request.getAttribute(
            		HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE);
	
				//权限校验。
				//此处略去20行代码
        		if (match) {
	  				return true;
				}
        		response.sendError(HttpStatus.FORBIDDEN.value());
				return false；	
   			}
	}


	1.我们设计一个安全的拦截器继承HandlerInterceptorAdapter做一个切面
	2.override preHandler方法，
	3.通过在request中得到现在访问的url会最佳匹配到得controller方法中requestmapping配置的路径
	如果用户访问的/user/1, pattern就会返回 /usr/{id} 这样就达成了我们想要的结果
	我们不直接保存用户访问的url在权限表中，而是保存请求url最后映射的controller方法的 requestmapping配置的url。

如果想得做log记录到访问的http参数，可以分两部分获取：

	1.//获取@PathVariable的值
	Object attribute = request.getAttribute(HandlerMapping.URI_TEMPLATE_VARIABLES_ATTRIBUTE);

	2.各种getParameter部分：request.getParameterMap() //直接获取所有参数，返回Map



感想:

	此处掌声10s!
	大赞SpringMVC!
	绝对业界良心啊。架构上得灵活性，给予我等码农最大的灵活性，话说在Web端我最爱的两个框架，SpringMVC，Stripes。
	纳尼？你不正在学习Clojure。。，嘘。。。。！



页面安全url校验的tag：
此处一坑是，如果我们直接继承TagSupport去实现我们的功能时，发现惨了，我们的Service不能被Spring autoWeired了。
So我们还是乖乖使用Spring封装的这个RequestContextAwareTag吧。


	public class SecurityUrlTag extends RequestContextAwareTag {
    	private String url;

    /**
     * 权限判断
     * 
     * hufeng(of730)
     * @return
     * @throws JspException
     */
    @Override
    @SuppressWarnings("unchecked")
    public int doStartTagInternal() throws JspTagException {
        WebApplicationContext ctx = getRequestContext().getWebApplicationContext();
        EmployeeService employeeService = (EmployeeService) ctx.getBean("employeeService");
        DeptRightsService deptRightsService = (DeptRightsService) ctx.getBean("deptRightsService");


        String empNo = (String) pageContext.getAttribute("empNo", PageContext.SESSION_SCOPE);


        if (StringUtils.isNotEmpty(empNo)) {

            Dept dept = employeeService.getDept(empNo);

            if (dept != null) {
                List<String> allRights = deptRightsService.findAllRightsUrlsByDeptId(dept.getId());
                if (allRights.contains(url)) {
                    return EVAL_BODY_INCLUDE;
                }
            }
        }

        return SKIP_BODY;
    }

    //~~~~~~~~~~~~~setter&&getter~~~~~~~~~~~~~~
    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

}

在WEB-INFO下面：SecurityUrl.tld,你懂的！
<?xml version="1.0" encoding="UTF-8" ?>
<taglib
        xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
        version="2.1">

    <tlib-version>1.0</tlib-version>
    <short-name>pc</short-name><!-- 标签库名称 -->
    <uri>http://xxx/tags/pc</uri><!-- 导入标签库时会用到 -->

    <!-- 定义标签 -->
    <tag>
        <name>securityUrl</name> <!-- 标签名称 -->
        <tag-class>xxx.SecurityUrlTag</tag-class> <!-- 对应类 -->
        <body-content>scriptless</body-content>
        <attribute>
            <name>url</name>
            <required>true</required>
        </attribute>
    </tag>
</taglib>


jsp页面导入自定义标签:).

	<%--自定义标签--%>
	<%@taglib prefix="pc" uri="http://xx/tags/pc" %>
	<pc:securityUrl url="/product/deleteProductById">
		<a href="javascript:del('{{id}}');" class="btn btn-default toggle-detail">
                <i class="icon icon-trash-o"></i>
                删除
        </a>
    </pc:securityUrl>