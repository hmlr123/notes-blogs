---
title: Mybatis基类生成
date: 2019-03-21 16:45:18
update: 2019-03-21
tags: Mybatis
---

Mybatis基类生成，减少开发代码量

<!--more-->

这里只是记录个人总结，详细请移步<a href="http://bbs.gupaoedu.com/forum.php?mod=viewthread&tid=62">这里</a>



今天跟着视频写淘淘商城的时候，发现有部分基类Mapper 基类Service 没有视频讲解，网上查了一下，有些了解，还需深入理解

BaseMapper

```java
import java.util.List;

import org.apache.ibatis.annotations.Param;

public interface BaseMapper<T,Example,ID> {
	
	/**
	 * 条件查询数量
	 * @param example
	 * @return
	 */
	int countByExample(Example example);
	
	/**
	 * 条件删除
	 * @param example
	 * @return
	 */
	int deleteByExample(Example example);
	 
	/**
	 * 主键删除
	 * @param id
	 * @return
	 */
    int deleteByPrimaryKey(ID id);
 
    /**
     * insert selectiveinsert区别
     * selective 选择性
     * selectiveInsert选择性保留数据
     * insertSelective执行时只插入对应设置的字段，（主键是自动添加，默认为空）
     * insert 不管你设置多少字段，全部插入个便
     */
    
    /**
     * 插入
     * @param record
     * @return
     */
    int insert(T record);
 
    /**
     * 插入
     * @param record
     * @return
     */
    int insertSelective(T record);
 
    /**
     * 条件查询数据
     * @param example
     * @return
     */
    List<T> selectByExample(Example example);
 
    /**
     * 依靠逐渐查询
     * @param id
     * @return
     */
    T selectByPrimaryKey(ID id);
 
    /**
     * 条件修改 设置多少字段，修改多少字段
     * @param record
     * @param example
     * @return
     */
    int updateByExampleSelective(@Param("record") T record, @Param("example") Example example);
 
    /**
     * 同上，全部修改
     * @param record
     * @param example
     * @return
     */
    int updateByExample(@Param("record") T record, @Param("example") Example example);
 
    /**
     * 依靠主键修改
     * @param record
     * @return
     */
    int updateByPrimaryKeySelective(T record);
 
    /**
     * 依靠主键修改
     * @param record
     * @return
     */
    int updateByPrimaryKey(T record);
}
```

知识点：

1. 泛型是可以多种的，只要表示不同即可

2. <a href="https://blog.csdn.net/hello_word2/article/details/80560725">insert 和selectiveInsert区别</a>

   > 两者的区别在于如果选择insert 那么所有的字段都会添加一遍即使没有值
   >
   > 但是如果使用inserSelective就会只给有值的字段赋值（会对传进来的值做非空判断）

   

BaseExample

这里可以添加扩展功能，比如排序、分页等。。。

```java
/**
 * 基类
 * 修改扩展功能
 * @author liwei
 *
 */
public abstract class BaseExample {
	//protected PageInfo pageInfo;
	
}
```



BaseService

命名规则和BaseMapper一致

```java
import java.util.List;

import com.taotao.pojo.BaseExample;

import org.apache.ibatis.annotations.Param;

public interface BaseService<T, Example extends BaseExample, ID> {

	/**
	 * 条件查询数量
	 * @param example
	 * @return
	 */
	int countByExample(Example example);
	
	/**
	 * 条件删除
	 * @param example
	 * @return
	 */
	int deleteByExample(Example example);
	 
	/**
	 * 主键删除
	 * @param id
	 * @return
	 */
    int deleteByPrimaryKey(ID id);
 
    /**
     * insert selectiveinsert区别
     * selective 选择性
     * selectiveInsert选择性保留数据
     * insertSelective执行时只插入对应设置的字段，（主键是自动添加，默认为空）
     * insert 不管你设置多少字段，全部插入个便
     */
    
    /**
     * 插入
     * @param record
     * @return
     */
    int insert(T record);
 
    /**
     * 插入
     * @param record
     * @return
     */
    int insertSelective(T record);
 
    /**
     * 条件查询数据
     * @param example
     * @return
     */
    List<T> selectByExample(Example example);
 
    /**
     * 依靠逐渐查询
     * @param id
     * @return
     */
    T selectByPrimaryKey(ID id);
 
    /**
     * 条件修改 设置多少字段，修改多少字段
     * @param record
     * @param example
     * @return
     */
    int updateByExampleSelective(@Param("record") T record, @Param("example") Example example);
 
    /**
     * 同上，全部修改
     * @param record
     * @param example
     * @return
     */
    int updateByExample(@Param("record") T record, @Param("example") Example example);
 
    /**
     * 依靠主键修改
     * @param record
     * @return
     */
    int updateByPrimaryKeySelective(T record);
 
    /**
     * 依靠主键修改
     * @param record
     * @return
     */
    int updateByPrimaryKey(T record);
}
```



BaseServiceImpl

