# 使用说明


## 1.搭建solr环境
&ensp;&ensp; **利用提供的solr4.9压缩包，搭建solr运行环境，从而为程序提供运行环境。**

* `Solr`服务正常启动
    * 确保本地配置Java环境变量，具体教程可以参考 `http://jingyan.baidu.com/article/f96699bb8b38e0894e3c1bef.html`
    * 解压缩提供的`solr4.9`包
    * 进入用cmd模式，进入解压完的solr目录下的example文件夹下
    * 使用`java -jar start.jar`启动solr服务
    * 在浏览器中访问`localhost:8983\solr`，出现solr界面，说明成功启动。
    
&ensp;&ensp; **关于solr服务器的其他一些配置**

* 自定义字段
    * 配置schema.xml,位于solrconfig同一目录下，用于配置data-config.xml文件中用到的字段类型，可以在179行开始，便于以后查看。文件的绝对路径为：
	`solr-4.9.0\example\solr\collection1\conf\schema.xml`
	* 定义格式为：`<field name="geo_name" type="text_general" indexed="true" stored="true" /> `
	* 其中name为自定义字段名称，type为字段类型以及索引、存储等设置
* 改写配置文件，连接到数据库（以Oracle为例）
    * 下载Oracle连接的驱动包
    * 将压缩包拷贝到solr目录下，路径为`solr-4.9.0\contrib\dataimporthandler\lib`
    * 配置solrconfig.xml，路径为`solr-4.9.0\example\solr\collection1\conf`,在第87~91行加入下面内容,从而导入引用Oracle的驱动包和DataImport所需的包

<code>    
 
	  <lib dir="../../../contrib/dataimporthandler/lib" regex=".*\.jar" />
      <lib dir="../../../dist/" regex="solr-dataimporthandler-4.9.0.jar" />
      <lib dir="../../../dist/" regex="solr-dataimporthandler-extras-4.9.0.jar" />

</code>

   * 在1210行加入下面内容，保证solr读取访问连接数据的文件data-config.xml

<code>

	<requestHandler name="/dataimport" 	class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
    <str name="config">data-config.xml</str> 
    </lst> 
   </requestHandler>
</code>

   * 在solrconfig.xml同文件夹下创建data-config.xml文件，其中dataSource中为数据库连	接的内容，entity中的内容为所要查询的表以及建立索引的列，field中column的值为数据库中列名，name为要映射到solr中的建立索引的列名需要保证两者数据类型相同， **需要注意这里使用到的name字段名称必须在schema.xml中已经定义，同时必须保证name中有一列为id**，示例内容：

<code>
	
	<dataConfig>

	<dataSource type="JdbcDataSource" driver="oracle.jdbc.driver.OracleDriver"  
    url="jdbc:oracle:thin:@60.30.69.61:1521:adc"  
    user="CVDEV2"  
    password="CVDEV2"/>  


	<document>

	<entity name="T_AUTOMAKER_INFO" query="SELECT AUTOMAKER_ID, AUTOMAKER_NAME, 
	CREATE_TIME from T_AUTOMAKER_INFO">

	<field column="AUTOMAKER_ID" name="id"/> 
	<field column="AUTOMAKER_NAME" name="use_cname"/> 
	<field column="CREATE_TIME" name="geo_name"/> 


	</entity>>
	</document>

	</dataConfig>

</code>

* 分词器
	* 下载IK Analyzer 2012FF_hf1，需要自行下载
	* IKAnalyzer2012FF_u1.jar拷贝到solr服务的solr\WEB-INF\lib下面，路径为：`solr-4.9.0\example\solr-webapp\webapp\WEB-INF\lib`
	* 把IKAnalyzer.cfg.xml、stopword.dic拷贝到需要使用分词器的core的conf下面，和core的schema.xml文件一个目录,路径为：`solr-4.9.0\example\solr\collection1\conf`
	* core的schema.xml，在<types></types>配置项间加一段如下配置：

<code>

	<fieldType name="text_ik" class="solr.TextField">   
     <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>   
	</fieldType>
  
