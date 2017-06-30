---
layout: post
title: 'Jave IO流之压缩流（zip,Gzip）'
categories:
  - Study
  - Java
tags: Java
keywords:
  - Java
  - zip
  - Gzip
  - 压缩
abbrlink: 837682646
date: 2015-09-01 00:00:00
---


JAVAIO流是java的一个很重要的部分，清晰有很复杂，各种各样的流分管不同的功能。正确使用IO流可以让你的输入输出效率增加，这篇博客主要说一下压缩流的使用，使用JAVA内置API压缩解压缩文件。

<!--more-->
## ZIP压缩
### 主要的用到的API是
```java
  ZipFile
  ZipInputStream
  ZipOutputStream
```
### 完整代码
```java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Enumeration;
import java.util.zip.ZipEntry;
import java.util.zip.ZipException;
import java.util.zip.ZipFile;
import java.util.zip.ZipInputStream;
import java.util.zip.ZipOutputStream;

/**
 * 
 * @author chendong
 * 
 *         </br>使用java内置APIZipFile完成对文件的压缩解压缩操作
 * 
 *         </br>文件中如果有中文名称加压后的文件会出现乱码，但是解压之后又ok了，可以使用中文
 * 
 *         </br>提供了大文件异步加压解压的方法，使用回调检测是否完成
 * 
 *         </br>提供小文件同步加压解压更加方便
 * 
 */
public class ZipUtils {

	/**
	 * 用于大文件加压解压的回调监听
	 * 
	 * @author chendong
	 * 
	 */
	public interface OnZipOverListener {
		void onZipOver();
	}

	public interface OnUnZipOverListener {
		void onUnZipOver();
	}

	/**
	 * 压缩 </br>构造源文件
	 * 
	 * @param src
	 *            源文件路径
	 * @param dest
	 *            目标文件路径
	 * @throws FileNotFoundException
	 */
	public static void zip(String src, String dest, OnZipOverListener listener) {
		File srcFile = new File(src);
		zip(srcFile, dest, listener);

	}

	/**
	 * 压缩 </br>生成压缩输出文件流
	 * 
	 * @param srcFile
	 * @param dest
	 * @throws FileNotFoundException
	 */
	public static void zip(File srcFile, String dest, OnZipOverListener listener) {
		ZipOutputStream destOs = null;
		try {
			destOs = new ZipOutputStream(new FileOutputStream(dest));
			zip(srcFile, destOs, "");
			close(destOs);
			if (listener != null)
				listener.onZipOver();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(destOs);
		}

	}

	/**
	 * 压缩,使用递归 </br>写文件 </br>主要操作在这个函数中，使用递归如果是一个文件，将其写入流中否则进行递归 </br>ZipEntry
	 * 类是java.util.zip包下的一个类， ZipEntry 类用于表示 ZIP 文件条目。 利用这个类压缩和解压zip文件
	 * 
	 * @param srcFile
	 * @param destOs
	 * @param string
	 * @throws IOException
	 */
	private static void zip(File srcFile, ZipOutputStream destOs, String base) {
		BufferedInputStream bis = null;

		try {
			if (srcFile.isDirectory()) {
				/* 如果源文件是目录 */
				File[] files = srcFile.listFiles();
				destOs.putNextEntry(new ZipEntry(base + "/"));
				base = base.length() == 0 ? "" : base + "/";
				for (int i = 0; i < files.length; i++) {
					zip(files[i], destOs, base + files[i].getName());
				}
			} else {
				/* 如果是文件 */
				destOs.putNextEntry(new ZipEntry(base));
				bis = new BufferedInputStream(new FileInputStream(srcFile));
				byte[] buffer = new byte[1024];
				int len = 0;
				while ((len = bis.read(buffer)) != -1) {
					destOs.write(buffer, 0, len);
				}
				close(bis);
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(bis);
		}
	}

	/**
	 * 解压
	 * 
	 * @param src
	 *            源文件，需要是zip文件
	 * @param dest
	 *            目标文件，
	 */
	public static void unzip(String src, String dest,
			OnUnZipOverListener listener) {

		File destFile = new File(dest);
		/* 目标文件不存在，创建之 */
		if (!destFile.exists()) {
			destFile.mkdirs();
		}
		/* 构造源文件 */
		File srcFile = new File(src);
		if (!srcFile.exists()) {
			return;
		}
		unzip(srcFile, dest, listener);
	}

	/**
	 * 解压
	 * 
	 * @param srcFile
	 * @param destFile
	 */
	public static void unzip(File srcFile, String dest,
			OnUnZipOverListener listener) {
		BufferedInputStream bis = null;
		BufferedOutputStream bos = null;
		try {
			ZipFile srcZipFile = new ZipFile(srcFile);
			/* 获得zipentry的枚举 */
			Enumeration e = srcZipFile.entries();
			ZipEntry entry = null;
//			ZipInputStream zis = new ZipInputStream(
//					new FileInputStream(srcFile));
			// while((entry=zis.getNextEntry())!=null){
			while (e.hasMoreElements()) {
				entry = (ZipEntry) e.nextElement();
				if (entry.toString().equals("/")) {
					continue;
				}
				bis = new BufferedInputStream(srcZipFile.getInputStream(entry));
				/* 构建对应输出流 */
				bos = new BufferedOutputStream(new FileOutputStream(dest + "/"
						+ entry.getName()));
				int len = 0;
				byte[] buffer = new byte[1024];
				while ((len = bis.read(buffer)) != -1) {
					bos.write(buffer, 0, len);
				}
				bos.flush();
				close(bis);
				close(bos);
			}

			if (listener != null)
				listener.onUnZipOver();
		} catch (ZipException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(bis);
			close(bos);
		}
	}

	/**
	 * 压缩单个或者多个文件,但是不可以同时压缩文件和文件夹
	 * 
	 * @param src
	 *            想要压缩的文件路径，不可以是文件夹
	 * @param dest
	 *            目标路径，xx.zip
	 */
	public static void zipFile(String[] src, String dest,
			OnZipOverListener listener) {

		File parent = new File(new File(dest).getParent() + "/temp");
		if (!parent.exists()) {
			parent.mkdirs();
		}
		BufferedInputStream bis = null;
		BufferedOutputStream bos = null;
		File file = null;
		try {
			int len = 0;
			byte[] buffer = new byte[1024];
			for (String path : src) {
				file = new File(path);
				bis = new BufferedInputStream(new FileInputStream(file));
				bos = new BufferedOutputStream(new FileOutputStream(
						parent.getAbsolutePath() + "/" + file.getName()));
				while ((len = bis.read(buffer)) != -1) {
					bos.write(buffer, 0, len);
				}
				bos.flush();
				close(bos);
				close(bis);
			}
			zip(parent, dest, listener);
			for (File ff : parent.listFiles()) {
				ff.delete();
			}
			parent.delete();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(bos);
			close(bis);
		}

	}

	private static void close(Closeable close) {
		if (close != null) {
			try {
				close.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

}

```