```java
import java.util.List;

import com.taotao.mapper.BaseMapper;
import com.taotao.pojo.BaseExample;
import com.taotao.service.BaseService;

public class BaseServiceImpl<T, Example extends BaseExample,ID> implements BaseService<T, Example, ID> {

	private BaseMapper<T,Example,ID> mapper;

	/**
	 * 为什么不用注入的方式？
	 * @param mapper
	 */
	public void setMapper(BaseMapper<T, Example, ID> mapper) {
		this.mapper = mapper;
	}
	

	@Override
	public int countByExample(Example example) {
		return mapper.countByExample(example);
	}

	@Override
	public int deleteByExample(Example example) {
		return mapper.deleteByExample(example);
	}

	@Override
	public int deleteByPrimaryKey(ID id) {
		return mapper.deleteByPrimaryKey(id);
	}

	@Override
	public int insert(T record) {
		return mapper.insert(record);
	}

	@Override
	public int insertSelective(T record) {
		return mapper.insertSelective(record);
	}

	@Override
	public List<T> selectByExample(Example example) {
		return mapper.selectByExample(example);
	}

	@Override
	public T selectByPrimaryKey(ID id) {
		return mapper.selectByPrimaryKey(id);
	}

	@Override
	public int updateByExampleSelective(T record, Example example) {
		return mapper.updateByExampleSelective(record, example);
	}

	@Override
	public int updateByExample(T record, Example example) {
		return mapper.updateByExample(record, example);
	}

	@Override
	public int updateByPrimaryKeySelective(T record) {
		return mapper.updateByPrimaryKeySelective(record);
	}

	@Override
	public int updateByPrimaryKey(T record) {
		return mapper.updateByPrimaryKey(record);
	}
}
```



后面的对官方的generator扩展部分不懂，先摘下来



---



​	    现在,可以开始对官方generator进行扩展,并对对应的mapper.xml进行相应变动.生成期望中的代码.

   	 generator本身预留了插件扩展功能.所以主要通过继承PluginAdapter来完成我们想要的东西.

​            另外,generator本身的生成注释不是很符合我们的习惯.我们期望的是,可通过db表和字段信息来生成对应的注释,在这里进行扩展生成我们想要的注释信息.



CommentGenerator 不包含实体类的注释描述生成.

