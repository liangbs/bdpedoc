# 5.前台查看日志详情查看的非上次运行的日志

> 问题描述：在前台web端点击“开启监控”后，在“历史日志”栏看到的日志不是job上次运行的历史日志。

### a.原因分析：
	代码逻辑里是根据jobID获取所有当前job的历史日志，然后取了第一个，所以出错。
	造成的原因是以前版本只保留一个历史日志文件，所以可以正确显示，而现在改为保留30个历史日志文件，所以每次都随机取一个，故而显示错误。
![image](/images/his_log_show_0.png) 

### b.解决办法：
	修改了代码逻辑，改为获取最新创建的那个历史日志文件。

### 相关代码：
##### 修改了下列代码：
![image](/images/his_log_show_1.png) 

	/*
	for (File file : allFiles)
	{
		logger.info("JOB[" + webMonitor.getJobId() + "]File[" + file + "]");
		
		//otherTimestamp = Long.valueOf(StringUtils.substring(baseName, 33, baseName.length()));
		logger.info("JOB[" + webMonitor.getJobId() + "]otherTimestamp[" + otherTimestamp + "]");
		// 当前时候JOB还未运行
		if (null == thisTimestamp)
		{
			jobHisLogFile = file;
			break;
		}
		// 当前JOB正在运行
		String baseName = FilenameUtil.getBaseName(file.getAbsolutePath());
		otherTimestamp = getJobTimestamp(baseName);

		if (otherTimestamp < thisTimestamp)
		{
			jobHisLogFile = file;
			break;
		}
		// 当前JOB运行失败了
		else if (otherTimestamp.equals(thisTimestamp))
		{
			jobHisLogFile = file;
			break;
		}
	}
	*/
	jobHisLogFile = this.getJobHisLogFile(allFiles);//注释上面for循环，改为这句
	logger.info("JOB[" + webMonitor.getJobId() + "]File[" + jobHisLogFile + "]");
##### 新增了下列代码：
![image](/images/his_log_show_2.png) 

	/**
	 * 取最新的日志文件
	 * @param allFiles
	 * @return
	 */
	private File getJobHisLogFile(File[] allFiles){
		File result = null;
		if (ArrayUtils.isNotEmpty(allFiles)){
			long distance = 0;
			long currTimestamp = System.currentTimeMillis();
			for (File file : allFiles){
				String baseName = FilenameUtil.getBaseName(file.getAbsolutePath());
				long jobTimestamp = getJobTimestamp(baseName);
				long dis = Math.abs(currTimestamp - jobTimestamp);//取绝对值
				if (distance == 0 || dis < distance){
					distance = dis;
					result = file;
				}
			}
		}
		return result;
	}