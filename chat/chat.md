文件夹:C:\Local_dev\test_v10\app\src\main\java\com\example\test_v10
    - 这是 AppDatabase.kt
        - ```javascript
package com.example.test_v10

import androidx.room.Database
import androidx.room.RoomDatabase

@Database(entities = [Project::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun projectDao(): ProjectDao
}
        ```
    - 这是 GameWidgetProvider.kt
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
        views.setTextViewText(R.id.goal_text, gameData.goal)
        views.setProgressBar(R.id.boss_health_bar, 100, gameData.bossHp.toInt(), false)
        views.setTextViewText(R.id.project_text, gameData.project)
        views.setTextViewText(R.id.kill_num_text, gameData.record)

        // 设置点击事件，跳转到 MainActivity 输入
        val mainIntent = Intent(context, MainActivity::class.java)
        val mainPendingIntent = PendingIntent.getActivity(context, appWidgetId, mainIntent, PendingIntent.FLAG_UPDATE_CURRENT)
        views.setOnClickPendingIntent(R.id.input_info_text, mainPendingIntent)

        // 设置点击事件，设置项目，跳转到 ProjectActivity
        val projectIntent = Intent(context, ProjectActivity::class.java)
        val projectPendingIntent = PendingIntent.getActivity(context, appWidgetId, projectIntent, PendingIntent.FLAG_UPDATE_CURRENT)
        views.setOnClickPendingIntent(R.id.project_text, projectPendingIntent)


        //替换项目标题
        val sharedPreferences = context.getSharedPreferences("current_project", Context.MODE_PRIVATE)
        val gson = Gson()
        val json = sharedPreferences.getString("current_project", null)
        val currentProject = gson.fromJson(json, Project::class.java)



        views.setTextViewText(R.id.project_text, currentProject?.name)



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
    - 这是 HelloworldWidget.kt
        - ```javascript
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
    - 这是 MainActivity.kt
        - ```javascript
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


        // 设置前景输入框
        val foregroundView = findViewById<RelativeLayout>(R.id.foreground_view)
        layoutInflater.inflate(R.layout.input_layout, foregroundView, true)

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
    - 这是 Project.kt
        - ```javascript
package com.example.test_v10

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity
data class Project(
    @PrimaryKey(autoGenerate = true) val id: Int,
    val name: String,
    val miro_board_id: String,
    val miro_board_link: String,
    val total_goals_num: Int,
    val completed_goals_num: Int,
    val process: Double,
    val dead_line: String,
    val start_date: String,
    val end_date: String,
    val time_schedule: Double
)

        ```
    - 这是 ProjectActivity.kt
        - ```javascript
package com.example.test_v10

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.example.test_v10.ProjectListFragment



class ProjectActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_project)

        val projectName = intent.getStringExtra("project_name")

        //显示projectlist fragment
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragment_container, ProjectListFragment())
            .commit()

    }
}

        ```
    - 这是 ProjectAdapter.kt
        - ```javascript
package com.example.test_v10

import android.view.View
import android.view.ViewGroup
import android.view.LayoutInflater
import android.widget.TextView
import androidx.recyclerview.widget.RecyclerView


class ProjectAdapter(private val projects: List<Project>, private val listener: OnItemClickListener) : RecyclerView.Adapter<ProjectAdapter.ProjectViewHolder>() {

    inner class ProjectViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView), View.OnClickListener {
        val projectName: TextView = itemView.findViewById(R.id.project_name)

        init {
            itemView.setOnClickListener(this)
        }

        override fun onClick(v: View?) { //sdsd
            val position = adapterPosition
            if (position != RecyclerView.NO_POSITION) {
                listener.onItemClick(projects[position])
            }
        }
    }

    interface OnItemClickListener {
        fun onItemClick(project: Project)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ProjectViewHolder {
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.project_item, parent, false)
        return ProjectViewHolder(itemView)
    }

    override fun onBindViewHolder(holder: ProjectViewHolder, position: Int) {
        val currentProject = projects[position]
        holder.projectName.text = currentProject.name
    }

    override fun getItemCount() = projects.size
}

        ```
    - 这是 ProjectDao.kt
        - ```javascript
package com.example.test_v10

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query

@Dao
interface ProjectDao {
    @Query("SELECT * FROM project")
    fun getAll(): List<Project>

    @Insert
    fun insertAll(vararg projects: Project)
}

        ```
    - 这是 ProjectListFragment.kt
        - ```javascript
package com.example.test_v10

import android.view.LayoutInflater

import android.content.Context
import android.os.Bundle
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import androidx.recyclerview.widget.RecyclerView
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.room.Room
import com.google.gson.Gson


class ProjectListFragment : Fragment(), ProjectAdapter.OnItemClickListener {

    private lateinit var projectDao: ProjectDao

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        val view = inflater.inflate(R.layout.fragment_project_list, container, false)

        val recyclerView = view.findViewById<RecyclerView>(R.id.recycler_view)
        recyclerView.layoutManager = LinearLayoutManager(context)

        val db = Room.databaseBuilder(
            requireContext(),
            AppDatabase::class.java, "database-name"
        ).build()
        projectDao = db.projectDao()

        val projects = projectDao.getAll()
        val adapter = ProjectAdapter(projects, this)
        recyclerView.adapter = adapter

        return view
    }

    override fun onItemClick(project: Project) {
        // Save the current project to internal storage
        val sharedPreferences = requireActivity().getSharedPreferences("current_project", Context.MODE_PRIVATE)
        val editor = sharedPreferences.edit()
        val gson = Gson()
        val json = gson.toJson(project)
        editor.putString("current_project", json)
        editor.apply()
    }
}

        ```
    - 这是 WidgetProvider.kt
        - ```javascript
package com.example.test_v10

import android.appwidget.AppWidgetManager
import android.appwidget.AppWidgetProvider
import android.content.Context
import android.widget.RemoteViews

class WidgetProvider : AppWidgetProvider() {
    override fun onUpdate(context: Context, appWidgetManager: AppWidgetManager, appWidgetIds: IntArray) {
        for (appWidgetId in appWidgetIds) {
            val views = RemoteViews(context.packageName, R.layout.widget_layout)
            appWidgetManager.updateAppWidget(appWidgetId, views)
        }
    }
}

        ```
