# 期中作业 记事本                                   
>                                                                               软工（1）班 黄磊 123012016005
## 1.实验内容
阅读NotePad的源代码并做相应的扩展。


**已实现功能汇总：**


**基础功能拓展：**


1.NoteList中显示条目增加时间戳显示


2.添加笔记查询功能（根据标题进行模糊查询）

**附加功能拓展：**

1.UI美化-改变每条便签颜色以代表不同心情

2.导出笔记

3.更改文本字体参数（大小、颜色）


![主界面](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/Screenshot_2019-%E4%B8%BB%E7%95%8C%E9%9D%A2.png)

**图1 记事本主界面**

![更新标题](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E6%A0%87%E9%A2%98.png?raw=true)

**图2 更新标题**

![长按弹出菜单](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E9%95%BF%E6%8C%89%E5%BC%B9%E5%87%BA%E8%8F%9C%E5%8D%95.png)

**图3 长按弹出菜单**

## 2.基础扩展


**1.NoteList中显示条目增加时间戳显示 **


![记事本主界面（时间戳）](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/Screenshot_2019-%E4%B8%BB%E7%95%8C%E9%9D%A2.png)

**图4 记事本主界面（时间戳）**

实现时间戳可以在原有代码的线性布局的基础上增加一个textview，用于显示时间。

**关键实现代码：**



**notelist_item.xml**


 
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView
            android:id="@android:id/text1"
            android:layout_width="match_parent"
            android:layout_height="?android:attr/listPreferredItemHeight"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:gravity="center_vertical"
            android:paddingLeft="5dip"
            android:singleLine="true"/>
        <TextView
            android:id="@+id/text2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:textColor="@color/black"
            android:textSize="20sp"
            />
    </LinearLayout>

**NotePad.java**

     private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
        };

    
**2.添加笔记查询功能（根据标题进行模糊查询）**

![查询界面](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%9F%BA%E4%BA%8E%E6%A0%87%E9%A2%98%E6%A8%A1%E7%B3%8A%E6%9F%A5%E8%AF%A2.png?raw=true)

**图5 模糊查询界面**

实现了通过标题进行模糊查询

**关键实现代码：**

**NoteSearch.java**

    @Override
    public boolean onQueryTextChange(String s) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+s+"%" };//查询条件参数，配合selection参数使用,%通配多个字符

        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。
        //managedQuery(Uri, String[], String, String[], String)等同于Context.getContentResolver().query()
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.用于ContentProvider查询的URI，从这个URI获取数据
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date.用于标识uri中有哪些columns需要包含在返回的Cursor对象中
                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择
                selectionArgs,                    // 查询条件参数，配合selection参数使用
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.查询结果的排序方式，按照某个columns来排序，例：String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE
        );

        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        lListView.setAdapter(adapter);
        return true;
    }


**note_search.xml**

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="请输入便签标题..."
        android:iconifiedByDefault="true">
    </SearchView>
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/list"></ListView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>


## 3.附加功能扩展

**1.UI美化-改变每条便签颜色以代表不同心情**

不同颜色代表你的不同心情。
珠光白：明快；天空蓝：淡雅；樱草金：欢快；草原绿：放松；烈焰红：激情。

![心情便签（颜色）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%BF%83%E6%83%85%E4%BE%BF%E7%AD%BE%EF%BC%88%E9%A2%9C%E8%89%B2%EF%BC%89.png)

**图6 心情便签（颜色）1**

![心情便签（颜色）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%BF%83%E6%83%85%E4%BE%BF%E7%AD%BE%EF%BC%88%E9%A2%9C%E8%89%B2%EF%BC%892.png)

**图7 心情便签（颜色）2**

![心情便签（颜色）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%BF%83%E6%83%85%E4%BE%BF%E7%AD%BE%EF%BC%88%E9%A2%9C%E8%89%B2%EF%BC%893.png)

**图8 心情便签（颜色）3**

**关键实现代码：**



**note_color.xml**


<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button
        android:id="@+id/color_white"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="珠光白:明快"
        android:layout_weight="1"
        android:background="@color/white"
        android:onClick="white"/>
    <Button
        android:id="@+id/color_yellow"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="樱草金:欢快"
        android:layout_weight="1"
        android:background="@color/yellow"
        android:onClick="yellow"/>
    <Button
        android:id="@+id/color_blue"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="天空蓝:淡雅"
        android:layout_weight="1"
        android:background="@color/royalblue"
        android:onClick="blue"/>
    <Button
        android:id="@+id/color_green"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="草原绿:放松"
        android:layout_weight="1"
        android:background="@color/green"
        android:onClick="green"/>
    <Button
        android:id="@+id/color_red"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="烈焰红:激情"
        android:layout_weight="1"
        android:background="@color/red"
        android:onClick="red"/>
</LinearLayout>

