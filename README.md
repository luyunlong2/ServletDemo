# ServletDemo
Servlet的初步封装
import java.io.IOException;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.jxnd.bean.Users;
import org.jxnd.biz.IUsersBiz;
import org.jxnd.biz.impl.RolesBiz;
import org.jxnd.biz.impl.UsersBiz;
import org.jxnd.tools.Tools;


@WebServlet("/UsersServlet")
@SuppressWarnings("all")
public class UsersServlet extends HttpServlet {
	IUsersBiz biz=new UsersBiz();
	Class c=UsersServlet.class;
	
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		String type=request.getParameter("type");
		for (Method mt: c.getDeclaredMethods()) {
			if (mt.getName().equalsIgnoreCase(type)) {
				try {
					mt.invoke(this, request,response);
				} catch (IllegalAccessException e) {
					e.printStackTrace();
				} catch (IllegalArgumentException e) {
					e.printStackTrace();
				} catch (InvocationTargetException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	private void updateUsersById(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		int id=Integer.parseInt(request.getParameter("roleid"));
		request.setAttribute("info", biz.getUsersById(id));
		request.setAttribute("list", new RolesBiz().getRolesAll());
		request.getRequestDispatcher("update.jsp").forward(request, response);
	
	}

	private void doUpdate(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		Users info=new Users();
		
		String name=request.getParameter("name");
		String pass=request.getParameter("pass");
		int roleid=Integer.parseInt(request.getParameter("rid"));
		
		
		info.setName(name);
		info.setPass(pass);
		info.setRoleid(roleid);
		
		if(biz.updateUsers(info)<1)
			request.setAttribute("error", "修改失败,请联系管理员");
		//request.getRequestDispatcher("GetUsersPage").forward(request, response);
		getUsersPage(request, response);
	}

	private void updatePwd(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		Users info=(Users) request.getSession().getAttribute("info");
		String name=info.getName();
		String op=request.getParameter("op");
		String np=request.getParameter("np1");
		String msg="";
		int i=biz.updatePwd(name, op, np);
		if (i==3){
			info.setPass(np);
			request.getSession().setAttribute("info", info);
			response.sendRedirect("UsersServlet?type=getUsersPage&pageIndex=1");
			return;
		}if (i==1)
			msg="原始密码错误";
		if(i==2)
			msg="服务器错误";
		request.setAttribute("errormsg", msg);
		request.getRequestDispatcher("updatepwd.jsp").forward(request, response);
	
	}

	private void getUsersPage(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		int pageIndex=Integer.parseInt(request.getParameter("pageIndex"));
		int pageSize=Integer.parseInt(request.getParameter("pageSize")==null?"10":request.getParameter("pageSize"));
		int count=biz.getUsersCount();
		List<Users> list=biz.getUsersPage(pageIndex, pageSize);
		request.setAttribute("list", list);
		request.setAttribute("pageIndex", pageIndex);
		request.setAttribute("pageSize", pageSize);
		request.setAttribute("count",count);
		request.setAttribute("pageMsg", Tools.paging(count, pageSize, pageIndex, "UsersServlet?type=getUsersPage"));
		request.getRequestDispatcher("manager.jsp").forward(request, response);
	
	}

	private void deleteUsersById(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		int id=Integer.parseInt(request.getParameter("roleid"));
		if (biz.deleteUsersById(id)>0) 
			//request.getRequestDispatcher("GetUsersPage").forward(request, response);
			getUsersPage(request, response);
		else{
			request.setAttribute("msg", "删除失败,请联系管理员!");
			getUsersPage(request, response);
			//request.getRequestDispatcher("GetUsersAll").forward(request, response);
		}		
	}

	public void login(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
		String name=request.getParameter("name");
		String pass=request.getParameter("pass");
		
		//基本验证
		if (name!=null&&!"".equals(name)&&pass!=null&&!"".equals(pass)) {
			//内容验证 (过滤特殊字符,脏数据) 此处省略部分代码
			Users info= biz.login(name, pass);
			if (info!=null) {
				request.getSession().setAttribute("info", info);
				response.sendRedirect("UsersServlet?type=GetUsersPage&pageIndex=1&pageSize=10");
			}else {
				request.setAttribute("msg", "用户名或者密码错误!");
				request.getRequestDispatcher("login.jsp").forward(request, response);
			}
		}
	}
	
	
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		doGet(request, response);
	}
}
