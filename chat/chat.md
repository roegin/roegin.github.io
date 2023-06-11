<!-- coding:utf-8 -->

- path:C:\Local_dev\Ad_game_java\app\src\main\java\com\example\ad_game_java
    - here is code with GameData.java
        - ```java
package com.example.ad_game_java;

public class GameData {
    private String player_hp;
    private String player_mp;
    private String lv;
    private String ep;
    private String inputInfo;
    private String goal;
    private String bossHp;
    private String project;
    private String record;

    public GameData(String player_hp, String player_mp, String lv, String ep, String inputInfo, String goal, String bossHp, String project, String record) {
        // 构造函数的实现
    }

    public String getPlayer_hp() {
        return player_hp;
    }

    public void setPlayer_hp(String player_hp) {
        this.player_hp = player_hp;
    }

    public String getPlayer_mp() {
        return player_mp;
    }

    public void setPlayer_mp(String player_mp) {
        this.player_mp = player_mp;
    }

    public String getLv() {
        return lv;
    }

    public void setLv(String lv) {
        this.lv = lv;
    }

    public String getEp() {
        return ep;
    }

    public void setEp(String ep) {
        this.ep = ep;
    }

    public String getInputInfo() {
        return inputInfo;
    }

    public void setInputInfo(String inputInfo) {
        this.inputInfo = inputInfo;
    }

    public String getGoal() {
        return goal;
    }

    public void setGoal(String goal) {
        this.goal = goal;
    }

    public String getBossHp() {
        return bossHp;
    }

    public void setBossHp(String bossHp) {
        this.bossHp = bossHp;
    }

    public String getProject() {
        return project;
    }

    public void setProject(String project) {
        this.project = project;
    }

    public String getRecord() {
        return record;
    }

    public void setRecord(String record) {
        this.record = record;
    }
}

        ```
    - here is code with GameWidgetProvider.java
        - ```java
package com.example.ad_game_java;

import android.app.PendingIntent;
import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.util.Log;
import android.widget.RemoteViews;

import com.example.ad_game_java.GameData;
import com.fasterxml.jackson.databind.ObjectMapper;

import java.util.Timer;
import java.util.TimerTask;
import java.io.IOException;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import com.example.ad_game_java.R;



public class GameWidgetProvider extends AppWidgetProvider {
    private Timer timer;
    private TimerTask updateTask;

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            update_game_widget(context, appWidgetManager, appWidgetId);
        }
    }

    @Override
    public void onEnabled(Context context) {
        super.onEnabled(context);

        if (updateTask != null) {
            updateTask.cancel();
        }

        if (timer != null) {
            timer.cancel();
        }

        timer = new Timer();
        updateTask = new TimerTask() {
            @Override
            public void run() {
                Log.d("GameWidgetUpdate", "start onenable");
                AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
                int[] appWidgetIds = appWidgetManager.getAppWidgetIds(new ComponentName(context.getPackageName(), getClass().getName()));

                for (int appWidgetId : appWidgetIds) {
                    update_game_widget(context, appWidgetManager, appWidgetId);
                }
            }
        };

        timer.schedule(updateTask, 0, 60000);  // Update every minute
    }

    @Override
    public void onDisabled(Context context) {
        super.onDisabled(context);

        if (updateTask != null) {
            updateTask.cancel();
        }

        if (timer != null) {
            timer.cancel();
        }
    }

    public void update_game_widget(Context context, AppWidgetManager appWidgetManager, int appWidgetId) {
        Log.d("GameWidgetUpdate", "await fetchdata");
        new Thread(() -> {
            GameData gameData = fetchData();  // Fetch data from the API
            Log.d("GameWidgetUpdate", "Fetched game data: " + gameData); // 输出解析后的 GameData 对象
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.game_widget_layout);
            // Update views with fetched data
            views.setTextViewText(R.id.lv_text, "LV: " + gameData.getLv());
            views.setTextViewText(R.id.ep_text, "EP: " + gameData.getEp());
            views.setProgressBar(R.id.health_bar, 100, Integer.parseInt(gameData.getPlayer_hp()), false);
            views.setProgressBar(R.id.blue_bar, 100, Integer.parseInt(gameData.getPlayer_mp()), false);
            views.setCharSequence(R.id.input_info_button, "setText", gameData.getInputInfo());
            views.setTextViewText(R.id.goal_text, gameData.getGoal());
            views.setProgressBar(R.id.boss_health_bar, 100, Integer.parseInt(gameData.getBossHp()), false);
            views.setTextViewText(R.id.project_text, gameData.getProject());
            views.setTextViewText(R.id.kill_num_text, gameData.getRecord());

            // Create an Intent that will open MainActivity
            Intent intent = new Intent(context, MainActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_IMMUTABLE);
            views.setOnClickPendingIntent(R.id.input_info_button, pendingIntent);

            appWidgetManager.updateAppWidget(appWidgetId, views);
        }).start();
    }



    public GameData fetchData() {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url("https://alex.shinestu.com/ad_game/get_data")  // Replace with your API URL
                .build();

        Log.d("GameWidgetUpdate", "new request");
        try {
            Response response = client.newCall(request).execute();
            Log.d("GameWidgetUpdate", "already get fetchdata");
            String json = response.body().string();

            Log.d("GameDataFetch", "JSON from server: " + json); // 输出获取的 JSON 数据

            ObjectMapper mapper = new ObjectMapper();

            return mapper.readValue(json, GameData.class);
        } catch (IOException e) {
            Log.e("GameWidgetUpdate", "Failed to fetch data", e);
            return new GameData("", "", "", "", "", "", "", "", "");  // Return a default GameData if fetch failed
        }
    }
}


        ```
    - here is code with MainActivity.java
        - ```java
package com.example.ad_game_java;

import android.os.Bundle;
import android.view.KeyEvent;
import android.view.inputmethod.EditorInfo;
import android.widget.EditText;
import android.widget.FrameLayout;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    private EditText inputEditText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 设置背景
        FrameLayout backgroundView = findViewById(R.id.background_view);
        backgroundView.setBackgroundResource(R.drawable.project); // 替换为你的背景图片
        backgroundView.setClickable(true);

        // 获取输入框
        inputEditText = findViewById(R.id.input_edit_text);
        inputEditText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public boolean onEditorAction(TextView textView, int actionId, KeyEvent keyEvent) {
                if (actionId == EditorInfo.IME_ACTION_DONE) {
                    sendInputToServer(inputEditText.getText().toString()); // 将输入发送到远端服务器
                    inputEditText.clearFocus();
                    return true;
                } else {
                    return false;
                }
            }
        });
    }

    private void sendInputToServer(String input) {
        // 将输入发送到远端服务器的逻辑
        // ...
    }
}

        ```
- path:C:\Local_dev\Ad_game_java\app\src\main\res\layout
    - here is code with activity_main.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/background_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/project"
    android:clickable="true"
    tools:context=".MainActivity">

    <RelativeLayout
        android:id="@+id/foreground_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <EditText
            android:id="@+id/input_edit_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="请输入内容"
            android:padding="8dp"
            android:textSize="16sp" />

    </RelativeLayout>

</FrameLayout>

        ```
    - here is code with game_widget_layout.xml
        - ```xml
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
                <Button
                    android:id="@+id/input_info_button"
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
