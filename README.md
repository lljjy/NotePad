# NotePad
android期中作业

拓展功能解析 NoteList中显示条目增加时间显示 在NotePad原应用中，笔记列表只显示了笔记的标题。如图3、图6。要对它做时间扩展，可以把时间放在标题的下方。 首先，找到列表中item的布局：noteslist_item.xml。 在这个布局文件中，能看到一个TextView，这个便是笔记列表的标题item了。


要在标题下方加时间显示，就要在标题的TextView下再加一个时间的TextView。但是由于原应用列表item只需要一个标题，所以不需要用上别的布局，而我要多加一个时间TextView，就要把标题TextView和时间TextView放入垂直的线性布局。 由于要美化UI，所以将TextView原字体颜色改为黑色。新加的时间TextView字体大小小于标题TextView。


通过SimpleCursorAdapter装填：

String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
SimpleCursorAdapter adapter
    = new SimpleCursorAdapter(
            this, // The Context for the ListView
            R.layout.noteslist_item, // Points to the XML for a list item
            cursor,   // The cursor to get items from
            dataColumns,
            viewIDs
    );
// Sets the ListView's adapter to be the cursor adapter that was just created.
setListAdapter(adapter);

要将时间显示，首先要在PROJECTION中定义显示的时间，原应用有两种时间，我选择修改时间作为显示的时间。颜色部分先忽略，下文会涉及。

Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);

SimpleCursorAdapter不变（在改笔记列表颜色时需要做出改变，下文会涉及）。做完这些，标题下确实会多显示一行数据，但是，这并不是我们平常所见到的时间格式，而是时间戳，需要对这些数据进行转化，使人能看的懂。 我选则的方法时把时间戳改为以时间格式存入，改动地方分别为NotePadProvider中的insert方法和NoteEditor中的updateNote方法。前者为创建笔记时产生的时间，后者为修改笔记时产生的时间。下面代码中的dateTime即为转化后的时间格式，将其用ContentValues的put方法存入数据库。

Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);

运行效果（修改主题后，界面颜色与之前不一样，下文会提到）：

再添加一条笔记：

修改第一条笔记：

笔记查询（按标题查询） 要添加笔记查询功能，就要在应用中增加一个搜索的入口。找到菜单的xml文件，list_options_menu.xml，添加一个搜索的item，搜索图标用安卓自带的图标，设为总是显示：

<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
在NoteList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:

//添加搜素

case R.id.menu_search:
    Intent intent = new Intent();
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;
菜单：

在case语句中写跳转activity代码之前要先写好搜索的activity，新建一个名为NoteSearch的activity。由于搜索出来的也是笔记列表，所以可以模仿NoteList的activity继承ListActivity。在安卓中有个用于搜索控件：SearchView，可以把SearchView跟ListView相结合，动态地显示搜索结果。

在上面的ListView的id命名方式与往常“@+id/”的方式有些不同，之前用“@+id/”方式尝试过，但是运行会出错，可能是由于是继承ListAcitivity的缘故。 要动态地显示搜索结果，就要对SearchView文本变化设置监听，NoteSearch除了要继承ListView外还要实现SearchView.OnQueryTextListener接口。

即使我不需要使用onQueryTextSubmit方法，onQueryTextSubmit和onQueryTextChange两个方法也是实现接口必须写的方法。onListItemClick方法是点击NoteList的item跳转到对应笔记编辑界面的方法，NoteList中有这个方法，搜索出来的笔记跳转原理与NoteList中笔记一样，所以可以直接从NoteList中复制过来直接使用。 使用PROJECTION，Cursor，adapter方法与时间显示的原理一样，这里不多描述，但是可以注意到adapter是用MyCursorAdapter声明的，MyCursorAdapter是继承SimpleCursorAdapter，对其中个别方法进行重写，下文UI部分会提到。 动态搜索的实现最主要的部分在onQueryTextChange方法中，在使用这个方法，要先为SearchView注册监听：

SearchView searchview = (SearchView)findViewById(R.id.search_view);
searchview.setOnQueryTextListener(NoteSearch.this);  

而onQueryTextChange方法作用是，当SearchView中文本发生变化时，执行其中代码，搜索还有一个重要的部分就是要做到模糊匹配而不是严格匹配，可以使用数据库查询语句中的LIKE和%结合来实现，newText为输入搜索的内容：

String[] selectionArgs = { "%"+newText+"%" }; 1 最后要在AndroidManifest.xml注册NoteSearch：

<!--添加搜索activity-->
    <activity
        android:name="NoteSearch"
        android:label="@string/title_notes_search">
    </activity>


![image](https://github.com/lljjy/NotePad/blob/master/pictures/1.png)

![image](https://github.com/lljjy/NotePad/blob/master/pictures/2.png)

![image](https://github.com/lljjy/NotePad/blob/master/pictures/3.png)
