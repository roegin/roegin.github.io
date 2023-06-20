/*
 * @Author: roegin roe.gin@qq.com
 * @Date: 2022-04-19 01:35:17
 * @LastEditors: roegin roe.gin@qq.com
 * @LastEditTime: 2023-05-15 23:11:34
 * @FilePath: \miro-sync-todoist-v1\src\miroserve.js
 * @Description: 
 * 
 * Copyright (c) 2022 by roegin roe.gin@qq.com, All Rights Reserved. 
 */
//var express = require('express')
//var bodyparser = require('body-parser')
import axios from 'axios'
import { Miro } from './miro';
import { Jsserve,Jsserve_api } from './jsserve';

//jssop
import org from 'org'
//import { Title } from './regex';
import {Title,Regex_text,Miro_mind_title, Regex, Title_goal} from './regex'
import { Todoist_task } from './todoist';
import { Axios_plus } from './axios_plus';
import { Asana } from './asana';

//引入JSDOM引入DOCUMENT
//import jsdom from "jsdom"
//var JSDOM = jsdom.JSDOM;
//var document = new JSDOM(body).window.document;

//import element from 'element'
//import document from 'document'
var jsserve=new Jsserve
//var express = require('express')
//var axios = require('axios')

//miromec定义
let miromec={}
//let Miro={}
miromec.widget={}
miromec.widget.wrotemeta=async function(cardid,key,value){
  let metadata={}
  console.log('metadata') //调试
  console.log(cardid) //调试
  console.log(await miro.board.widgets.get({id:cardid}) ) //调试

  //根据id取回widget
  let widgets=await miro.board.widgets.get({id:cardid})
  let widget=widgets[0]
  console.log(widget) //调试

  //修正meta写入流程
  let widgetmetadata = widget.metadata['3458764523466410540']   
  if(!widgetmetadata){
    widget.metadata['3458764523466410540']={} 
    widgetmetadata=widget.metadata['3458764523466410540']
    console.log(widget.metadata['3458764523466410540'])
  }
  
  console.log('wiget meta') //调试
  console.log(widgetmetadata) //调试

  //写入要写入的参数到metadata
  widgetmetadata[key]=value

  //复写修正过的metadata
  widget.metadata['3458764523466410540']=widgetmetadata

  console.log(widget.metadata) //调试

  //将复写值写回节点
  await miro.board.widgets.update(widget)
}

//getmeta


miromec.board={}
miromec.board.inil=async function(boardinfo){
  //boardinfo=miromec.board.getboardinfo()
  if(!!miromec.board.getboardinfo()){
    await miro.board.widgets.update([
      {id:boardinfo.id , text:'boardinfo'+'\n'+'boardinfo.id:'+boardinfo.id+' '+'boardinfo.des:'+boardinfo.description},
      
  
    ])


  }
  else{
    await miro.board.widgets.create([
        {type: 'sticker', text:'boardinfo'+'\n'+'boardinfo.id:'+boardinfo.id+' '+'boardinfo.des:'+boardinfo.description},
        
    
      ])
  }
}

//读取boardinfo备份
miromec.board.getboardinfo=async function(){
  let stickers= await miro.board.widgets.get({type: 'sticker'});
  stickers.filter(x=>{
    return !!x.plainText.match(/boardinfo/g)
  })
  let boardinfosticker=stickers[0]
  let boardinfo={}
  boardinfo.id=/boardinfo.id:(.*?)(\s|$)/g.test(str) ? RegExp.$1 : null;
  boardinfo.description=/boardinfo.des:(.*?)(\s|$)/g.test(str) ? RegExp.$1 : null;
  return boardinfo
}

//get toggl token
miromec.oauth2auth={}
miromec.oauth2auth.togglauth={}
miromec.oauth2auth.togglauth.get=function(){
  return new Promise((resolve,reject)=>{
    //https://alex.shinestu.com:3002/togglplan/auth
    axios.get('https://alex.shinestu.com:3002/togglplan/auth '
      ,{
        responseType:'document'
      
      }
    )
    .then(async res => {
      console.log(res.data)
      let htmldoc=res.data
      //htmlDoc.innerhtml(res.data);
      //显示AUTH页面
      miro.board.ui.openModal('./auth.html', { width: 400, height: 400 }).then(() => {
        miro.showNotification("modal closed");
      });

      //console.log(res)
    })
    .catch(function (error) {
      // handle error
      console.log(error);
    })
    .then(function () {
      // always executed
    });
  })
}
miromec.alex_serve_api=function (action,data){
  return new Promise((resolve,reject)=>{
    //let tempproject={project:this.inputentry}
    //console.log(this.apiobject.name)
    axios({
      headers: {
        //Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
        'Content-Type':'application/json',
        },  
      method: 'post',
      url: `https://alex.shinestu.com/miroserve`,
      data: {
        action:action,
        data:data
        /*{
          title:title_data.pure_text,
          dueString:`${title_data.start_date} ${!!title_data.start_time?title_data.start_time:''}`,
          statue:title_data.statue,
          miro_link_text:task.out_put_link(),

          }*/
        }
      }
    )
    .then(res => {
      //console.log(`--------------------测试中断---------------------`)
      //console.log(JSON.stringify(res.data))
      

      resolve(console.log(`TODOIST API 链接完成`));
    })
    .catch(error=>{
      // handle error
      console.log(` TODOIST APIerror`)
      console.log(error.response?.data)

      

      //console.log(this.inputproject)
      //this.inputproject.name=(this.inputproject.name+' 重名修正')
    // reject()
      //reject(this.inputentry.name)
      
    })
    .then(function () {
      // always executed
    });
})
}

class miro_mind{
  constructor(data){
    this.init=function(mind){

    }
    this.init(mind)



  }
}

