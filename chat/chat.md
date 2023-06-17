- 以下是我todoist hook触发计时器的代码
    - ```javascript
        Todoist.activitywatcher.set_trigger(hook,'item:updated',async data=>{
          //let event=new Gcal.Event()
          
          /*
          //google日历同步
          try{
            console.log(`----------------date.due is--`)
            console.log(data.due)
            event.get_gcal_from_todoist_and_ai_adjust(data.content,data.due.date,data.due.timezone,data.due.lang,data.due.is_recurring,data.priority).then(x=>{}).catch(e=>{console.log(e)})
          }
          catch(e){
            console.log(e)
            console.log(`due date error read`)
            console.log(data)
          }
          //google日历同步
          */
          
 
            if(hook.event_data.labels.includes('行动')==true){ //唯一标签触发

              // 防止重复触发
              let id1 = Jsserve.common.read_single_data_2_file('action_id');
              if (Number(data.id) === Number(id1)) {
                //console.log('重复action标签触发,中止执行');
                return console.log('重复action标签触发,中止执行');
              }
              else{
                console.log(`触发计时HOOK`,data.content)
              }
              
              //标签唯一化
              await Todoistapi.delete_all_except_choice_label(data.id, '行动');
              

              // 将 data 转换为与 task_info 匹配的格式
              let title1=new Regex_v2.Ai_Title(data.content)
              let data1=title1.property
              //console.log('data-------',data)
              let task_info = {
                todoist_task_id: data.id,
                todoist_project_id: data.project_id,
                todoist_task_name:data1.task,
                // 其他属性根据需要从 data 中提取
                total_time_limit: data1.timeLimitSet*3600??900,
                //daily_time_limit: 1200,
              };
              
              // 开启项目计时
              let timer_001 = Timer.Timer_001;
              await timer_001.start(task_info);

              //写入防止重复标签
              await Jsserve.common.write_single_data_2_file('action_id', data.id);
                

              

            }

          //当切换到测试时等同完成计时

        })```