```java
/**
 * Comment Generator
 * @ClassName CommentGenerator 
 * @Description 
 * @author Marvis
 */
public class CommentGenerator extends DefaultCommentGenerator {
        private Properties properties;
        private boolean suppressDate;
        private boolean suppressAllComments;
 
        public CommentGenerator() {
                this.properties = new Properties();
                this.suppressDate = false;
                this.suppressAllComments = false;
        }
 
        public void addJavaFileComment(CompilationUnit compilationUnit) {
                 
                compilationUnit.addFileCommentLine("/*** copyright (c) 2017 Marvis  ***/");
        }
        /**
         * XML file Comment
         */
        public void addComment(XmlElement xmlElement) {
                if (this.suppressAllComments) {
                        return;
                }
 
        }
 
        public void addRootComment(XmlElement rootElement) {
        }
 
        public void addConfigurationProperties(Properties properties) {
                this.properties.putAll(properties);
 
                this.suppressDate = StringUtility.isTrue(properties.getProperty("suppressDate"));
 
                this.suppressAllComments = StringUtility.isTrue(properties.getProperty("suppressAllComments"));
        }
 
        protected void addJavadocTag(JavaElement javaElement, boolean markAsDoNotDelete) {
                StringBuilder sb = new StringBuilder();
                sb.append(" * ");
                sb.append("@date");
//                if (markAsDoNotDelete) {
//                        sb.append(" do_not_delete_during_merge");
//                }
                String s = getDateString();
                if (s != null) {
                        sb.append(' ');
                        sb.append(s);
                }
                javaElement.addJavaDocLine(sb.toString());
        }
 
        protected String getDateString() {
                if (this.suppressDate) {
                        return null;
                }
                return new Date().toString();
        }
        /** 
         *  Comment of Example inner class(GeneratedCriteria ,Criterion)
         */
        public void addClassComment(InnerClass innerClass, IntrospectedTable introspectedTable) {
                if (this.suppressAllComments) {
                        return;
                }
 
                innerClass.addJavaDocLine("/**");
                innerClass.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable().getDomainObjectName()+ "<p/>");
                innerClass.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable().toString());
                addJavadocTag(innerClass, false);
                innerClass.addJavaDocLine(" */");
        }
 
        public void addEnumComment(InnerEnum innerEnum, IntrospectedTable introspectedTable) {
                if (this.suppressAllComments) {
                        return;
                }
 
                StringBuilder sb = new StringBuilder();
 
                innerEnum.addJavaDocLine("/**");
                innerEnum.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable().getAlias()+ "<p/>");
                innerEnum.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable());
                innerEnum.addJavaDocLine(sb.toString());
 
                addJavadocTag(innerEnum, false);
 
                innerEnum.addJavaDocLine(" */");
        }
        /**
         * entity filed Comment
         */
        public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                        IntrospectedColumn introspectedColumn) {
                if (this.suppressAllComments) {
                        return;
                }
 
//                if(introspectedColumn.getRemarks() != null && !introspectedColumn.getRemarks().trim().equals(""))
                 
                field.addJavaDocLine("/**");
                field.addJavaDocLine(" * " + introspectedColumn.getRemarks());
                field.addJavaDocLine(" * @author Marvis" );
                field.addJavaDocLine(" * @date " + getDateString() );
                field.addJavaDocLine(" * @return");
                field.addJavaDocLine(" */");
        }
        /**
         *  Comment of EXample filed 
         */
        public void addFieldComment(Field field, IntrospectedTable introspectedTable) {
                if (this.suppressAllComments) {
                        return;
                }
 
                field.addJavaDocLine("/**");
                addJavadocTag(field, false);
                field.addJavaDocLine(" */");
        }
        /**
         * Comment of Example method
         */
        public void addGeneralMethodComment(Method method, IntrospectedTable introspectedTable) {
                if (this.suppressAllComments) {
                        return;
                }
 
 
//                method.addJavaDocLine("/**");
 
//                addJavadocTag(method, false);
 
//                method.addJavaDocLine(" */");
        }
         
        /**
         * 
         * entity Getter Comment
         */
        public void addGetterComment(Method method, IntrospectedTable introspectedTable,
                        IntrospectedColumn introspectedColumn) {
                if (this.suppressAllComments) {
                        return;
                }
 
                method.addJavaDocLine("/**");
 
                 
                method.addJavaDocLine(" * @return " + introspectedTable.getFullyQualifiedTable().getAlias() + " : " + introspectedColumn.getRemarks());
//                addJavadocTag(method, false);
                method.addJavaDocLine(" */");
        }
 
        public void addSetterComment(Method method, IntrospectedTable introspectedTable,
                        IntrospectedColumn introspectedColumn) {
                if (this.suppressAllComments) {
                        return;
                }
 
                StringBuilder sb = new StringBuilder();
 
                method.addJavaDocLine("/**");
 
                Parameter parm = (Parameter) method.getParameters().get(0);
                sb.append(" * @param ");
                sb.append(parm.getName());
                sb.append(" : ");
                sb.append(introspectedColumn.getRemarks());
                method.addJavaDocLine(sb.toString());
 
//                addJavadocTag(method, false);
 
                method.addJavaDocLine(" */");
        }
         
        /**
         * Comment of Example inner class(Criteria)
         */
        public void addClassComment(InnerClass innerClass, IntrospectedTable introspectedTable, boolean markAsDoNotDelete) {
                if (this.suppressAllComments) {
                        return;
                }
 
                innerClass.addJavaDocLine("/**");
                innerClass.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable().getAlias()+ "<p/>");
                innerClass.addJavaDocLine(" * " + introspectedTable.getFullyQualifiedTable().toString());
                addJavadocTag(innerClass, markAsDoNotDelete);
 
                innerClass.addJavaDocLine(" */");
        }
}
```



EntityCommentPlugin

```java
public class EntityCommentPlugin extends PluginAdapter {
         
         
         
        @Override
        public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
                addModelClassComment(topLevelClass, introspectedTable);
                return super.modelBaseRecordClassGenerated(topLevelClass, introspectedTable);
        }
 
        @Override
        public boolean modelRecordWithBLOBsClassGenerated(TopLevelClass topLevelClass,
                        IntrospectedTable introspectedTable) {
 
                addModelClassComment(topLevelClass, introspectedTable);
                return super.modelRecordWithBLOBsClassGenerated(topLevelClass, introspectedTable);
        }
 
        protected void addModelClassComment(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
 
                FullyQualifiedTable table = introspectedTable.getFullyQualifiedTable();
                String tableComment = getTableComment(table);
 
                topLevelClass.addJavaDocLine("/**");
                if(StringUtility.stringHasValue(tableComment))
                        topLevelClass.addJavaDocLine(" * " + tableComment + "<p/>");
                topLevelClass.addJavaDocLine(" * " + table.toString() + "<p/>");
                topLevelClass.addJavaDocLine(" * @date " + new Date().toString());
                topLevelClass.addJavaDocLine(" *");
                topLevelClass.addJavaDocLine(" */");
        }
 
        /**
         * @author Marvis
         * @date Jul 13, 2017 4:39:52 PM
         * @param table
         */
        private String getTableComment(FullyQualifiedTable table) {
                String tableComment = "";
                Connection connection = null;
                Statement statement = null;
                ResultSet rs = null;
                try {
                        connection = ConnectionFactory.getInstance().getConnection(context.getJdbcConnectionConfiguration());
                        statement = connection.createStatement();
                        rs = statement.executeQuery("SHOW CREATE TABLE " + table.getIntrospectedTableName());
 
                        if (rs != null && rs.next()) {
                                String createDDL = rs.getString(2);
                                int index = createDDL.indexOf("COMMENT='");
                                if (index < 0) {
                                        tableComment = "";
                                } else {
                                        tableComment = createDDL.substring(index + 9);
                                        tableComment = tableComment.substring(0, tableComment.length() - 1);
                                }
                        }
 
                } catch (SQLException e) {
 
                } finally {
                        closeConnection(connection, statement, rs);
                }
                return tableComment;
        }
         
        /**
         * 
         * @author Marvis
         * @date Jul 13, 2017 4:45:26 PM
         * @param connection
         * @param statement
         * @param rs
         */
        private void closeConnection(Connection connection, Statement statement, ResultSet rs) {
                try {
                        if (null != rs)
                                rs.close();
                } catch (SQLException e) {
 
                        e.printStackTrace();
                } finally {
                        try {
                                if (statement != null)
                                        statement.close();
                        } catch (Exception e) {
                                e.printStackTrace();
 
                        } finally {
 
                                try {
                                        if (connection != null)
                                                connection.close();
                                } catch (SQLException e) {
                                        e.printStackTrace();
                                }
                        }
                }
        }
        /**
         * This plugin is always valid - no properties are required
         */
        @Override
        public boolean validate(List<String> warnings) {
                return true;
        }
}
}
```