**NoteColor.java**

    @Override
    protected void onPause() {
        //选完后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

**Notelist.java**

    public class MyCursorAdapter extends SimpleCursorAdapter {
        public MyCursorAdapter(Context context, int layout, Cursor c,
                               String[] from, int[] to) {
            super(context, layout, c, from, to);
        }
        @Override
        public void bindView(View view, Context context, Cursor cursor){
            super.bindView(view, context, cursor);
            //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
            int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));

            //应用已选颜色
            switch (x){
                case NotePad.Notes.DEFAULT_COLOR:       //珠光白
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
                case NotePad.Notes.YELLOW_COLOR:       //樱草金
                    view.setBackgroundColor(Color.rgb(255, 255, 0));
                    break;
                case NotePad.Notes.BLUE_COLOR:       //天空蓝
                    view.setBackgroundColor(Color.rgb(0, 191, 255));
                    break;
                case NotePad.Notes.GREEN_COLOR:       //草原绿
                    view.setBackgroundColor(Color.rgb(0, 255, 127));
                    break;
                case NotePad.Notes.RED_COLOR:       //烈焰红
                    view.setBackgroundColor(Color.rgb(255, 0, 0));
                    break;
                default:
                    view.setBackgroundColor(Color.rgb(255, 255, 255));
                    break;
            }
        }
    }

 **2.导出笔记**
 
 由于手机安卓9.0版本太高，部分情况下无法正常运行，所以此功能在虚拟机上运行测试。

![导出笔记（1）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%AF%BC%E5%87%BA%E6%96%87%E4%BB%B6%EF%BC%881%EF%BC%89.png?raw=true)

**图9 导出笔记（1）**

![导出笔记（2）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E5%AF%BC%E5%87%BA%E6%96%87%E4%BB%B6%EF%BC%882%EF%BC%89.png?raw=true)

**图10 导出笔记（2）**

**关键实现代码：**

首先要在清单中注册相关读取权限

**AndroidManifest.xml**

   <uses-sdk>
        android:minSdkVersion="11"
    </uses-sdk>

    <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"
        tools:ignore="ProtectedPermissions" />

    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>

OutputText.java

    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File(sdCardDir.getCanonicalPath()+ "/" + mName.getText() + ".txt");
                if(!targetFile.exists()){
                    targetFile.createNewFile();
                }
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }

**output_text.java**

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="确认导出"
        android:onClick="OutputOk" />
</LinearLayout>

**3.更改文本字体参数（大小、颜色）**

![更改字体大小（1）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E5%AD%97%E4%BD%93%E5%A4%A7%E5%B0%8F%EF%BC%881%EF%BC%89.png)

**图11 更改字体大小（1）**

![更改字体大小（2）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E5%AD%97%E4%BD%93%E5%A4%A7%E5%B0%8F%EF%BC%882%EF%BC%89.png)

**图12 更改字体大小（2）**

![更改字体大小（3）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E5%AD%97%E4%BD%93%E5%A4%A7%E5%B0%8F%EF%BC%883%EF%BC%89.png)

**图13 更改字体大小（3）**

**关键实现代码：**

**editor_options_menu.xml**

    <item
    android:id="@+id/font_size"
    android:title="字体大小">
    <!--子菜单-->
    <menu>
        <!--定义一组单选菜单项-->
        <group>
            <!--定义多个菜单项-->
            <item
                android:id="@+id/font_10"
                android:title="小"
                />

            <item
                android:id="@+id/font_16"
                android:title="中" />
            <item
                android:id="@+id/font_20"
                android:title="大" />
        </group>
    </menu>

**NoteEditor.java**

        case R.id.menu_revert:
            cancelNote();
            break;
            case R.id.font_10:
                mText.setTextSize(20);
                Toast toast =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast.show();
                break;

            case R.id.font_16:
                mText.setTextSize(32);
                Toast toast2 =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast2.show();
                break;
            case R.id.font_20:
                mText.setTextSize(40);
                Toast toast3 =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast3.show();
                break;


![更改字体颜色（1）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E5%AD%97%E4%BD%93%E9%A2%9C%E8%89%B2%EF%BC%881%EF%BC%89.png)

**图14 更改字体颜色（1）**

![更改字体大小（2）.png](https://github.com/Huanglei666/NotePad-master_improved/blob/master/%E8%BF%90%E8%A1%8C%E6%88%AA%E5%9B%BE/%E6%9B%B4%E6%94%B9%E5%AD%97%E4%BD%93%E9%A2%9C%E8%89%B2%EF%BC%882%EF%BC%89.png)

**图15 更改字体颜色（2）**

**关键实现代码：**

**NoteEditor.java**

            case R.id.red_font:
                mText.setTextColor(Color.RED);
                Toast toast4 =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast4.show();
                mCursor.moveToFirst();
                color=mCursor.getInt(NotePad.Notes.RED_COLOR);
                ContentValues values = new ContentValues();
                values.put(NotePad.Notes.COLUMN_NAME_TEXT_COLOR, color);
                getContentResolver().update(mUri, values, null, null);
                mCursor.close();
                break;
            case R.id.black_font:
                mText.setTextColor(Color.BLACK);
                Toast toast5 =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast5.show();
                break;
            case R.id.blue_font:
                mText.setTextColor(Color.BLUE);
                Toast toast6 =Toast.makeText(NoteEditor.this,"心情便签：修改成功",Toast.LENGTH_SHORT);
                toast6.show();
                break;
