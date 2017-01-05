---
layout: post
title: Mybatis CRUD (一)
category: 技术
tags: Reids
keywords: 
description: 
---



### CURD
CURD就是增删改查的缩写，如果不对增删改查进行封装难免会多写一些重复的code，不仅会影响工作效率，也会使人在编码时感到很无味。之前在武汉写的项目中没有用到持久层的框架，都是用原生的SQL语句来完成持久层的操作，用过的都知道在处理实体类，连接关闭数据库等操作大部分感觉都是些无意义但是非写不可的代码片（虽然现在觉得也可以优化去简化操作，但是现阶段一些持久层框架比其有太多的可取之处）。上个项目中用的hibernate架构人员将CURD的封装做的非常好，以至于只有用到一些复杂的报表才需要去写SQL，调用也很方便。目前正在写的项目是用的mybatis，架构人员在持久层这块做的很简洁，每建一个有持久层操作的实体类就要配置其dao类接口和xml，然后每个xml中都要拼写很多增删改查的SQL语句。这样做为了保证其结构的清晰和业务的直观性我认为是有必要的，但是作为一个系统架构我觉得弊大于利。结合之前对CURD封装的认识和在谷歌上一些mybaits用例的了解做了一个小demo。
#### 主要思路
根据mybaits的持久层操作在XML中，简化xml代码量就只能动态生成SQL当成变量传入SQL中，在对一个实体类的操作时我们用反射可以很轻松的取到对象的当前状态，在实体类变量的定义中添加一些注解可以在拼接SQL时排除非数据库字段和转换成注解的数据库字段。简单来说就是在公共的Service通过反射得到变量名和值动态的拼接SQL传入公共的Dao调用公共的XML完成处理数据库的操作。
#### BaseMapper.xml
公共xml ， 减少了大量满足此增删改查的XML编写。

``` xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper  
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.framework.base.BaseDao">
	<select id="getCode" resultType="String">
	${sql}
	</select>
	<insert id="baseSave">
	${sql}
	</insert>
	<delete id="baseParDel">
	${sql}
	</delete>
	<delete id="baseLogParDel">
	${sql}
	</delete> 
	<update id="baseUpdate">
	${sql}
	</update>
	<select id="baseParSelect" resultType="hashmap">
	${sql}
	</select>
	<select id="baseSelectEq" resultType="hashmap">
	${sql}
	</select>
	<select id="baseSelectLike" resultType="hashmap">
	${sql}
	</select>
	<select id="baseGetCode" resultType="String">
	${sql}
	</select>
	<select id="baseSelectAll" resultType="hashmap">
	${sql}
	</select>
</mapper>

```

####BaseDao
BaseDao接口， 减少了大量满足此增删改查的Dao接口编写。

``` java

package com.framework.base;
import java.util.List;
import java.util.Map;
import org.apache.ibatis.annotations.Param;

/**
 * MYBATIS CRUD BaseDao
 * @author yyf
 *
 * @param <T>
 */
public interface BaseDao<T> {
	public boolean baseSave(@Param("sql") String sql);
	public boolean baseParDel(@Param("sql") String sql);
	public boolean baseLogParDel(@Param("sql") String sql);
	public boolean baseUpdate(@Param("sql") String sql);
	public Map<String, Object> baseParSelect(@Param("sql") String sql);
	public List<Map<String, Object>> baseSelectEq(@Param("sql") String sql);
	public List<Map<String, Object>> baseSelectLike(@Param("sql") String sql);
	public String baseGetCode(@Param("sql") String sql);
	public List<Map<String,Object>> baseSelectAll(@Param("sql") String sql);
}


```
####BaseService
BaseService  将CURD方法进行封装，现阶段都是直接传入的实体类的方法，后续会添加一些更加方便的封装操作。