ClientDaoPlugin

```java
/**
 * javaClient("XMLMAPPER") extended 
 * @ClassName ClientDaoPlugin 
 * @Description 
 * @author Marvis
 */
public class ClientDaoPlugin extends EntityCommentPlugin {
 
        @Override
        public boolean clientGenerated(Interface interfaze, TopLevelClass topLevelClass,
                        IntrospectedTable introspectedTable) {
                 
                JavaTypeResolver javaTypeResolver = new JavaTypeResolverDefaultImpl();
                FullyQualifiedJavaType calculateJavaType = javaTypeResolver.calculateJavaType(introspectedTable.getPrimaryKeyColumns().get(0));
                 
                FullyQualifiedJavaType superInterfaceType = new FullyQualifiedJavaType("BaseMapper<"
                                + introspectedTable.getBaseRecordType() + ","
                                + introspectedTable.getExampleType() + ","
                                + calculateJavaType.getShortName()+ ">");
                 
                FullyQualifiedJavaType baseMapperInstance = FullyQualifiedJavaTypeProxyFactory.getBaseMapperInstance();
                 
                interfaze.addSuperInterface(superInterfaceType);
                interfaze.addImportedType(baseMapperInstance);
                 
//                interfaze.getAnnotations().clear();
                List<Method> methods = interfaze.getMethods();
                List<Method> changeMethods = new ArrayList<Method>();
                for (Method method : methods) {
                        if(method.getName().endsWith("BLOBs"))
                                changeMethods.add(method);
                }
                interfaze.getMethods().clear();
                for (Method changeMethod : changeMethods) {
                        interfaze.addMethod(changeMethod);
                }
                 
                return super.clientGenerated(interfaze, topLevelClass, introspectedTable);
        }
         
}
```



PaginationPlugin

