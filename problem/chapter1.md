# 1.时间触发配置触发器时，点击清除按钮，然后修改为条件触发，触发类型数据库中存储的不是1

> 问题描述：给job配置触发器的时候，一旦配置了时间触发器，就不能清除干净。

### a.原因分析：
	修改代码逻辑，在点击“清除”按钮时，清除报错的时间触发器，并修改的相关状态。

### b.解决办法：

##### 1.修改flex代码：修改JobAttrPanel.mxml文件，修改第582行代码如下：
	//===================begin======================
	case InitParamUtil.JOB_TRIGGER_FIXEDTIME:
	{//周期性定时
		if(_jobattrValue.condtion && _jobattrValue.condtion.jobs && _jobattrValue.condtion.jobs.length>0){//有条件触发
			if(timeBox.cycleTrigger.attr.condtions.length<1){
				_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_BEFORE_CONDITION;//条件触发
				_jobattrValue.time.cronExpression="";
			}else{
				_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_FIXECONDITION_COMB;//定时+条件触发
			}
		}else{//无条件触发
			_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_FIXEDTIME;//定时触发
		}
		break;
	}
	case InitParamUtil.JOB_TRIGGER_FREQTIME:
	{//频次
		if(_jobattrValue.condtion && _jobattrValue.condtion.jobs && _jobattrValue.condtion.jobs.length>0){//有条件触发
			if(timeBox.continuallyTrigger.attr.endStr=="000000" 
				&& timeBox.continuallyTrigger.attr.operateStatus==1 
				&& timeBox.continuallyTrigger.attr.timerId==null){
				_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_BEFORE_CONDITION;//条件触发
				_jobattrValue.time.cronExpression="";
			}else{
				_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_FREQCONDITION_COMB;//频次+条件触发
			}
		}else{//无条件触发
			_jobattrValue.jobTriggerType=InitParamUtil.JOB_TRIGGER_FREQTIME;//频次触发
		}
		break;
	}
	//====================end=====================

##### 2.修改java代码：修改ScheduleMgrServiceImpl.java文件里的saveTimeScheduleAttrOfBatchLock方法第734行如下：
	//===begin====
	if(newJobTriggerType == Constant.JOB_TRIGGER_BEFORE_CONDITION){//如果JobTriggerType为1，则为条件触发，删除时间属性
	    jobScheduleMap.deleteTimeSchedule(attr.getJobId());
	}else{//如果JobTriggerType不为1，更新时间属性
	    jobScheduleMap.updateJobTimeSchedule(attr);
	}
	//===end======
	//jobScheduleMap.updateJobTimeSchedule(attr);