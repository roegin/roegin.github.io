- 请根据参考,帮我完成以下要求,给我详细步骤和代码
    - 给我一个新的类SynchronizerMiroProjectsAsTasks
        - 该类将项目概览卡片作为任务同步到miro的任务集合数据库
        - 同步属性对应关系
            - 项目名title对应notion的任务标题
- 参考
    - Bigpicture类介绍
        - 以下是bigpicture类的说明
            - 有一个nodejs的bigpicture类,该类负责管理bigpicture的miro板块上的项目概览包括其中的项目概览卡片
            - 获得所有项目概览卡片的函数
                - ```javascript
async function fetch_app_cards(board_id) {
  //console.log('board_id------------------------',board_id)
  let appCards = [];
  let cursor = '';

  while (true) {
    try {
      let params = {
        limit: '50',
        type: 'app_card',
        board_id: board_id,
      };

      // 如果 cursor 有值，才添加到请求参数中
      if (cursor) {
        params.cursor = cursor;
      }

      const response = await sdk.getItems(params);


      const resdata = response.data;
      const data=resdata.data;
      //console.log('data---------------------',data,)
      if (data.length === 0) {
        break;
      } else {
        appCards = appCards.concat(data);
      }
      
      // 获取下一页的 cursor
      cursor = resdata.cursor;

      // 如果 cursor 为 null 或者无效，那么就停止请求
      if (!cursor || cursor === 'null' || cursor === 'undefined') {
        break;
      }
    } catch (error) {
      console.error(error);
      break;
    }
  }

  //过滤appcard
  appCards = appCards.filter(card => {return card.type == 'app_card'});
  //console.log('appCards---------------------',appCards,)

  return appCards;
}```
            - 代码
                - ```javascript
const { axios,cheerio,he} = require('./libraries');

const sdk = require('api')('@miro-ea/v2.0#jbaw3n6li1osaw6');
sdk.auth('eyJtaXJvLm9yaWdpbiI6ImV1MDEifQ_KRLLlm7GAmsDYGUHRM9QBMqBfcU');

const Miroserve = require('./miroserve');
const Miro_api = Miroserve.Api;
const Bigpicture_board_id='uXjVMOeXY0E='
const Miro = require('./Miro');
miroApi = new Miro_api();




//BigPciture类
class BigPicture {
  constructor() {
    this.teamId = '3458764552996161564';

    //引用模块
    this.miroApi = new Miro_api();
  }
  async _getProjectsWithSummary(){
    const projects = await this.miroApi.getProjectsAtTeam(this.teamId);

    const projectOverviewPromises = projects.map(async project => {
      try {
        let board = new Miro.MiroBoard(project.id);
        let summary = await board.GetProjectSummary();
        return {
          projectId: project.id,
          ...summary,
          ...project
        };
      } catch (error) {
        console.error(`获取 ${project.name}:项目概览失败,可能项目目标标签不存在`);
        return {
          projectId:project.id,
          projectTitle:project.name,
          projectStatus:'待办',
          ...project
        };
      }
    });
    
    const _projectOverviews = await Promise.allSettled(projectOverviewPromises);
    
    const projectOverviews = _projectOverviews
      .filter(result => result.status === 'fulfilled')
      .map(result => result.value);

    return projectOverviews
  }

  async _filterCardsForProjectCreation() {
    let allCards = await fetch_normal_cards(process.env.BigPictureBoardId); // 获取所有卡片
    function extractPlainText(htmlString) {
      const $ = cheerio.load(htmlString);
      return $('p').text();
    }

    allCards=allCards.map(card=>{
          card.data.title=extractPlainText(card.data.title)
          return card
        }
      )
    //console.log('allCards', allCards.map (card=>card.data.title))
    const projectCreationCards = allCards.filter(card => card.data.title.includes('#creatProject')); // 过滤出标题含有 #creatProject 的卡片
    
    console.log('将要创建的项目卡片', projectCreationCards.map (card=>card.data.title))
    return projectCreationCards;
  }
  
  async _createMiroProjectFromCards() {
    const cardsToCreateProject = await this._filterCardsForProjectCreation();
    for (const card of cardsToCreateProject) {
      card.data.title = card.data.title; // 将标题中的 HTML 实体转换为普通字符
      const projectTitle = card.data.title.replace('#creatProject', ''); // 删除#creatProject标签，只保留标题
      const project = new Miroserve.project({
        name: projectTitle,
        statue: "进行中"
      });
      const board_data = await project.creat_from_goal_template();
      const board_id = board_data.id;
      card.boardId = board_id; // 将新创建的项目id保存到卡片对象中
    }
    return cardsToCreateProject
  }

  async _createProjectCardsFromCards() {
    let cardsToCreateProject=await this._createMiroProjectFromCards();
    //const cardsToCreateProject = await this.filterCardsForProjectCreation();
    for (const card of cardsToCreateProject) {
      const projectCard = new ProjectCard(card.data.title, card.boardId, "待办");

      //console.log('xy坐标',card)
      await projectCard.createIndependent(card.position.x,card.position.y); // 假设ProjectCard类有一个创建卡片的方法
      
      //删除原始卡片
      sdk.deleteCardItem({board_id: process.env.BigPictureBoardId, item_id: card.id})
      .then(({ data }) => console.log(`Card ${card.id} deleted successfully`))
      .catch(err => console.error(err));
    }
  }
  
  

  async manageProjects() {
    
    //检查触发创建
    await this._createProjectCardsFromCards();

    const projects = await this.miroApi.getProjectsAtTeam(this.teamId);

    const projectOverviewPromises = projects.map(async project => {
      try {
        let board = new Miro.MiroBoard(project.id);
        let summary = await board.GetProjectSummary();
        return {
          projectId: project.id,
          ...summary,
          ...project
        };
      } catch (error) {
        console.error(`Failed to get project summary for projectId ${project.name}:`);
        return {
          projectId:project.id,
          projectTitle:project.name,
          projectStatus:'待办',
          ...project
        };
      }
    });
    
    const _projectOverviews = await Promise.allSettled(projectOverviewPromises);
    
    const projectOverviews = _projectOverviews
      .filter(result => result.status === 'fulfilled')
      .map(result => result.value);

    //console.log('projectoverviews', projectOverviews.map(project=>`${project.projectStatus} => ${project.projectTitle} `))


    //获取所有的卡片
    const appCards = await fetch_app_cards(Bigpicture_board_id);


    //console.log('success--------------');

    //获取卡片位置信息
    let leftmostCard = appCards[0]; // 假设第一个卡片是最靠左边的
    let bottommostCard = appCards[0]; // 假设第一个卡片是最下面的

    for (const card of appCards) {
      const cardPosition = card.position;
      //console.log('测试中断-----------------------')
      if (cardPosition.x < leftmostCard.position.x) {
        leftmostCard = card; // 更新最靠左边的卡片
      }
      if (cardPosition.x === leftmostCard.position.x && cardPosition.y > bottommostCard.position.y) {
        bottommostCard = card; // 更新最下面的卡片
      }
}

    let positionY = 0;

    if (leftmostCard && bottommostCard) {
      positionY = bottommostCard.position.y + bottommostCard.geometry.height + 50; // 获取最下面卡片的下方位置
    }


    //循环检查创建或者更新
    for (const projectOveriew of projectOverviews) {
      const projectcard_from_projectdata = {
        projectTitle: projectOveriew.projectTitle??project.name,
        projectStatus: projectOveriew.projectStatus??'待办', 
        miroBoardId: projectOveriew.id,
        ...projectOveriew
      };

      const existingCard = appCards.find(card => {
        if (card.data.description) {
          const decodedDescription = he.decode(card.data.description);
          return JSON.parse(decodedDescription).miroBoardId === projectcard_from_projectdata.miroBoardId;
        }
        return false;
      });

      const projectCard = new ProjectCard(projectcard_from_projectdata.projectTitle, projectcard_from_projectdata.miroBoardId,projectcard_from_projectdata.projectStatus);

      if (existingCard) {

        //比较存在的card和summary的标题和状态,不同则更新
        let ifDiff=(projectcard_from_projectdata.projectTitle!=existingCard.data.fields.find(field=>field.tooltip=='title')?.value)
        ||(projectcard_from_projectdata.projectStatus!=existingCard.data.fields.find(field=>field.tooltip=='status')?.value);
        //console.log('ifDiff',projectCard.projectTitle, ifDiff,projectcard_from_projectdata.projectStatus,existingCard.data.fields.find(field=>field.tooltip=='status')?.value)

        if(ifDiff){
          //检测不同值比较
          /*
          console.log(existingCard,existingCard.data.fields)
          console.log(projectcard_from_projectdata.projectTitle,existingCard.data.fields.find(field=>field.tooltip=='title'),
            projectcard_from_projectdata.projectStatus,existingCard.data.fields.find(field=>field.tooltip=='status')?.value
          )*/
          await projectCard.update(existingCard.id);
          console.log(`项目 ${projectcard_from_projectdata.projectTitle} 的卡片已更新`)
        }


      } else {
        await projectCard.create(positionY,leftmostCard); // 使用计算得到的位置创建卡片
        positionY += 50; // 每次新建卡片都向下移动 50 像素
      }
    }
  }

  async getInProgressProjectsByCard() {
    // 使用已有的 getProjectsAtTeam 方法获取所有项目
    //const allProjects = await this._getProjectsWithSummary()

    //从card集合获取进行中状态的项目id集合
    const appCards = await fetch_app_cards(process.env.BigPictureBoardId);
    const inProgressBoardIds=appCards
    .filter(card=>{
      if(card.data.fields.find(field=>field.tooltip=='status')?.value=='进行中'){
        return true
      }
      return false
    })
    .map(card=>{
      if (card.data.description) {
        const decodedDescription = he.decode(card.data.description);
        return JSON.parse(decodedDescription).miroBoardId 
      }
      else{
        return null
      }
    })

    //拉取进行中项目任务集合
    let inProgressProjects;
    const projects = await this.miroApi.getProjectsAtTeam(this.teamId);
    //过滤projects中id和inProgressBoardIds中的id相同的项目
    inProgressProjects=projects.filter(project=>{
      if(inProgressBoardIds.includes(project.id)){
        return true
      }
      return false
    })
    
    // 返回这些项目
    return inProgressProjects;
  }
  async getInProgressProjectsIndependent() {
    // 使用已有的 getProjectsAtTeam 方法获取所有项目
    const allProjects = await this._getProjectsWithSummary()
    
    // 使用 filter 方法获取状态为“进行中”的项目
    const inProgressProjects = allProjects.filter(project => project.projectStatus === '进行中');
    
    // 返回这些项目
    return inProgressProjects;
  }
  async getInProgressProjectTasksByCard(){ //写入了projectname
    let projects=await this.getInProgressProjectsByCard()
    let tasks=[]
    for(const project of projects){
      let miroboard=new Miro.MiroBoard(project.id)
      let miroTasks = await miroboard.getTasks()

      //写入projectname
      miroTasks=miroTasks.map(node=>{
        node.miroBoardName=project.name
        node.miroBoardId=project.id
        return node
      })

      tasks.push(...miroTasks)

    }
    return tasks
  }
}

//测试代码 打印所有进行中任务

/*
const bigPicture = new BigPicture();

async function testGetInProgressProjects() {
  try {
    let inProgressProjects = await bigPicture.getInProgressProjectTasksByCard();
    console.log('进行中的项目任务集合：');

    // 删除项目根节点
    const rootNode = inProgressProjects.find(node => node.node_type === 'root');
    inProgressProjects = inProgressProjects.filter(node => node.node_type !== 'root');
    console.log(inProgressProjects);
  } catch (error) {
    console.error('获取进行中项目时出错：', error);
  }
}

testGetInProgressProjects();
*/




// 创建 ProjectCard 类
class ProjectCard {
  constructor(projectTitle, miroBoardId,projectStatus) {

    //console.log('newcard-------',projectTitle,miroBoardId)
    this.projectTitle = projectTitle;
    this.miroBoardId = miroBoardId;
    this.projectStatus = projectStatus;
    this.fields = [
      {
        iconShape: 'round',
        value: this.projectTitle,
        tooltip: 'title',
      },
      {
        iconShape: 'round',
        value: `https://miro.com/app/board/${this.miroBoardId}/`,
        //tooltip: '点击访问miro板块',
      },
      {
        iconShape: 'round',
        value: this.projectStatus,
        tooltip: 'status',
      },
    ];
    this.description = JSON.stringify({
      miroBoardId: this.miroBoardId,
    });
  }

  async create(positionY, leftmostCard) {
    try {
      console.log("data", this.fields);
      const response = await sdk.createAppCardItem({
        data: {
          fields: this.fields,
          status: 'connected',
          title: this.projectTitle,
          description: this.description,
        },
        position: {
          origin: 'center',
          x: leftmostCard.position.x, // 使用最左边卡片的水平位置
          y: positionY,
        },
      }, {
        board_id: Bigpicture_board_id,
      });
  
      console.log(response.data);
      return response.data.id;
    } catch (error) {
      console.error(error);
    }
  }


  async createIndependent(positionX,positionY ) {
    try {
      console.log("data", this.fields);
      const response = await sdk.createAppCardItem({
        data: {
          fields: this.fields,
          status: 'connected',
          title: this.projectTitle,
          description: this.description,
        },
        position: {
          origin: 'center',
          x: positionX, // 使用最左边卡片的水平位置
          y: positionY,
        },
      }, {
        board_id: Bigpicture_board_id,
      });
  
      console.log(response.data);
      return response.data.id;
    } catch (error) {
      console.error(error);
    }
  }
  

  async _update(itemId) {
    try {

      //计算项目颜色
      function getProjectStatusColor(status) {
        let statusColor;
      
        switch (status) {
          case '待办':
            statusColor = '#007BFF'; // 深蓝色
            break;
          case '进行中':
            statusColor = '#FFA500'; // 橙色
            break;
          case '完成':
            statusColor = '#008000'; // 绿色
            break;
          case '失败':
            statusColor = '#FF0000'; // 红色
            break;
          case '放弃':
            statusColor = '#808080'; // 灰色
            break;
          case '基本完成':
            statusColor = '#FFFF00'; // 黄色
            break;
          default:
            statusColor = '#000000'; // 默认颜色（黑色）
            break;
        }
      
        return statusColor;
      }
      let projectStatus=this.fields.find(field=>field.tooltip=='status')?.value??'待办';
      let projectStatusColor=getProjectStatusColor(projectStatus);


      const response = await sdk.updateAppCardItem({
        data: {
          fields: this.fields,
          status: 'connected',
          title: this.projectTitle,
          description: this.description,
        },
        style:{
          fillColor:projectStatusColor
        }
        /*
        position: {
          origin: 'center',
          x: 0,
          y: 0,
        },*/
      }, {
        board_id: Bigpicture_board_id,
        item_id: itemId,
      });

      console.log(response.data);
    } catch (error) {
      console.error(error);
    }
  }

  async _get(itemId) {
    try {
      const response = await sdk.getAppCardItem({
        board_id: Bigpicture_board_id,
        item_id: itemId,
      });

      console.log(response.data);
      return response.data;
    } catch (error) {
      console.error(error);
    }
  }

  async update() {
    const appCards = await fetch_app_cards(Bigpicture_board_id);

    // 测试 app card 信息
    //console.log("appCards", appCards);
    const targetCard = appCards.find(card => {
      if (card.data.description) {
        const decodedDescription = he.decode(card.data.description);
        return JSON.parse(decodedDescription).miroBoardId === this.miroBoardId;
      }
      return false;
    });
    if (targetCard) {
      await this._update(targetCard.id);
      
    } else {
      console.error('Target app card not found during update');
    }
  }

  async get() {
    const appCards = await fetch_app_cards(Bigpicture_board_id);
    const targetCard = appCards.find(card => {
      if (card.data.description) {
        const decodedDescription = he.decode(card.data.description);
        return JSON.parse(decodedDescription).miroBoardId === this.miroBoardId;
      }
      return false;
    });
    if (targetCard) {
      return await this._get(targetCard.id);
    } else {
      console.error('Target app card not found during get');
      return null;
    }
  }
}

