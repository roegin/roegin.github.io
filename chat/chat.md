- 以下是ionic项目相关代码
    - 以下是widget对象的代码
        - ```javascript
package io.ionic.starter;

import android.app.PendingIntent;
import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.widget.RemoteViews;
import android.util.Log;
//引入log.d的 依赖

//引入网络依赖
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class GameWidgetProvider extends AppWidgetProvider {
    // 小部件的属性对象
    private static class WidgetElement {
        public String goalText;
        public int health;
        public String projectTitle;
        public String level;
        public String money;
        public String inputText;
        public int bossHealth;
        public String killCount;
    }

    // 内部存储小部件的属性对象
    private static WidgetElement widgetElement = new WidgetElement();

    //更新项目标题的方法
    public static void setCurrentProjectTitle(String projectTitle) {
        widgetElement.projectTitle = projectTitle;
    }

    //projecttiele
    public static void updateProjectTitle(Context context, String projectTitle) {
        // 更新项目标题
        widgetElement.projectTitle = projectTitle;

        // 获取小部件管理器
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);

        // 获取小部件的唯一标识符
        int[] appWidgetIds = appWidgetManager.getAppWidgetIds(new ComponentName(context, GameWidgetProvider.class));

        // 更新小部件属性
        for (int appWidgetId : appWidgetIds) {
            update_widget_element(context, appWidgetManager, appWidgetId, widgetElement);
        }
    }



    // 更新小部件的属性内容
    public static void update_widget_element(Context context, AppWidgetManager appWidgetManager, int appWidgetId, WidgetElement element) {
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.game_widget_layout);

        // 更新任务目标文本
        if (element.goalText != null) {
            views.setTextViewText(R.id.goal_text, element.goalText);
        }

        // 更新血量
        if (element.health >= 0) {
            views.setProgressBar(R.id.boss_health_bar, 100, element.health, false);
        }

        // 更新项目标题
        if (element.projectTitle != null) {
            views.setTextViewText(R.id.project_text, element.projectTitle);
        }

        // 更新等级文本
        if (element.level != null) {
            views.setTextViewText(R.id.lv_text, element.level);
        }

        // 更新金钱文本
        if (element.money != null) {
            views.setTextViewText(R.id.ep_text, element.money);
        }

        // 更新输入框文本
        if (element.inputText != null) {
            views.setTextViewText(R.id.input_info_text, element.inputText);
        }

        // 更新BOSS血条
        if (element.bossHealth >= 0) {
            views.setProgressBar(R.id.boss_health_bar, 100, element.bossHealth, false);
        }

        // 更新击杀计数
        if (element.killCount != null) {
            views.setTextViewText(R.id.kill_num_text, element.killCount);
        }

        // 更新小部件
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }

    // 在小部件创建时打印测试信息
    @Override
    public void onEnabled(Context context) {
        super.onEnabled(context);
        Log.d("test_tag", "这是一条调试日志消息");
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // 不执行默认的更新操作，将更新逻辑放在外部调用的方法中
    }
}

```
    - 以下是bridge的代码
        - ```javascript
package io.ionic.starter;

import android.content.Context;
import android.appwidget.AppWidgetManager;

public class WidgetBridge {
    public static void updateProjectTitle(Context context, int appWidgetId, String projectTitle) {
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
        GameWidgetProvider.updateProjectTitle(context, projectTitle);
    }
}
```
    - 以下是react的projectlistpage组件
        - ```javascript
import React from 'react';
import { IonContent, IonList, IonItem, IonLabel } from '@ionic/react';
import {  Plugins } from '@capacitor/core';
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';
import { useState, useEffect } from 'react';
const FilesystemDirectory = Directory;

//const { Filesystem } = Plugins;
const defaultProject = {
    name: "默认项目",
    miro_board_id: "",
    miro_board_link: "",
    total_goals_num: 0,
    completed_goals_num: 0,
    process: 0,
    dead_line: "",
    start_date: "",
    end_date: "",
    time_schedule: 0
  };
  

const ProjectsListPage: React.FC = () => {
  const [projectStack, setProjectStack] = useState([]);

  // 读取本地存储的项目数据
  const readProjectStack = async () => {
    try {
      const result = await Filesystem.readFile({
        path: 'file_stack/project_stack.json',
        directory: FilesystemDirectory.External
      });
      const projectStackData = JSON.parse(result.data);
      setProjectStack(projectStackData);
    } catch (error) {
      console.error('Error reading project stack:', error);
      setProjectStack([]);
    }
  };

  // 点击项目时更新小部件的项目标题
    const updateWidgetProjectTitle = async (projectTitle: string) => {
        try {
        await Filesystem.invoke('GameWidgetProvider.updateProjectTitle', { projectTitle });
        console.log('Project title updated in widget.');
        } catch (error) {
        console.error('Error updating project title in widget:', error);
        }
    };

  // 点击项目时弹出确认对话框
  const handleProjectClick = async (project: any) => {
    const confirmDialog = window.confirm(`确定选择项目 ${project.name}?`);
    if (confirmDialog) {
        updateWidgetProjectTitle(project.name);
    }
  };

  useEffect(() => {
    readProjectStack();
  }, []);

  return (
    <IonContent>
      <IonList>
        {projectStack.map((project: any) => (
          <IonItem key={project.name} onClick={() => handleProjectClick(project)}>
            <IonLabel>{project.name}</IonLabel>
          </IonItem>
        ))}
        <IonItem key={defaultProject.name} onClick={() => handleProjectClick(defaultProject)}>
          <IonLabel>{defaultProject.name}</IonLabel>
        </IonItem>
      </IonList>
    </IonContent>
  );
};

export default ProjectsListPage;

```
- 测试结果 是点击项目 并 确认并没有更新widget的项目标题,帮我检查代码给我解决步骤