</code>
我们就多了一种text_ik的field类型了，该类型使用的分词器就是ik-analyzer。

* 重启solr服务，访问`http://localhost:8983/solr/#/collection1/analysis`，即可对新添加的text_ik分词效果测试。

&ensp;&ensp; **这一部分内容为扩展内容，如果开发者想添加一些新的分词或者功能的时候参考这一部分内容。**

## 2.使用图形界面建立文件索引

&ensp;&ensp;将提供的searchengine项目导入到IDEA中，启动运行，在浏览器中访问：`http://localhost:8080/manage`，即可进入到管理者界面，对于文件建立索引。


* 对于数据库建立索引
	* 数据库连接栏输入类似：`jdbc:oracle:thin:@60.30.69.61:15211:qwewe`，即数据库的地址；
	* 表名栏输入要建立索引的表名，注意一次只能对一个表数据建立索引；
	* 字段名栏输入要建立索引的字段名，如果有多个字段，使用`；`隔开；
	* 单击`建立`，即可完成对与这张表指定字段索引建立，完成建立后会返回一个提示框。

	<img src="https://github.com/chengxuyuanjunjujn/imageurl/blob/master/a6.jpg?raw=true" width="400" />


* 对于文档建立索引
	* 单击`选择文件`，选择你要建立索引的文档
	* 单击`上传并建立`，完成建立工作，成功建立索引后返回一个提示框
	* 注意一次只能上传单个文档
	
	<img src="https://github.com/chengxuyuanjunjujn/imageurl/blob/master/a2.jpg?raw=true" width="400" />

* 使用xml文件建立相应文件索引
	* 用户改写提供项目下的TNBSolrDataSourcesConfig.xml文件；
		* 数据库类型，按照格式写入需要建立数据库表名以及字段名；
		* 文件类型，按照格式写入需要建立的文档路径；
	* 单击`选择文件`，导入提供项目下的`TNBSolrDataSourcesConfig.xml`
	* 单击`导入`，对于配置文件中的数据库和文件建立索引；

	<img src="https://github.com/chengxuyuanjunjujn/imageurl/blob/master/a3.jpg?raw=true" width="400" />

* 对于整个目录下的文件建立索引
	* 将要建立索引目录的路径填入，类似：`E:\ACM`;
	* 如果对于整个目录下的文件建立索引，第二栏空白即可；
	* 如果选择某一类型文件，可以在第二栏添上`md`、`csv`、`txt`等文件类型，注意每次只能选择一种文件类型进行建立索引；
	* 单击`建立`按钮，即可对选择目录建立索引；

	<img src="https://github.com/chengxuyuanjunjujn/imageurl/blob/master/a4.jpg?raw=true" width="400" />


* 对于已经建立索引的文件夹进行更新
	* 更新的目的提高目录索引更新速度，同时保证索引时效性；
	* 将需要更新的索引目录的路径填入，类似：`E:\ACM`;
	* 如果对于整个目录下的文件进行索引更新，第二栏空白即可；
	* 如果选择某一类型文件，可以在第二栏添上`md`、`csv`、`txt`等文件类型，注意每次只能选择一种文件类型进行建立索引；
	* 单击`更新`按钮，即可对选择目录建立索引；
	
	<img src="https://github.com/chengxuyuanjunjujn/imageurl/blob/master/a5.jpg?raw=true" width="400" />



## 3.组件提供jar包中可用的方法函数
&ensp;&ensp;&ensp;&ensp; **将需要使用搜索功能组件的项目导入提供的所有jar包，从而在项目中实现对于数据库文件以及文档建立索引，并进行搜索和结果展示的操作，而不需要使用图形界面。**


### Document.java中的方法
* **public static void documentIndex(String indexFile, String type)**方法，用于构建指定文件下的索引，
	* indexFile为文件路径;
	* type指定哪种类型文件，""为默认为所有文件 
	* 调用了本身提供的public方法 **private static void indexFilesSolrCell(String fileName, String solrId, String docType)**，用于对于文件构建列表索引；
	* 调用private方法 **private static LinkedList<String> getAllDoc(String indexFile, String type)**，用于确定整个目录结构并构建为一个列表，方便documentIndex使用；
	* **该方法在后续调整中被废除**

