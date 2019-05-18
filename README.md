# NotePad
拓展功能





1）NoteList中显示条目增加时间显示

2）笔记查询（按标题查询）

3）改变文本字体大小

4）改变文本字体颜色

5）导出

6）插入了自己的背景图片



1）NoteList中显示条目增加时间显示

运行效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019051819231890.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

noteslist_item.xml

添加显示时间的TextView,把标题TextView和时间TextView放入垂直的线性布局

```
<?xml version="1.0" encoding="utf-8"?>
<!-- Copyright (C) 2010 The Android Open Source Project

     Licensed under the Apache License, Version 2.0 (the "License");
     you may not use this file except in compliance with the License.
     You may obtain a copy of the License at
  
          http://www.apache.org/licenses/LICENSE-2.0
  
     Unless required by applicable law or agreed to in writing, software
     distributed under the License is distributed on an "AS IS" BASIS,
     WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     See the License for the specific language governing permissions and
     limitations under the License.
-->

<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@drawable/p">


<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
/>
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        />
</LinearLayout>
```


NotesList.java

在PROJECTION添加NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
```
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
};
```


dataColumns，viewIDs中补充时间部分

```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

// The view IDs that will display the cursor columns, initialized to the TextView in
// noteslist_item.xml
int[] viewIDs = { android.R.id.text1 , R.id.text1_time};
```

在NotePadProvider中的insert方法和NoteEditor中的updateNote方法中添加

```
Long now = Long.valueOf(System.currentTimeMillis());

Date date = new Date(now);

SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

String dateTime = format.format(date);


```

2）笔记查询（按标题查询）

运行效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019051819320730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

在list_options_menu.xml中添加一个搜索的item

```
<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
```

在NoteList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:

```
case R.id.menu_search:
    Intent intent = new Intent();
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;

```

新建一个note_search_list.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/p">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView

        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>



```

新建一个NoteSearch.java

```
package com.example.android.notepad;

import android.app.ListActivity;

import android.content.ContentUris;

import android.content.Intent;

import android.database.Cursor;

import android.net.Uri;

import android.os.Bundle;

import android.view.View;

import android.widget.ListView;

import android.widget.SearchView;

import android.widget.SimpleCursorAdapter;



/**

 * Created by panhongwen on 2018/5/24.

 */

public class NoteSearch extends ListActivity implements SearchView.OnQueryTextListener {

    private static final String[] PROJECTION = new String[] {

            NotePad.Notes._ID, // 0

            NotePad.Notes.COLUMN_NAME_TITLE, // 1

            //扩展 显示时间 颜色

            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 2

            //NotePad.Notes.COLUMN_NAME_BACK_COLOR

    };

    @Override

    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

        setContentView(R.layout.note_search_list);

        Intent intent = getIntent();

        if (intent.getData() == null) {

            intent.setData(NotePad.Notes.CONTENT_URI);

        }

        SearchView searchview = (SearchView)findViewById(R.id.search_view);

        //为查询文本框注册监听器

        searchview.setOnQueryTextListener(NoteSearch.this);

    }

    @Override

    public boolean onQueryTextSubmit(String query) {

        return false;

    }

    @Override

    public boolean onQueryTextChange(String newText) {

        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";

        String[] selectionArgs = { "%"+newText+"%" };

        Cursor cursor = managedQuery(

                getIntent().getData(),            // Use the default content URI for the provider.

                PROJECTION,                       // Return the note ID and title for each note. and modifcation date

                selection,                        // 条件左边

                selectionArgs,                    // 条件右边

                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.

        );

        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };

        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

        SimpleCursorAdapter adapter = new SimpleCursorAdapter(

                this,

                R.layout.noteslist_item,

                cursor,

                dataColumns,

                viewIDs

        );

        setListAdapter(adapter);

        return true;

    }

    @Override

    protected void onListItemClick(ListView l, View v, int position, long id) {

        // Constructs a new URI from the incoming URI and the row ID

        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

        // Gets the action from the incoming Intent

        String action = getIntent().getAction();

        // Handles requests for note data

        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            // Sets the result to return to the component that called this Activity. The

            // result contains the new URI

            setResult(RESULT_OK, new Intent().setData(uri));

        } else {

            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The

            // Intent's data is the note ID URI. The effect is to call NoteEdit.

            startActivity(new Intent(Intent.ACTION_EDIT, uri));

        }

    }

}




