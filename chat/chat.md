- 以下是我的MainActivity.kt
- 以下是activity_main.xml
- 以下是GameWidgetProvider.kt
    - ```javascript
package com.example.test_v10

import android.annotation.SuppressLint
import android.appwidget.AppWidgetManager
import android.appwidget.AppWidgetProvider
import android.content.ComponentName
import android.content.Context
import android.widget.RemoteViews
import com.google.gson.Gson
import okhttp3.OkHttpClient
import okhttp3.Request
import okhttp3.Response


//
import kotlinx.coroutines.*

import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.isActive
import kotlinx.coroutines.delay

//输出日志用
import android.util.Log // 这个是新加的，用于输出日志

//输入框
import android.app.PendingIntent
import android.content.Intent




class GameWidgetProvider : HelloworldWidget() {
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray
    ) {
        for (appWidgetId in appWidgetIds) {
            update_game_widget(context, appWidgetManager, appWidgetId)
        }
    }

    private var updateJob: Job? = null

    override fun onEnabled(context: Context) {
        super.onEnabled(context)

        updateJob?.cancel()
        updateJob = GlobalScope.launch(Dispatchers.IO) {
            while (isActive) {
                Log.d("GameWidgetUpdate", "start onenable")
                val appWidgetManager = AppWidgetManager.getInstance(context)
                val appWidgetIds = appWidgetManager.getAppWidgetIds(ComponentName(context.packageName, javaClass.name))
    
                for (appWidgetId in appWidgetIds) {
                    update_game_widget(context, appWidgetManager, appWidgetId)
                }
    
                delay(60_000L)  // Wait for 1 minute before the next update
            }
        }
    }
    
    override fun onDisabled(context: Context) {
        super.onDisabled(context)
    
        updateJob?.cancel()
    }
}

//@SuppressLint("RemoteViewLayout")
internal fun update_game_widget(
    context: Context,
    appWidgetManager: AppWidgetManager,
    appWidgetId: Int
) {
    GlobalScope.launch(Dispatchers.Main) {
        Log.d("GameWidgetUpdate", "await fetchdata")
        val gameData = fetchData()  // Fetch data from the API

        Log.d("GameWidgetUpdate", "Fetched game data: $gameData") // 输出解析后的 GameData 对象

        val views = RemoteViews(context.packageName, R.layout.game_widget_layout)

        // Update views with fetched data
        views.setTextViewText(R.id.lv_text, "LV: ${gameData.lv.toString()}")
        views.setTextViewText(R.id.ep_text, "EP: ${gameData.ep.toString()}")
        views.setProgressBar(R.id.health_bar, 100, gameData.player_hp.toInt(), false)
        views.setProgressBar(R.id.blue_bar, 100, gameData.player_mp.toInt(), false)
        views.setTextViewText(R.id.input_info_text, gameData.inputInfo)

        // 设置点击事件和跳转到 MainActivity
        val intent = Intent(context, MainActivity::class.java)
        val pendingIntent = PendingIntent.getActivity(context, appWidgetId, intent, PendingIntent.FLAG_UPDATE_CURRENT)
        views.setOnClickPendingIntent(R.id.input_info_text, pendingIntent)

        views.setTextViewText(R.id.goal_text, gameData.goal)
        views.setProgressBar(R.id.boss_health_bar, 100, gameData.bossHp.toInt(), false)
        views.setTextViewText(R.id.project_text, gameData.project)
        views.setTextViewText(R.id.kill_num_text, gameData.record)

        appWidgetManager.updateAppWidget(appWidgetId, views) //
    }
}



//点击输入辅助函数
private fun getPendingIntent(context: Context, appWidgetId: Int): PendingIntent {
    val intent = Intent(context, MainActivity::class.java) // 替换为你的MainActivity类
    intent.action = AppWidgetManager.ACTION_APPWIDGET_UPDATE
    intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, intArrayOf(appWidgetId))
    return PendingIntent.getActivity(context, appWidgetId, intent, PendingIntent.FLAG_UPDATE_CURRENT)
}



//OKHTTP
data class GameData(
    val player_hp: String,
    val player_mp: String,
    val lv: String,
    val ep: String,
    val inputInfo: String,
    val goal: String,
    val bossHp: String,
    val project: String,
    val record: String
)

//fetchdata
suspend fun fetchData(): GameData = withContext(Dispatchers.IO) {
    val client = OkHttpClient()

    val request = Request.Builder()
        .url("https://alex.shinestu.com/ad_game/get_data")  // Replace with your API URL
        .build()

    Log.d("GameWidgetUpdate", "new request")
    try {
        client.newCall(request).execute().use { response ->
            Log.d("GameWidgetUpdate", "already get fetchdata")
            val json = response.body?.string()

            Log.d("GameDataFetch", "JSON from server: $json") // 输出获取的 JSON 数据

            val gson = Gson()

            gson.fromJson(json, GameData::class.java)
        }
    } catch (e: Exception) {
        Log.e("GameWidgetUpdate", "Failed to fetch data", e)
        GameData("", "", "", "", "", "", "", "", "")  // Return a default GameData if fetch failed
    }
}




```
- 以下是game_widget_layout.html
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
