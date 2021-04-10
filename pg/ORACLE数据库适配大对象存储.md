![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png

ORACLE数据库适配大对象存储






 

 

目录

[1.  Oracle中Blob和Clob的作用？... 2](#_Toc47452536)

[2．PostgreSQL大对象存储方式... 2](#_Toc47452537)

[3.  Oracle /PostgreSQL大对象存储结构及正常用例... 3](#_Toc47452538)

[3.1 Oracle大对象存储... 3](#_Toc47452539)

[3.2  PostgreSQL 大对象存储... 5](#_Toc47452540)

[4. 项目适配... 8](#_Toc47452541)

[4.1 背景... 8](#_Toc47452542)

[4.2 适配修改... 10](#_Toc47452543)

[4.2.1解决方法... 10](#_Toc47452544)

[4.2.2具体实现方法... 11](#_Toc47452545)

[5. 参考文档... 13](#_Toc47452546)

 



 

# 1.   Oracle中Blob和Clob的作用？

Blob是指二进制大对象也就是英文Binary Large Object的缩写，

Clob是指大字符对象也就是英文Character Large Object的缩写。

由此可见这辆个类型都是用来存储大量数据而设计的，其中BLOB是用来存储大量二进制数据的，可存储的最大大小为4G字节，应用：通常像图片、文件、音乐等信息就用BLOB字段来存储，先将文件转为二进制再存储进去。；CLOB用来存储大量文本数据，可存储的最大大小为4G字节，。应用：而像文章或者是较长的文字，就用CLOB存储。

那么有人肯定要问既然已经有VARCHAR和VARBINARY两中类型，为什么还要再使用另外的两种类型呢？其实问题很简单，VARCHAR 和VARBINARY两种类型是有自己的局限性的。首先说这两种类型的长度还是有限的不可以超过一定的限额，以VARCHAR再ORA中为例长度不可以超过4000；

那么有人又要问了，LONGVARCHAR类型作为数据库中的一种存储字符的类型可以满足要求，存储很长的字符，那为什么非要出现CLOB类型呢？其实如果你用过LONGVARCHAR类型就不难发现，该类型的一个重要缺陷就是不可以使用LIKE这样的条件检索。（稍候将介绍在CLOB中如何实现类似LIKE的模糊查找）另外除了上述的问题外，还又一个问题，就是在数据库中VARCHAR和VARBINARY的存取是将全部内容从全部读取或写入，对于100K或者说更大数据来说这样的读写方式，远不如用流进行读写来得更现实一些。

# 2．PostgreSQL大对象存储方式

PostgreSQL的[大数据类型](http://www.raincent.com/list-10-1.html)只有两种，就是存储二进制数据的bytea和存储字符类型的text。下面介绍一下它们之间的对应和迁移时的一些注意事项。

Oracle的Blob类型主要内容是二进制的大对象。最大长度是(4G-1)*database block size。在PostgreSQL中，与之对应的是bytea。最大长度是1G。虽然最大长度小于Blob，但是在实际应用中已经足够了。

Oracle的Clob类型，主要存储基于数据库字符集的单字节或多字节文本信息，最大长度是(4G-1)*database block size。PostgreSQL中，可以使用text来对应。text的最大长度是1G，比Oracle的小。但是，实际应用中，1G已经足够。

Oracle的NClob类型，主要存储固定长度的UNICODE字符串，最大长度是(4G-1)*database block size。PostgreSQL中，可以使用text来对应。text的最大长度是1G，比Oracle的小。但是，实际应用中，1G已经足够。

迁移的时候，文本信息转成text，二进制信息转成bytea。特殊类型BFILE形式的，可以额外写一些代码把数据从文件中读出转换成bytea。这样就可以完成大数据类型的迁移。

注意： PostgreSQL对应的[大数据类型](http://www.raincent.com/list-10-1.html)还有一个对象标识符类型(oid)。它是一个标识符，指向在pg_largeobject 系统表中的一个bytea类型的对象。由于它是用一个四字节的无符号整数实现，不能够提供[大数据库](http://www.raincent.com/list-10-1.html)范围内的唯一性保证。因此，postgreSQL不推荐使用oid类型。加上它的内部实现，也是使用bytea类型，所以就不单独介绍了。

# 3.   Oracle /PostgreSQL大对象存储结构及正常用例

## 3.1 Oracle大对象存储

（1）Oracle插入blob类型数据的类。代码如下：

\4.   **class** InsertBlobData{ 

\5.     **private** ResultSet rs = **null**; 

\6.     **private** InitDB idb = **null**; 

\7.    

\8.     InsertBlobData() 

\9.     { 

\10.     idb = **new** InitDB(); 

\11.   } 

\12.  

\13.   **public** **void** insertBlob(String sql1) **throws** SQLException 

\14.   { 

\15.     Connection con = idb.getCon(); 

\16.     **try** 

\17.     { 

\18.       con.setAutoCommit(**false**);// 不设置自动提交 

\19.       BLOB blob = **null**; // 插入空的Blob 

\20.       PreparedStatement pstmt = con 

\21.           .prepareStatement("insert into cdl_test(sid,img) values(?,empty_blob())"); 

\22.       pstmt.setString(1, "100"); 

\23.       pstmt.executeUpdate(); 

\24.       pstmt.close(); 

\25.       rs = con.createStatement().executeQuery(sql1); 

\26.       **while** (rs.next()) 

\27.       { 

\28.         System.out.println("rs length is:"); 

\29.         oracle.sql.BLOB b = (oracle.sql.BLOB) rs.getBlob("img"); 

\30.         System.out.println("cloblength is:" + b.getLength()); 

\31.         File f = **new** File("d:\\1.jpg"); //1.jpg一张QQ的截图 

\32.         System.out.println("file path is:" + f.getAbsolutePath()); 

\33.         BufferedInputStream in = **new** BufferedInputStream( 

\34.             **new** FileInputStream(f)); 

\35.         BufferedOutputStream out = **new** BufferedOutputStream(b 

\36.             .getBinaryOutputStream()); 

\37.         **int** c; 

\38.         **while** ((c = in.read()) != -1) 

\39.         { 

\40.           out.write(c); 

\41.         } 

\42.         in.close(); 

\43.         out.close(); 

\44.       } 

\45.       con.commit(); 

\46.     } 

\47.     **catch** (Exception e) 

\48.     { 

\49.       con.rollback();// 出错回滚 

\50.       e.printStackTrace(); 

\51.     } 

\52.   } 

\53. } 

（2）Oracle插入clob数据类型的，代码如下：

\1.   **class** InsertClobData 

\2.   { 

\3.     **private** ResultSet rs = **null**; 

\4.     **private** InitDB idb = **null**; 

\5.    

\6.     InsertClobData() 

\7.     { 

\8.       idb = **new** InitDB(); 

\9.     } 

\10.  

\11.   **public** **void** insertClob(String sql1) **throws** SQLException 

\12.   { 

\13.     Connection con = idb.getCon(); 

\14.     **try** 

\15.     { 

\16.       con.setAutoCommit(**false**);// 不设置自动提交 

\17.       BLOB blob = **null**; // 插入空的Clob 

\18.       PreparedStatement pstmt = con 

\19.           .prepareStatement("insert into cdl_test(sid,doc) values(?,empty_clob())"); 

\20.       pstmt.setString(1, "101"); 

\21.       pstmt.executeUpdate(); 

\22.       pstmt.close(); 

\23.       rs = con.createStatement().executeQuery(sql1); 

\24.       **while** (rs.next()) 

\25.       { 

\26.         System.out.println("sdfasdfas"); 

\27.         oracle.sql.CLOB cb = (oracle.sql.CLOB) rs.getClob("doc"); 

\28.         File f = **new** File("d:\\1.txt"); //1.txt一本小说《风云》马荣成 

\29.         System.out.println("file path is:" + f.getAbsolutePath()); 

\30.         BufferedWriter out = **new** BufferedWriter(cb 

\31.             .getCharacterOutputStream()); 

\32.         BufferedReader in = **new** BufferedReader(**new** FileReader(f)); 

\33.         **int** c; 

\34.         **while** ((c = in.read()) != -1) 

\35.         { 

\36.           out.write(c); 

\37.         } 

\38.         in.close(); 

\39.         out.close(); 

\40.       } 

\41.       con.commit(); 

\42.     } 

\43.     **catch** (Exception e) 

\44.     { 

\45.       con.rollback();// 出错回滚 

\46.       e.printStackTrace(); 

\47.     } 

\48.   } 

\49. } 

## 3.2 PostgreSQL 大对象存储

方式1：二进制文件直接存储成bytea，代码如下：

CREATE TABLE images (imgname text, img bytea);

（只涉及一张表）

 

存储：

File file = new File("myimage.gif");

FileInputStream fis = new FileInputStream(file);

PreparedStatement ps = conn.prepareStatement("INSERT INTO images VALUES (?, ?)");

ps.setString(1, file.getName());

ps.setBinaryStream(2, fis, (int)file.length());

ps.executeUpdate();

ps.close();

fis.close();

读取

PreparedStatement ps = conn.prepareStatement("SELECT img FROM images WHERE imgname = ?");

ps.setString(1, "myimage.gif");

ResultSet rs = ps.executeQuery();

while (rs.next()) {

  byte[] imgBytes = rs.getBytes(1);

  // use the data in some way here

}

rs.close();

ps.close();

 

方式2：二进制文件直接存储成oid（类似于一个id）,真实数据存储在pg_largeObject表

 

```
CREATE TABLE imageslo (imgname text, imgoid oid);
```

存储

```
// All LargeObject API calls must be within a transaction block
conn.setAutoCommit(false);
 
// Get the Large Object Manager to perform operations with
LargeObjectManager lobj = ((org.postgresql.PGConnection)conn).getLargeObjectAPI();
 
// Create a new large object
int oid = lobj.create(LargeObjectManager.READ | LargeObjectManager.WRITE);
 
// Open the large object for writing
LargeObject obj = lobj.open(oid, LargeObjectManager.WRITE);
 
// Now open the file
File file = new File("myimage.gif");
FileInputStream fis = new FileInputStream(file);
 
// Copy the data from the file to the large object
byte buf[] = new byte[2048];
int s, tl = 0;
while ((s = fis.read(buf, 0, 2048)) > 0) {
    obj.write(buf, 0, s);
    tl += s;
}
```

PreparedStatement ps = conn.prepareStatement("INSERT INTO imageslo VALUES (?, ?)");

ps.setString(1, file.getName());

ps.setInt(2, oid);

ps.executeUpdate();

ps.close();

fis.close();

conn.commit();

 

读取

LargeObjectManager lobj = ((org.postgresql.PGConnection)conn).getLargeObjectAPI();

 

PreparedStatement ps = conn.prepareStatement("SELECT imgoid FROM imageslo WHERE imgname = ?");

ps.setString(1, "myimage.gif");

ResultSet rs = ps.executeQuery();

while (rs.next()) {

  // Open the large object for reading

  int oid = rs.getInt(1);

  LargeObject obj = lobj.open(oid, LargeObjectManager.READ);

 

  // Read the data

  byte buf[] = new byte[obj.size()];

  obj.read(buf, 0, obj.size());

  // Do something with the data read here

 

  // Close the object

  obj.close();

}

rs.close();

ps.close();

 

// Finally, commit the transaction.

conn.commit();

# 4. 项目适配

## 4.1 背景

在GFKD数据库适配项目中，由于客户应用之前用的是oracle数据库，要转换成咱们的uxdb数据库存储。

（1）数据存储转换方式：是按3.2中的第一种方式存储的，即oracle的blob转换成posrgre 的bytea类型；oracle的clob转换成posrgre 的text类型。即单个类型存储只涉及一张表结构。

( 2 ) 数据读取方式：在客户应用中，读取数据的方式是oracle数据库大对象读取数据方式（即getBlob,getClob方法）,在不改变客户应用的场景下，使用postgre的相应方法出现适配问题，问题如下：

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

原因分析：在postgre中getBlob，getClob方法适用于标题3.2中的数据存储结构，即两张表的数据结构。参照postgre jdbc源码分析可知：postgre中getBlob，getClob方法会先根据查询列名或者列数，先去计算oid,

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

再根据oid去创建PgBolb,PgClob对象。

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

Blob类型通过getBinaryStream()方法获取具体二进制数据信息，具体实现如下：

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image014.jpg)

## 4.2 适配修改

### 4.2.1解决方法 

增加几个适配oracle JDBC对大对象存储的处理类;对于bytea类型大对象数据，再调用getBlob或者getClob方法时，进行格式转换处理。使数据可以通过getBlob，或者getClob正常获取出来，并生成数据/文件,适配结果如下：

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image016.png)

### 4.2.2具体实现方法

(1).新增适配类型UXByteaBlob，UXByteaClob，UXByteaBlobOutputStream，UXByteaClobOutputStream，UXByteaClobWriter,实现具体转换逻辑。

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image018.jpg)
 (2).在原有UXResultSet类加入Blob类型适配，修改如下：

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image020.png)

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image022.png)

(3).在原有UXResultSet类加入Clob类型适配，修改如下：

 

![img](/C:/Users/thsfz/AppData/Local/Temp/msohtmlclip1/01/clip_image024.png)![img](https://cdn.jsdelivr.net/gh/zfsndtl/picmarkdown@main/picture/clip_image026.png)

注意：Clob和Blob是有区别的，前者是字符流，后者是字节流，get获取实现方式不同

# 5. 参考文档

https://blog.csdn.net/housonglin1213/article/details/86508452

https://blog.csdn.net/weixin_30485799/article/details/97176619

https://blog.csdn.net/phantomes/article/details/8077930

https://jdbc.postgresql.org/documentation/80/binary-data.html