实际使用示例：  
  
<code>

    @RequestMapping(value="/import-dir", method=RequestMethod.POST)
    @ResponseBody
    public String importDir(String dir, String type) {
        try {
            documentIndex(dir, type);
            return "success";
        }catch (Exception e){
            e.printStackTrace();
            return "error";
        }
    }

</code>        

* **protected static boolean needUpdate(String url)**方法，用于判定一个文件目录是否更新，从而确定是否对索引进行更新
	* url为文件路径

实际调用示例：

```
 if(needUpdate(url)){

        indexFilesSolrCell2(fileName, solrId, docType);
 }
``` 

* **public static void indexFilesSolrCell(String fileName, String solrId, String docType)**方法，用于为pdf类型文件构建索引
	* filename为指定文件名；
	* solrId为文件绝对路径；
	* docType为文件类型；

* **public static void indexFilesSolrCell2(String fileName, String solrId, String docType)**方法，用于为txt, md, csv类型文件构建索引
	* filename为指定文件名；
	* solrId为文件绝对路径；
	* docType为文件类型；

* **public static String getModifiedTime(String url)**,用于获取文件的最后更新时间
	* url为文件的绝对路径；
	* 返回的string格式为："yyyy-MM-dd HH:mm:ss"，即标准时间格式；


### FileCharsetDetector.java中的方法
* **public String guessFileEncoding(File file)** 用于获取文件编码，其中file为文件类型变量，这个方法在为文件构建索引的时候使用，检查编码从而防止乱码。
	* 调用了私有化方法 **private String guessFileEncoding(File file, nsDetector det)**方法，进行检查编码，这个方法是透明的，不需要开发者了解。
	* 返回string为文件编码，eg：UTF-8,GBK,GB2312形式(不确定的时候，返回可能的字符编码序列)；若无，则返回null；

实际代码调用：

```
try{
            String encoding;
            File file=new File(solrId);
            String doctype = new FileCharsetDetector().guessFileEncoding(file);
            if(file.isFile() && file.exists()){

                encoding = doctype.split(",")[0];
                InputStreamReader read = new InputStreamReader(new FileInputStream(file), encoding);
                BufferedReader bufferedReader = new BufferedReader(read);
                String lineTxt = null;
                while((lineTxt = bufferedReader.readLine()) != null)
                    text=text + lineTxt;
                read.close();

            }
```

### FileUpload.java中的方法
* **public static String upload(MultipartFile file, String filePath)**方法，用于文件上传，	
	* file为前端向后传输文件，前端标签为 `<input type="file"></>`；
	* filePath为目标路径；

实际使用示例(下面为配置文件更新使用代码示例)：  
  
```
 public String configupload(@RequestParam("test") MultipartFile file) {
        String prefix = "D:\\NEXT\\searchengine\\configSave\\";
        String returnString = FileUpload.upload(file, prefix);
        String[] realFileName = file.getOriginalFilename().split("\\\\");
        String fileName = prefix + realFileName[realFileName.length-1];
        ImportExportHelper.setConfigFileName(fileName);
        if(!returnString.equals("文件为空")){
            try{
                ImportExportHelper.TNBSolrDataSourcesConfigParser();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return returnString;
    }
   
```

下面为上传单个文件更新使用示例：
	
```
    @RequestMapping(value="/upload-file", method=RequestMethod.POST)
    @ResponseBody
    public String fileupload(@RequestParam("file") MultipartFile file) {
        String prefix = "D:\\NEXT\\searchengine\\fileSave\\";
        String returnString = FileUpload.upload(file, prefix);
        String[] realFileName = file.getOriginalFilename().split("\\\\");
        String fileName = prefix + realFileName[realFileName.length-1];
        if(!returnString.equals("文件为空")){
            try{
                System.out.println(fileName);
                importFromFileImpl(fileName, true);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return returnString;
    }
```

### FileVisitor.java中的方法