```java
public class PaginationPlugin extends ClientDaoPlugin {
        @Override
        public boolean modelExampleClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
                // add field, getter, setter for limit clause
//                addLimit(topLevelClass, introspectedTable, "pageInfo");
//                addLimitString(topLevelClass, introspectedTable, "groupByClause");
                 
//                topLevelClass.addImportedType(FullyQualifiedJavaTypeProxy.getPageInfoInstanceInstance());
 
                FullyQualifiedJavaType baseExampleType = FullyQualifiedJavaTypeProxyFactory.getBaseExampleInstance();
                topLevelClass.setSuperClass(baseExampleType);
                 
                topLevelClass.addImportedType(baseExampleType);
                return super.modelExampleClassGenerated(topLevelClass, introspectedTable);
        }
         
        @Override
        public boolean sqlMapSelectByExampleWithBLOBsElementGenerated(XmlElement element,
                        IntrospectedTable introspectedTable) {
 
                XmlElement isNotNullElement1 = new XmlElement("if"); 
                isNotNullElement1.addAttribute(new Attribute("test", "groupByClause != null")); 
                isNotNullElement1.addElement(new TextElement("group by ${groupByClause}"));
                element.addElement(5, isNotNullElement1);
                XmlElement isNotNullElement = new XmlElement("if");
                isNotNullElement.addAttribute(new Attribute("test", "pageInfo != null")); 
                isNotNullElement.addElement(new TextElement("limit #{pageInfo.pageBegin} , #{pageInfo.pageSize}"));
                element.addElement(isNotNullElement);
 
                return super.sqlMapUpdateByExampleWithBLOBsElementGenerated(element, introspectedTable);
        }
 
        @Override
        public boolean sqlMapSelectByExampleWithoutBLOBsElementGenerated(XmlElement element,
                        IntrospectedTable introspectedTable) {
 
                // XmlElement isParameterPresenteElemen = (XmlElement) element
                // .getElements().get(element.getElements().size() - 1);
                XmlElement isNotNullElement1 = new XmlElement("if");
                isNotNullElement1.addAttribute(new Attribute("test", "groupByClause != null"));
                // isNotNullElement.addAttribute(new Attribute("compareValue", "0"));
                isNotNullElement1.addElement(new TextElement("group by ${groupByClause}"));
                // isParameterPresenteElemen.addElement(isNotNullElement);
                element.addElement(5, isNotNullElement1);
 
                // XmlElement isParameterPresenteElemen = (XmlElement) element
                // .getElements().get(element.getElements().size() - 1);
                XmlElement isNotNullElement = new XmlElement("if"); 
                isNotNullElement.addAttribute(new Attribute("test", "pageInfo != null"));
                // isNotNullElement.addAttribute(new Attribute("compareValue", "0"));
                isNotNullElement.addElement(new TextElement("limit #{pageInfo.pageBegin} , #{pageInfo.pageSize}"));
                // isParameterPresenteElemen.addElement(isNotNullElement);
                element.addElement(isNotNullElement);
 
                return super.sqlMapUpdateByExampleWithoutBLOBsElementGenerated(element, introspectedTable);
        }
 
        @Override
        public boolean sqlMapCountByExampleElementGenerated(XmlElement element, IntrospectedTable introspectedTable) {
 
                XmlElement answer = new XmlElement("select");
 
                String fqjt = introspectedTable.getExampleType();
 
                answer.addAttribute(new Attribute("id", introspectedTable.getCountByExampleStatementId()));
                answer.addAttribute(new Attribute("parameterType", fqjt));
                answer.addAttribute(new Attribute("resultType", "java.lang.Integer"));
 
                this.context.getCommentGenerator().addComment(answer);
 
                StringBuilder sb = new StringBuilder();
                sb.append("select count(1) from ");
                sb.append(introspectedTable.getAliasedFullyQualifiedTableNameAtRuntime());
 
                XmlElement ifElement = new XmlElement("if");
                ifElement.addAttribute(new Attribute("test", "_parameter != null"));
                XmlElement includeElement = new XmlElement("include");
                includeElement.addAttribute(new Attribute("refid", introspectedTable.getExampleWhereClauseId()));
                ifElement.addElement(includeElement);
 
                element.getElements().clear();
                element.getElements().add(new TextElement(sb.toString()));
                element.getElements().add(ifElement);
                return super.sqlMapUpdateByExampleWithoutBLOBsElementGenerated(element, introspectedTable);
        }
 
        @SuppressWarnings("unused")
        private void addLimit(TopLevelClass topLevelClass, IntrospectedTable introspectedTable, String name) {
                CommentGenerator commentGenerator = context.getCommentGenerator();
                FullyQualifiedJavaType pageInfoInstance = FullyQualifiedJavaTypeProxyFactory.getPageInfoInstanceInstance();
//                Field field = new Field();
//                field.setVisibility(JavaVisibility.PROTECTED);
                // field.setType(FullyQualifiedJavaType.getIntInstance());
                // PrimitiveTypeWrapper integerInstance =
                // PrimitiveTypeWrapper.getIntegerInstance();
//                field.setType(pageInfoInstance);
//                field.setName(name);
//                // field.setInitializationString("-1");
//                commentGenerator.addFieldComment(field, introspectedTable);
//                topLevelClass.addField(field);
                char c = name.charAt(0);
                String camel = Character.toUpperCase(c) + name.substring(1);
                Method method = new Method();
                method.setVisibility(JavaVisibility.PUBLIC);
                method.setName("set" + camel);
                method.addParameter(new Parameter(pageInfoInstance, name));
                method.addBodyLine("this." + name + "=" + name + ";");
                commentGenerator.addGeneralMethodComment(method, introspectedTable);
                topLevelClass.addMethod(method);
                method = new Method();
                method.setVisibility(JavaVisibility.PUBLIC);
                method.setReturnType(pageInfoInstance);
                method.setName("get" + camel);
                method.addBodyLine("return " + name + ";");
                commentGenerator.addGeneralMethodComment(method, introspectedTable);
                topLevelClass.addMethod(method);
        }
 
        @SuppressWarnings("unused")
        private void addLimitString(TopLevelClass topLevelClass, IntrospectedTable introspectedTable, String name) {
                CommentGenerator commentGenerator = context.getCommentGenerator();
                FullyQualifiedJavaType stringInstance = FullyQualifiedJavaType.getStringInstance();
//                Field field = new Field();
//                field.setVisibility(JavaVisibility.PROTECTED);
                // field.setType(FullyQualifiedJavaType.getIntInstance());
                // PrimitiveTypeWrapper integerInstance =
                // PrimitiveTypeWrapper.getIntegerInstance();
//                field.setType(stringInstance);
//                field.setName(name);
//                // field.setInitializationString("-1");
//                commentGenerator.addFieldComment(field, introspectedTable);
//                topLevelClass.addField(field);
                char c = name.charAt(0);
                String camel = Character.toUpperCase(c) + name.substring(1);
                Method method = new Method();
                method.setVisibility(JavaVisibility.PUBLIC);
                method.setName("set" + camel);
                method.addParameter(new Parameter(stringInstance, name));
                method.addBodyLine("this." + name + "=" + name + ";");
                commentGenerator.addGeneralMethodComment(method, introspectedTable);
                topLevelClass.addMethod(method);
                method = new Method();
                method.setVisibility(JavaVisibility.PUBLIC);
                method.setReturnType(stringInstance);
                method.setName("get" + camel);
                method.addBodyLine("return " + name + ";");
                commentGenerator.addGeneralMethodComment(method, introspectedTable);
                topLevelClass.addMethod(method);
                introspectedTable.getFullyQualifiedTable();
        }
 
 
}
```