async function fetch_app_cards(board_id) {
  //console.log('board_id------------------------',board_id)
  let appCards = [];
  let cursor = '';

  while (true) {
    try {
      let params = {
        limit: '50',
        type: 'app_card',
        board_id: board_id,
      };

      // 如果 cursor 有值，才添加到请求参数中
      if (cursor) {
        params.cursor = cursor;
      }

      const response = await sdk.getItems(params);


      const resdata = response.data;
      const data=resdata.data;
      //console.log('data---------------------',data,)
      if (data.length === 0) {
        break;
      } else {
        appCards = appCards.concat(data);
      }
      
      // 获取下一页的 cursor
      cursor = resdata.cursor;

      // 如果 cursor 为 null 或者无效，那么就停止请求
      if (!cursor || cursor === 'null' || cursor === 'undefined') {
        break;
      }
    } catch (error) {
      console.error(error);
      break;
    }
  }

  //过滤appcard
  appCards = appCards.filter(card => {return card.type == 'app_card'});
  //console.log('appCards---------------------',appCards,)

  return appCards;
}

async function fetch_normal_cards(board_id) {
  //console.log('board_id------------------------',board_id)
  let appCards = [];
  let cursor = '';

  while (true) {
    try {
      let params = {
        limit: '50',
        type: 'card',
        board_id: board_id,
      };

      // 如果 cursor 有值，才添加到请求参数中
      if (cursor) {
        params.cursor = cursor;
      }

      const response = await sdk.getItems(params);
      //console.log('response---------------------',response.data)

      const resdata = response.data;
      const data=resdata.data;
      //console.log('data---------------------',data,)
      if (data.length === 0) {
        break;
      } else {
        appCards = appCards.concat(data);
      }
      
      // 获取下一页的 cursor
      cursor = resdata.cursor;

      // 如果 cursor 为 null 或者无效，那么就停止请求
      if (!cursor || cursor === 'null' || cursor === 'undefined') {
        break;
      }
    } catch (error) {
      console.error(error);
      break;
    }
  }

  //过滤appcard
  appCards = appCards.filter(card => {return card.type == 'card'});
  //console.log('appCards---------------------',appCards,)

  return appCards;
}