* **public static void getDoc(String args, String doctype, boolean update)**方法，用于遍历文件目录并构建索引，或更新已建立索引并进行修改过的文件
	* args为目录地址；
	* doctype为构建索引的文件类型，默认为所有类型文件；
	* 是否开启更新，如果为true会对目录地址进行更新而不是从新构建；

使用示例：

对目录首次建立时使用：

```
	@RequestMapping(value="/import-dir", method=RequestMethod.POST)
    @ResponseBody
    public String importDir(String dir, String type) {
        try {
            documentIndex(dir, type, false);
            return "success";
        }catch (Exception e){
            e.printStackTrace();
            return "error";
        }
    }
```

对目录进行更新时使用：

```
	@RequestMapping(value="/update-dir", method=RequestMethod.POST)
    @ResponseBody
    public String updateDir(String dir, String type) {
        try {
            getDoc(dir, type, true);
            return "success";
        }catch (Exception e){
            e.printStackTrace();
            return "error";
        }
    }
}

```

### ImportExportHelper.java中的方法
* **public static void TNBSolrDataSourcesConfigParser()**方法，用于将配置文件中的内容导入并建立索引，调用 **public static void importData(@NotNull Document xmlDoc)**方法。

使用示例：    

```
 @RequestMapping(value = "/fullimporttest",method = RequestMethod.GET)
    @ResponseBody
    public void fullImport() throws Exception {
        ImportExportHelper.TNBSolrDataSourcesConfigParser();
    }

```

* **public static void importData(@NotNull Document xmlDoc)**方法，调用真正导入配置文件的 **public static void importData(@NotNull Document xmlDoc, boolean fromDatabases, boolean fromDirectory, boolean fromFile)**方法。

* **public static void importData(@NotNull Document xmlDoc, boolean fromDatabases, boolean fromDirectory, boolean fromFile)**方法
	* xmlDoc为将要导入的配置文件；
	* 其他几个参数为配置文件中包含几方面的即将建立索引类型，如果存在就将值设为true；
	* fromDatabases数据库类型数据，fromDirectory为从目录导入，fromFile为单个文件；
	* 上述三种数据类型分别调用了各自的导入数据方法，分别为public static void importFromDatabases(Document xmlDoc)、  public static void importFromDirectory(Document xmlDoc,boolean saveConfig)和public static void importFromFile(Document xmlDoc, boolean saveConfig)；



* **public static void exportToDatabaseConfig(String url, String user, String password, String table, Object[] objects)**方法，用于将对于某个数据表的内容建立索引保存到配置文件中；

* **public static void exportToDirectoryConfig(String url)**方法，用于将对于目录建立索引保存到配置文件中；

* **public static void exportToFileConfig(String url)**方法，用于将某个文件建立索引保存到配置文件中；

* 配置文件格式为：

```
数据库：

<database password="CVDEV2" url="jdbc:oracle:thin:@60.30.69.61:1521:adc" user="CVDEV2">
<table tableName="T_AUTOMAKER_INFO">
<column indexed="true" name="AUTOMAKER_ID" solrAlias="AUTOMAKER_ID"/>
<column indexed="true" name="AUTOMAKER_NAME" solrAlias="AUTOMAKER_NAME"/>
<column indexed="true" name="CREATE_TIME" solrAlias="CREATE_TIME"/>
</table>
</database>

文件：
file url="D:\NEXT\searchengine\fileSave\LICENSE.md"/>
<file url="D:\NEXT\searchengine\fileSave\毛概论文.pdf"/>

```


### loadFile.java中的方法
* **public static String loadFileService(String url)**方法，该方法用于在点击搜索结果展示框的时候将整个文件作为字节流传入前端使用
	* 如果文件建立索引与文件当前状态不一致，会进行更新，从而保证文件内容准确一致，如果文件应景被删除，会将索引删除，从而保持一致性。

实际调用示例：

```
 @RequestMapping("/loadFile")
    @ResponseBody
    public String loadFile(String url){
        return loadFile.loadFileService(url);
    }
```

