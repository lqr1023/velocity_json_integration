### velocity_json_integration   2018.9.3整理如下   
本项目主要完成json内容文件的读取和html变量值的替换  
## 关于velocity
velocity 可以在需要进行替换的模板里进行指定变量值的替换    
需要引入的maven文件如下：  
```
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
<dependency>
  <groupId>org.apache.velocity</groupId>
  <artifactId>velocity-engine-core</artifactId>
  <version>2.0</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.5</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-simple -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
    <scope>test</scope>
</dependency>
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.7</version>
</dependency>
<!-- https://mvnrepository.com/artifact/net.sf.json-lib/json-lib -->
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.4</version>
    <classifier>jdk15</classifier>
</dependency>
```
需要替换的文件：  aa.html
```
<div>aaa</div>
<p>b</p>  
```   
模板文件: demo.vm  注意这里自定义的key不能含有. .在velocity里有别的作用 之后再整理
```  
<div>${key1}</div> 
<p>${key2}</p>  
```   
java  
```

//linux 和 windows路径处理
public static  Path getPath(String first, String... more) {
   if(System.getProperty( "os.name" ).contains( "indow" ) && first.matches("^/\\w:.*")){
        first=first.substring(1);
   }
   return FileSystems.getDefault().getPath(first, more);
}
public void create(){
  //获取引擎
  VelocityEngine ve = new VelocityEngine();
  Path vmFile=getPath(vmPath);
  //异常处理
  if(Files.exists(vmFile)){
	htmlPath=vmFile.toAbsolutePath().toString();  
	System.out.println("模板文件:"+vmPath);  
  }else{
	System.out.println("未找到模板文件:"+vmPath);
  }
  //判断是绝对路径还是相对路径 绝对路径下需要设置FILE_RESOURCE_LOADER_PATH
  if(vmPath.startsWith("/")||vmPath.indexOf(":") > 0){
	ve.setProperty(VelocityEngine.FILE_RESOURCE_LOADER_PATH, "");
  }

  //编码设置
  ve.setProperty(VelocityEngine.ENCODING_DEFAULT, "utf-8");
  ve.setProperty("input.encoding", "utf-8");
  ve.setProperty("output.encoding", "utf-8");
  ve.init();

  Template t = ve.getTemplate(vmPath);
  VelocityContext ctx = new VelocityContext();
  ctx.put("key1","a");
  ctx.put("key2","b");
  //创建输出流 
	BufferedWriter writer = null;
  String outputPath = "xxx";
	try {
		writer = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(outputPath,false),"utf-8"));
		t.merge(ctx, writer);
		writer.flush();
		System.out.println("create successfully!");
	} catch (FileNotFoundException e) {
		 System.out.println("error!");
		 e.printStackTrace();
	} finally {
		 writer.close();
	}
}
```  
## 关于json文件  
判断json对象的类型,利用递归对json文件的内容进行读取,最终返回键值对，示例文件：  
``` 
{
  "str1":"a",                            key:root_str1  -> value:a
  "object1":{
  	array1:[
	  {
	  	"str2":"b",              key:root_object1_array1_0_str2    -> value:b
	  	"object2":{              
	  	   str3:"c"              key:root_object2_array1_0_object2_str3 ->value:c
		}
	  },
	  {
	  	
	  	"str2":"d",              key:root_object1_array1_1_str2    -> value:d
	  	"object2":{
	  	   str3:"e"              key:root_object1_array1_1_object2_str3 ->value:e
		}	
	   }
	]
  }
}
....   
依次类推   

java   
读取json字符串   

public static String readJsonFile(String Path) {
        BufferedReader reader = null;
        String laststr = "";
        try {
            reader = new BufferedReader(new InputStreamReader(new FileInputStream(Path), "UTF-8"));
            String tempString = null;
            while ((tempString = reader.readLine()) != null) {
                laststr += tempString + "\n";
            }
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (reader != null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return laststr;
    }
}

Object json = (Object)JSONObject.fromObject(FileUtil.readJsonFile(jsonPath));   

//所需静态变量
//拼接key
private static String KEY = "root";
//最后得到的map键值对
private static Map<String,String> m = new HashMap<String, String>();

public void getString(Object o){
     if(o instanceof JSONObject){
	JSONObject temp = JSONObject.fromObject(o);
	Set<String> keys = temp.keySet();
	Object[] a =  keys.toArray();
	for(int i = 0;i<a.length;i++){
		Object obj = temp.get(a[i].toString());
		//拼接key值
		KEY = KEY + "_" + a[i].toString();
		getString(obj);
	}
	//循环完成后 回退key值 到上一层 继续拼接 剩下的操作相同
	if(KEY.lastIndexOf("_")>0){
		KEY = KEY.substring(0, KEY.lastIndexOf("_"));
	}
    }else if(o instanceof JSONArray){
	JSONArray arrays = (JSONArray) o;
	for(int i=0;i<arrays.size();i++){
	    Object a = arrays.get(i);
	    KEY = KEY + "_" + i;
	    getString(a);
	}
	if(KEY.lastIndexOf("_")>0){
	   KEY = KEY.substring(0, KEY.lastIndexOf("_"));
	}
	}else{
	   m.put(KEY, o.toString());
	   if(KEY.lastIndexOf("_")>0){
		KEY = KEY.substring(0, KEY.lastIndexOf("_"));
	   }
	}
}   
```