SerializablePlugin:

```java
public class SerializablePlugin extends PluginAdapter {
        private FullyQualifiedJavaType serializable;
        private FullyQualifiedJavaType gwtSerializable;
        private boolean addGWTInterface;
        private boolean suppressJavaInterface;
 
        public SerializablePlugin() {
                this.serializable = new FullyQualifiedJavaType("java.io.Serializable");
                this.gwtSerializable = new FullyQualifiedJavaType("com.google.gwt.user.client.rpc.IsSerializable");
        }
 
        public boolean validate(List<String> warnings) {
                return true;
        }
 
        public void setProperties(Properties properties) {
                super.setProperties(properties);
                this.addGWTInterface = Boolean.valueOf(properties.getProperty("addGWTInterface")).booleanValue();
                this.suppressJavaInterface = Boolean.valueOf(properties.getProperty("suppressJavaInterface")).booleanValue();
        }
 
        public boolean modelBaseRecordClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
                makeSerializable(topLevelClass, introspectedTable);
                return true;
        }
 
        public boolean modelPrimaryKeyClassGenerated(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
                makeSerializable(topLevelClass, introspectedTable);
                return true;
        }
 
        public boolean modelRecordWithBLOBsClassGenerated(TopLevelClass topLevelClass,
                        IntrospectedTable introspectedTable) {
                makeSerializable(topLevelClass, introspectedTable);
                return true;
        }
 
        protected void makeSerializable(TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {
                if (this.addGWTInterface) {
                        topLevelClass.addImportedType(this.gwtSerializable);
                        topLevelClass.addSuperInterface(this.gwtSerializable);
                }
 
                if (!(this.suppressJavaInterface)) {
                        topLevelClass.addImportedType(this.serializable);
                        topLevelClass.addSuperInterface(this.serializable);
                        topLevelClass.addAnnotation("@SuppressWarnings(\"serial\")");
                         
                }
        }
}
```



FullyQualifiedJavaTypeProxyFactory:

```java
public class FullyQualifiedJavaTypeProxyFactory  extends FullyQualifiedJavaType{
         
        private static FullyQualifiedJavaType pageInfoInstance = null;
        private static FullyQualifiedJavaType baseExampleInstance = null;
        private static FullyQualifiedJavaType baseMapperInstance = null;
        private static FullyQualifiedJavaType baseServiceInstance = null;
        private static FullyQualifiedJavaType baseServiceImplInstance = null;
         
        public FullyQualifiedJavaTypeProxyFactory(String fullTypeSpecification) {
                super(fullTypeSpecification);
        }
         
        public static final FullyQualifiedJavaType getPageInfoInstanceInstance() {
                if (pageInfoInstance == null) {
                        pageInfoInstance = new FullyQualifiedJavaType("cn.xxx.core.base.model.PageInfo");
                }
 
                return pageInfoInstance;
        }
        public static final FullyQualifiedJavaType getBaseExampleInstance() {
                if (baseExampleInstance == null) {
                        baseExampleInstance = new FullyQualifiedJavaType("cn.xxx.core.base.model.BaseExample");
                }
                 
                return baseExampleInstance;
        }
         
        public static final FullyQualifiedJavaType getBaseMapperInstance() {
                if (baseMapperInstance == null) {
                        baseMapperInstance = new FullyQualifiedJavaType("cn.xxx.core.base.dao.BaseMapper");
                }
                 
                return baseMapperInstance;
        }
        public static final FullyQualifiedJavaType getBaseServiceInstance() {
                if (baseServiceInstance == null) {
                        baseServiceInstance = new FullyQualifiedJavaType("cn.xxx.core.base.service.BaseService");
                }
                 
                return baseServiceInstance;
        }
        public static final FullyQualifiedJavaType getBaseServiceImplInstance() {
                if (baseServiceImplInstance == null) {
                        baseServiceImplInstance = new FullyQualifiedJavaType("cn.xxx.core.base.service.impl.BaseServiceImpl");
                }
                 
                return baseServiceImplInstance;
        }
}
```