/*
const appCards = fetch_app_cards(Bigpicture_board_id).then(appCards => {
  //console.log('appCards---------------------',appCards,)
  let card=appCards.find(card => {
   // console.log(card.data.title)
    return card.data.title == '完成财务纪律'
  })
  console.dir(card, { depth: null })
})
*/





/*
//获取appcard
async function fetch_app_cards(board_id) {
  let appCards = [];
  let cursor = null;

  while (true) {
    try {
      const response = await axios.get(`https://api.miro.com/v2/boards/${board_id}/widgets`, {
        headers: {
          Authorization: 'Bearer '+Miro_token,
          Accept: 'application/json',
          'Content-Type': 'application/json',
        },
        params: {
          limit: '50',
          type: 'app_card',
          cursor: cursor,
        },
      });

      const data = response.data;
      if (data.length === 0) {
        break;
      } else {
        appCards = appCards.concat(data);
        cursor = response.headers['x-next-page-cursor'];
        if (!cursor) {
          break;
        }
      }
    } catch (error) {
      console.error(error);
      break;
    }
  }

  return appCards;
}
*/


//let bp=new BigPicture();
//bp.manageProjects()





  

module.exports = {BigPicture,ProjectCard};
```
        - 以下是作为项目概括卡片以appcard形式存在于bigpicture上的卡片的数据结构
            - ```javascript
{
  id: '3458764558075981876',
  type: 'app_card',
  data: {
    description: '{&#34;miroBoardId&#34;:&#34;uXjVM7Uik4s&#61;&#34;}',
    fields: [
      {
        fillColor: '#ffffff',
        iconShape: 'round',
        textColor: '#1a1a1a',
        tooltip: 'title',
        value: '完成财务纪律'
      },
      {
        fillColor: '#ffffff',
        iconShape: 'round',
        textColor: '#1a1a1a',
        value: 'https://miro.com/app/board/uXjVM7Uik4s=/'
      },
      {
        fillColor: '#ffffff',
        iconShape: 'round',
        textColor: '#1a1a1a',
        tooltip: 'status',
        value: '进行中'
      }
    ],
    owned: true,
    status: 'connected',
    title: '完成财务纪律'
  },
  style: { fillColor: '#ffa500' },
  geometry: { width: 320, height: 60 },
  position: {
    x: -7977.317941148725,
    y: 228.94425619530472,
    origin: 'center',
    relativeTo: 'canvas_center'
  },
  links: {
    self: 'https://api.miro.com/v2/boards/uXjVMOeXY0E=/app_cards/3458764558075981876'
  },
  createdAt: '2023-06-27T11:30:08Z',
  createdBy: { id: '3074457364464330478', type: 'user' },
  modifiedAt: '2023-07-06T08:21:17Z',
  modifiedBy: { id: '3074457364464330478', type: 'user' }
}```


- 同步miro任务到notion任务集合的类
    - ```javascript