``` java

package com.framework.base;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.framework.core.domain.Core;
import com.framework.util.NumberFormat;
/**
 * MYBATIS CURD BaseService 
 * 一些增删改查的封装
 * @author yyf
 * 
 * @param <T>
 * @param <D>
 */
@Service
public class BaseService<T extends BaseEntity, D extends BaseDao<T>> {

	@Autowired
	public D dao;

	/**
	 * 保存
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public boolean baseSave(T t) throws Exception {
		return dao.baseSave(BaseSQL.getInstance().baseSave(t));
	}

	/**
	 * 保存 用于保存List bean
	 * 
	 * @param ls
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public boolean baseSave(List<T> ls) throws Exception {
		Boolean bool = false;
		for (T t : ls) {
			bool = dao.baseSave(BaseSQL.getInstance().baseSave(t));
			if (bool == false)
				return bool;
		}
		return bool;
	}

	/**
	 * 删除 根据实体类的Primary值或者ID 实体类中有一个唯一值即可（Primary 或 ID） 注：物理删除
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public boolean baseParDel(T t) throws Exception {
		return dao.baseParDel(BaseSQL.getInstance().baseParDel(t));
	}

	/**
	 * 删除 根据实体类的Primary值或者ID 实体类中有一个唯一值即可（Primary 或 ID） 注：逻辑删除
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public boolean baseParLogicDel(T t) throws Exception {
		return dao.baseLogParDel(BaseSQL.getInstance().baseLogParDel(t));
	}

	/**
	 * 更新 根据实体类的Primary值或者ID（实体类顺序查找的第一个作为跟新依据） 实体类中有一个唯一值即可（Primary 或 ID）
	 * 注1：如果根据查到的第一个之后的唯一值来跟新前面的唯一值 此方法无效 注2:
	 * 由于实体类默认String为"",不支持更新""操作,如有必要可以更新" "
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public boolean baseUpdate(T t) throws Exception {
		return dao.baseUpdate(BaseSQL.getInstance().baseUpdate(t));
	}

	/**
	 * 查询 根据实体类的Primary值或者ID（实体类顺序查找的第一个作为跟新依据） 实体类中有一个唯一值即可（Primary 或 ID）
	 * 注：此sql返回值为一个对象
	 * 
	 * @param t
	 * @return t
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public T baseParSelect(T t) throws Exception {
		Map map = dao.baseParSelect(BaseSQL.getInstance().baseParSelect(t));
		t = (T) Core.M2P(map, t);
		return t;
	}

	/**
	 * 数据库查询操作 根据实体类的值进行绝对查询 注：无时间大小比较
	 * 
	 * @param t
	 * @return list<t>
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public List<T> baseSelectEq(T t) throws Exception {
		List<Map<String, Object>> maps = dao.baseSelectEq(BaseSQL.getInstance()
				.baseSelectEq(t));
		List<Object> os = Core.ML2PL(maps, t);
		List<T> ts = new ArrayList<T>();
		for (int i = 0; i < os.size(); i++) {
			ts.add((T) os.get(i));
		}
		return ts;
	}

	/**
	 * 数据库查询操作 查询实体类数据库所有的信息 注：有flag为0限制
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public List<T> baseSelectAll(T t) throws Exception {
		List<Map<String, Object>> maps = dao.baseSelectAll(BaseSQL
				.getInstance().baseSelectAll(t));
		List<Object> os = Core.ML2PL(maps, t);
		List<T> ts = new ArrayList<T>();
		for (int i = 0; i < os.size(); i++) {
			ts.add((T) os.get(i));
		}
		return ts;
	}

	/**
	 * 数据库查询操作 根据实体类的值进行模糊查询 注：无时间大小比较
	 * 
	 * @param t
	 * @return list<t>
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public List<T> baseSelectLike(T t) throws Exception {
		List<Map<String, Object>> maps = dao.baseSelectLike(BaseSQL
				.getInstance().baseSelectLike(t));
		List<Object> os = Core.ML2PL(maps, t);
		List<T> ts = new ArrayList<T>();
		for (int i = 0; i < os.size(); i++) {
			ts.add((T) os.get(i));
		}
		return ts;
	}

	/**
	 * 通过注解获取Code操作 注：人为定义code格式 注：实体T数据库code字段必须要有@iscode注解 此注解的value值为Code头部
	 * 注：实体T数据库必须有@Tabel注解
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	@SuppressWarnings("unchecked")
	public String baseGetCode(T t) throws Exception {
		Map<String, String> m = BaseSQL.getInstance().baseGetCode(t);
		String code = dao.baseGetCode(m.get("sql"));
		if ("".equals(code) || code == null)
			code = m.get("date") + "0001";
		else {
			code = "1" + code.substring(m.get("type").length());
			double d = Double.parseDouble(code) + 1;
			code = m.get("type")
					+ NumberFormat.formatNum(String.valueOf(d), 0, false)
							.substring(1);
		}
		return code;
	}

}


```

