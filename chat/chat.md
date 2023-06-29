- 项目参考
    - adgame对象相关
        - 服务器网址
            - https://alex.shinestu.com
            - 所以adgame接口的完整网址是如 https://alex.shinestu.com/current_project/set 
        - AdGame对象
            - ```javascript
const fs = require('fs');
const path = require('path');
const apiServer = require('./ApiServer');
const Habi=require('./habi');

class AdGame {
  constructor(apiServer) {
    this.name = 'adGame';
    this.apiServer = apiServer;
    this.currentProject = null;
    this.currentGoal = null;
    this.player = null; // Player对象
    this.dataFolderPath = 'fs_data_stack/AdGame'; // 数据文件夹路径
    this.dataFilePath = path.join(this.dataFolderPath, 'adGame_data.json'); // 数据文件路径
    this.boss = new Habi.Boss();

    // 创建数据文件夹
    this.createDataFolder();

    // 从文件中加载数据
    this.loadFromStorage();

    // 初始化Player对象
    this.player = new Proxy(new Player(), {
      set: (target, property, value) => {
        if (target.hasOwnProperty(property)) {
          target[property] = value;
        } else {
          // 继承已有的属性值
          if (typeof value !== 'undefined') {
            target[property] = value;
          }
        }
        
        // 将Player对象保存到本地
        this.saveToStorage();
        return true;
      }
    });

    // 将Player对象保存到本地
    this.saveToStorage();

    // 初始化
    this.init();
  }

  static getInstance(apiServer) {
    if (!AdGame.instance) {
      AdGame.instance = new AdGame(apiServer);
    }
    return AdGame.instance;
  }

  setApiServer(apiServer) {
    this.apiServer = apiServer;
  }

  init() {
    this.apiServer.setApi('/ad_game/current_project/set', 'post', (reqdata) => {
      console.log('ad_game/current_project/set', reqdata);
      this.currentProject = reqdata;
      this.saveToStorage(); // 保存数据到文件
      return 'set ok';
    });

    this.apiServer.setApi('/ad_game/current_project/get', 'get', () => {
      return this.currentProject;
    });

    this.apiServer.setApi('/ad_game/current_goal/set', 'post', (reqdata) => {
      console.log('ad_game/current_goal/set', reqdata);
      this.currentGoal = reqdata;
      this.saveToStorage(); // 保存数据到文件
      return 'set ok';
    });

    this.apiServer.setApi('/ad_game/current_goal/get', 'get', () => {
      return this.currentGoal;
    });

    this.apiServer.setApi('/ad_game/current_boss/get', 'get', async () => {
        let bossHealth=await this.boss.getHp();
        let bossMaxHp=await this.boss.getMaxHp();
        let bossHp=await this.boss.getHp();
        let boss_data={
            bossHealth:bossHealth,
            maxHp:bossMaxHp,
            hp:bossHp,
        }

      return boss_data;
    });

    this.apiServer.setApi('/ad_game/current_player/get', 'get', () => {
      return this.player;
    });

    console.log('adGame init');
  }

  createDataFolder() {
    if (!fs.existsSync(this.dataFolderPath)) {
      fs.mkdirSync(this.dataFolderPath, { recursive: true });
    }
  }

  saveToStorage() {
    const data = {
      currentProject: this.currentProject,
      currentGoal: this.currentGoal,
      player: this.player // 保存Player对象到数据中
    };

    // 将数据写入文件
    fs.writeFileSync(this.dataFilePath, JSON.stringify(data));
  }

  loadFromStorage() {
    try {
      const data = fs.readFileSync(this.dataFilePath, 'utf-8');
      const parsedData = JSON.parse(data);
      this.currentProject = parsedData.currentProject;
      this.currentGoal = parsedData.currentGoal;
      if (parsedData.player) {
        this.player = new Proxy(parsedData.player, {
          set: (target, property, value) => {
            if (target.hasOwnProperty(property)) {
              target[property] = value;
            } else {
              // 继承已有的属性值
              if (typeof value !== 'undefined') {
                target[property] = value;
              }
            }
            
            // 将Player对象保存到本地
            this.saveToStorage();
            return true;
          }
        });
      }
      console.log('adGame data loaded from storage:', parsedData);
    } catch (error) {
      console.log('Error loading data from storage:', error);
    }
  }
}

class Player {
  constructor() {
    this.hp = 0;
    this.mp = 0;
    this.exp = 0;
    this.gp = 0;
    this.lv = 0;
    this.maxExp = 0;
    this.maxHp = 0;
    this.maxMp = 0;
  }
}

const adGameInstance = AdGame.getInstance(apiServer);
module.exports = adGameInstance;
```
    - widget项目说明
        - widget说明
            - widget界面说明
                - 该widget左边是玩家的图像,下面是玩家的属性
                - 该widget右边是boss的图像,下面是boss的血条,血条下方是当前目标所属的项目
                    - 项目的数据源是来源于https://alex.shinestu.com的服务器处理后发送的抽象的miro板块的项目属性
                - boss左上方是目标模块,模块下半部分是成功和失败按钮,模块上半部分是当前目标显示
                - boss右下方是历史击杀数的显示
            - widget功能说明
                - 项目
                    - widget所有的项目和目标都来源于miro板块的脑图,脑图整体代表一个项目,其中的节点代表目标
    - 项目的GameWidgetProvider.java代码
        - 代码
            - ```javascript
package io.ionic.starter;

import android.app.PendingIntent;
import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;
import android.os.Handler;
import android.os.Looper;
import android.widget.RemoteViews;
import android.util.Log;

import org.json.JSONObject;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class GameWidgetProvider extends AppWidgetProvider {
    private AdGame adGame;
    private static WidgetElement widgetElement = new WidgetElement();

    @Override
    public void onEnabled(Context context) {
        super.onEnabled(context);
        Log.d("test_tag", "onEnabled");
        adGame = new AdGame(context, widgetElement);
        adGame.startAlarm();
    }

    @Override
    public void onDisabled(Context context) {
        super.onDisabled(context);
        adGame.stopAlarm();
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // Update the widgets as you originally planned.
    }

    // Define your WidgetElement class here
    public static class WidgetElement {
        public String goalText;
        public int health;
        public String projectTitle;
        public String level;
        public String money;
        public String inputText;
        public int bossHealth;
        public String killCount;
        public String miroLink; //项目目标链接
    }

    // Define your updateWidgetElement function here
    public static void updateWidgetElement(Context context, WidgetElement element) {
        // Fill in with your widget update code.
        //game_widget_layout = new RemoteViews(context.getPackageName(), R.layout.game_widget_layout);
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.game_widget_layout);

        // 更新任务目标文本
        if (element.goalText != null) {
            views.setTextViewText(R.id.goal_text, element.goalText.replace("\\n", "\n"));

        }

        // 更新血量
        if (element.health >= 0) {
            views.setProgressBar(R.id.boss_health_bar, 100, element.health, false);
        }

        // 更新项目标题
        if (element.projectTitle != null) {
            views.setTextViewText(R.id.project_text, element.projectTitle);

            // 如果miro链接存在，创建一个PendingIntent来打开链接
            if (element.miroLink != null) {
            Log.d("test_tag", "miroLink: " + element.miroLink);
                try {
                    Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(element.miroLink));
                    PendingIntent pendingIntent;
                    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.M) {
                        pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE);
                    } else {
                        pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);
                    }
                    views.setOnClickPendingIntent(R.id.project_text, pendingIntent);

                } catch (Exception e) {
                    Log.e("test_tag", "Failed to create PendingIntent for miroLink", e);
                }
            }
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

        // 获取小部件管理器
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);

        // 获取小部件的唯一标识符
        ComponentName componentName = new ComponentName(context, GameWidgetProvider.class);
        int[] appWidgetIds = appWidgetManager.getAppWidgetIds(componentName);

        // 更新小部件属性
        for (int appWidgetId : appWidgetIds) {
            appWidgetManager.updateAppWidget(appWidgetId, views);
        }
    }

    public static class UpdateReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d("test_tag", "Alarm received");
            WidgetElement widgetElement = new WidgetElement();
            AdGame adGame = new AdGame(context, widgetElement);
            adGame.update();
        }
    }
}
```
    - widget对应的game_widget_layout.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="280dp">
    <!-- Player FrameLayout -->
    <FrameLayout
        android:id="@+id/player_container"
        android:layout_width="150dp"
        android:layout_height="200dp"
        android:layout_alignParentStart="true"
        android:layout_alignParentBottom="true"
        android:layout_gravity="center_horizontal|bottom">

        <!-- Avatar 图片 -->
        <ImageView
            android:id="@+id/avatar_image"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:src="@drawable/avatar"
            android:scaleType="fitCenter"
            android:layout_gravity="center_horizontal|bottom" />


        <!-- 玩家属性区域 和输入区-->
        <RelativeLayout
            android:id="@+id/attribute_layout"
            android:layout_width="wrap_content"
            android:layout_height="60dp"
            android:orientation="vertical"
            android:layout_gravity="center_horizontal|bottom"
            android:layout_marginBottom="10dp">

            <!--血条蓝条-->
            <RelativeLayout
                android:id="@+id/health_bar_container"
                android:layout_width="100dp"
                android:layout_height="wrap_content"
                android:layout_alignParentTop="true"
                android:layout_alignParentStart="true">

                <ProgressBar
                    android:id="@+id/health_bar"
                    style="?android:attr/progressBarStyleHorizontal"
                    android:layout_width="match_parent"
                    android:layout_height="10dp"
                    android:layout_alignParentTop="true"
                    android:max="100"
                    android:progress="50"
                    android:progressTint="@color/health_color" />

                <ProgressBar
                    android:id="@+id/blue_bar"
                    style="?android:attr/progressBarStyleHorizontal"
                    android:layout_width="match_parent"
                    android:layout_height="10dp"
                    android:layout_below="@+id/health_bar"
                    android:max="100"
                    android:progress="70"
                    android:progressTint="@color/light_blue_900" />

            </RelativeLayout>

            <!--属性文本框-->
            <RelativeLayout
                android:id="@+id/attribute_text_layout"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_alignParentTop="true"
                android:layout_toEndOf="@id/health_bar_container"
                >

                <!-- LV文本 -->
                <FrameLayout
                    android:id="@+id/lv_text_container"
                    android:layout_width="60dp"
                    android:layout_height="15dp"
                    android:background="@drawable/project">

                    <TextView
                        android:id="@+id/lv_text"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:gravity="center_vertical|start"
                        android:paddingStart="2dp"
                        android:text="LV: 91"
                        android:textSize="9sp" />
                </FrameLayout>


                <!-- 金钱文本 -->
                <FrameLayout
                    android:id="@+id/money_text_container"
                    android:layout_width="60dp"
                    android:layout_height="15dp"
                    android:layout_below="@id/lv_text_container"
                    android:background="@drawable/project">
                    <TextView
                        android:id="@+id/ep_text"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:textSize="9sp"
                        android:paddingStart="2dp"
                        android:text="EP: 8765" />
                </FrameLayout>

            </RelativeLayout>

            <!-- 输入框 -->
            <FrameLayout
                android:id="@+id/input_container"
                android:layout_width="130dp"
                android:layout_height="30dp"
                android:layout_below="@id/attribute_text_layout"
                android:layout_centerHorizontal="true"
                android:layout_marginTop="5dp"
                android:background="@drawable/project">

                <TextView
                    android:id="@+id/input_info_text"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:paddingStart="2dp"
                    android:text="这里输入结果 "
                    android:textSize="11sp" />
            </FrameLayout>






        </RelativeLayout>





        <!-- 输入框 -->
        <!--
        <EditText
            android:id="@+id/input_text"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:hint="Enter failure result"
            android:layout_below="@id/attribute_layout"
            android:layout_alignStart="@id/attribute_layout"
            android:layout_marginTop="16dp" />
        -->
    </FrameLayout>


    <!--BOSS区整体框架-->
    <FrameLayout
        android:id="@+id/boss_area_container"
        android:layout_width="300dp"
        android:layout_height="280dp"
        android:layout_alignParentEnd="true"
        android:layout_alignParentBottom="true"
        >

        <!--任务交互区域-->
        <RelativeLayout
            android:id="@+id/action_area"
            android:layout_width="150dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="30dp"
            android:elevation="3dp"
            android:layout_gravity="left" >

            <!-- 任务目标 -->
            <FrameLayout
                android:id="@+id/goal_text_container"
                android:layout_width="wrap_content"
                android:maxLines="2"
                android:layout_height="50dp"
                android:layout_alignParentTop="true"
                android:layout_centerHorizontal="true"
                android:background="@drawable/project">
                <TextView
                    android:id="@+id/goal_text"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:paddingStart="2dp"
                    android:text="无目标"
                    android:textStyle="bold" 
                    android:maxLines="2"
                    android:ellipsize="end"
                    android:textSize="15sp" />
            </FrameLayout>

            <RelativeLayout
            android:id="@+id/action_area_button"
            android:layout_width="100dp"
            android:layout_height="50dp"
            android:layout_below="@id/goal_text_container"
            android:layout_centerHorizontal="true" >
                <!-- 完成按钮 -->
                <FrameLayout
                    android:id="@+id/goal_success_container"
                    android:layout_width="50dp"
                    android:layout_height="50dp"
                    android:background="@drawable/project">
                    <TextView
                        android:id="@+id/goal_success_text"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_gravity="center"
                        android:paddingStart="2dp"
                        android:text="✔"
                        android:textSize="29sp" />
                </FrameLayout>

                <!-- 失败按钮 -->
                <FrameLayout
                    android:id="@+id/goal_fail_container"
                    android:layout_width="50dp"
                    android:layout_height="50dp"
                    android:layout_toEndOf="@id/goal_success_container"
                    android:background="@drawable/project">
                    <TextView
                        android:id="@+id/goal_fail_text"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_gravity="center"
                        android:paddingStart="2dp"
                        android:text="❌"
                        android:textSize="22sp" />
                </FrameLayout>
            </RelativeLayout>

        </RelativeLayout>


        <!-- BOSS 本身区域 -->
        <FrameLayout
            android:id="@+id/boss_container"
            android:layout_width="170dp"
            android:layout_height="280dp"
            android:layout_alignParentEnd="true"
            android:layout_alignParentBottom="true"
            android:elevation="2dp"
            android:layout_gravity="center|bottom">

            <!-- BOSS 图片 -->
            <ImageView
                android:id="@+id/boss_image"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="fitCenter"
                android:src="@drawable/boss"
                android:layout_gravity="center_horizontal|bottom" />



            <!--项目框-->
            <FrameLayout
                android:id="@+id/project_area"
                android:layout_width="80dp"
                android:layout_height="20dp"
                android:background="@drawable/project"
                android:layout_marginBottom="43dp"
                android:layout_gravity="center|bottom">
                <!-- 项目标题 -->
                <TextView
                    android:id="@+id/project_text"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:paddingStart="2dp"
                    android:text="项目完成"
                    android:textSize="13sp" />
            </FrameLayout>


            <!--BOSS血条 -->
            <FrameLayout
                android:id="@+id/boss_hp_container"
                android:layout_width="100dp"
                android:layout_height="10dp"
                android:background="@drawable/project"
                android:layout_marginBottom="30dp"
                android:layout_gravity="center|bottom">
                <!-- HP -->
                <ProgressBar
                    android:id="@+id/boss_health_bar"
                    style="?android:attr/progressBarStyleHorizontal"
                    android:layout_width="match_parent"
                    android:layout_height="10dp"
                    android:layout_gravity="center"
                    android:max="100"
                    android:progress="50"
                    android:progressTint="@color/health_color" />
            </FrameLayout>
        </FrameLayout>

        <!--BOSS记录-->
        <FrameLayout
            android:id="@+id/kill_history_area"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:background="@drawable/project"
            android:layout_marginBottom="60dp"
            android:elevation="1dp"
            android:layout_gravity="right|bottom" >

            <!-- 标题 -->
            <TextView
                android:id="@+id/kill_title_text"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:layout_marginTop="7dp"
                android:text="历史击杀"
                android:textSize="10sp" />

            <!-- 标题 -->
            <TextView
                android:id="@+id/kill_num_text"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:layout_below="@id/kill_title_text"
                android:layout_marginTop="7dp"
                android:text="28"
                android:textSize="13sp" />
        </FrameLayout>
    </FrameLayout>






</RelativeLayout>
```
- 以上是我的项目相关参考代码,帮我完成以下要求,给我具体步骤和代码
    - 自动将玩家和boss的属性写入widget layout的对应文本