const sqlite3 = require('sqlite3').verbose();

const BigPicture = require('./BigPicture').BigPicture;
const Miro=require('./Miro')
const Notion=require('./notion')


const { notionClient,Jsserve,axios,moment,path,AiTitleMiro} = require('./libraries');


class SynchronizerMiroNotionTasks {
    constructor() {
        // 确保只有一个实例被创建
        /*
        if (SynchronizerMiroNotionTasks.instance) {
            return SynchronizerMiroNotionTasks.instance;
        }
        */
        //this.MiroBoardId = MiroBoardId;
        this.NotionDatabaseId = process.env.NOTION_TASKS_DATABASE_ID//NotionDatabaseId;
        this.teamId = process.env.GOALS_TEAM_ID;
        this.dbPath = process.env.DB_PATH;
        this.Miro = Miro;
        this.db = new sqlite3.Database(path.join(__dirname, process.env.DB_PATH));
        this.db.run(`CREATE TABLE IF NOT EXISTS MiroTasksUpdateRecord (id TEXT PRIMARY KEY, modifiedAt TEXT)`);
        this.notionClient = notionClient
        /*
        SynchronizerMiroNotionTasks.instance = this;
        */
    }

    extractmiroNodeId(notionTask) {
        try{
            return notionTask.properties.miroNodeId.rich_text[0].plain_text

        }
        catch(e){
            return null
        }
    }