generatorConfig.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
 
<generatorConfiguration>
 
        <context id="ables" targetRuntime="MyBatis3">
         
                <plugin type="run.override.ServiceHierarchyPlugin" /> 
                <plugin type="run.override.PaginationPlugin" /> 
                <plugin type="run.override.SerializablePlugin" />
                <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
                <commentGenerator type="run.override.CommentGenerator">
                        <property name="suppressAllComments" value="false" />
                        <property name="suppressDate" value="false" />
                </commentGenerator>
                 
                <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://xx.xx.xx.xx:3306/crowdsourcing?characterEncoding=utf8"
                         userId="xxx"
                        password="xxx">
                </jdbcConnection>
                <javaTypeResolver>
                        <property name="forceBigDecimals" value="false" />
                </javaTypeResolver>
 
                  <javaModelGenerator targetPackage="cn.xxx.xxx.task.module.model" targetProject=".\src">   
                        <property name="enableSubPackages" value="false" />
                        <property name="trimStrings" value="true" />
                </javaModelGenerator>
                 
                 <sqlMapGenerator targetPackage="mapper.cn.xxx.xxx.task.module.dao" targetProject=".\src"> 
                        <property name="enableSubPackages" value="false" />
                </sqlMapGenerator>
                 
                <javaClientGenerator type="XMLMAPPER" targetPackage="cn.xxx.xxx.task.module.dao" targetProject=".\src">  
                        <property name="enableSubPackages" value="false" />
                </javaClientGenerator>
                <!-- 指定数据库表 -->
                <!-- tableName:用于自动生成代码的数据库表；domainObjectName:对应于数据库表的javaBean类名   -->
                <table tableName="a_task_quest_sub" domainObjectName="TaskQuestSub" alias="taskQuestSub">
                 <generatedKey column="id" sqlStatement="MySql" identity="true" /> 
                </table>
        </context>
</generatorConfiguration>
```



service层代码生成

ServiceHierarchyPlugin

```java
public class ServiceHierarchyPlugin extends PluginAdapter {
     
 
    @Override
    public List<GeneratedJavaFile> contextGenerateAdditionalJavaFiles(IntrospectedTable introspectedTable) {
        CompilationUnit addServiceInterface = addServiceInterface(introspectedTable);
        CompilationUnit addServiceImplClazz = addServiceImplClazz(introspectedTable);
         
        JavaModelGeneratorConfiguration javaModelGeneratorConfiguration = this.context.getJavaModelGeneratorConfiguration();
         
        String targetProject = javaModelGeneratorConfiguration.getTargetProject();
         
        GeneratedJavaFile gjfServiceInterface = new GeneratedJavaFile(addServiceInterface, targetProject, this.context.getProperty("javaFileEncoding"), this.context.getJavaFormatter());
        GeneratedJavaFile gjfServiceImplClazz = new GeneratedJavaFile(addServiceImplClazz, targetProject, this.context.getProperty("javaFileEncoding"), this.context.getJavaFormatter());
         
         
        List<GeneratedJavaFile> list= new ArrayList<>();
        list.add(gjfServiceInterface);
        list.add(gjfServiceImplClazz);
        return list;
    }
 
    protected CompilationUnit addServiceInterface(IntrospectedTable introspectedTable) {
         
        String entityClazzName = introspectedTable.getBaseRecordType();
        String javaModelTargetPackage = entityClazzName.substring(0,entityClazzName.lastIndexOf("."));
        String serviceSuperPackageName = javaModelTargetPackage.substring(0, javaModelTargetPackage.lastIndexOf("."));
         
        String entityExampleClazzName = introspectedTable.getExampleType();
        String domainObjectName = introspectedTable.getFullyQualifiedTable().getDomainObjectName();
         
        JavaTypeResolver javaTypeResolver = new JavaTypeResolverDefaultImpl();
        FullyQualifiedJavaType calculateJavaType = javaTypeResolver
                .calculateJavaType(introspectedTable.getPrimaryKeyColumns().get(0));
         
        FullyQualifiedJavaType superInterfaceType = new FullyQualifiedJavaType(
                "BaseService<" + entityClazzName + "," + entityExampleClazzName + ","
                        + calculateJavaType.getShortName() + ">");
 
 
        Interface serviceInterface = new Interface(serviceSuperPackageName + ".service." + domainObjectName + "Service");
        serviceInterface.addSuperInterface(superInterfaceType);
        serviceInterface.setVisibility(JavaVisibility.PUBLIC);
         
         
        FullyQualifiedJavaType baseServiceInstance = FullyQualifiedJavaTypeProxyFactory.getBaseServiceInstance();
        FullyQualifiedJavaType modelJavaType = new FullyQualifiedJavaType(entityClazzName);
        FullyQualifiedJavaType exampleJavaType = new FullyQualifiedJavaType(entityExampleClazzName);
        serviceInterface.addImportedType(baseServiceInstance);
        serviceInterface.addImportedType(modelJavaType);
        serviceInterface.addImportedType(exampleJavaType);
        return serviceInterface;
    }
 