- 这是我timer相关对象
    - ```javascript
class Timer_Goal {
  constructor(callback_start, callback_circle, callback_stop) {
    // callback function default to noop
    this.callback_start = callback_start || (() => {});
    this.callback_circle = callback_circle || (() => {});
    this.callback_stop = callback_stop || (() => {});
    this.mongo_collect_name = 'task_cost';
    this.mongo = new Mongo.Atlas_task_cost(this.mongo_collect_name);
    this.interval = null;
    //this.current_task_file = 'current_task';
  }

  // Add new methods for reading/writing timer state to local file.
  readTimerStateFromFile() {
    let read_result=Jsserve.common.read_single_data_2_file('pre_task');
    try{
      read_result=JSON.parse(read_result)
    }
    catch(e){
      return null
    }
    return read_result
  }

  writeTimerStateToFile(taskInfo) {
      let node1=new Jsserve.Node(taskInfo)
      let taskinfo=node1.toString()
      return Jsserve.common.write_single_data_2_file('pre_task', taskinfo);
  }

  async start(task_info) {
    // Get the current timer state from file.
    const previousTask = this.readTimerStateFromFile();
    if (previousTask) {
        // Stop the previous timer and pass its info to the stop callback.
        await this.stop(previousTask);
    }

    // Initialize/reset all task data
    this.todoist_task_id = task_info.todoist_task_id;
    this.todoist_project_id = task_info.todoist_project_id;
    this.miro_task_id = task_info.miro_task_id;
    this.miro_board_id = task_info.miro_board_id;
    this.total_time_limit = task_info.total_time_limit;
    this.daily_time_limit = task_info.daily_time_limit;
    this.todoist_task_name=task_info.todoist_task_name,
    this.miro_task_name=task_info.miro_task_name,
    this.total_time = 0;
    this.single_time = 0;
    this.daliy_time = 0;
    this.total_money = 0;
    this.mongo_iden_id = null;

    // fetch task data from db
    let task_cost = await this.mongo.read(task_info);
    //console.log('拉取taskcost',task_cost,task_info)
    //console.log('mongo读取的信息----------------------------',task_cost)
    if (!task_cost) {
      // if no task data in db, create new one
      task_cost = await this.mongo.ai_write(task_info);
    }

    // start timer and update properties
    this.total_time = task_cost.total_time || 0;
    //console.log('拉取totaltime',task_cost.total_time,this.total_time)
    this.single_time = 0;
    this.daliy_time = task_cost.daliy_time || 0;
    this.total_money = task_cost.total_money || 0;
    this.mongo_iden_id = task_cost._id;
    
    try{
      //console.log('启动时写入 的taskinfo------------------',this.getTaskInfo())
    await this.callback_start(this.getTaskInfo());
    }catch(e){
      console.log(e?.message)
    }
    console.log(`${this.todoist_task_id}, total time: ${this.total_time}/limit:${this.total_time_limit}, 计时器启动`);

    let counter = 0;
      this.interval = setInterval(async () => {
          this.single_time += 15;
          this.total_time += 15;
          this.daliy_time += 15;
          //console.log('测试中断-------------------------------------')
          //console.log('运行中写入 的taskinfo',this.getTaskInfo())

          await this.mongo.ai_write(this.getTaskInfo());

          counter++;
          if (counter === 4) {
              console.log(`${this.todoist_task_id}, total time: ${this.total_time}, 计时器运行中`);
              counter = 0;
          }

          try{
            await this.callback_circle(this.getTaskInfo());
          }
          catch(e){
            console.log(e?.message)
          }
          this.writeTimerStateToFile(this.getTaskInfo());
        }, 15000);

  }

  async stop(previousTask) {
    console.log('计时器开始停止',previousTask)
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
  
      // 重置计时器
      this.total_time = 0;
      this.single_time = 0;
      this.daliy_time = 0;
      this.total_money = 0;
      this.todoist_task_id = null;
      this.todoist_project_id = null;
      this.miro_task_id = null;
      this.miro_board_id = null;
      this.total_time_limit = null;
      this.daily_time_limit = null;
      this.mongo_iden_id = null;
      this.todoist_task_name=null;
      this.miro_task_name=null;
  
      try{
        await this.callback_stop(previousTask);
      }catch(e){
        console.log(e?.message)
      }
  
      console.log(`计时器已停止，所有信息已重置。`);
  
      // 在重置后写入数据库
      //await this.mongo.ai_write(this.getTaskInfo());
    }
  }
  

  getTaskInfo() {
    return {
      todoist_task_id: this.todoist_task_id,
      todoist_project_id: this.todoist_project_id,
      todoist_task_name:this.todoist_task_name,
      miro_task_name:this.miro_task_name,
      miro_task_id: this.miro_task_id,
      miro_board_id: this.miro_board_id,
      total_time: this.total_time,
      single_time: this.single_time,
      daliy_time: this.daliy_time,
      total_money: this.total_money,
      total_time_limit: this.total_time_limit,
      daily_time_limit: this.daily_time_limit,
      mongo_iden_id: this.mongo_iden_id,
    };
  }
}


Timer.Timer_Goal=Timer_Goal

//计时器实例
// 用于更新任务标题的函数
async function updateTaskTitle(task_info,stop_flag) {
  let todoist=new Todoist_v2.TodoistAPI
  let Ai_Title=Regex_v2.Ai_Title
  let task = await todoist.read(task_info.todoist_task_id);
  let taskTitle = new Ai_Title(task.content);
  let totalTimeLimit = taskTitle.property.timeLimitSet*3600||task_info.total_time_limit||0;
  let total_time = task_info.total_time? task_info.total_time : 0;

  if (totalTimeLimit > 0) {
      const hoursLimit = Math.floor(totalTimeLimit/3600);
      const minutesLimit = Math.floor(totalTimeLimit%3600/60);

      let timeLimit = "";
      if (hoursLimit > 0) {
          timeLimit += `${hoursLimit}h`;
      }
      if (minutesLimit > 0 || timeLimit === "") {
          timeLimit += `${minutesLimit}m`;
      }

      const hoursDuration = Math.floor(total_time/3600);
      const minutesDuration = Math.floor(total_time%3600/60);

      let duration = "";
      if (hoursDuration > 0) {
          duration += `${hoursDuration}h`;
      }
      if (minutesDuration > 0 || duration === "") {
          duration += `${minutesDuration}m`;
      }

      taskTitle.update_value({
          timeLimit: timeLimit,
          duration: duration
      });
  } else {
      const hoursDuration = Math.floor(total_time/3600);
      const minutesDuration = Math.floor(total_time%3600/60);

      let duration = "";
      if (hoursDuration > 0) {
          duration += `${hoursDuration}h`;
      }
      if (minutesDuration > 0 || duration === "") {
          duration += `${minutesDuration}m`;
      }

      taskTitle.update_value({
          duration: duration,
          timeLimit: '15m',
      });
  }

  //判断是否停止 
  let newTitle
  //console.log(' 判断是否停止----------------------------',stop_flag,task_info)
  if(stop_flag==true){
    newTitle = taskTitle.out_put_title();
    console.log('stop------------------title',newTitle)
  }
  else{
    newTitle = '⌛'+taskTitle.out_put_title();
  }
  await todoist.update(task_info.todoist_task_id, {content: newTitle});
}

function new_timer() {
  const todoist = new Todoist_v2.TodoistAPI();
  const toggltrack = Toggltrack.togglentry.start_action_entry_view_total_v8;
  const timer = new Timer_Goal(
      // timer start callback
      async function(task_info) { //开始时回调
          try {
              // 使用新的updateTaskTitle函数来更新任务标题
              await updateTaskTitle(task_info);
              console.log('task_info.todoist_task_name',task_info)
              // 启动toggl追踪
              toggltrack(task_info.todoist_task_name, task_info.total_time , task_info.total_time_limit);
          } catch (error) {
              console.error('An error occurred in start callback:', error);
          }
      },
      // timer circle callback
      async function(task_info) { //运行中回调
          try {
              // 使用新的updateTaskTitle函数来更新任务标题
              await updateTaskTitle(task_info);
              console.log(`计时器任务 ${task_info.todoist_task_name}运行中---`)
          } catch (error) {
              console.error('An error occurred in circle callback:', error);
          }
      },
      // timer stop callback
      async function(task_info) { //停止时回调
          try {
              // 使用新的updateTaskTitle函数来更新任务标题
              await updateTaskTitle(task_info,true);
          } catch (error) {
              console.error('An error occurred in stop callback:', error);
          }
      }
  );
  return timer;
}

  
  // 辅助函数，将秒数转换为 "h小时m分" 的格式
  function formatTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    return `${hours}h${minutes}m`;
  }

Timer.Timer_001=new_timer()
  



//导出
module.exports=Timer```
- 以下是我的循环托管服务器对象
    - ```javascript
const express = require('express');
const bodyParser = require('body-parser');
const history = require('connect-history-api-fallback');
const path = require('path');
const https = require('https');
const axios = require('axios');
const fs = require('fs');
const cors = require('cors');
const schedule = require('node-schedule');
const sdk = require('api')('@miro-ea/v2.0#jbaw3n6li1osaw6');
const Miroserve = require('./miroserve');
const Miro_api = Miroserve.Miro_api;
const ProjectCard = Miroserve.Project_card;
const BigPicture = require('./BigPicture').BigPicture;


//

//循环服务
/*
使用方式：

创建一个循环托管服务器对象 LoopServer 的实例，命名为 serverCircle。
创建一个循环服务对象 LoopService 的实例，传入一个需要管理的实例和循环间隔。例如，示例中创建了一个需要管理的大图片服务实例 bigPictureService，循环间隔为 5 分钟。
将循环服务实例添加到循环托管服务器的服务列表中，使用 addService 方法。可以添加多个循环服务实例。
调用 startAllServices 方法，启动所有的循环服务。每个循环服务会按照设置的循环间隔执行相应的任务（通过调用实例的 manage 方法）。
循环托管服务器会持续运行，定时执行每个循环服务的任务，直到进程结束或手动停止。
*/



class LoopService {
  constructor(runInstance,methodname, loopInterval) {
    this.runInstance = runInstance; // 运行实例
    this.loopInterval = loopInterval; // 循环间隔
    this.methodname = methodname;
  }

  start() {
    //开始
      
    console.log(`
    CCCCC   -RRRRR-  UU   UU  NNN   NN
    CC        RR   RR  UU   UU  NNNN  NN
    CC        RRRRRR   UU   UU  NN NN NN
    CC        RR  RR   UU   UU  NN  NNNN
    CCCCC   RR   RR   UUUUU   NN   NNN
    `);

  
    // 创建定时任务，按照循环间隔执行任务
    schedule.scheduleJob(`*/${this.loopInterval} * * * *`, async () => {
      try {
        if ( this.methodname) {
          let instance=new this.runInstance();
          //let fun=instance[this.methodname];
          await instance[this.methodname](); // 执行运行实例的 manage 方法
        }
      } catch (err) {
        console.error(err);
      }
    });
  }
}


class LoopServer { //
  constructor() {
    this.loopServices = []; // 存储循环服务的数组
  }

  addService(service) {
    this.loopServices.push(service); // 添加循环服务到数组
  }

  startAllServices() {
    this.loopServices.forEach(service => service.start()); // 启动所有循环服务
  }
}

const serverCircle = new LoopServer();

// 示例：添加一个需要管理的大图片服务实例，该实例应包含一个 manage 方法。
const bigPictureService = new LoopService(BigPicture,'manageProjects', 5);
serverCircle.addService(bigPictureService);

// 添加更多服务实例...

// 开始所有服务
serverCircle.startAllServices();

```
- 完成我如下要求
    - 目前的代码的功能是,hook返回变更的todoist任务,如哦todoist任务含有行动标签则触发timer计时,并删除其他行动标签
    - 要改成
        - 变更的任务不经由todoist hook触发,而是由循环代理服务器每隔五分钟拉取所有的todoist任务并检测最新的含有行动标签的任务,然后再进入同样的计时流程(如删除其他任务的行动标签并开始计时)