    _extractTaskTitle(notionTask) {
        try{
            return ((item) => {
            if (item.properties['Task name'] && item.properties['Task name'].title && item.properties['Task name'].title[0]) {
                let title = item.properties['Task name'].title[0].plain_text;
                //console.log('title', title);
                return title;
            } else {
                return 'N/A'; // 或者你可以选择返回任何其他默认值
            }
        })(notionTask)
        }
        catch(e){
            return null
        }
    }

    async sync() {
        try {
            const bigPicture = new BigPicture();

            // 获取Miro中的所有任务
            let miroTasks = await bigPicture.getInProgressProjectTasksByCard();

            // 删除项目根节点
            /*
            const rootNode = miroTasks.find(node => node.node_type === 'root');
            if (rootNode) {
                const index = miroTasks.indexOf(rootNode);
                miroTasks.splice(index, 1);
            } */
            miroTasks = miroTasks.filter(node => {
                if (node.node_type === 'root') {
                    let aititle=new AiTitleMiro(node.title)
                    let pro=aititle.getProperties()
                    let actionStartDate=pro.actionStartDate
                    return actionStartDate
                }
                return true
            });

            // 获取Notion数据库中的所有任务
            let notionTasks = await this.getNotionTasks(this.NotionDatabaseId);

            //打印存在任务列表
            let existTasks=[]

            // 同步中的创建和更新
            for (const miroTask of miroTasks) {
                // 获取Notion对应的miroid的task




                const existingNotionTask = notionTasks.find(t => this.extractmiroNodeId(t) === miroTask.id);

                //控制台打印检测id匹配结果以检查
                //console.log('miroTask.id',miroTask.id)

                existTasks.push(this._extractTaskTitle(existingNotionTask))




                let { modifiedAt } = await this.getTaskLastModifiedAt(miroTask.id);

                if (existingNotionTask) {
                    if (!miroTask) {
                        await this.deleteNotionTask(existingNotionTask.id);
                        await this.deleteTaskLastModifiedAt(miroTask.id);
                        console.log(`Deleted Notion Task: ${existingNotionTask.properties.title}`);
                    } else {
                        if (modifiedAt === null) {
                            await this.updateTaskLastModifiedAt(miroTask.id, miroTask.modifiedAt);
                            modifiedAt = miroTask.modifiedAt;
                            console.log(`Created database record for Miro Task: ${miroTask.pureTitle}`);
                        }
                        if (new Date(modifiedAt) < new Date(miroTask.modifiedAt)) {
                            await this.updateNotionTask(existingNotionTask.id, miroTask);
                            await this.updateTaskLastModifiedAt(miroTask.id, miroTask.modifiedAt);
                            console.log(`Updated Notion Task: ${existingNotionTask.properties.title}`);
                        }
                    }
                } else if (miroTask) {
                    await this.createNotionTask(miroTask, this.NotionDatabaseId);
                    if (modifiedAt === null) {
                        await this.updateTaskLastModifiedAt(miroTask.id, miroTask.modifiedAt);
                        console.log(`Created database record for Miro Task: ${miroTask.pureTitle}`);
                    }
                    console.log(`Created Notion Task: `, miroTask.text);
                }
            }

            console.log('existingNotionTask',existTasks.join('\n'))

            // 删除失效任务
            for (const notionTask of notionTasks) {
                const miroNodeId = this.extractmiroNodeId(notionTask);

                // Skip if there is no miroNodeId
                if (!miroNodeId) {
                    continue;
                }

                const correspondingMiroTask = miroTasks.find(t => t.id === miroNodeId);

                // Delete the Notion task if there is no corresponding Miro task
                if (!correspondingMiroTask) {
                    await this.deleteNotionTask(notionTask.id);
                    await this.deleteTaskLastModifiedAt(miroNodeId);
                    console.log(`Deleted Notion Task: ${_extractTaskTitle(notionTask)}`);
                    continue;
                }
            }

        } catch (error) {
            console.error(error);
        }
    }
    