## GZIP压缩
- 这个没太搞懂，一直在弄加压解压有点烦了，改天再说吧，贴一下已经实现的简单功能。就是加压一个文件，本身GZIP就是一对一的，也就是说每次只能压缩一个文件，我们需要压缩多个问价的时候要使用tar先打包，看到网上有实现这个功能的，但是使用的阿帕奇的第三方库，就没去试,这部分的有点仓促，主要是烦了，写完这个我就去看点别的了，老看一个东西烦得慌，改天会在完善吧。


### 代码实现
```java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.zip.GZIPInputStream;
import java.util.zip.GZIPOutputStream;

/**
 * 
 * 使用java内置API完成文件的加压解压</br>
 * 使用gzip压缩压缩效果更好，但是gzip压缩只能是一对一的，也就是说一个压缩包只能解压出一个文件，所以想压缩多个文件时就需要先使用tar压成一个包
 * ，再用gzip压缩
 * 
 * @author chendong
 * 
 */
public class GZipUtils {

	/**
	 * gzip压缩，将一个文件压缩到制定包
	 * 
	 * @param src
	 * @param dest
	 */
	public static void gzip(String src) {
		File srcFile = new File(src);
		if (!srcFile.exists()) {
			return;
		}
		BufferedInputStream bis = null;
		GZIPOutputStream gos = null;
		File destFile = new File(src + ".gz");
		try {
			/* 获得输入流 */
			bis = new BufferedInputStream(new FileInputStream(src));
			/* 获得压缩输出流 */
			gos = new GZIPOutputStream(new BufferedOutputStream(
					new FileOutputStream(destFile)));
			int len = 0;
			byte[] buffer = new byte[1024];
			while ((len = bis.read(buffer)) != -1) {
				gos.write(buffer, 0, len);
			}
			gos.flush();
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(gos);
			close(bis);
		}
	}

	/**
	 * 解压
	 * @param src
	 * @param dest
	 */
	public static void ungzip(String src, String dest) {
		File destFile = new File(dest);
		File srcFile = new File(src);
		GZIPInputStream gis = null;
		BufferedOutputStream bos = null;
		try {
			if (!destFile.exists()) {
				destFile.createNewFile();
			}
			if (!srcFile.exists()) {
				return;
			}
			bos = new BufferedOutputStream(new FileOutputStream(destFile));
			gis = new GZIPInputStream(new FileInputStream(srcFile));
			int len = 0;
			byte[] buffer = new byte[1024];
			while ((len = gis.read(buffer)) != -1) {
				bos.write(buffer, 0, len);
			}

		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			close(bos);
			close(gis);
		}
	}

	public static void pack(File files[]) {

	}

	private static void close(Closeable close) {
		if (close != null) {
			try {
				close.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}

```

  