//触发器
class Mind_trigger{
  constructor(trigger_statue,callback,if_mul_trigger,if_on_load_all){
      this.trigger_statue=trigger_statue
      this.callback=callback
      this.if_on_load_all=if_on_load_all||this.if_on_load_all||false
      this.if_mul_trigger=if_mul_trigger||this.if_mul_trigger||false

      //增加刷新监控触发
      miro.addListener('SELECTION_UPDATED', async(widgets) => {
        
        //获得全部对象
        let selectedItems=widgets.data
        let fullstack= new Array
        for( let x in selectedItems)
        {
          //console.log('x.id')
          //console.log(selectedItems[x].id)
          let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
          let fullobject=fullobject0[0]
          //
          //console.log('get object is')
          //console.log(fullobject)
          //
          fullstack.push(fullobject)
        }
        
        //过滤mindnode
        fullstack=fullstack.filter(node=>node.type=='TEXT'   )     

        let mindnodes=fullstack;
        //-----------

        //过滤单触发
        //console.log(this.if_mul_trigger)
        if(this.if_mul_trigger==false){
          mindnodes=mindnodes.filter(node=>{
            let tags=node.plainText.match(/#/g)
            let tag_num=tags?.length
            //console.log(tag_num)
            try{
            if(tag_num>1){
              return false

            }
            }
            catch(e){}
            return true


        })

        }


        //--------------------------


        //触发回调
        mindnodes=mindnodes.filter(node=>trigger_statue(node))
        if(
        JSON.stringify(mindnodes) != '[]'
        ){
          for(let x in mindnodes){

            callback(mindnodes[x])
          
           }
        }


      })
      //

      //增加刷新监控触发
      async function onAllWidgetsLoaded(temp_callback) {
        const areAllWidgetsLoaded = await miro.board.widgets.areAllWidgetsLoaded()
        if (areAllWidgetsLoaded) {
          temp_callback()
        } else {
          miro.addListener('ALL_WIDGETS_LOADED', temp_callback)
        }
      }

      if(this.if_on_load_all===false){return}
      onAllWidgetsLoaded(async() => { //tempcallback
        
        
        //console.log(`----------------------------#tag click--------------------------------`)
        let mindnodes=await miro.board.widgets.get({type: 'TEXT'})

        //触发回调
        mindnodes=mindnodes.filter(node=>trigger_statue(node))
        if(
        JSON.stringify(mindnodes) != '[]'
        ){
          for(let x in mindnodes){

            callback(mindnodes[x])
          
           }
        }

        

      
        
      
      })
      //

      
    }
}

miromec.Mind_trigger=Mind_trigger    

//miroserve定义
function miroserve()
{


    //mirowidget
    

    let boardinfo
    let boardpara={}
    this.boardinfoget=async function(){
     // boardinfo =await (async function(){
        
        boardinfo=await miro.board.info.get();//let boardinfo=await miro.board?.info?.get();
        console.log('--------MIRO板块信息读取完成--------')
        console.log(boardinfo)
      /* if(!!boardinfo){
           //wroteboardinfo2widget,boardinfoinil
          miromec.board.inil(boardinfo)
          return boardinfo
        }
      })()??await (async function(){
        return await miromec.board.getboardinfo()
      })();
      })()*/

     

      //getboardpara
      boardpara.togglprojectid=(function(){
        let regextogglprojectid=/togglprojectid:(.*?)(\s|$)/g
        return regextogglprojectid.test(boardinfo.description)?RegExp.$1 : null;

      })()
      //miro.showNotification('toggl project id is '+boardpara.togglprojecti)
      return boardpara.togglprojectid
      
      }
    this.board_data_get=async function(){
      // boardinfo =await (async function(){
          
          let board_data=await miro.board.info.get();//let boardinfo=await miro.board?.info?.get();
          return board_data
        
    }
    this.board_des_get=async function(){
      let boardinfo=await miro.board.info.get()
      return boardinfo.description
    }
    //this.boardinfoinit=async function(){
      //this.boardinfo=await miro.board.info.get()
    //}//
    //20220425 miroserve.watch
    this.watch=watch
    //this.filtag=filtag
    //20220427
    this.creathabitodo=creathabitodo
    this.multicreathabitodo=multicreathabitodo
    this.mirostack2goalstack=mirostack2goalstack
    //20220425
    this.nodestack=new Array();
    let todoisttasks=new Array();
    this.savestack= function(stack){
      this.nodestack=stack
    };
    this.creatmiroboard=creatmiroboard
    this.multicreatmiroboard=async function(goalstack){
      for (let x in goalstack){
        //console.log(goalstack[x].title+goalstack[x].id)
        
        //2022-05-03 获得节点前置线条
        if(goalstack[x].type=='CARD'){
          await this.creatmiroboard(goalstack[x].title,goalstack[x].metadata,goalstack[x].id)
          /*
          let line = await miro.board.widgets.get({endWidgetId:goalstack[x].id,type:"LINE"})
          line[0].style.lineColor='#B75EFF'
          
          await miro.board.widgets.update({
            id:line[0].id,
            style:line[0].style
            //style:{lineColor:'#B75EFF'},

          })
          console.log('lineis')
          console.log(line)
          */
          //console.log(goalstack[x].title)
          //取消project
          
          //取消project end
          

        }
        ////2022-05-03 获得节点前置线条 end

      }
    }
    async function creatmiroboard(projectname,metadata,cardid){
      //console.log(todoisttask.reqstatue);
      //if(req.body.event_name='update'){
         // return
      //}
      //2022-5-07 start
      //let projectid
      try{
        //let metadata0=mindnode.metadata['3458764523466410540']
          if(!metadata.projectid){
            throw new Error('project id blank');
          }
        }
        catch(err)
        {
          console.log('error')
          let newtext=projectname.replace('#project','')   
          let creatresultdata=await (function(){    
            return new Promise((resolve,reject)=>{
                
                  //let regexshortdate=/(?<=-).*$/mi
                  
                  axios({url:'https://alex.shinestu.com:3002/miroserve/creatmiroboard',
                  //headers:{'Content-Type':'text/plain'},
                  method:'post',
                  data:{projectname:newtext}
                    
                  })
                  .then(res => {
                    //console.log(JSON.stringify(res.data))
                    resolve(res.data)
                  })
                  .catch(function (error) {
                    // handle error
                    console.log(error);
                  })
                  .then(function () {
                    // always executed
                  });
            })
          })()

          
          projectname.replace('#project','') 
          //创建URL
          let des=(function(){
            let link='https://miro.com/app/board/'+creatresultdata.id+'/';
            let text='<p><a href='+link+">PROJECT-LINK</a></p><p><br></p>";
            return text
          })()
          console.log('update'+cardid)
          await miro.board.widgets.update({
            id:cardid,
            
            title:'<p>'+newtext+'</p>',
            

            description:des,
            metadata:{
              '3458764523466410540':{
                projectid:creatresultdata.id,
                boardid:boardinfo.id
              }
            }

          })
          console.log('updete finnished')
                             
        }
      //2022-05-07 end 
      /*
      let boardquery=(await function(projectid){
                  return new Promise((resolve,reject)=>{
                    
                    let regexshortdate=/(?<=-).*$/mi
                    
                    axios({
                      headers: {
                        Authorization: 'Bearer eyJtaXJvLm9yaWdpbiI6ImV1MDEifQ_2msHtUIgmZ2RSElTVIjeJnHKr3M',
                        'Content-Type':'application/json',
                      },
                      method: 'get',
                      url: 'https://api.miro.com/v2/boards/'+projectid,
                      /*
                      data: {
                        name:projectname+' #board:project',
                      
                        "policy": {
                            "permissionsPolicy": {
                                  "copyAccess": "anyone"
                            },
                            "sharingPolicy": {
                                  "access": "private",
                                  "inviteToAccountAndBoardLinkAccess": "no_access",
                                  "organizationAccess": "private",
                                  "teamAccess": "private"
                            }
                        },
                        "description": "11",
                        "teamId": "3074457364465016759"
                        } // *

                        
                        
                      
                    })
                    .then(res => {
                      //console.log(JSON.stringify(res.data))
                      //console.log(res)
                      resolve(res);
                    })
                    .catch(function (error) {
                      // handle error
                      resolve('error')
                      console.log(error);
                    })
                    .then(function () {
                      // always executed
                    });
                  })
              
      })()
      if(boardquery.code=='error'){
        return 'error'

      } 
      */
      
      
    }  
    this.gettasks=function(){
      return new Promise((resolve,reject)=>{
          axios({
              headers: {
              Authorization: 'Bearer e0c87d823e4ae4493bcfb04d8c13fbbb496da628',
              'Content-Type':'application/json',
              },
              method: 'get',
              url: 'https://api.todoist.com/rest/v1/tasks',
              /*data: {
              content: con,
              description:des,
              project_id:id,
              due_date:due,
              
              }*/
          })
          .then(res => {
              //console.log(JSON.stringify(res.data))
              //console.log(res)
              //获取网址
              //console.log(document.URL);
              //
              todoisttasks=res.data;
              resolve(res);


          })
          .catch(function (error) {
              // handle error
              console.log(error);
          })
          .then(function () {
              // always executed
          });
      })
    } 
    this.creattask=creattask  //change2gcal
    function creattask(con,des,id,due) {
      return new Promise((resolve,reject)=>{
            axios({
              headers: {
                Authorization: 'Bearer e0c87d823e4ae4493bcfb04d8c13fbbb496da628',
                'Content-Type':'application/json',
              },
              method: 'post',
              url: 'https://api.todoist.com/rest/v1/tasks',
              data: {
                content: con,
                description:des,
                project_id:id,
                due_date:due,
                
              }
            })
            .then(res => {
              //console.log(JSON.stringify(res.data))
              //console.log(res)
              resolve(res);
            })
            .catch(function (error) {
              // handle error
              console.log(error);
            })
            .then(function () {
              // always executed
            });
      })
    };
    this.updatetask=function(id,con,pjid,due){
      return new Promise((resolve,reject)=>{
        
        axios({
          headers: {
            Authorization: 'Bearer e0c87d823e4ae4493bcfb04d8c13fbbb496da628',
            'Content-Type':'application/json',
          },
          method: 'post',
          url: 'https://api.todoist.com/rest/v1/tasks/'+id,
          data: {
            content: con,
            //description:des,
            project_id:pjid,
            due_date:due,
            
          }
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          //console.log(res)
          resolve(res);
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        })
        .then(function () {
          // always executed
        });
      })
    }
    this.deletetask=function(id){
      return new Promise((resolve,reject)=>{
        
        axios({
          headers: {
            Authorization: 'Bearer e0c87d823e4ae4493bcfb04d8c13fbbb496da628',
            'Content-Type':'application/json',
          },
          method: 'delete',
          url: 'https://api.todoist.com/rest/v1/tasks/'+id,
          
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          //console.log(res)
          resolve(res);
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        })
        .then(function () {
          // always executed
        });
      })
    }
    this.listname2id={
      miro:'2289847097',
      inbox:'2275239200',
      日历:'2289823583',
    }
    this.wrotetodoist=async function(mindnodestack){

      let regexmiroid=/(?=miro-id:).*?(?=\\n|\s|$)/mi // i g m
      await this.gettasks()
      for(let x in mindnodestack) //对
      {
        let mindnode=mindnodestack[x]

        //获取end
         //跳过无ID无DATE项目
         //console.log('now')
        if(!mindnode.actiondate){
         
          try{
          let metadata0=mindnode.metadata
          let todoisttaskid0=metadata0.todoisttaskid
          }
          catch(err)
          {
            continue;
          }
          //删除有ID无DATE的项目
          let metadata1=mindnode.metadata
          let todoisttaskid1=metadata1.todoisttaskid;
          this.deletetask(todoisttaskid1);
          continue;
        }
        
        let todoistdes=(function(){
          let link='https://miro.com/app/board/'+boardinfo.id+'/?moveToWidget='+mindnode.id+'&cot=14'
          let text='--------\nmiro-id:'+mindnode.id+'\nmiro-board-id:'+boardinfo.id+'\n[miro-link]('+link+')\n--------';
          return text
        })()
        

        //mindnodedate
        //console.log(regexdate.exec(mindnode.text)[0]);
        let mindnodedate=mindnode.actiondate.replace(/(\d{4})(\d{2})(\d{2})/g,'$1-$2-')      

        //确认node在todoist 的存在情况
        let todoitem=todoisttasks.find(a=>{
            //console.log(a.description)
            try{
            return regexmiroid.exec(a.description)[0].replace(/miro-id:(.*?)/,'$1')==mindnode.id
            }
            catch(err)
            {
              return false
            }
          })
        //console.log(todoitem)

        //判断mindnode在todoist 的存在情况,并进行对应操作
        if(
          !!todoitem
        )
          {
            //存在则更新 
            let metadata=mindnode.metadata
            let todoisttaskid=metadata.todoisttaskid
            await this.updatetask(todoisttaskid,mindnode.puretitle,'2289847097',mindnodedate)
          }
        else
          //todo不存在下的操作,创建
          {
            //判断日期存在与否
            if(!!mindnodedate)
            {
              //miro.board.showNotification('creat'+mindnodedate)
              let main=mindnode.puretitle              
              if(!mindnode.deadline)
              {
                main=main+actiondate
              }

              //创建TODOIST TASK
              let res=await this.creattask(main,todoistdes,'2289847097',mindnodedate)

              //更新MIRO脑图节点颜色
              //增加对不同类型的更新处理 202220420
              await(async function updateback(){

                //对单行输入处理
                if(mindnode.type=='TEXT')
                {
                  await miro.board.widgets.update(
                    {
                      id:mindnode.id,
                      text:'<p>'+main+' '+'<span style=\"background-color:rgb(230,230,230);color:rgb(128,128,128)\">'+mindnode.actiondate+'</span></p>',
                      metadata:{
                        '3458764523466410540':{
                          todoisttaskid:res.data.id,
                          boardid:boardinfo.id
                        }
                      }
                    }
                  )
                }
                else if(mindnode.type=='CARD')
                {
                  await miro.board.widgets.update(
                    {
                      id:mindnode.id,
                      title:'<p>'+main+' '+'<span style=\"background-color:rgb(230,230,230);color:rgb(128,128,128)\">'+mindnode.actiondate+'</span></p>',
                      metadata:{
                        '3458764523466410540':{
                          todoisttaskid:res.data.id,
                          boardid:boardinfo.id
                        }
                      }
                    }
                  )
                }
                else
                {

                }
              })()
              //更新end
              console.log('update completed '+mindnode.puretitle)
            }
            //
            //await this.creattask(mindnode.)
            //console.log('failed')
          }
        //
        
      }
    }
    //定义区
    this.multiope=async function(func,goalstack){
      for (let x in goalstack){
        //console.log(goalstack[x].title+goalstack[x].id)
        await func(goalstack[x])
        //2022-05-03 获得节点前置线条
        
        ////2022-05-03 获得节点前置线条 end

      }
    }


    async function watch(tag){ //观察选择行为,根据设定的标签调用对应函数 | return tagstack
      miro.addListener('SELECTION_UPDATED', async(widget) => {
       
        ////如果输入函数为空中止
        /*if(!func){
          return
        }*/
        //
        let selectedItems=widget.data;
        //
        console.log(selectedItems)
        //获得完整对象
        let fullstack= new Array
        for( let x in selectedItems)
        {
          //console.log('x.id')
          //console.log(selectedItems[x].id)
          let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
          let fullobject=fullobject0[0]
          //
          //console.log('get object is')
          //console.log(fullobject)
          //
          fullstack.push(fullobject)
        }
        //
        //console.log(fullstack)
        //获得该标签下对象
        let regextag=/#\S*(?=(\s|$|\\n))/gi;
        let tagstack =new Array
        for(let x in fullstack){
          //console.log(fullstack[x])
          if(fullstack[x].type=='CARD')
          {
            if(regextag.test(fullstack[x].title)==true){
              let tagexec=(jsserve.filterhtml(fullstack[x].title)).match(regextag).find(input=>input==tag);
              if(!!tagexec)
              {
                tagstack.push(fullstack[x])
              }
            }
          }
          else if(fullstack[x].type=='TEXT')
          {
            
            if(regextag.test(fullstack[x].text)==true){
              //console.log(fullstack[x].text.match(regextag)) //打印所有标签
              let tagexec=(jsserve.filterhtml(fullstack[x].text)).match(regextag).find(input=>input==tag);
              
              if(!!tagexec)
              {
                tagstack.push(fullstack[x])
              }
            }
          }
          else
          {

          }
        }
        //console.log(tagstack)
        console.log('tag is '+tag)
        console.log(tagstack.length)
        if(tagstack.length!=0) //满足条件的TAG集合存在
        {
         if(tag=='#ac'){ //#ac的运行函数和处理
          //
          console.log(tag)
          console.log('tagstackis')
          console.log(tagstack)
          //
          let goalstack=this.mirostack2goalstack(tagstack)
          //console.log('mirocircle')
          //console.log(goalstack)
          await this.multicreathabitodo(goalstack)//func(goalstack); 
          //2022-05-02
          
         }
         //2022-05-04
          
          
         if(tag=='#project'){
          console.log('tagstackis') //调试
          console.log(tagstack) //调试
          let goalstack=this.mirostack2goalstack(tagstack)
          await this.multicreatmiroboard(goalstack)
         }

         if(tag=='#accom'){
          let goalstack=this.mirostack2goalstack(tagstack)
          await this.multiope(this.scoredhabi,goalstack)
         }
         //2022-05-04-end
        }
        //console.log('tagstack is')
        //console.log(tagstack)

        

        //this.filtag(this.wrotetodoist,'#sync',para)

        //console.log(widget)
      })
      
      
      //添加MIRO监视器监视选择操作
      

      
      //func(para)
    }
    
    

    function creathabitodo(title,aliasid){ //goalsynchabitido
      //await (function(){
        return new Promise((resolve,reject)=>{
          let des=(function(){
            let link='https://miro.com/app/board/'+boardinfo.id+'/?moveToWidget='+aliasid+'&cot=14'
            //let text='--------\nmiro-id:'+aliasid+'\nmiro-board-id:'+boardinfo.id+'\n[miro-link]('+link+')\n--------';
            let text='[miro-link]('+link+')\n------------';
            return text
          })()
          let mirotag=new Array
          mirotag[0]='a0413f69-4eaf-47ff-8e5f-f97573cfc124'
          axios({
            headers: {
              'Content-Type': 'application/json',
              'x-client':'23d75568-fa21-4411-be2d-c70d748e1628-mirosynchabitica',
              'x-api-user':'23d75568-fa21-4411-be2d-c70d748e1628',
              'x-api-key':'ea9547ef-6a26-452b-974a-594f9c0771a7'
            },
            method: 'post',
            url: 'https://habitica.com/api/v3/tasks/user',
            data: {
              text:title,
              type:'todo',
              alias:aliasid,
              notes:des,
              tags:mirotag,
            }
          })
          .then(async res => {
            //await miromec.widget.wrotemeta(aliasid,'habiticataskid',aliasid)
            resolve(res);
          })
          .catch(function (error) {
            // handle error
            console.log(error.response.data);
          })
          .then(function () {
            // always executed
          });
        })
      //})()

      
    }
    //creathabitodo('testmiroserve','dsfdsfse23')
    async function multicreathabitodo(goalstack){
      
      for (let x in goalstack){
        console.log(`goalstack[x].title is ${goalstack[x].title}+goalstack[x].id`) //调试中
        await this.creathabitodo(goalstack[x].title,goalstack[x].id)
        //2022-05-03 获得节点前置线条
        if(goalstack[x].type=='TEXT'){
          /*
          let line = await miro.board.widgets.get({endWidgetId:goalstack[x].id,type:"LINE"})
          line[0].style.lineColor='#B75EFF'
          
          await miro.board.widgets.update({
            id:line[0].id,
            style:line[0].style
            //style:{lineColor:'#B75EFF'},

          })
          console.log('lineis')
          console.log(line)
          */
          //console.log(goalstack[x].title)
          //取消ac
          let newtext=goalstack[x].title.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
          newtext=newtext.replace(/^/,'#目标\/进行中 ') 
          //goalstack[x].title.replace('#ac','') 
          await miro.board.widgets.update({
            id:goalstack[x].id,
            
            text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'

          })
          //取消ac end
          

        }
        ////2022-05-03 获得节点前置线条 end

      }
    }

    function mirostack2goalstack(mirostack){
      let goalstack=new Array
      let goalstacknozero=new Array
      /*let regexmiroid=/(?=miro-id:).*?(?=\\n|\s|$)/mi // i g m
      //let regexdate=/(?<=\);\">).*?(?=<\/span>)|(?<=\d\d-\d\d\s)\d.*?\d(?=<\/p>)/mi
      let regexplainmain=/.*\s(?=(\d\d.*\d\d$))/mi
      let regexplaindate=/(\s\d\d\d\d|\s\d\d-\d\d|\s\d\d\d\d-\d\d-\d\d)$/mi
      //let mindnodestack= nodestack.filter(widget=>widget.type === 'TEXT'||widget.type=='CARD')*/
      
      for(let x in mirostack){

        //跳过非CARD和NODE元素
        if((mirostack[x].type!='TEXT')&&(mirostack[x].type!='CARD')){
          continue;
        }

        //获取初始标题
        let plaintext

        //判断类型不同从不同位置提取文本
          if (mirostack[x].type=='TEXT')
          {
            plaintext=mirostack[x].plainText
          }
          else if(mirostack[x].type==='CARD')
          {
            plaintext=jsserve.filterhtml(mirostack[x].title)
          }
          else
          {

          }
        
       
        

        console.log('plaintest') //调试
        

        goalstack[x]={};
        goalstack[x].title=plaintext;

         //获取原始标题
        //取消原正则 let regexpuretitle=/.*?(?=(\s(\d{8}\s\d{4}|\d{4})))/g
        let regexpuretitle=/(.*?)(((\s(\d{8}|\d{4})\s)|\s)(\d{4})($|\s\d{2}:\d{2}-\d{2}:\d{2})|$)/g    
        goalstack[x].puretitle=plaintext.replace(regexpuretitle,'$1')
        console.log('replace1') //调试
        console.log('puretitle is'+goalstack[x].puretitle) //调试
        
        //获得显示截止时间
        let regexdeadline=/(\s(\d{8}|\d{4})(\s|$))/g
        if(!!plaintext.match(regexdeadline)){
          console.log('time') //调试
          goalstack[x].showdeadline=(plaintext.match(regexdeadline)[0]).match(/(\d{8}|\d{4})/g)[0]
          console.log('regexdeadline') //调试
        

        //获得记录截止时间
          if(!goalstack[x].showdeadline.match(/\d{8}/g)){
            goalstack[x].deadline=jsserve.getyear()+(plaintext.match(regexdeadline)[0]).match(/(\d{8}|\d{4})/g)[0]
          }
          else
          {
            goalstack[x].deadline=goalstack[x].showdeadline
          }
        

          //获取执行日期
          let regexactiondate=/(\d{4}(\s|$))/g
          if(!!plaintext.match(regexactiondate)){           
            goalstack[x].showactiondate=plaintext.match(regexactiondate).pop().match(/(\d{8}|\d{4})/)[0]

            //获得记录执行日期
            goalstack[x].actiondate=jsserve.getyear()+goalstack[x].showactiondate
          }

          //获取带截止日期标题
          goalstack[x].titlewithdeadline=goalstack[x].puretitle+' '+goalstack[x].showdeadline
        }

        console.log('开始时间') //调试
        //获取开始时间
        let regextimetext=/\d{2}\:\d{2}-\d{2}\:\d{2}/g
        if(!!plaintext.match(regextimetext)){   
          let regextime=/\d{2}\:\d{2}/g
          goalstack[x].starttime=plaintext.match(regextimetext).match(regextime)[0]

          //获取结束时间
          goalstack[x].endtime=plaintext.match(regextimetext).match(regextime).pop() //alert
        }

        goalstack[x].id=mirostack[x].id
        goalstack[x].type=mirostack[x].type
        goalstack[x].metadata=mirostack[x].metadata['3458764523466410540']??{}

        /*//调试 MIRO2GOAL META
        console.log('miro2goal metadate is')
        console.log(mirostack[x].metadata)
        console.log(goalstack[x].metadata) //调试*/

        //goalstack[x].startdate=goalstack[x].actiondate
        //goalstack[x].starttime
        goalstack[x].actionenddate
        
        goalstack[x].color="#db9a9a"
        goalstack[x].estimatedhours
        goalstack[x].focused

        //获得状态
        goalstack[x].status=(plaintext=>{
          if(!!plaintext.match(/#goal\/finished\s|#目标\/完成\s|#完成/g)){
            return "completed"
          }
          else if(!!plaintext.match(/#goal\/doing\s/g)){
            return "doing"
          }
          else{
            return "todo"
          }
        })(plaintext)

        //获得projectid
        goalstack[x].projectid

        
        /*
        console.log(goalstack[x])//调试
        console.log(goalstack[x].metadata)*/

        //console.log('goalstack circle')
        //console.log(goalstack[x])
        goalstacknozero.push(goalstack[x])
          
      }

      


      

      return goalstacknozero
      
      
    }


    //filgoal
    function fildatechangegoal(goalstack){
        
        return goalstack.filter(goal=>{
          return goal?.actiondate!=goal.metadata?.goalcalidata?.actiondate||goal?.deadline!=goal.metadata?.goalcalidata?.deadline||goal?.puretitle!=goal.metadata?.goalcalidata?.puretitle
        })
      }
      
    


    //2022-05-04 start
    /**
     * @description: 
     * goal=[id,title],only v1mindmap
     * @param {*} goal
     * @return {*}
     */
    this.scoredhabi=async function(goal){

      //跳过非脑图节点
      /*
      if(goal.type!='TEXT'){
        return;
      }
      */


      //验证ATLAS存在
      /*
      if((!goal.metadata.habiticataskid)&&(goal.type=='TEXT')){

        //为空值时创建habi
        await creathabitodo(goal.title,goal.id)

      }
      */

      //if(goal.type=='TEXT'){
        
        //取消ac
        /*
        let newtext=goal.title.replace('#accom','') 
        goal.title.replace('#accom','') 
        */
        //复写
        /*
        await miro.board.widgets.update({
          id:goal.id,
          
          text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'

        })
        */
        //取消ac end
        

      //}

      await (function(){return new Promise((resolve,reject)=>{
        
          axios({
            headers: {
              'Content-Type': 'application/json',
              'x-client':'23d75568-fa21-4411-be2d-c70d748e1628-mirosynchabitica',
              'x-api-user':'23d75568-fa21-4411-be2d-c70d748e1628',
              'x-api-key':'ea9547ef-6a26-452b-974a-594f9c0771a7'
            },
            method: 'post',
            url: 'https://habitica.com/api/v3/tasks/'+goal.id+'/score/up',
          
          })
          .then(res => {
            //console.log(JSON.stringify(res.data))
            console.log(res.data)
            resolve(res.data);
          })
          .catch(function (error) {
            // handle error
            console.log(error.response);
          })
          .then(function () {
            // always executed
          });
        })
      })()
      
      
    }
    
    //2022-05-04 end

    //创建单个TOGGLEPLAN
    function creattogglplan(goal){
      let tptask=goal2tptask(goal)
      tptask.project_id=goal.projectid
      return new Promise((resolve,reject)=>{
        axios({
          headers: {
            Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
            'Content-Type':'application/json',
          },
          method: 'post',
          url: 'https://api.plan.toggl.com/api/v5/847322/tasks',
          data: {
            "name": tptask.name,
            "start_date": tptask.start_date,
            "end_date": tptask.end_date,
            "start_time": tptask.start_time,
            "end_time": tptask.end_time,
            //"color": "#9575CD",
            "estimated_hours": '0.25',
            "pinned": false,
            "status": tptask.status,
            "workspace_members": [
              7636620
            ],
            "project_id": tptask.project_id

            //tags:'dfb6eea7-9e26-40a1-a0e3-e7c545ef8df8',
          }
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          console.log('togglid '+res.data.id)
          resolve(res.data.id);
        })
        .catch(function (error) {
          // handle error
          console.log(error.response);
        })
        .then(function () {
          // always executed
        });
      })
    }

    //更新单个TOGGL-plan
    function updatatogglplan(goal){
      let tptask=goal2tptask(goal)
      tptask.project_id=goal.projectid
      return new Promise((resolve,reject)=>{
        axios({
          headers: {
            Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
            'Content-Type':'application/json',
          },
          method: 'put',
          url: 'https://api.plan.toggl.com/api/v5/847322/tasks/'+goal.metadata.togglplanid,
          data: {
            "name": tptask.name,
            "start_date": tptask.start_date,
            "end_date": tptask.end_date,
            "start_time": tptask.start_time,
            "end_time": tptask.end_time,
            //"color": "#9575CD",
            "estimated_hours": '0.25',
            "pinned": false,
            //"status": tptask.status,
            "workspace_members": [
              'roe.gin'
            ],
            "project_id": tptask.project_id

            //tags:'dfb6eea7-9e26-40a1-a0e3-e7c545ef8df8',
          }
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          //console.log(res)
          resolve(res.data.id);
        })
        .catch(function (error) {
          // handle error
          console.log(error.response);
        })
        .then(function () {
          // always executed
        });
      })
    }

    //与服务器通讯
    function send2alex(meth,value){
      return new Promise((resolve,reject)=>{
                
        //let regexshortdate=/(?<=-).*$/mi
        
        axios({url:'https://alex.shinestu.com:3002/miroserve/'+meth,
        //headers:{'Content-Type':'text/plain'},
        method:'post',
        data:{para:value}
          
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          resolve(res.data)
        })
        .catch(function (error) {
          // handle error
          console.log(error);
        })
        .then(function () {
          // always executed
        });
      })
    }

    //写入BOARD参数
    async function wroteboardpara(name,para){
      //根据NAME生成JSON
      let jsonpara
      if (name='togglprojectid'){
        jsonpara={togglprojectid:para,boardid:boardinfo.id}
        //jsonpara.togglprojectid
      }
      
      
      await send2alex('updatamiroboarddes',jsonpara)

    }

    //同步BOARD和PROJECT
    async function synctogglproject(){
      console.log('synctogglprojectstated &toggl projectid is '+boardpara.togglprojectid) //调试
      if(!!boardpara.togglprojectid){
        return boardpara.togglprojectid
      }
      else
      {
        console.log(boardinfo.title) //调试
        let projectid=await creattogglproject(boardinfo.title)
        await wroteboardpara('togglprojectid',projectid)
        boardpara.togglprojectid=projectid
        return projectid

        //wroteback
        //getpara
        /*let regexpara={}
        regexpara.togglprojectid=/(?=togglprojectid:).*(\n|$)/mg
        boardpara.togglprojectid=boardinfo.description.match(regexpara.togglprojectid)[0].replace('togglprojectid:','')*/
        

      }
    }

    function fixgoalwithdate (goal){
      if(goal.title.search(/\d\d-\d\d\s\d\d-\d\d/g)!=-1){
        let date=goal.title.match(/\d\d-\d\d/g)
        goal.title=goal.title+' '+date
        return goal

      }
    }


    //同步单个GOAL到TOGGL
    this.synctoggltask=async function(goal){
      console.log('synctoggltask') //调试
      goal.projectid =await synctogglproject();
      console.log('syncprojectoutpro')
      //let goalfil=fixgoalwithdate(goal);
      console.log(goal?.metadata)
      if(!!goal.metadata.togglplanid){ //存在UPDATA
        
        updatatogglplan(goal)
        
      }
      else
      {
        let togglplanid=await creattogglplan(goal)
        console.log(togglplanid) //调试
        await miromec.widget.wrotemeta(goal.id,'togglplanid',togglplanid)
      }

      //recordmeta
      let goalx=Object.assign({}, goal)
      function filcalidata(goalx){
        delete goalx.metadata
        return goalx
      }

      console.log('write cali') //调试
      await miromec.widget.wrotemeta(goal.id,'goalcalidata',filcalidata(goalx))

    }

    
    this.filtergoalwithdate=function(goalstack){
      let regexplainmain=/.*\s(?=(\d\d.*\d\d$))/mi
      let regexplaindate=/(\s\d\d\d\d|\s\d\d-\d\d|\s\d\d\d\d-\d\d-\d\d)$/mi
      return goalstack.filter(function(goal){
        return goal.search(regexplaindate)!=-1
      })
    }

    

    

    

    

    function creattogglproject (projectname){
      return new Promise((resolve,reject)=>{
        /*
        let des=(function(){
          let link='https://miro.com/app/board/'+boardinfo.id+'/?moveToWidget='+aliasid+'&cot=14'
          //let text='--------\nmiro-id:'+aliasid+'\nmiro-board-id:'+boardinfo.id+'\n[miro-link]('+link+')\n--------';
          let text='[miro-link]('+link+')\n------------';
          return text
        })()*/
        axios({
          headers: {
            Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
            'Content-Type':'application/json',
          },
          method: 'post',
          url: 'https://api.plan.toggl.com/api/v5/847322/projects',
          //params:{},
          data: {
            name:projectname,
            //color:'#9575CD',
            //tags:'dfb6eea7-9e26-40a1-a0e3-e7c545ef8df8',
          }
        })
        .then(res => {
          //console.log(JSON.stringify(res.data))
          //console.log(res)
          resolve(res.data.id);
        })
        .catch(function (error) {
          // handle error
          console.log(error.response);
        })
        .then(function () {
          // always executed
        });
      })
    }
    
    

    //转换goal对象为toggleplantask对象
    function goal2tptask (goal){
      let tptask={}
      tptask.name=goal.puretitle

      //转换为start_date
      let regexline=/(\d{4})(\d{2})/g      
      tptask.start_date=goal.actiondate.replace(regexline,'$1-$2-')
      console.log('replace2') //调试

      //转换为end_date
      tptask.end_date=tptask.start_date
      //转换为start_time
      if(!!goal.starttime){
      tptask.start_time=goal.starttime
      }
      //转换为end_time
      if(!!goal.endtime){
        tptask.start_time=goal.endtime
        }
      //转换为color
      
      //转换为estimated_hours

      //转换为pinned
      //转换为status
      if(!!goal.status)
      {
        //转换状态
        tptask.status=(function(){
          if(goal.status=='completed')
          {return 'done'}
          else if(goal.status=='doing')
          {return 'in-progress'}
          else if(goal.status=='todo')
          {return 'to-do'}
          
          
        })()??'to-do'
      }
      //转换为workspace_members

      //转换为project_id
      //输出返回
      return tptask

    }
    
    this.watchdate=async function (){ //观察date标签 | return tagstack

      //增加刷新监控触发
      async function onAllWidgetsLoaded(callback) {
        const areAllWidgetsLoaded = await miro.board.widgets.areAllWidgetsLoaded()
        if (areAllWidgetsLoaded) {
          callback()
        } else {
          miro.addListener('ALL_WIDGETS_LOADED', callback)
        }
      }
      onAllWidgetsLoaded(async() => {
        
       
        //获得所有对象
        let cards=await miro.board.widgets.get({type: 'CARD'})
        let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
        let fullstack=cards.concat(mindnodes)


        /*//获得选择对象
        let selectedItems=widget.data;
        console.log(selectedItems)*/

        //从选择对象获得完整对象
        /*
        let fullstack= new Array
        for( let x in selectedItems)
        {
          let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
          let fullobject=fullobject0[0]
          fullstack.push(fullobject)
        }*/
        console.log('watchdate-fullstack-checked is') //调试
        console.log(fullstack)

        //获得有日期的下对象
        let regexdate=/((.*)((\s(\d{8}|\d{4})\s)|\s)(\d{4})($|\s\d{2}:\d{2}-\d{2}:\d{2}))/g;
        let goalstack=mirostack2goalstack(fullstack)

        console.log('goalstack is') //#调试
        console.log(goalstack) //调试


        let datestack=goalstack.filter(goal=>!!goal.title.match(regexdate)
        )
        console.log('date is ') //调试
        console.log(datestack) //调试

        console.log(datestack.length)
        console.log('datestack checked')//调试
        

        
        

        //对存在的集合操作
        if(datestack.length!=0) 
        {

          //过滤获取变更的集合
          datestack=fildatechangegoal(datestack)
          console.log('filter datestack is ')
          console.log(datestack)

          //对变更的日期集合写入到toggleplan
          this.multiope(this.synctoggltask,datestack)

          //对变更的日期集合写入到todoist
          //this.wrotetodoist(datestack)

          console.log('已经写入todoist')
      
        }
        
      })

      //选择触发
      /*
      miro.addListener('SELECTION_UPDATED', async(widget) => {
       
        //获得选择对象
        let selectedItems=widget.data;
        console.log(selectedItems)

        //从选择对象获得完整对象
        let fullstack= new Array
        for( let x in selectedItems)
        {
          let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
          let fullobject=fullobject0[0]
          fullstack.push(fullobject)
        }
        console.log('watchdate-fullstack-checked is') //调试
        console.log(fullstack)

        //获得有日期的下对象
        let regexdate=/((.*)((\s(\d{8}|\d{4})\s)|\s)(\d{4})($|\s\d{2}:\d{2}-\d{2}:\d{2}))/g;
        let goalstack=mirostack2goalstack(fullstack)

        console.log('goalstack is') //#调试
        console.log(goalstack) //调试


        let datestack=goalstack.filter(goal=>!!goal.title.match(regexdate)
        )
        console.log('date is ') //调试
        console.log(datestack) //调试

        console.log(datestack.length)
        console.log('datestack checked')//调试
        

        
        

        //对存在的集合操作
        if(datestack.length!=0) 
        {

          //过滤获取变更的集合
          datestack=fildatechangegoal(datestack)
          console.log('filter datestack is ')
          console.log(datestack)

          this.multiope(this.synctoggltask,datestack)
      
        }
        
      })*/
      
       
      
    }

    /*
    this.watch_tag=async function (){ //观察date标签 | return tagstack

      //增加刷新监控触发
      async function onAllWidgetsLoaded(callback) {
        const areAllWidgetsLoaded = await miro.board.widgets.areAllWidgetsLoaded()
        if (areAllWidgetsLoaded) {
          callback()
        } else {
          miro.addListener('ALL_WIDGETS_LOADED', callback)
        }
      }
      onAllWidgetsLoaded(async() => {
        
        
        //console.log(`----------------------------#tag click--------------------------------`)
        let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
      
        
        //console.log(mindnodes)
        //获得tag为#目标的集合
        let mindnodes_withtags_goal=mindnodes.filter(node=>{
          //console.log(node)
          return !!node.text.match(/#目标\s/)
        })

        //console.log(mindnodes_withtags_goal)
          //清除并覆写特定tag
        if(JSON.stringify(mindnodes_withtags_goal) != '[]'){
          for( let x in mindnodes_withtags_goal){
            let node=mindnodes_withtags_goal[x]
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            console.log('replace #目标')
            newtext=newtext.replace(/^/,'#目标 ') 
            //goalstack[x].title.replace('#ac','') 
            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color:transparent;color:rgb(45,155,240)\">'+newtext+'</span></p>' 

            })
          }
        }//
          


        //活儿都tag为#目标/进行中的集合
        let mindnodes_withtags_doing=mindnodes.filter(node=>{
          return (!!node.text.match(/#目标\/进行中\s/))&&(!node.text.match(/color:rgb\(219,174/))
        })
          //清除并覆写特定tag

        if(JSON.stringify(mindnodes_withtags_doing) != '[]'){
          for( let x in mindnodes_withtags_doing){
            let node=mindnodes_withtags_doing[x]
            console.log(node)
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            console.log(newtext)
            console.log('replace #目标/进行中')
            newtext=newtext.replace(/^/,'#目标\/进行中 ') 
            //goalstack[x].title.replace('#ac','') 


            //创建Habi对像
            //this.creathabitodo(node.plainText,node.id)
            

            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'

            })
          }
        }

        //获得tag为#目标/完成的集合
        let mindnodes_withtags_done=mindnodes.filter(node=>{
          return (!!node.text.match(/#目标\/完成\s/))&&(!node.text.match(/color:rgb\(230,249,148\)/))
        })
          //清除并覆写特定tag
        
        if(JSON.stringify(mindnodes_withtags_done) != '[]'){
          for( let x in mindnodes_withtags_done){
            let node=mindnodes_withtags_done[x]
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            newtext=newtext.replace(/^/,'#目标\/完成 ') 
            //goalstack[x].title.replace('#ac','') 

            //完成HABI对象
            this.creathabitodo(node.plainText,node.id).then(()=>{
              let node_repair={
                id:node.id,
                title:node.plainText,
              }
              this.scoredhabi(node_repair)
              console.log(`--------${node.plainText} is Completed at Habitica!--------`)
            })
            

            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color:rgb(230,249,148);color:rgb(0,0,0)\">'+newtext+'</span></p>',

            })


          }
        }

      
       
      
      })
    }*/
    
    this.watch_click_tags=async function(){  //错误的,是点击图标触发这个
      //console.log(`----------------------------#tag click--------------------------------`)
      let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
    
      
      //console.log(mindnodes)
      //获得tag为#目标的集合
      let mindnodes_withtags_goal=mindnodes.filter(node=>{
        //console.log(node)
        return !!node.text.match(/#目标\s/)
      })

      //console.log(mindnodes_withtags_goal)
        //清除并覆写特定tag
      if(JSON.stringify(mindnodes_withtags_goal) != '[]'){
        for( let x in mindnodes_withtags_goal){
          let node=mindnodes_withtags_goal[x]
          let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
          console.log('replace #目标')
          newtext=newtext.replace(/^/,'#目标 ') 
          //goalstack[x].title.replace('#ac','') 
          //改变颜色
          await miro.board.widgets.update({
            id:node.id,
            
            text:'<p><span style=\"background-color:rgb(170,216,255);color:rgb(0,0,0)\">'+newtext+'</span></p>'

          })
        }
      }//
        


      //活儿都tag为#目标/进行中的集合
      let mindnodes_withtags_doing=mindnodes.filter(node=>{
        return (!!node.text.match(/#目标\/进行中\s/))&&(!node.text.match(/color:rgb\(219,174/))
      })
        //清除并覆写特定tag

      if(JSON.stringify(mindnodes_withtags_doing) != '[]'){
        for( let x in mindnodes_withtags_doing){
          let node=mindnodes_withtags_doing[x]
          console.log(node)
          let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
          console.log(newtext)
          console.log('replace #目标/进行中')
          newtext=newtext.replace(/^/,'#目标\/进行中 ') 
          //goalstack[x].title.replace('#ac','') 

            //创建Habi对象
            
            //创建HABI对象
            //this.creathabitodo(node.plainText,node.id)
        //改变颜色
          await miro.board.widgets.update({
            id:node.id,
            
            text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'

          })
        }
      }

      //获得tag为#目标/完成的集合
      let mindnodes_withtags_done=mindnodes.filter(node=>{
        return (!!node.text.match(/#目标\/完成\s/))&&(!node.text.match(/color:rgb\(230,249,148\)/))
      })
        //清除并覆写特定tag
      
      if(JSON.stringify(mindnodes_withtags_done) != '[]'){
        for( let x in mindnodes_withtags_done){
          let node=mindnodes_withtags_done[x]
          let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
          newtext=newtext.replace(/^/,'#目标\/完成 ') 
          //goalstack[x].title.replace('#ac','') 




          //完成HABI对象
          this.creathabitodo(node.plainText,node.id).then(()=>{
            let node_repair={
              id:node.id,
              title:node.plainText,
            }
            this.scoredhabi(node_repair)
            console.log(`--------${node.plainText} is Completed at Habitica!--------`)
          }) //



          //改变颜色
          await miro.board.widgets.update({
            id:node.id,
            
            text:'<p><span style=\"background-color:rgb(230,249,148);color:rgb(0,0,0)\">'+newtext+'</span></p>',

          })



        }
      }
    }
    this.click_to_get_summary=async function(){
      
    }
    this.on_click_icon=async function(board_data){  //是点击图标触发这个 ,同步到ASANA
      
      //console.log(`----------------------------#tag click--------------------------------`)
      //节点树生成
      let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
      let lines=await miro.board.widgets.get({type: 'LINE'})

      return `节点数目:${mindnodes.length}/1000`

      //停止同步树
      return 
      // 1. Original data
      async function init_filtered_tree_from_cloud_rest_v1(tree_filter_func){
        /*
        树的生成更改
        1.data的 获取
        2.根节点的获取 
        3.get son node的函数
        4.fitel_func(node)=>true通过,false不通过
        */

        let data=mindnodes.concat(lines)
        
    
        //如果data不存在则返回错误
        if (!data){return false} 
        //---------------------------------------------------------------------------------
        //获取脑图部分
    
    
        //生成带根节点的同步堆栈
        class mec_miro_get_rootnode_v1{
          constructor(all_items_stack_v1){
              this.items_stack=all_items_stack_v1
              this.goal_ids=[]
              this.line_end_ids=[]
              this.root_nodes_ids
              
          }
          get_first_root_node=function(){
              //获得目标节点集合
              for(let x in this.items_stack){
                  if(this.items_stack[x]?.type=='TEXT'){
                      this.goal_ids.push(this.items_stack[x].id)
                  }
              }
              //console.log(this.goal_ids)
              //----------------
    
              //获得线的终点集合
              for(let x in this.items_stack){
                  if(this.items_stack[x]?.type=='LINE'){
                      this.line_end_ids.push(this.items_stack[x].endWidgetId)                }
              }
              //console.log(this.line_end_ids)
              //---------------
    
              //获得根节点id
              let a=this.goal_ids
              let b=this.line_end_ids
              this.root_nodes_ids =  a.filter(function(v){ return b.indexOf(v) == -1 }) // [1,3,4,5]
              //console.log(this.root_node_data)
              //-------------------
              if(!!this.root_nodes_ids[0]){
                  let first_root_node=this.items_stack.find(node=>{
                      return node.id==this.root_nodes_ids[0]
                  })
                  return first_root_node
    
              }else{
                  return false
              }
              
              
    
          }
    
        }
        let mec_miro_get_rootnode_v1_1=new mec_miro_get_rootnode_v1(data)
        let root_node=mec_miro_get_rootnode_v1_1.get_first_root_node()
    
        let sync_stack=[]
    
        sync_stack.push(root_node)
        //生成带根节点的同步堆栈
    
    
        //定义获取子节点函数
        /*
        function get_son_nodes(fnode,nodes){
            let son_nodes=[]
            return son_nodes=nodes.filter(node=>node.pid==fnode.id)
        }
        */
        class mec_miro_get_sonnodes_rest_v1{
          constructor(all_items_stack_v1){
              this.items_stack=all_items_stack_v1
              this.goal_ids=[]
              this.line_end_ids=[]
              this.root_node_data
          }
          get_son_nodes=function(parent_node){
              //console.log(this.items_stack)
              //获取子节点id集合
              let son_nodes_ids=[]
              this.items_stack.filter(node=>{
                  if((node.type=='LINE')&&(node.startWidgetId==parent_node.id)){
                      //console.log(node)
                      son_nodes_ids.push(node.endWidgetId)
                      return true
                  }
                  else{
                      return false
                  }
              })
              //onsole.log(son_nodes_ids)
              //-------------------------------------
    
              //获取子节点集合
              let son_nodes=[]
              this.items_stack.every(node=>{
                  //console.log(!!son_nodes_ids.find(id=>{return node.id==id}))
                  if(!!son_nodes_ids.find(id=>{return node.id==id})){
                      son_nodes.push(node)
                  }
                  return true
              })
    
              
              //console.log(son_nodes)
              return son_nodes
              //------------------------
    
    
          }
        }
        
        
        
        //定义获取子节点函数
    
    
        //定义筛选函数
        let filter_func=tree_filter_func
        function filter_func_default(node){
            /*
            let title1=new Title_goal( node.text)
            let data1=title1.out_put_data()
            if(!!data1.start_date){
                return true
            }
            else{
                return false
            }
            */
           return true
          
    
            
        }
        if(!filter_func){
          filter_func=filter_func_default
        }
        //定义筛选函数
    
    
        //console.log('分割线-----------------------------')
        let mec_miro_get_sonnodes_rest_v1_1=new mec_miro_get_sonnodes_rest_v1(data)
        //识别并在同步队列里加入子节点
        function push_son_nodes_2_sync_stack(key_node,node_stack){
    
    
            //形成初始遍历队列
            let bfs_queue=[]
            //bfs_queue.push(key_node)
    
    
            let first_son_nodes=mec_miro_get_sonnodes_rest_v1_1.get_son_nodes(key_node)
            first_son_nodes.every(node=>{
                bfs_queue.push(node)
                return true
            })
            //形成初始遍历队列
    
    
            //特殊函数定义
    
    
    
            //特殊函数定义
    
    
            for(let x=0;!!bfs_queue[x];x++){
                
    
    
                let node=bfs_queue[x]
    
                //console.log(`测试中断`)
                //console.log(node)
    
                let filter_result=filter_func(node)
                //console.log(filter_result)
    
                if(filter_result==true){
                    //写入新树父节点并推送到同步堆栈
                    
                    let new_node=node
                    delete new_node.pid
                    new_node.fid=key_node.id
                    sync_stack.push(new_node)
                    
                    //写入新树父节点并推送到同步堆栈
                }
                else{
    
                    let s_nodes=mec_miro_get_sonnodes_rest_v1_1.get_son_nodes(node)
                    //let s_nodes=get_son_nodes(node,node_stack)
    
                    s_nodes.every(node=>{
                        bfs_queue.push(node)
                        return true
                    })
                }
    
    
                
            }
    
        }
    
        //识别并在同步队列里加入子节点
    
        //循环加入子节点
        for(let x=0;!!sync_stack[x];x++){
            push_son_nodes_2_sync_stack (sync_stack[x])//,data)
        }
        //循环加入子节点
    
    
        //----------------------------------------------
    
    
    
    
        return sync_stack
        //console.log(data)
      }
      let data = await init_filtered_tree_from_cloud_rest_v1()

      
      //console.log(data)
      // 2. Function to get root node
      function get_root(data) {
        return data.find(node => !node.fid);
      }


      // 4. Filter function 1
      function filter_1(node0) {
        //return true
        const input = node0.plainText;
        const node = new Jsserve_api.Miro_mindmap_node_plaintext(input); // 创建一个实例并传入需要提取日期的输入
        
        const result = node.output_date_object();

        //node.start_date=result.start_date
        node0.start_date=result.start_date
        node0.end_date=result.end_date

        //检查filter_1的过滤结果
        //console.log('!!result.start_date&&!!result.end_date',node0)
        //console.log(result)
        //console.log(!!result.start_date&&!!result.end_date)



        if(!!result.start_date&&!!result.end_date) return true;


        return false;
      }

      // 5. Filter function 2
      function filter_2(node0) {
        
        //return true
        const input = node0.plainText;
        const node = new Jsserve_api.Miro_mindmap_node_plaintext(input); // 创建一个实例并传入需要提取日期的输入
        
        const result = node.output_date_object();
        
        //node.start_date=result.start_date
        //console.log('!!result.end_date&&!result.start_date',!!result.end_date&&!result.start_date)

        if(!!result.end_date&&!result.start_date) return true;

        return false;

      }

      // Create a new instance of Node_tree
      let tree = new Jsserve_api.Node_tree(data, get_root, filter_1, filter_2);

      let nodes_tree1 = tree.get_new_tree1();
      //console.log('nodes_tree1-----------------')
      //console.log('nodes_tree1-----------------',nodes_tree1)
      let nodes_tree = tree.get_new_tree2();
      //console.log(nodes_tree)
      
      //节点树生成````````````````````````````
      console.log('节点树生成---------------------------------------------',nodes_tree)
      //精简node-tree
      function shorten_nodes_tree(nodes_tree){
        nodes_tree.forEach(node => {
          delete node.bounds
          delete node.type
          delete node.style
          delete node.metadata
          delete node.capabilities
          delete node.createdUserId
          delete node.lastModifiedUserId
          delete node.clientVisible
          delete node.scale
          delete node.rotation
          delete node.width
          
          

        });

        return nodes_tree
        
      }
      nodes_tree=shorten_nodes_tree(nodes_tree)
      //精简node-tree------------------------------------
      //console.log(nodes_tree)
      console.log('board_data--------------',board_data)
      let asana_task=new Asana.Asana_action (nodes_tree,board_data.id,board_data.title)
      console.log('board_data_name------------',board_data)
      await asana_task.sync_2_asana()
      console.log('ASANA同步完毕---------------------------------------------')

      //发送数据到接口
      
      //------------------------------------------------------

      
    }
    this.on_load_tag=async function (){ //观察date标签 | return tagstack

      //增加刷新监控触发
      async function onAllWidgetsLoaded(callback) {
        const areAllWidgetsLoaded = await miro.board.widgets.areAllWidgetsLoaded()
        if (areAllWidgetsLoaded) {
          callback()
        } else {
          miro.addListener('ALL_WIDGETS_LOADED', callback)
        }
      }
      onAllWidgetsLoaded(async() => {
        
        
        //console.log(`----------------------------#tag click--------------------------------`)
        let mindnodes=await miro.board.widgets.get({type: 'TEXT'})

        //测试显示线条
        /*
        let lines=await miro.board.widgets.get({type: 'LINE'})
        console.log(lines)
        */
        //------------------------------
      
        
        //console.log(mindnodes)
        //获得tag为#目标的集合
        let mindnodes_withtags_goal=mindnodes.filter(node=>{
          //console.log(node)
          return (!!node.text.match(/>#目标\s/))&&(!node.text.match(/color:rgb\(146,236,255/))
        })

        //console.log(mindnodes_withtags_goal)
          //清除并覆写特定tag//
        if(JSON.stringify(mindnodes_withtags_goal) != '[]'){
          for( let x in mindnodes_withtags_goal){
            let node=mindnodes_withtags_goal[x]
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            console.log('replace #目标')
            newtext=newtext.replace(/^/,'#目标 ') 
            //goalstack[x].title.replace('#ac','') 
            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color: rgb(146, 236, 255); color: rgb(0, 0, 0);\">'+newtext+'</span></p>' 

            })
          }
        }//
          


        //活儿都tag为#目标/进行中的集合 //
        let mindnodes_withtags_doing=mindnodes.filter(node=>{
          return (!!node.text.match(/>#目标\/进行中\s/))&&(!node.text.match(/color:rgb\(219,174/))
        })
          //清除并覆写特定tag

        if(JSON.stringify(mindnodes_withtags_doing) != '[]'){
          for( let x in mindnodes_withtags_doing){
            let node=mindnodes_withtags_doing[x]
            console.log(node)
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            console.log(newtext)
            console.log('replace #目标/进行中')
            newtext=newtext.replace(/^/,'#目标\/进行中 ') 
            //goalstack[x].title.replace('#ac','') 


            //创建Habi对像
            //this.creathabitodo(node.plainText,node.id)
            

            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'

            })
          }
        }

        //获得tag为#目标/完成的集合
        let mindnodes_withtags_done=mindnodes.filter(node=>{
          return (!!node.text.match(/>#目标\/完成\s/))&&(!node.text.match(/color:rgb\(230,249,148\)/))
        })
          //清除并覆写特定tag
        
        if(JSON.stringify(mindnodes_withtags_done) != '[]'){
          for( let x in mindnodes_withtags_done){
            let node=mindnodes_withtags_done[x]
            let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
            newtext=newtext.replace(/^/,'#目标\/完成 ') 

            //改变颜色
            await miro.board.widgets.update({
              id:node.id,
              
              text:'<p><span style=\"background-color:rgb(230,249,148);color:rgb(0,0,0)\">'+newtext+'</span></p>',

            })

            //goalstack[x].title.replace('#ac','') 

            //完成HABI对象
            this.creathabitodo(node.plainText,node.id).then(()=>{
              let node_repair={
                id:node.id,
                title:node.plainText,
              }
              this.scoredhabi(node_repair)
              console.log(`--------${node.plainText} is Completed at Habitica!--------`)
            })
            




          }
        }

      
       
      
      })
    }
    this.watch_selected_tags=async function(board_data_input){ //点击触发器
            //console.log(`----------------------------#tag click--------------------------------`)
            //let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
      this.board_data=this.board_data??{}
      this.board_data=board_data_input??this.board_data

      miro.addListener('SELECTION_UPDATED', async(widget) => {
       
        
            ////如果输入函数为空中止
            /*if(!func){
              return
            }*/
            //
            let selectedItems=widget.data
            let fullstack= new Array
            for( let x in selectedItems)
            {
              //console.log('x.id')
              //console.log(selectedItems[x].id)
              let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
              let fullobject=fullobject0[0]
              //
              //console.log('get object is')
              //console.log(fullobject)
              //
              fullstack.push(fullobject)
            }
            let mindnodes=fullstack.filter(node=>node.type=='TEXT');
            //写入board_id
            mindnodes=mindnodes.map(node=>{
              node=Object.assign(node,{board_id:this.board_data.id})
              return node
            })
            //写入board_id
            //console.log(`--------------------------mindenodes--------------------测试中断----------------开始-----------------`) 
            //打印全部对象
            console.log(fullstack)

            //进行更新或者创建判断
            
              //完成触发创建行为
              /*
            for(let x in fullstack ){
              //let miro_task=new Miro.Task(fullstack[x])
              //console.log(miro_task)
              /*
              //let todoist_data=miro_task.out_put_todoist()
              let metadata=new miro_mind_metadata(fullstack[x])
              if(metadata.check==false){
                let title=new Title(node.text,{est:null,'#目标/进行中':null,'#目标/阻塞:':null,date:null,'#目标/完成':null,'#目标':null})
                let miro_task_data=title.out_put_data()
                if(!!title.out_put_data().start_date){
                  let todoist_task=new Todoist.task({
                    purename:miro_task_data.purename,
                    description:`[Miro-Link](${task.get_link()})`,//Jsserve.common.fixdes_n(task.des_recog_para.puredes),
                    dueString:`${miro_task_data.start_date} ${!!miro_task_data.start_time?miro_task_data.start_time:''}`,
                    //order:task.apiobject.position,
                    project_id:todopid,
                    //toggl_plan_task_id:task.apiobject.id,
                    estimate_time:miro_task_data.est,
    
                  })
                  await todoist_task.create()
                  //覆写metadata
                  await metadata.write()
                }
              }
              
            }
            */
              

            //
            
              //完成更新触发

            //console.log(mindnodes)
            //获得tag为#目标的集合
            
            let mindnodes_withtags_goal=mindnodes.filter(node=>{
              //console.log(node)
              return (!!node.text.match(/>#目标\s/))&&(!node.text.match(/color:rgb\(146,236,255/))
            })
            
            //console.log(mindnodes_withtags_goal)
              //清除并覆写特定tag
            if(JSON.stringify(mindnodes_withtags_goal) != '[]'){
              for( let x in mindnodes_withtags_goal){
                let node=mindnodes_withtags_goal[x]
                let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
                console.log('replace #目标')
                newtext=newtext.replace(/^/,'#目标 ') 
                //goalstack[x].title.replace('#ac','') 
                //改变颜色
                await miro.board.widgets.update({
                  id:node.id,
                  
                  text:'<p><span style=\"background-color: rgb(146, 236, 255); color: rgb(0, 0, 0);\">'+newtext+'</span></p>'  ////
                  //<p><span style=\"background-color:rgb(170,216,255); color:rgb(0,0,0);\">#目标 游戏测试到没问题</span></p>
                })
              }
            }//
            
              
      
            
            //活儿都tag为#目标/进行中的集合
            let mindnodes_withtags_doing=mindnodes.filter(node=>{
              return (!!node.text.match(/>#目标\/进行中\s/))&&(!node.text.match(/color:rgb\(219,174/))
            })
              //清除并覆写特定tag
      
            if(JSON.stringify(mindnodes_withtags_doing) != '[]'){
              for( let x in mindnodes_withtags_doing){
                let node=mindnodes_withtags_doing[x]
                console.log(node)
                let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
                console.log(newtext)
                console.log('replace #目标/进行中')
                newtext=newtext.replace(/^/,'#目标\/进行中 ') 
                //goalstack[x].title.replace('#ac','')

            //创建Habi对象

            //创建habi对象
            //this.creathabitodo(node.plainText,node.id)
                /*
                //开始追踪当前目标/并将页面所有其他的设为待办
                //开始追踪
                //console.log(`测试中断1-------------------------------`)
                let task=new Miro.Task(node)
                let title=new Regex.Title_goal(node.plainText)
                function mark_todo(){
                    return new Promise((resolve,reject)=>{
                      //let tempproject={project:this.inputentry}


                      //识别标题
                      //console.log(`-----------------------测试中断2进行中---------------${mindnode.plainText}-------------`)
                      //console.log(`测试中断2-------------------------------`)
                      
                      let est=title.out_put_data().est
                      let pure_text=title.out_put_data().pure_text
                      console.log(title.out_put_data())
                     
                      axios({
                        headers: {
                          //Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
                          'Content-Type':'application/json',
                          },  
                        method: 'post',
                        url: `https://alex.shinestu.com/automate`,
                        data: {
                          action:'set_action_task',
                          data:task.out_put_mark_todo()
                        }
                      })
                      .then(res => {
                        //console.log(`--------------------测试中断---------------------`)
                        //console.log(JSON.stringify(res.data))
                        

                        resolve(console.log(`追踪创建完成`));
                      })
                      .catch(error=>{
                        // handle error
                        console.log(` automate追踪error`)
                        console.log(error.response?.data)
                
                        
                
                        //console.log(this.inputproject)
                        //this.inputproject.name=(this.inputproject.name+' 重名修正')
                      // reject()
                        //reject(this.inputentry.name)
                        
                      })
                      .then(function () {
                        // always executed
                      });
                  })
                }
                if(!!(title.out_put_data().start_date)&&!!(title.out_put_data().start_time)){
                  //console.log(`测试中断1.5-------------------------------`)
                  await mark_todo()
                }
                */
                
              
              //改变颜色
                await miro.board.widgets.update({
                  id:node.id,
                  
                  text:'<p><span style=\"background-color:rgb(219,174,2555);color:rgb(0,0,0)\">'+newtext+'</span></p>'
      
                })//
              }
            }

                        
            //活儿都tag为#目标/执行的集合
            let mindnodes_withtags_action=mindnodes.filter(node=>{
              let title=new Regex.Title_goal(node.plainText)
              let data=title.out_put_data()
              //console.log(`测试-start_date-----------------------${data.start_date}`)
              return (!!node.text.match(/>#目标\/执行\s/))
              &&(!node.text.match(/color:rgb\(255,32,226/)
              //&&(!!data.start_date)
              )
            })
              //清除并覆写特定tag
           // console.log(`------------------------------测试中断-----------------------------`)
            if(JSON.stringify(mindnodes_withtags_action) != '[]'){
              //console.log(`------------------------------测试中断2-----------------------------`)
              for( let x in mindnodes_withtags_action){
                let node=mindnodes_withtags_action[x]
                //console.log(`------------------------------测试中断3-----------------------------`)
                console.log(node)
                let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
                console.log(newtext)
                console.log('replace #目标/执行')
                newtext=newtext.replace(/^/,'#目标\/执行 ') 
                //goalstack[x].title.replace('#ac','')

            //创建Habi对象

            //创建habi对象
            //this.creathabitodo(node.plainText,node.id)
                
                //开始追踪当前目标/并将页面所有其他的设为待办
                //开始追踪
                function chasing(){
                      return new Promise((resolve,reject)=>{
                        //let tempproject={project:this.inputentry}


                        //识别标题
                        //console.log(`-----------------------测试中断2进行中---------------${mindnode.plainText}-------------`)
                        let title=new Regex.Title_goal(node.plainText)
                        let est=title.out_put_data().est
                        let pure_text=title.out_put_data().pure_text
                        console.log(title.out_put_data())
                        
                        axios({
                          headers: {
                            //Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
                            'Content-Type':'application/json',
                            },  
                          method: 'post',
                          url: `https://alex.shinestu.com/automate`,
                          data: {
                            action:'update_chasing',
                            data:{
                              //past_real_do_name:'unknown',
                              now_action:{
                                name:pure_text??node.plainText,
                                pre_est:est??0.5
                              }
                            }
                          }
                        })
                        .then(res => {
                          //console.log(`--------------------测试中断---------------------`)
                          //console.log(JSON.stringify(res.data))
                          

                          resolve(console.log(`追踪创建完成`));
                        })
                        .catch(error=>{
                          // handle error
                          console.log(` automate追踪error`)
                          console.log(error.response?.data)
                  
                          
                  
                          //console.log(this.inputproject)
                          //this.inputproject.name=(this.inputproject.name+' 重名修正')
                        // reject()
                          //reject(this.inputentry.name)
                          
                        })
                        .then(function () {
                          // always executed
                        });
                    })
                }
                  //chasing() //暂时清除执行追踪 2023-02-26-await 
                function action(){
                    return new Promise((resolve,reject)=>{
                      //let tempproject={project:this.inputentry}
                      //node=Object.assign(node,{board_id:board_data.id})
                      console.log('----------------board_data____________test-')
                      console.log(node)
                      let task=new Miro.Task(node)
                      let link=task.out_put_link()

                      //识别标题
                      //console.log(`-----------------------测试中断2进行中---------------${mindnode.plainText}-------------`)
                      let title=new Regex.Title_goal(node.plainText)
                      let title_data=title.out_put_data()
                      let est=title.out_put_data().est
                      let pure_text=title.out_put_data().pure_text
                      let duedate=title.out_put_data().start_date
                      let due_text=`${title_data.start_date} ${title_data.start_time}`
                      console.log(`-----due测试中断-------------`)
                      console.log(due_text)
                      console.log(title.out_put_data())
                      
                      axios({
                        headers: {
                          //Authorization: 'Bearer QjxAyceFmAu6hcGRWh55m1eoy5s-CY0Y_6T09VwCvqUNbhaeh35Sl-puA_SrIK3_CSL33DBokIeCuJdvnz9DVxGAJUKkMHx_v-rZ8k9Drkw4zOVIBvLUsJJ7Xm3NyWA-cI98NKa4BN0E2iR0X2r719Ig4zRGy6P05l4J2mktYiqnb4rpbMCTl5vc3vGqprCFCG3uNj5EEvw0AFej9C4eHArZE1PRmxZlEReE6T8HiXSm2vU2SlZntfFXQXz4gKcrdhpmS7I8w9pk6PRMRbIYS6vqlH0PAdY8SApncLBttJ8vzdkssleQDB_G2MLc_SY5YnjBs97EKfMGrb-4bFLSCg==',
                          'Content-Type':'application/json',
                          },  
                        method: 'post',
                        url: `https://alex.shinestu.com/automate`,
                        data: {
                          action:'start_action',
                          data:{
                            //past_real_do_name:'unknown',
                          title:pure_text,
                          duedate:due_text,
                          miro_id:node.id,
                          miro_link:link,

                          }
                        }
                      })
                      .then(res => {
                        //console.log(`--------------------测试中断---------------------`)
                        //console.log(JSON.stringify(res.data))
                        console.log(duedate)

                        resolve(console.log(`追踪创建完成`));
                      })
                      .catch(error=>{
                        // handle error
                        console.log(` automate追踪error`)
                        console.log(error.response?.data)
                
                        
                
                        //console.log(this.inputproject)
                        //this.inputproject.name=(this.inputproject.name+' 重名修正')
                      // reject()
                        //reject(this.inputentry.name)
                        
                      })
                      .then(function () {
                        // always executed
                      });
                  })
                }


                //有日期的开启同步
                let title=new Regex.Title_goal(node.plainText)
                let data=title.out_put_data()
                let due_date=data.start_date
                let due_time=data.start_time
                let due_text=`${due_date} ${due_time}`
                if(!!due_date&&!!due_time){
                  console.log('------------------------action-----------------------------------------------')
                  action()
                }

                //有日期的开启同步
                
                
              
              //改变颜色
                await miro.board.widgets.update({
                  id:node.id,
                  
                  text:'<p><span style=\"background-color:rgb(255,32,226);color:rgb(0,0,0)\">'+newtext+'</span></p>'
      
                })//
              }
            }


            //console.log(`--------------------------mindenodes--------------------测试中断---------------------------------`)
            //获得tag为#目标/完成的集合
            let mindnodes_withtags_done=mindnodes.filter(node=>{
              return (!!node.text.match(/>#目标\/完成\s/))&&(!node.text.match(/color:rgb\(230,249,148\)/))
            })
              //清除并覆写特定tag
            
            if(JSON.stringify(mindnodes_withtags_done) != '[]'){

              for( let x in mindnodes_withtags_done){
                let node=mindnodes_withtags_done[x]
                let newtext=node?.plainText.replace(/#ac|(#目标\s|#目标\/执行\s|#目标\/进行中\s|#目标\/完成\s|#目标\/测试)/g,'')
                newtext=newtext.replace(/^/,'#目标\/完成 ') 

                //改变颜色
                await miro.board.widgets.update({
                  id:node.id,
                  
                  text:'<p><span style=\"background-color:rgb(230,249,148);color:rgb(0,0,0)\">'+newtext+'</span></p>',
      
                })

                let title=new Regex.Title_goal(node.plainText)
                let out_data=title.out_put_data()

                //google_sheet写入
                let mes=new Axios_plus.Web_message({
                  method:'post',
                  url:'https://alex.shinestu.com/miroserve',
                  json:{
                    action:'record_task',
                    data:{
                      goal:out_data.pure_text,
                      finished_time:jsserve.getnowtime_normal(),
                    }
                  }
                })
                mes.trans().catch(e=>{console.log(e)})


              
                //完成HABI对象
                let habi_uuid=jsserve.gen_uuid_no_line()
                //-------------------完成HABI任务 
                async function completed_habi_todo(alias){
                  return new Promise((resolve,reject)=>{
                            
                              axios({
                                headers: {
                                  'Content-Type': 'application/json',
                                  'x-client':'23d75568-fa21-4411-be2d-c70d748e1628-mirosynchabitica',
                                  'x-api-user':'23d75568-fa21-4411-be2d-c70d748e1628',
                                  'x-api-key':'ea9547ef-6a26-452b-974a-594f9c0771a7'
                                },
                                method: 'post',
                                url: 'https://habitica.com/api/v3/tasks/'+alias+'/score/up',
                              
                              })
                              .then(res => {
                                //console.log(JSON.stringify(res.data))
                                console.log(res.data)
                                resolve(res.data);
                              })
                              .catch(function (error) {
                                // handle error
                                console.log(error.response.data);
                              })
                              .then(function () {
                                // always executed
                              });
                            })
                  
                  }
                completed_habi_todo(node.id)
                //-----------------------------------
                this.creathabitodo(node.plainText,habi_uuid).then(()=>{
                  
                  let node_repair={
                    id:habi_uuid,
                    title:node.plainText,
                  }
                  this.scoredhabi(node_repair)
                  console.log(`--------${node.plainText} is Completed at Habitica!--------`)
                }).catch(e=>console.log(e))

                //完成todoist
                let complete_data={
                  miro_id:node.id
                }
                await miromec.alex_serve_api('complete_task',complete_data)
                /*
                let temp_title=new Title.Title_goal(node.plainText)
                let temp_start_date=temp_title.out_put_data().start_date
                if(!temp_start_date){ //如果存在日期则不完成
                  let complete_data={
                    miro_id:node.id
                  }
                  await miromec.alex_serve_api('complete_task',complete_data)
                }
                else{
                  miro.showNotification('完成todo')
                }*/
                //

                //写入变更
                let te_task=new Miro.Task(node)
                //console.log(`-------------------测试中断--------------------------------------------------------`)
                await te_task.write_checkdata()


                //console.log(`-------------------测试中断--------------------------------`)


               
              }
            }
       })  
    }
    //监视点击更新日期
    this.watchclickwithdateasync= async function (){ //观察date标签 | return tagstack

      //增加刷新监控触发
      
        
        
        //获得所有对象
        let cards=await miro.board.widgets.get({type: 'CARD'})
        let mindnodes=await miro.board.widgets.get({type: 'TEXT'})
        let fullstack=cards.concat(mindnodes)


        /*//获得选择对象
        let selectedItems=widget.data;
        console.log(selectedItems)*/

        //从选择对象获得完整对象
        /*
        let fullstack= new Array
        for( let x in selectedItems)
        {
          let fullobject0=await miro.board.widgets.get({id:selectedItems[x].id})
          let fullobject=fullobject0[0]
          fullstack.push(fullobject)
        }*/
        console.log('watchdate-fullstack-checked is') //调试
        console.log(fullstack)

        //获得有日期的下对象
        let regexdate=/((.*)((\s(\d{8}|\d{4})\s)|\s)(\d{4})($|\s\d{2}:\d{2}-\d{2}:\d{2}))/g;
        let goalstack=mirostack2goalstack(fullstack)

        console.log('goalstack is') //#调试
        console.log(goalstack) //调试


        let datestack=goalstack.filter(goal=>!!goal.title.match(regexdate)
        )
        console.log('date is ') //调试
        console.log(datestack) //调试

        console.log(datestack.length)
        console.log('datestack checked')//调试
        

        
        

        //对存在的集合操作
        if(datestack.length!=0) 
        {

          //过滤获取变更的集合
          datestack=fildatechangegoal(datestack)
          console.log('filter datestack is ')
          console.log(datestack)

          this.multiope(this.synctoggltask,datestack)
      
        }
        
      

    }
    

    
    //定义区end
}








//按钮触发全体
/*
class All_mind_analaysis{
  constructor(){
    let mindnodes=new Promise(resolve=>{
      miro.board.widgets.get({type: 'TEXT'})
      .then(mindnodes=>resolve(mindnodes))
    })
    return mindnodes
    
  }
}

miromec.All_mind_analaysis=All_mind_analaysis
*/

//miroserve01.creattask()
export {miroserve};
export {miromec}
//export {Miro}
//axpost()