    async getTaskLastModifiedAt(taskId) {
        return new Promise((resolve, reject) => {
            this.db.get(`SELECT * FROM MiroTasksUpdateRecord WHERE id = ?`, [taskId], (err, row) => {
                if (err) {
                    reject(err);
                }
                resolve(row || { id: taskId, modifiedAt: null });
            });
        });
    }

    async updateTaskLastModifiedAt(taskId, modifiedAt) {
        return new Promise((resolve, reject) => {
            this.db.run(`REPLACE INTO MiroTasksUpdateRecord (id, modifiedAt) VALUES (?, ?)`, [taskId, modifiedAt], function (err) {
                if (err) {
                    reject(err);
                }
                resolve();
            });
        });
    }

    async deleteTaskLastModifiedAt(taskId) {
        return new Promise((resolve, reject) => {
            this.db.run(`DELETE FROM MiroTasksUpdateRecord WHERE id = ?`, [taskId], function (err) {
                if (err) {
                    reject(err);
                }
                resolve();
            });
        });
    }

    async getNotionTasks(databaseId) {
        // 使用Notion SDK获取所有任务
        const response = await this.notionClient.databases.query({
            database_id: databaseId,
        });

        return response.results;
    }

    //测试
    /*
    const { notionClient,Jsserve,axios,moment,path} = require('./libraries');
    getNotionTasks=async function(databaseId) {
        // 使用Notion SDK获取所有任务
        const response = await notionClient.databases.query({
            database_id: databaseId,
        });

        return response.results;
    }
    //测试代码
    getNotionTasks(process.env.NOTION_TASKS_DATABASE_ID).then(x=>console.log(x))
    */