文件夹:C:\Local_dev\test_v10\app\src\main\res\layout
    - 这是 activity_main.xml
        - ```javascript
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
    - 这是 activity_project.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#333333">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragment_container"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/bottom_navigation"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/bottom_navigation_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>

        ```
    - 这是 fragment_project_list.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<androidx.recyclerview.widget.RecyclerView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />

        ```
    - 这是 game_widget_layout.xml
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
    - 这是 input_layout.xml
        - ```javascript
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
    - 这是 project_item.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/project_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp" />

    <!-- Add more views here to display other project data -->

</LinearLayout>

        ```
    - 这是 widget_layout.xml
        - ```javascript
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
文件夹:C:\Local_dev\test_v10\app\src\main\res\menu
    - 这是 bottom_navigation_menu.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/navigation_project_list"
        android:icon="@drawable/ic_project_list"
    android:title="项目列表" />

    <item
        android:id="@+id/navigation_project_overview"
        android:icon="@drawable/ic_project_overview"
    android:title="项目总览" />
</menu>

        ```
文件夹:C:\Local_dev\test_v10\app\src\main\res\xml
    - 这是 backup_rules.xml
        - ```javascript
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
    - 这是 data_extraction_rules.xml
        - ```javascript
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
    - 这是 game_widget_info.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="350dp"
    android:minHeight="100dp"
    android:updatePeriodMillis="6000"
    android:initialLayout="@layout/game_widget_layout" />

        ```
    - 这是 widget_provider.xml
        - ```javascript
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="100dp"
    android:minHeight="100dp"
    android:updatePeriodMillis="60000"
    android:initialLayout="@layout/widget_layout" />

        ```