####BaseSQL
BaseSQL 核心类，将SQL进行拼接

```java

package com.framework.base;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import com.framework.core.domain.Po;
import com.framework.core.domain.Pram;
import com.framework.core.domain.SqlCore;
import com.framework.util.DateUtils;

/**
 * CURD SQL拼接操作类
 * 
 * @author yyf
 *
 * @param <T>
 */
public class BaseSQL<T extends Po> {

	private SqlCore<?> sc;

	private BaseSQL() {
		this.sc = SqlCore.getInstance();
	}

	private static BaseSQL<?> instance = null;

	@SuppressWarnings("unchecked")
	public static synchronized BaseSQL getInstance() {
		if (instance == null)
			instance = new BaseSQL();
		return instance;
	}

	/**
	 * 数据库新增操作
	 * 
	 * @param po
	 * @return
	 * @throws Exception
	 */
	public String baseSave(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		String sql = "insert into " + tableName + " (";
		String prams = "";
		String values = "";
		List<Pram> pramList = this.sc.getPramList(t);
		for (int i = 0; i < pramList.size(); i++) {
			prams += pramList.get(i).getFile();
			if (pramList.get(i).getValue() == null) {
				values += "null";
			} else {
				values += "'" + pramList.get(i).getValue() + "'";
			}
			if (i < pramList.size() - 1) {
				prams += ",";
				values += ",";
			}
		}
		sql += prams + ") value (" + values + ");";
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库删除操作 根据实体类的Primary值或者ID 实体类中有一个唯一值即可（Primary 或 ID） 注：物理删除
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseParDel(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		Pram pram = this.sc.getPrimaryName(t);
		if (pram == null) {
			throw new Exception("实体类无查询条件数据！");
		}
		String sql = "delete from " + tableName + " where " + pram.getFile()
				+ "='" + pram.getValue() + "'";
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库删除操作 根据实体类的Primary值或者ID 实体类中有一个唯一值即可（Primary 或 ID） 注：逻辑删除
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseLogParDel(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		Pram pram = this.sc.getPrimaryName(t);
		if (pram == null) {
			throw new Exception("实体类无查询条件数据！");
		}
		String sql = "update " + tableName + " set flag ='1' where "
				+ pram.getFile() + "='" + pram.getValue() + "'";
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库跟新操作 根据实体类的Primary值或者ID（实体类顺序查找的第一个作为跟新依据） 实体类中有一个唯一值即可（Primary 或 ID）
	 * 注1：如果根据查到的第一个之后的唯一值来跟新前面的唯一值 此方法无效 注2:
	 * 由于实体类默认String为"",不支持更新""操作,如有必要可以更新" "
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseUpdate(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		Pram pram = this.sc.getPrimaryName(t);
		if (pram == null) {
			throw new Exception("实体类无查询条件数据！");
		}
		List<Pram> pramList = this.sc.getPramList(t);
		String s = "";
		for (int i = 0; i < pramList.size(); i++) {
			if (pramList.get(i).getValue() != null
					&& !"".equals(pramList.get(i).getValue().toString().trim())) {
				s += pramList.get(i).getFile() + "= '"
						+ pramList.get(i).getValue() + "'";
				if (i < pramList.size() - 1) {
					s += ",";

				}
			}
		}
		if ("".equals(s)) {
			throw new Exception("实体类无更新数据！");
		}
		if (s.endsWith(",")) {
			s = s.substring(0, s.length() - 1);
		}
		String sql = "update " + tableName + " set " + s + " where "
				+ pram.getFile() + "='" + pram.getValue() + "'";
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库查询操作 根据实体类的Primary值或者ID（实体类顺序查找的第一个作为跟新依据） 实体类中有一个唯一值即可（Primary 或 ID）
	 * 注：此sql返回值为一个对象
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseParSelect(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		Pram pram = this.sc.getPrimaryName(t);
		if (pram == null) {
			throw new Exception("实体类无查询条件数据！");
		}
		String sql = "select * from " + tableName + " where " + pram.getFile()
				+ "='" + pram.getValue() + "'";
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库查询操作 查询实体类数据库所有的信息
	 * 注：有flag为0限制
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseSelectAll(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		String sql = "select * from " + tableName + " where flag ='0'" ;
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库查询操作 根据实体类的值进行绝对查询 注：无时间大小比较
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseSelectEq(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		List<Pram> pramList = this.sc.getAllPramList(t);
		String sql = "select * from " + tableName + " where 1=1 ";
		for (int i = 0; i < pramList.size(); i++) {
			if ("id".equals(pramList.get(i).getFile().toLowerCase())) {
				if (!"0".equals(pramList.get(i).getValue().toString()))
					sql += "and " + pramList.get(i).getFile() + " = '"
							+ pramList.get(i).getValue() + "' ";
			} else {
				if (pramList.get(i).getValue() != null
						&& !"".equals(pramList.get(i).getValue().toString()
								.trim()))
					sql += "and " + pramList.get(i).getFile() + " = '"
							+ pramList.get(i).getValue() + "' ";
			}
		}
		System.out.println(sql);
		return sql;
	}

	/**
	 * 数据库查询操作 根据实体类的值进行模糊查询 注：无时间大小比较
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public String baseSelectLike(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		List<Pram> pramList = this.sc.getAllPramList(t);
		String sql = "select * from " + tableName + " where 1=1 ";
		for (int i = 0; i < pramList.size(); i++) {
			if ("id".equals(pramList.get(i).getFile().toLowerCase())) {
				if (!"0".equals(pramList.get(i).getValue().toString()))
					sql += "and " + pramList.get(i).getFile() + " = '"
							+ pramList.get(i).getValue() + "' ";
			} else {
				if (pramList.get(i).getValue() != null
						&& !"".equals(pramList.get(i).getValue().toString()
								.trim()))
					sql += "and " + pramList.get(i).getFile() + " like '%"
							+ pramList.get(i).getValue() + "%' ";
			}
		}
		System.out.println(sql);
		return sql;
	}

	/**
	 * 用于获取code操作 进行了数据库比较操作 注：人为定义code格式 注：实体T数据库code字段必须要有@iscode注解
	 * 此注解的value值为Code头部 注：实体T数据库必须有@Tabel注解
	 * 
	 * @param t
	 * @return
	 * @throws Exception
	 */
	public Map<String, String> baseGetCode(T t) throws Exception {
		String tableName = this.sc.getTableName(t);
		Map<String, String> cm = this.sc.getCodeMess(t);
		Map<String, String> rm = new HashMap<String, String>();
		String date = DateUtils.getStringDateShort().replaceAll("-", "")
				.substring(2, 6);
		date = cm.get("type") + date;
		String sql = "select max(" + cm.get("name") + ") from " + tableName
				+ " where " + cm.get("name") + " like '" + date + "%'";
		rm.put("sql", sql);
		rm.put("date", date);
		rm.put("type", cm.get("type"));
		return rm;
	}

}

```
#### Bean Controller  Service 
可以很直观的看出对一个实体类进行增删改查的简洁性。