    _mapTaskToNotionProperties(miroTask) {
        // 生成日期
        let pDue = this._convertToNotionDate(miroTask.actionStartDate, miroTask.actionEndDate);
        if (!pDue.date.end) { delete pDue.date.end; }
        let p = {};
        if (pDue) {
            p = { Due: pDue };
        }
    
        return {
            ...p,
            "Task name": {
                title: [
                    {
                        text: {
                            content: miroTask.pureTitle ?? miroTask.text,
                        },
                    },
                ],
            },
            Status: {
                status: {
                    name: miroTask.status ?? 'Not Started', // 假设你有一个状态字段
                },
            },
            /* 
            Priority: {
                select: {
                    name: miroTask.priority ?? 'Medium', // 假设你有一个优先级字段
                },
            },
            */
            //附加miroNodeId
            miroNodeId: {
                rich_text: [
                    {
                        text: {
                            content: miroTask.id,
                        },
                    },
                ],
            },
            miroBoardName: {
                rich_text: [
                    {
                        text: {
                            content: miroTask.miroBoardName,
                        },
                    },
                ],
            },
            miroBoardId: {
                rich_text: [
                    {
                        text: {
                            content: miroTask.miroBoardId,
                        },
                    },
                ],
            }
        };
    }
    
    async createNotionTask(miroTask, databaseId) {
        // 使用Notion SDK创建一个新的任务
        const properties = this._mapTaskToNotionProperties(miroTask);
    
        await this.notionClient.pages.create({
            parent: { database_id: databaseId },
            properties,
            "icon": {
                "type": "emoji",
                "emoji": this._getProjectStatusEmoji(miroTask.tagStatus.目标 ?? '待办')
            },
        });
    }
    