    protected CompilationUnit addServiceImplClazz(IntrospectedTable introspectedTable) {
         
        String entityClazzName = introspectedTable.getBaseRecordType();
        String javaModelTargetPackage = entityClazzName.substring(0,entityClazzName.lastIndexOf("."));
        String serviceSuperPackageName = javaModelTargetPackage.substring(0, javaModelTargetPackage.lastIndexOf("."));
        String entityExampleClazzName = introspectedTable.getExampleType();
         
        String JavaMapperTypeName = introspectedTable.getMyBatis3JavaMapperType();
        String domainObjectName = introspectedTable.getFullyQualifiedTable().getDomainObjectName();
         
        JavaTypeResolver javaTypeResolver = new JavaTypeResolverDefaultImpl();
        FullyQualifiedJavaType calculateJavaType = javaTypeResolver
                .calculateJavaType(introspectedTable.getPrimaryKeyColumns().get(0));
        FullyQualifiedJavaType superClazzType = new FullyQualifiedJavaType(
                "BaseServiceImpl<" + introspectedTable.getBaseRecordType() + "," + introspectedTable.getExampleType() + ","
                        + calculateJavaType.getShortName() + ">");
        FullyQualifiedJavaType implInterfaceType = new FullyQualifiedJavaType(serviceSuperPackageName + ".service." + domainObjectName + "Service");
 
 
        TopLevelClass serviceImplClazz = new TopLevelClass(serviceSuperPackageName + ".service.impl." + domainObjectName + "ServiceImpl");
        serviceImplClazz.addSuperInterface(implInterfaceType);
        serviceImplClazz.setSuperClass(superClazzType);
        serviceImplClazz.setVisibility(JavaVisibility.PUBLIC);
        serviceImplClazz.addAnnotation("@Service");
         
        FullyQualifiedJavaType baseServiceInstance = FullyQualifiedJavaTypeProxyFactory.getBaseServiceImplInstance();
        FullyQualifiedJavaType modelJavaType = new FullyQualifiedJavaType(entityClazzName);
        FullyQualifiedJavaType exampleJavaType = new FullyQualifiedJavaType(entityExampleClazzName);
        serviceImplClazz.addImportedType(new FullyQualifiedJavaType("org.springframework.beans.factory.annotation.Autowired"));
        serviceImplClazz.addImportedType(new FullyQualifiedJavaType("org.springframework.stereotype.Service"));
        serviceImplClazz.addImportedType(baseServiceInstance);
        serviceImplClazz.addImportedType(modelJavaType);
        serviceImplClazz.addImportedType(exampleJavaType);
        serviceImplClazz.addImportedType(implInterfaceType);
         
         
         
        FullyQualifiedJavaType logType = new FullyQualifiedJavaType("org.apache.commons.logging.Log");
        FullyQualifiedJavaType logFactoryType = new FullyQualifiedJavaType("org.apache.commons.logging.LogFactory");
        Field logField = new Field();
        logField.setVisibility(JavaVisibility.PRIVATE);
        logField.setStatic(true);
        logField.setFinal(true);
        logField.setType(logType);
        logField.setName("log");
        logField.setInitializationString("LogFactory.getLog("+ entityClazzName.substring(entityClazzName.lastIndexOf(".") + 1) + "ServiceImpl.class)");
        logField.addAnnotation("");
        logField.addAnnotation("@SuppressWarnings(\"unused\")");
        serviceImplClazz.addField(logField);
        serviceImplClazz.addImportedType(logType);
        serviceImplClazz.addImportedType(logFactoryType);
         
         
        String mapperName = Character.toLowerCase(domainObjectName.charAt(0)) + domainObjectName.substring(1) + "Mapper";
        FullyQualifiedJavaType JavaMapperType = new FullyQualifiedJavaType(JavaMapperTypeName);
         
        Field mapperField = new Field();
        mapperField.setVisibility(JavaVisibility.PUBLIC);
        mapperField.setType(JavaMapperType);//Mapper.java
        mapperField.setName(mapperName);
        mapperField.addAnnotation("@Autowired");
        serviceImplClazz.addField(mapperField);
        serviceImplClazz.addImportedType(JavaMapperType);
         
         
        Method mapperMethod = new Method();
        mapperMethod.setVisibility(JavaVisibility.PUBLIC);
        mapperMethod.setName("setMapper");
        mapperMethod.addBodyLine("super.setMapper(" + mapperName+ ");");
        mapperMethod.addAnnotation("@Autowired");
        serviceImplClazz.addMethod(mapperMethod);
         
        return serviceImplClazz;
    }
     
    @Override
    public boolean validate(List<String> paramList) {
        return true;
    }
}
```



