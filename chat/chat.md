- 以下是gamewidget的java代码
    - ```javascript
package com.example.ad_game_java;

import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.Context;
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

    //在小部件创建时打印测试信息
    @Override
    public void onEnabled(Context context) {
        super.onEnabled(context);

        // 创建一个 ExecutorService 实例来处理后台任务
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // 使用 Timer 创建一个定时任务
        Timer timer = new Timer();

        // 设定每分钟执行一次的任务
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                // 创建一个 Runnable 对象来执行网络请求
                Runnable runnable = new Runnable() {
                    @Override
                    public void run() {
                        // 执行网络请求
                        try {
                            // 创建一个 URL 对象
                            URL url = new URL("https://alex.shinestu.com/ad_game/get_data");

                            // 打开 URL 的连接
                            HttpURLConnection connection = (HttpURLConnection) url.openConnection();

                            // 设置请求方法为 GET
                            connection.setRequestMethod("GET");

                            // 获取服务器返回的输入流
                            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));

                            // 读取返回的数据
                            StringBuilder response = new StringBuilder();
                            String line;
                            while ((line = reader.readLine()) != null) {
                                response.append(line);
                            }

                            // 关闭连接和读取器
                            reader.close();
                            connection.disconnect();

                            // 打印返回的数据
                            Log.d("test_tag", response.toString());

                        } catch (Exception e) {
                            Log.d("test_tag", "获取数据时出现错误", e);
                        }
                    }
                };

                // 将网络请求的 Runnable 提交给 ExecutorService 执行
                executor.execute(runnable);
            }
        }, 0, 60 * 1000);  // 第一次执行的延迟（0 表示立即执行），以及每次执行的间隔（60*1000 毫秒，即一分钟）

        Log.d("test_tag", "这是一条调试日志消息");
    }




    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            update_game_widget(context, appWidgetManager, appWidgetId);
        }
    }

    public void update_game_widget(Context context, AppWidgetManager appWidgetManager, int appWidgetId) {
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.game_widget_layout);

        // Update views
        // Here you can update your views as you like
        // views.setTextViewText(R.id.example_view, "Example text");

        appWidgetManager.updateAppWidget(appWidgetId, views);
    }
}
```
- 以下是layout文件
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
        android:layout_width="230dp"
        android:layout_height="280dp"
        android:layout_alignParentEnd="true"
        android:layout_alignParentBottom="true"
        >

        <!--任务交互区域-->
        <RelativeLayout
            android:id="@+id/action_area"
            android:layout_width="100dp"
            android:layout_height="75dp"
            android:layout_marginTop="30dp"
            android:elevation="3dp"
            android:layout_gravity="left" >

            <!-- 任务目标 -->
            <FrameLayout
                android:id="@+id/goal_text_container"
                android:layout_width="70dp"
                android:layout_height="25dp"
                android:layout_alignParentTop="true"
                android:layout_centerHorizontal="true"
                android:background="@drawable/project">
                <TextView
                    android:id="@+id/goal_text"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:paddingStart="2dp"
                    android:text="目标XX"
                    android:textSize="15sp" />
            </FrameLayout>

            <!-- 完成按钮 -->
            <FrameLayout
                android:id="@+id/goal_success_container"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_below="@id/goal_text_container"
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
                android:layout_below="@id/goal_text_container"
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
                <!-- test -->
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
- 给我如下要求的详细步骤
    - 点击widget的项目标题部分,跳转到一个空白的activity页面