```

在AndroidManifest.xml注册NoteSearch

```
<activity
    android:name="NoteSearch"
    android:label="@string/title_notes_search"
    android:theme="@android:style/Theme.Holo.Light">
</activity>
```

3）改变文本字体大小

运行效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518193837486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518194720986.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518194736736.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

在editor_options_menu.xml中添加

```
<item
    android:id="@+id/font_size"
    android:title="@string/font_size">
    <!--子菜单-->
    <menu>
        <!--定义一组单选菜单项-->
        <group>
            <!--定义多个菜单项-->
            <item
                android:id="@+id/font_10"
                android:title="@string/font10"
                />

            <item
                android:id="@+id/font_16"
                android:title="@string/font16" />
            <item
                android:id="@+id/font_20"
                android:title="@string/font20" />
        </group>
    </menu>
</item>


```

在NoteEditor的onOptionsItemSelected中添加

```
case R.id.font_10:

    mText.setTextSize(20);
    Toast toast =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast.show();
    break;

case R.id.font_16:
    mText.setTextSize(32);
    Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast2.show();
    break;
case R.id.font_20:
    mText.setTextSize(40);
    Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast3.show();
    break
```

在NotePadProvider的数据库中添加

```
db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("

        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"

        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"

        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"

        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"

        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"

        + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER,"

        + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + "INTEGER"


        + ");");
```
4）改变文本字体颜色

运行效果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518195340958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518195358800.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518195410177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

在editor_options_menu.xml中添加

```
<item
    android:title="@string/font_color"
    android:id="@+id/font_color"
    >
    <menu>
        <!--定义一组普通菜单项-->
        <group>
            <!--定义两个菜单项-->
            <item
                android:id="@+id/red_font"
                android:title="@string/red_title" />
            <item
                android:title="@string/black_title"
                android:id="@+id/black_font"/>
        </group>
    </menu>
</item>
```

在NoteEditor的onOptionsItemSelected中添加

```
case R.id.red_font:
    mText.setTextColor(Color.RED);
    Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast4.show();


    break;
case R.id.black_font:
    mText.setTextColor(Color.BLACK);
    Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast5.show();
    break;


```
在NotePadProvider的数据库中添加
```
db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("

        + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"

        + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"

        + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"

        + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"

        + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"

        + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER,"

        + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + "INTEGER"


        + ");");
```

5）导出

运行效果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518202217813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518202322647.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)
在editor_options_menu.xml中添加

```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```
在NoteEditor中的switch中添加：
```
case R.id.menu_output:
        outputNote();
        break;
```
在NoteEditor中添加函数outputNote()：
```
private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }

```
新建output_text.xml
```
<?xml version="1.0" encoding="utf-8"?>
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
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>

```
新建OutputText.java
```
package com.example.android.notepad;

import android.app.Activity;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;

public class OutputText extends Activity {
    
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };

    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
   
    private Cursor mCursor;

    private EditText mName;
 
    private Uri mUri;
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
           
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
          
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
         
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
               
                File sdCardDir = Environment.getExternalStorageDirectory();
               
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
             
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
}

```
在AndroidManifest.xml中添加
```
<activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
```
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
6）插入了自己的背景图片

运行效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190518203125703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdhZXRoZWxiYWxk,size_16,color_FFFFFF,t_70)
在noteslist_item.xml和note_search_list.xml中添加
```
android:background="@drawable/p"
```



