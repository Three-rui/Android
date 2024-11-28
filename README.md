# NotePad
基于NotePad应用的功能扩展

## 实验内容

## 一、实现功能

### 1.NoteList中显示条目增加时间戳显示

![时间戳](https://github.com/user-attachments/assets/173a24ba-0f8f-4a7c-9b95-832f8a8177e8)


### 2.添加笔记查询功能（根据标题查询）
![Screenshot_20241128_103940](https://github.com/user-attachments/assets/5f4bbbb9-3e73-4a06-81ad-80cd0c2120dd)
![Screenshot_20241128_104010](https://github.com/user-attachments/assets/56c454cd-93ca-44ee-a1e1-6a602d8b2da9)


### 3.笔记导出

![image](https://github.com/user-attachments/assets/dd3e691e-b493-4709-9c6c-bc1f120855ec)
![image](https://github.com/user-attachments/assets/3b3d9fc7-875d-4a80-b581-12a2cb69c2e1)
![image](https://github.com/user-attachments/assets/34ba94f7-f55d-4886-bafb-405cb2572968)
![image](https://github.com/user-attachments/assets/c8428adf-0dfa-4bf7-82a7-304904f901de)
### 4.UI美化


### 列表颜色变换
![image](https://github.com/user-attachments/assets/f568c715-3193-4c18-b6a3-b21b0f1f48b4)
![image](https://github.com/user-attachments/assets/4e8426ae-11ec-4f1c-986f-26f689e2a478)


### 字体颜色大小改变

![image](https://github.com/user-attachments/assets/5e0aed69-40c7-4c36-8d0c-411a32fe09f1)
![image](https://github.com/user-attachments/assets/b729557e-f250-4768-8f49-908c6eb8cdb8)


### 更换背景

![image](https://github.com/user-attachments/assets/8e5097ef-ff42-42df-bf0e-bda5596dfb39)

## 二、项目代码分析以及源码
### 1.NoteList中显示条目增加时间戳显示
首先从noteslist_item.xml增加一个textview 组件，因为有两个组件，所以要相应的为他们添加一个布局
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    
    
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:singleLine="true"
        />
    </LinearLayout>
然后在NotesList.java中PROJECTION 的内容，添加modif 字段，使其在后面的搜索中才能从SQLite 中读取修改时间的字段。

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
    };
    
    修改适配器内容，增加dataColumns 中装配到ListView 的内容，因此要同时增加一个文本框来存放时间。
    
        private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
          
	    NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
需要在Notepad.java中添加 
public static final String COLUMN_NAME_MODIFICATION_DATE = "modified";

最后在修改NoteEditor.java中updateNote 方法中的时间类型。

    private final void updateNote(String text, String title) {
    
        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        Long now = Long.valueOf(System.currentTimeMillis());
        SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");
        Date d = new Date(now);
        String format = sf.format(d);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

 ### 2.添加笔记查询功能（根据标题查询）
 首先新建一个查找笔记内容的布局文件note_search.xml。
 
     <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        />
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        />
    </LinearLayout>
    
在NotesList.java中的onOptionsItemSelected 方法中添加search 查询的处理。
        
	case R.id.menu_search:
        //Find function
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));
          Intent intent = new Intent(this, NoteSearch.class);
          this.startActivity(intent);
          return true;
	  
然后新建一个NoteSearch.java用于search 功能的功能实现。

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener {

    private ListView listView;
    private SQLiteDatabase sqLiteDatabase;

    // The columns needed by the cursor adapter
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID,             // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 2
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);

        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }

        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();

        // Configure the search view
        searchView.setSubmitButtonEnabled(true);
        searchView.setQueryHint("Search for notes...");
        searchView.setOnQueryTextListener(this);
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        // Handle search submission if needed
        Toast.makeText(this, "You searched for: " + query, Toast.LENGTH_SHORT).show();
        // Since we want to perform the search on text change, return false here
        return false;
    }

    @Override
    public boolean onQueryTextChange(String query) {
        // Avoid empty queries which might lead to performance issues
        if (query.isEmpty()) {
            listView.setAdapter(null);
            return true;
        }

        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " LIKE ? OR " +
                NotePad.Notes.COLUMN_NAME_NOTE + " LIKE ?";
        String[] selectionArgs = {"%" + query + "%", "%" + query + "%"};

        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                selection,
                selectionArgs,
                null, null,
                NotePad.Notes.DEFAULT_SORT_ORDER
        );

        if (cursor == null || cursor.getCount() == 0) {
            Toast.makeText(this, "No notes found for: " + query, Toast.LENGTH_SHORT).show();
            listView.setAdapter(null);
            return true;
        }

        // Configure the cursor adapter for the ListView
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        };
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };

        SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs,
                0
        );

        listView.setAdapter(adapter);
        return true;
    }
}
在list_options_menu.xml布局文件中添加搜索功能。

        <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        android:title="@string/menu_search"
        android:showAsAction="always" />
在清单文件AndroidManifest.xml里面注册NoteSearch 。

        <activity android:name=".NoteSearch" android:label="@string/search_note" />
### 3.笔记导出
新建布局output_text.xml，垂直线性布局放置EditText和Button,


<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
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
        android:scrollHorizontally="true"
        tools:ignore="Deprecated" />
    
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
	
</LinearLayout>

创建OutputText的Acitvity，用来选择颜色：

public class OutputText extends Activity {

    private static final int LOADER_ID = 1;
    
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_CREATE_DATE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };

    private String title;
    private String note;
    private String createDate;
    private String modificationDate;
    private EditText mName;
    private Uri mUri;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mName = findViewById(R.id.output_name);

        // Initialize Loader
        getLoaderManager().initLoader(LOADER_ID, null, new NoteLoaderCallbacks());
    }

    @Override
    protected void onResume() {
        super.onResume();
        // Move cursor to first position if loader is already initialized
        // This should be done inside onLoadFinished if needed
    }

    public void OutputOk(View v) {
        new WriteToFileTask().execute();
        finish();
    }

    private class NoteLoaderCallbacks implements LoaderManager.LoaderCallbacks<Cursor> {
        @Override
        public Loader<Cursor> onCreateLoader(int id, Bundle args) {
            return new CursorLoader(
                    OutputText.this,
                    mUri,
                    PROJECTION,
                    null,
                    null,
                    null
            );
        }

        @Override
        public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
            if (data != null && data.moveToFirst()) {
                title = data.getString(data.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
                note = data.getString(data.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
                createDate = data.getString(data.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
                modificationDate = data.getString(data.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
                mName.setText(title);
            }
        }

        @Override
        public void onLoaderReset(Loader<Cursor> loader) {
            // Handle cursor reset
        }
    }

    private class WriteToFileTask extends AsyncTask<Void, Void, Void> {
        @Override
        protected Void doInBackground(Void... voids) {
            if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
                File sdCardDir = Environment.getExternalStorageDirectory();
                File targetFile = new File(sdCardDir, mName.getText().toString() + ".txt");
                try (PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"))) {
                    ps.println(title);
                    ps.println(note);
                    ps.println("创建时间：" + createDate);
                    ps.println("最后一次修改时间：" + modificationDate);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            super.onPostExecute(aVoid);
            if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
                Toast.makeText(OutputText.this, "保存成功,保存位置：" + Environment.getExternalStorageDirectory().getAbsolutePath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(OutputText.this, "SD卡不可用", Toast.LENGTH_SHORT).show();
            }
        }
    }
}
在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：

    <item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
    
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加:

  '''java //导出笔记选项
   case R.id.menu_output:
   
        outputNote();
        break;
	
在NoteEditor中添加函数outputNote()：  
//跳转导出笔记的activity，将uri信息传到新的activity

   private final void outputNote() {
   
        Intent intent = new Intent(null,mUri);
	
        intent.setClass(NoteEditor.this,OutputText.class);
	
        NoteEditor.this.startActivity(intent);
    }
### 4.UI美化

### （1）列表颜色变换
在NotePad.java中添加
       
	public static final String COLUMN_NAME_BACK_COLOR = "color";
	创建数据库表地方添加颜色的字段。
	      
	public void onCreate(SQLiteDatabase db) {
           
	   db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + "   ("
                 + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //color
                   + ");");
       }
给NotesList 换个主题，把黑色换成白色，在AndroidManifest.xml中NotesList 的Activity 中添加。
        
	<activity android:name="NotesList" android:label="@string/title_notes_list"
                  android:theme="@android:style/Theme.Holo.Light">
在NotePadProvider.java中添加对其相应的处理，
static 中：
        
	sNotesProjectionMap.put(
               
		NotePad.Notes.COLUMN_NAME_BACK_COLOR,
               
		NotePad.Notes.COLUMN_NAME_BACK_COLOR);
		
  insert 中：
		       
	  // Create a new notepad. The background is white by default
        
	if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            
	    values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
在NotePad.java中定义：
      public class NotePad {
    public static final int DEFAULT_COLOR = 0; // white
    public static final int YELLOW_COLOR = 1; // yellow
    public static final int BLUE_COLOR = 2; // blue
    public static final int GREEN_COLOR = 3; // green
    public static final int RED_COLOR = 4; // red
}

自定义一个MyCursorAdapter.java继承SimpleCursorAdapter ，将颜色填充到ListView 。
    package com.example.android.notepad;
    
    import android.content.Context;
    import android.database.Cursor;
    import android.graphics.Color;
    import android.view.View;
    import android.widget.SimpleCursorAdapter;
    
    public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //Get the color data corresponding to the note list from the cursor read from the database, and set the note color
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /** * white 255 255 255
         * yellow 247 216 133
         * blue 165 202 237
         * green 161 214 174
         * red 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
    }
在NotesList.java中的PROJECTION 添加颜色项。
       
	private static final String[] PROJECTION = new String[] {
           
	    NotePad.Notes._ID, // 0
           
	    NotePad.Notes.COLUMN_NAME_TITLE, // 1
          
	    //Extended:display time, color
          
	    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
         
	    NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
将NotesList.java中用的SimpleCursorAdapter 改为使用MyCursorAdapter ：

       
	//Modify to a custom adapter that can be filled with color. The custom code is in MyCursorAdapter.java
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
在editor_options_menu.xml中添加一个更改背景的功能选项。
	    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_edit"
        android:showAsAction="always"/>
在NoteEditor.java中找到onOptionsItemSelected 方法，在菜单的switch 中添加：
       
	
         //Change background color option
        case R.id.menu_color:
            changeColor();
            break;
    // 在NoteEditor.java中添加函数changeColor
private final void changeColor() {
    Intent intent = new Intent(null, mUri);
     Intent intent = new Intent(NoteEditor.this, mUri);
    intent.setData(mUri); 
    NoteEditor.this.startActivity(intent);
}
     <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
    </LinearLayout>
    

在NotesList.java中为PROJECTION 中添加颜色项。

    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //Extended:display time, color
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
在editor_options_menu.xml中添加一个更改背景的功能选项。

    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/ic_menu_edit"
        android:showAsAction="always"/>
在NoteEditor.java中找到onOptionsItemSelected 方法，在菜单的switch 中添加：

        //Change background color option
        case R.id.menu_color:
            changeColor();
            break;
在NoteEditor.java中添加函数changeColor ：

    //Jump to the activity changing the color and transfer the URI information to the new activity
    private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
新建布局note_color.xml，垂直线性布局放置5个ImageButton ，对选择颜色界面进行布局。

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
    </LinearLayout>
新建资源文件color.xml,添加所需颜色。
        <?xml version="1.0" encoding="utf-8"?>
    <resources>
    
    <color name="colorWhite">#fff</color>
    <color name="colorYellow">#FFD885</color>
    <color name="colorBlue">#A5CAED</color>
    <color name="colorGreen">#A1D6AE</color>
    <color name="colorRed">#F49585</color>
    
    </resources>

创建NoteColor.java，用来选择颜色。
    public class NoteColor extends Activity {
         private Uri mUri;
          private int color;
         private Cursor mCursor;
        private NotePadDbHelper mDbHelper;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);

        mDbHelper = new NotePadDbHelper(this);
        mUri = getIntent().getData();

        // Manually manage the cursor (or use LoaderManager)
        SQLiteDatabase db = mDbHelper.getReadableDatabase();
        String[] projection = {
                NotePad.Notes._ID,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
        };

        try {
            mCursor = db.query(
                    NotePad.Notes.TABLE_NAME,
                    projection,
                    null, null, null, null, null
            );

            if (mCursor != null && mCursor.moveToFirst()) {
                color = mCursor.getInt(mCursor.getColumnIndexOrThrow(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
            }
        } catch (Exception e) {
            Toast.makeText(this, "Error retrieving note color: " + e.getMessage(), Toast.LENGTH_LONG).show();
            finish();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mCursor != null) {
            mCursor.close();
        }
        mDbHelper.close();
    }

    public void setColorAndFinish(int color) {
        this.color = color;
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);

        try {
            getContentResolver().update(mUri, values, null, null);
        } catch (Exception e) {
            Toast.makeText(this, "Error updating note color: " + e.getMessage(), Toast.LENGTH_LONG).show();
        } finally {
            finish();
        }
    }

    public void white(View view) {
        setColorAndFinish(NotePad.Notes.DEFAULT_COLOR);
    }

    public void yellow(View view) {
        setColorAndFinish(NotePad.Notes.YELLOW_COLOR);
    }

    public void blue(View view) {
        setColorAndFinish(NotePad.Notes.BLUE_COLOR);
    }

    public void green(View view) {
        setColorAndFinish(NotePad.Notes.GREEN_COLOR);
    }

    public void red(View view) {
        setColorAndFinish(NotePad.Notes.RED_COLOR);
    }
}
在清单文件AndroidManifest.xml里面注册NoteColor
        <!--Change background color-->
        <activity android:name="NoteColor"
            android:theme="@android:style/Theme.Holo.Light.Dialog"
            android:label="ChangeColor"
            android:windowSoftInputMode="stateVisible"/>

### （2）改变字体大小和颜色
在menu文件夹中editoe_option_menu.xml里添加两段代码，分别用于字体大小和颜色：
<item
        android:title="字体颜色">
        <menu>
            <group>
                <item android:id="@+id/red_font"
                    android:title="红色"/>
                <item android:id="@+id/black_font"
                    android:title="黑色"/>
                <item android:id="@+id/green_font"
                    android:title="绿色"/>
                <item android:id="@+id/blue_font"
                    android:title="蓝色"/>
                <item android:id="@+id/yellow_font"
                    android:title="黄色"/>
            </group>
        </menu>
    </item>   
    <item android:title="字体大小">
        <menu>
            <group>
                <item android:id="@+id/font_10"
                    android:title="小"/>
                <item android:id="@+id/font_16"
                    android:title="中"/>
                <item android:id="@+id/font_20"
                    android:title="大"/>
            </group>
        </menu>
在java文件夹中NoteEditor里的onOptionsItemSelected(MenuItem item)中添加代码，达到调节字体大小和颜色的作用，主要代码如下所示：

     case R.id.font_10:
     mText.setTextSize(20);
     break;
     case R.id.font_16:
     mText.setTextSize(32);
     break;
     case R.id.font_20:
     mText.setTextSize(40);
     break;     
     case R.id.red_font:
     mText.setTextColor(Color.RED);
     break;
     case R.id.black_font:
     mText.setTextColor(Color.BLACK);
     break;
     case R.id.green_font:
     mText.setTextColor(Color.GREEN);
     break;
     case R.id.blue_font:
     mText.setTextColor(Color.BLUE);
     break;
     case R.id.yellow_font:
     mText.setTextColor(Color.YELLOW);
     break;

     
### （3）主题背景更换


      <item
        android:title="更换背景">
        <menu>
            <group>
                <item android:id="@+id/one"
                    android:title="小狗"/>
                <item android:id="@+id/two"
                    android:title="卡通"/>
                <item android:id="@+id/three"
                    android:title="恢复"/>
            </group>
        </menu>
    </item>
在java文件夹中NoteEditor里的onOptionsItemSelected(MenuItem item)添加两行代码，达到引用上面代码的作用，主要代码如下所示：
     setContentView(R.layout.note_editor);
     tv = (TextView) findViewById(R.id.note);
    
继续在该onOptionsItemSelected(MenuItem item)中添加代码，达到更换背景的作用，主要代码如下所示：

    case R.id.one:
    tv.setBackgroundResource(R.drawable.dog);
    break;
    case R.id.two:
    tv.setBackgroundResource(R.drawable.cute);
    break;
    case R.id.three:
    tv.setBackgroundResource(R.drawable.bak_green);
    break;
