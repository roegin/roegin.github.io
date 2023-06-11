<!-- coding:utf-8 -->

- path:C:\Local_dev\test_v10\app\src\main\java\com\example\test_v10
    - here is code with GameWidgetProvider.kt
        - ```kotlin
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
    - here is code with HelloworldWidget.kt
        - ```kotlin
package com.example.test_v10

import android.appwidget.AppWidgetManager
import android.appwidget.AppWidgetProvider
import android.content.Context
import android.widget.RemoteViews

/**
 * Implementation of App Widget functionality.
 */
open class HelloworldWidget : AppWidgetProvider() {
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray
    ) {
        for (appWidgetId in appWidgetIds) {
            updateAppWidget(context, appWidgetManager, appWidgetId)
        }
    }


    override fun onEnabled(context: Context) {
        // Enter relevant functionality for when the first widget is created
    }

    override fun onDisabled(context: Context) {
        // Enter relevant functionality for when the last widget is disabled
    }
}

internal fun updateAppWidget(  //小部件更新对象
    context: Context,
    appWidgetManager: AppWidgetManager,
    appWidgetId: Int,
) {

    val views = RemoteViews(context.packageName, R.layout.widget_layout)

    //更新
    appWidgetManager.updateAppWidget(appWidgetId, views)
}
        ```
    - here is code with MainActivity.kt
        - ```kotlin
package com.example.test_v10

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle

//输入框
import android.app.PendingIntent
import android.content.Intent
import android.view.inputmethod.EditorInfo
import android.widget.FrameLayout
import android.widget.RelativeLayout
import android.widget.EditText



class MainActivity : AppCompatActivity() {
    private lateinit var inputEditText: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 设置背景
        val backgroundView = findViewById<FrameLayout>(R.id.background_view)
        backgroundView.setBackgroundResource(R.drawable.project) // 替换为你的背景图片
        backgroundView.isClickable = true


        /*
        // 设置前景输入框
        val foregroundView = findViewById<RelativeLayout>(R.id.foreground_view)
        layoutInflater.inflate(R.layout.input_layout, foregroundView, true)

         */

        // 获取输入框
        inputEditText = findViewById(R.id.input_edit_text)
        inputEditText.setOnEditorActionListener { _, actionId, _ ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                sendInputToServer(inputEditText.text.toString()) // 将输入发送到远端服务器
                inputEditText.clearFocus()
                true
            } else {
                false
            }
        }
    }

    private fun sendInputToServer(input: String) {
        // 将输入发送到远端服务器的逻辑
        // ...
    }
}


        ```
- path:C:\Local_dev\test_v10\app\src\main\res\layout
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
    - here is code with input_layout.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/input_edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:hint="请输入内容"
        android:padding="8dp"
        android:textSize="16sp" />

</RelativeLayout>

        ```
    - here is code with widget_layout.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/widget_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World2222!" />

</LinearLayout>

        ```
- path:C:\Local_dev\test_v10\app\src\main\res\xml
    - here is code with backup_rules.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?><!--
   Sample backup rules file; uncomment and customize as necessary.
   See https://developer.android.com/guide/topics/data/autobackup
   for details.
   Note: This file is ignored for devices older that API 31
   See https://developer.android.com/about/versions/12/backup-restore
-->
<full-backup-content>
    <!--
   <include domain="sharedpref" path="."/>
   <exclude domain="sharedpref" path="device.xml"/>
-->
</full-backup-content>
        ```
    - here is code with data_extraction_rules.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?><!--
   Sample data extraction rules file; uncomment and customize as necessary.
   See https://developer.android.com/about/versions/12/backup-restore#xml-changes
   for details.
-->
<data-extraction-rules>
    <cloud-backup>
        <!-- TODO: Use <include> and <exclude> to control what is backed up.
        <include .../>
        <exclude .../>
        -->
    </cloud-backup>
    <!--
    <device-transfer>
        <include .../>
        <exclude .../>
    </device-transfer>
    -->
</data-extraction-rules>
        ```
    - here is code with game_widget_info.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="350dp"
    android:minHeight="100dp"
    android:updatePeriodMillis="6000"
    android:initialLayout="@layout/game_widget_layout" />

        ```
    - here is code with widget_provider.xml
        - ```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="100dp"
    android:minHeight="100dp"
    android:updatePeriodMillis="60000"
    android:initialLayout="@layout/widget_layout" />

        ```
