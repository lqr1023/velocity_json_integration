### velocity_json_integration   2018.9.3整理如下
velocity 可以在需要进行替换的模板里进行指定变量值的替换   
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