``` java
package com.framework.crud.bean.user;
import com.framework.base.BaseEntity;
import com.framework.core.annotation.Code;
import com.framework.core.annotation.Primary;
import com.framework.core.annotation.TableName;
import com.framework.core.annotation.TempField;
/**
 * @author yyf
 * 
 * 用户Bean
 * Parimary 定义的有唯一性字段
 * Code 编码字段 一般是系统在插入时生成 如 U17010001（编码头 年 月 计数字段）
 * TempField 非数据库字段（在CRUD中会将其忽略）
 * @author yyf
 * 
 */

@TableName(name = "user")
public class User extends BaseEntity {
	private int id;
	
	/**
	 * 用户名 /唯一
	 */
	@Primary
	private String name;
	/**
	 * 用户编码 /唯一/U开头
	 */
	@Code(name = "U")
	@Primary
	private String code;
	/**
	 * 用户年龄/非数据库字段
	 */
	@TempField
	private String age;
	/**
	 * 用户联系方式
	 */
	private String phone;
	/**
	 * 用户备注
	 */
	private String remark;

	public void setUser(String name, String age, String phone, String remark) {
		this.name = name;
		this.age = age;
		this.phone = phone;
		this.remark = remark;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getCode() {
		return code;
	}

	public void setCode(String code) {
		this.code = code;
	}

	public String getAge() {
		return age;
	}

	public void setAge(String age) {
		this.age = age;
	}

	public String getPhone() {
		return phone;
	}

	public void setPhone(String phone) {
		this.phone = phone;
	}

	public String getRemark() {
		return remark;
	}

	public void setRemark(String remark) {
		this.remark = remark;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	@Override
	public String toString() {
		return "User [age=" + age + ", code=" + code + ", id=" + id + ", name="
				+ name + ", phone=" + phone + ", remark=" + remark + "]";
	}

}

package com.framework.crud.controller.user;

import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import com.framework.base.BaseController;
import com.framework.crud.bean.user.User;
import com.framework.crud.service.user.UserService;

/**
 * MYBAITS CRUD CONTROLLER
 * 
 * 在实体类USER上的简单的增删改查
 * 
 * 为了简洁（其实是懒）,无页面
 * 
 * 为了方便直接看出效果（其实是懒）,将实体类直接在controller中生成
 * 
 * @author yyf
 *
 */
@Controller
@RequestMapping(value = "/user")
public class UserController extends BaseController<User, UserService> {
	@Autowired
	private UserService userService;

	/**
	 * 增加操作
	 * @return
	 */
	@RequestMapping(value = "/add", method = { RequestMethod.GET })
	public String add() {
		try {
			User user = new User();
			user.setUser("name", "age", "phone", "remark");
			userService.addUser(user);
		} catch (Exception e) {
			//model 异常抛出操作
		}
		return "redirect:/user/get.do";
	}

	/**
	 * 删除操作
	 * @return
	 */
	@RequestMapping(value = "/del", method = { RequestMethod.GET })
	public String del() {
		try {
			User user = new User();
			user.setName("name"); 
			userService.delUser(user);
		} catch (Exception e) {
			//model 异常抛出操作
		}
		return "redirect:/user/get.do";
	}

	/**
	 * 修改操作
	 * @return
	 */
	@RequestMapping(value = "/update", method = { RequestMethod.GET })
	public String update() {
		try {
			User user = new User();
			user.setUser("name", "age2", "phone2", "remark2");
			user.setCode("U17010001"); //code在此bean中有些特殊
			userService.updateUser(user);
		} catch (Exception e) {
			//model 异常抛出操作
		}
		return "redirect:/user/get.do";
	}

	/**
	 * 查询操作
	 * 
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "/get", method = { RequestMethod.GET })
	public @ResponseBody String show() {
		List<User> users = null;
		try {
			users = userService.getUser();
		} catch (Exception e) {
			//model 异常抛出操作
		}
		return users.toString();
	}

}

package com.framework.crud.service.user;
import java.util.List;
import org.springframework.stereotype.Service;
import com.framework.base.BaseService;
import com.framework.crud.bean.user.User;
import com.framework.crud.dao.user.UserDao;

/**
 * MYBAITS CRUD SERVICE
 * 
 * 结合controller中的四个基本增删改查
 * 
 * 更多操作在BaseService
 * 
 * @author yyf
 *
 */
@Service
public class UserService extends BaseService<User, UserDao>{
	
	/**
	 * 增加
	 * baseGetCode (编号)
	 * @param user
	 * @throws Exception
	 */
	public void addUser(User user) throws Exception {
		user.setCode(baseGetCode(user));
		baseSave(user);
	}

	/**
	 * 删除（此处是物理删除）
	 * @param user
	 * @throws Exception
	 */
	public void delUser(User user) throws Exception {
		baseParDel(user);
		
	}

	/**
	 * 修改 
	 * @param user
	 * @throws Exception
	 */
	public void updateUser(User user) throws Exception {
		baseUpdate(user);
		
	}
	
	/**
	 * 查找（此处是查找全部）
	 * @return
	 * @throws Exception
	 */
	public List<User> getUser() throws Exception {
		return baseSelectAll(new User());
	}

}


```


> **完整项目地址：**[GitHub][1]。

[1]:https://github.com/yeyafei/YYF-JAVA/tree/master/crud