### search.java中的方法：
* **public static HttpSolrServer getServer(String urlString)**方法，与solr服务器建立连接
	* 方法返回值为HttpSolrServer类型；
	* 传入的ulrString为本地solr服务器地址，默认情况下为		`http://localhost:8983/solr`        

实际调用代码：  

```     
		server = getServer(SOLR_URL);    
```    
* **public static String query_by_page(String type, String query, int start, int row, boolean hightlight)** 方法，用于进行查询
	* type为所要查询的文件类型，默认为全部类型；
	* query为查询的内容；
	* start与row为指定查询结果的起始位置以及个数；
	* higtlight为指定结果是否高亮，默认为true；   
  
在实际调用代码示例：    

```
	@ResponseBody   
    public String search(String keyWords, int startIndex, int step){  
        String returnString = query_by_page("md", keyWords, startIndex, step, true);  
        return returnString;    
  
    }     
     
```
返回值类型为json串，需要符合下面格式：

```
  @Return String the json string formatted as
     {
      "results":[
          {
             "title":"",
             "content":"",
             "information":"2017-06-12 00:00:00",
             "url":"C:\README.md"
          }
       ],
       "runOutFlag":[
           {
               "flag":"false"
           }
       ]
      }
     
      the flag is whether it is the end of the index
      */
```

* **public static String[] autoComplete(String field, String prefix, int min)**方法，用于查询内容的自动补全，方便用户的查询
	* prefix为前缀，即实时的输入内容；
	* min为最大返回结果数，开发人员自定义，尽量不要太大，即不方便显示，后面的相关性也比较差；
* public static void buildStructual(String database, String user, String pwd, String table, String[] fieldName, boolean saveConfig)方法，用于对于数据库这种结构性数据建立索引
	* database为数据库连接池，格式类似：	`jdbc:oracle:thin:@60.30.69.61:15211:qwewe`；
	* user、pwd代表数据库的用户名和密码；
	* table内容为建立索引的表名；
	* String[] fieldName为需要建立索引的字段集合；
	* 是否存到配置文件中；   

实际调用代码示例：  
   
```
 public String createIndex(String database, String table, String fieldName){
        String [] fieldArr = fieldName.split(";");
        if(fieldArr == null)
            return "fail";
        try{

            buildStructual(database, Search.getUser(), Search.getPwd(), table, fieldArr,true);
            return "success";
        }catch (Exception e){
            return e.getMessage();
        }
    }
```
返回值类型为json串，假设输入为a，返回为：

```
"["and","as","an","are","all"]"
```

* **public static void buildStructual(String database, String user, String pwd, String table, String[] fieldName, boolean saveConfig)**方法，用于对于结构化数据，如指定的数据表的列建立索引
	* database为数据库连接池，user和pwd为用户名和密码
	* table指定用户表，filedName指定建立索引的列集合
	* saveconfig指定是否存入配置文件中，方便以后的导入；

调用代码示例：

```
 */
    private static void importFromDatabasesImpl(String url, String user, String password, String table, Object[] objects) throws Exception {
        String [] strings = new String[objects.length];
        for(int i=0;i<strings.length;i++){
            strings[i] = objects[i].toString();
        }
        Search.buildStructual(url,user,password,table,strings,false);
    }

```

* **public static void deleteByQuery(String query)**方法，用于删除某些不在需要的索引
	* query内容为定义删除哪些索引；
* **public static void deleteById(String id)**方法，用于删除指定索引号的索引，id内容为索引号，使用该方法需要已知需要删除的索引号
*  **public static void findInDatabase(String user, String pwd, QueryResponse rsp)**方法，用户根据查询结果中的索引集合去原数据库查询完整内容
	* user和pwd为数据库的用户名和密码；
	* rsp为查询后返回的索引集合； 
  
实际使用示例：  
  
```
 try{
            QueryResponse rsp = solrServer.query(solrQuery);
            SolrDocumentList docs = rsp.getResults();
            findInDatabase(user,pwd,rsp);
            for(SolrDocument doc:docs){
                System.out.println(doc.get("text").toString());
            }

        }catch (Exception e){
            e.printStackTrace();
        }
```




