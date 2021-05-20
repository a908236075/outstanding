### 问题描述:动态获取项目路径,然后找到想要执行的shell脚本,进行执行,发现命令拼接完成后,用Runtime.exec执行,不会产生预期的结果.

~~~java
 public void dynGetPath() {
        String sysSh = "/springboot/script/systemStatus.sh";
        log.info("=====================cpu使用率!!!!!=============================");
        Process pro = null;
        Runtime r = Runtime.getRuntime();
        //获取项目跟路径
//        String path = System.getProperty("user.dir");
        // 动态的获取路径  sysSh=/springboot/script/systemStatus.sh
        String curPath = System.getProperty("user.dir");
//        String curPath = ClassUtils.getDefaultClassLoader().getResource("").getPath();
        log.info("获取当前目录 curPath: " + curPath);
        String[] split = curPath.split(Matcher.quoteReplacement(File.separator));
//        String[] split = curPath.split(File.separator);
        String rootPath = "";
        if (split.length > 1) {
            rootPath = File.separator + split[1];
        }
        log.info("rootPath: " + rootPath);
        // 拼接后想要的语句 find /home -path '*/springboot/script/systemStatus.sh'
        String pathCom = "find " + rootPath + " -path '*" + sysSh + "'";
        log.info("拼接的执行命令为: " + pathCom);
     	// 注意 注意 注意 !!!!!!!!!
        String[] commandArr=new String[]{"sh","-c",pathCom};
        try {
            pro = r.exec(commandArr);
            BufferedReader in = new BufferedReader(new InputStreamReader(pro.getInputStream()));
            String line = null;
            StringBuilder sb = new StringBuilder();
            while ((line = in.readLine()) != null) {
                sb.append(line);
            }
            sysSh = sb.toString();
            log.info("================result=======动态获取的路劲为: " + sysSh + "=============result==========");
            in.close();
            pro.destroy();
        } catch (Exception e) {
            log.error("动态获取路径信息发生异常: " + e.getMessage());
        }
        Process pro2 = null;
        Runtime r2 = Runtime.getRuntime();
        try {
            pro2 = r2.exec(sysSh);
            BufferedReader in = new BufferedReader(new InputStreamReader(pro2.getInputStream()));
            String line = null;
            StringBuilder sb = new StringBuilder();
            while ((line = in.readLine()) != null) {
                sb.append(line);
            }
            String result = sb.toString();
            log.info("cup使用率相关的result 为: " + result);
            in.close();
            pro2.destroy();
        } catch (Exception e) {
            log.error("获取信息发生Exception: " + e.getMessage());
        }

    }
~~~

**解决办法:需要使用  String[] commandArr=new String[]{"sh","-c",pathCom}; 将命令进行拼接后执行即可.**