    async updateNotionTask(notionTaskId, miroTask) {
        // 使用Notion SDK更新任务
        const properties = this._mapTaskToNotionProperties(miroTask);
    
        await this.notionClient.pages.update({
            page_id: notionTaskId,
            properties,
            "icon": {
                "type": "emoji",
                "emoji": this._getProjectStatusEmoji(miroTask.tagStatus.目标 ?? '待办')
            },
        });
    }
    

    async deleteNotionTask(notionTaskId) {
        // 使用 Notion SDK 归档任务
        await this.notionClient.pages.update({
            page_id: notionTaskId,
            archived: true,
        });
    }



    _generateMiroElementLink(miroElementId, miroBoardId) {
        // 生成Miro元素的链接
        return `https://miro.com/app/board/${miroBoardId}/?moveToWidget=${miroElementId}`;
    }

    //const moment = require('moment-timezone');
    _getProjectStatusEmoji(status) {
        let statusEmoji;
      
        switch (status) {
          case '待办':
            statusEmoji = '📌'; // 待办
            break;
          case '进行中':
            statusEmoji = '⏳'; // 进行中
            break;
          case '完成':
            statusEmoji = '✅'; // 完成
            break;
          case '失败':
            statusEmoji = '❌'; // 失败
            break;
          case '放弃':
            statusEmoji = '🚫'; // 放弃
            break;
          case '基本完成':
            statusEmoji = '🔅'; // 基本完成
            break;
          default:
            statusEmoji = '❓'; // 默认图标
            break;
        }
      
        return statusEmoji;
      }

      /*
    _convertToNotionDate(startDateString, endDateString) {
        // 如果开始日期和结束日期都为空，则返回 null
        if (!startDateString) {
            return null;
        }
    
        const format = startDateString.includes(' ') ? 'YYYYMMDD HH:mm' : 'YYYYMMDD';
        const startDateObj = moment.tz(startDateString, format, 'Asia/Shanghai');
        const endDateObj = endDateString ? moment.tz(endDateString, format, 'Asia/Shanghai') : null;
    
        if (startDateObj.isValid()) {
            const start = startDateObj.utc().format('YYYY-MM-DDTHH:mm:ss') + 'Z';
    
            if (endDateObj && endDateObj.isValid() && endDateObj.isSameOrAfter(startDateObj)) {
                const end = endDateObj.utc().format('YYYY-MM-DDTHH:mm:ss') + 'Z';
                return {
                    type: 'date',
                    date: {
                        start,
                        end,
                        time_zone: 'Asia/Shanghai'
                    }
                };
            } else {
                return {
                    type: 'date',
                    date: {
                        start,
                        end: null,
                        time_zone: 'Asia/Shanghai'
                    }
                };
            }
        } else {
            return null;
        }
    }
    */
    _convertToNotionDate(startDateString, endDateString) {
        // 如果开始日期和结束日期都为空，则返回 null
        if (!startDateString) {
            return null;
        }
    
        const format = startDateString.includes(' ') ? 'YYYYMMDD HH:mm' : 'YYYYMMDD';
        const startDateObj = moment.tz(startDateString, format, 'Asia/Shanghai');
        const endDateObj = endDateString ? moment.tz(endDateString, format, 'Asia/Shanghai') : null;
    
        if (startDateObj.isValid()) {
            const start = startDateString.includes(' ') ? startDateObj.format('YYYY-MM-DDTHH:mm:ss') + '+08:00' : startDateObj.format('YYYY-MM-DD');
    
            if (endDateObj && endDateObj.isValid() && endDateObj.isSameOrAfter(startDateObj)) {
                const end = endDateString.includes(' ') ? endDateObj.format('YYYY-MM-DDTHH:mm:ss') + '+08:00' : endDateObj.format('YYYY-MM-DD');
                return {
                    type: 'date',
                    date: {
                        start,
                        end
                    }
                };
            } else {
                return {
                    type: 'date',
                    date: {
                        start,
                        end: null
                    }
                };
            }
        } else {
            return null;
        }
    }
    
}
    

module.exports = SynchronizerMiroNotionTasks ;

// Sync.test


const synchronizerMiroNotionTasks = new SynchronizerMiroNotionTasks ();

synchronizerMiroNotionTasks.sync();```
