---
layout: post
category: "设计模式"
title:  "单例模式"
tags: [单例]
---
##Properties存在的问题

	1.简化包名为 .prop
	2.根据不同操作系统加载不同的 properties 文件指示文件
	3.使用双重检查锁
	4.修改文件加载方式使相对路径不使用路径分隔符
	5.使用 RuntimeException 封装 checked Exception
	6.修改单元测试以断言异常

##单例模式

	/**
	 * 懒汉式单例模式将PropertiesParser类设置为单例
	 */
	public class SimpleProperties {
	
		private static volatile byte[] locker = new byte[0];
		private static PropertiesParser propertiesParser = null;
	
		private SimpleProperties() {
		}
	
		/**
		 * 得到PropertiesParser对象
		 * 
		 * @Title: getInstance
		 * @return PropertiesParser
		 */
		public static PropertiesParser getInstance() {
	
			if (propertiesParser != null) {
				return propertiesParser;
			}
	
			String appPropName = "app.properties";
	
			switch (Environment.getOS()) {
			case Unix:
			case Mac:
				appPropName = "app_unix.properties";
				break;
			case Windows:
				appPropName = "app_win.properties";
				break;
			}
	
			synchronized (locker) {
				if (propertiesParser != null) {
					return propertiesParser;
				}
	
				Properties appProp = new PropertiesParser(appPropName)
						.getProperties();
				String realPropName = appProp.getProperty("path");
				propertiesParser = new PropertiesParser(realPropName);
				return propertiesParser;
			}
		}
	
	}

##异常测试
	@Test
	public void test文件名应当合法() {
		try {
			new PropertiesParser("");
			Assert.fail("Should throw IllegalArgumentException");
		} catch (IllegalArgumentException ex) {

		}
	}	

	@Test
	public void test文件必须存在() {
		try {
			new PropertiesParser("notexist.properties");
			Assert.fail("Should throw RuntimeException");
		} catch (Exception ex) {
			Assert.assertTrue(ex.getCause() instanceof FileNotFoundException);
		}
	}

##运行环境判断
	
	public class Environment {
	
		public static OS getOS() {
			String os = System.getProperty("os.name").toLowerCase();
			if (OS.isUnix(os)) {
				return OS.Unix;
			}
			if (OS.isWindows(os)) {
				return OS.Windows;
			}
			if (OS.isMac(os)) {
				return OS.Mac;
			}
	
			throw new UnsupportedOperationException("Unsupported OS name: " + os);
		}
	
		public enum OS {
			Windows, Unix, Mac;
	
			static boolean isWindows(String os) {
				return (os.indexOf("win") >= 0);
			}
	
			static boolean isMac(String os) {
				return (os.indexOf("mac") >= 0);
			}
	
			static boolean isUnix(String os) {
				return (os.indexOf("nix") >= 0 || os.indexOf("nux") >= 0 || os
						.indexOf("aix") > 0);
			}
	
			static boolean isSolaris(String os) {
				return (os.indexOf("sunos") >= 0);
			}
		}
	}