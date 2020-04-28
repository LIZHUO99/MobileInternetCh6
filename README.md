# Mobile Internet (android part) Chapter6 Assignment

BUPT Mobile Internet Course android part chapter6

- Android Studio：3.6.1
- Gradle：5.6.4
- Gradle Plugin：3.6.1

学号：2017211428

## 1. 在DebugActivity里增加了写入并读出Internal Storage和 External Private Storage的部分

Internal Storage 部分用了AsyncTask实现：

```java
    final Button fileWriteBtnIn = findViewById(R.id.btn_write_files_internal);
    final TextView fileTextIn = findViewById(R.id.text_files_internal);

    @SuppressLint("StaticFieldLeak")
    class MyTask extends AsyncTask<String, Integer, List<String>> {
        @Override
        protected List<String> doInBackground(String... strings) {
            File dir = getFilesDir();
            File file = new File(dir, "test");
            FileUtils.writeContentToFile(file, "#internal internal \ntest content.");

            return FileUtils.readContentFromFile(file);
        }

        @Override
        protected void onPostExecute(List<String> strings) {
            fileTextIn.setText("");
            for (String string : strings){
                fileTextIn.append(string + "\n");
            }
        }
    }

    fileWriteBtnIn.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            new MyTask().execute();
        }
    });
}
```

External Private Storage 部分和已经写好的Public部分基本相同。

```java
final Button fileWriteBtnPri = findViewById(R.id.btn_write_files_private);
final TextView fileTextPri = findViewById(R.id.text_files_private);
fileWriteBtnPri.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                File dir = getExternalFilesDir(null);
                File file = new File(dir, "test");
                FileUtils.writeContentToFile(file, "#external private \ntest content.");
                final List<String> contents = FileUtils.readContentFromFile(file);
                DebugActivity.this.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        fileTextPri.setText("");
                        for (String content: contents){
                            fileTextPri.append(content + "\n");
                        }
                    }
                });
            }
        }).start();
    }
});
```



实现如下图：

<img src="https://github.com/LIZHUO99/MobileInternetCh6/blob/master/snapshots/Screenshot1.png" width="300" />



## 2. 实现简单的todoList（pro）

#### 从数据库中查询数据，并转换成 JavaBeans

```java
private List<Note> loadNotesFromDatabase() {
    SQLiteDatabase database = dbHelper.getReadableDatabase();

    String[] projection = {
            TodoEntry._ID,
            TodoEntry.COLUMN_NAME_CONTENT,
            TodoEntry.COLUMN_NAME_PRIORITY,
            TodoEntry.COLUMN_NAME_STATE,
            TodoEntry.COLUMN_NAME_DATE
    };

    String sortOrder = TodoEntry.COLUMN_NAME_PRIORITY;

    Cursor cursor = database.query(
            TodoEntry.TABLE_NAME,
            projection,
            null,
            null,
            null,
            null,
            sortOrder
    );

    List<Note> result = new ArrayList<>();

    while(cursor.moveToNext()){
        Note note = new Note(cursor.getLong(cursor.getColumnIndexOrThrow(TodoEntry._ID)));

        note.setContent(cursor.getString(cursor.getColumnIndexOrThrow(TodoEntry.COLUMN_NAME_CONTENT)));

        try {
            note.setDate(SIMPLE_DATE_FORMAT.parse(cursor.getString(cursor.getColumnIndexOrThrow(TodoEntry.COLUMN_NAME_DATE))));
        } catch (ParseException e) {
            Log.i(TAG, "ERROR: Parse date format fail.");
            //
        }

        note.setPriority(cursor.getInt(cursor.getColumnIndexOrThrow(TodoEntry.COLUMN_NAME_PRIORITY)));
        note.setState(State.from(cursor.getInt(cursor.getColumnIndexOrThrow(TodoEntry.COLUMN_NAME_STATE))));

        result.add(note);
    }

    return result;
}
```

#### 删除数据

```java
private void deleteNote(Note note) {
    SQLiteDatabase database = dbHelper.getReadableDatabase();

    String selection = TodoEntry._ID + " LIKE ?";

    String[] selectionArgs = {String.valueOf(note.id)};

    int deletedRows = database.delete(TodoEntry.TABLE_NAME, selection, selectionArgs);
    Log.i(TAG, "perform delete data, result:" + deletedRows);
    
    //删完刷新一下
    notesAdapter.refresh(loadNotesFromDatabase());
}
```

#### 更新数据

```java
private void updateNode(Note note) {
    SQLiteDatabase database = dbHelper.getReadableDatabase();

    ContentValues values = new ContentValues();
    values.put(TodoEntry.COLUMN_NAME_STATE, note.getState().intValue);

    String selection = TodoEntry._ID + " LIKE ?";
    String[] selectionArgs = {String.valueOf(note.id)};

    int count = database.update(TodoEntry.TABLE_NAME, values, selection, selectionArgs);
    Log.i(TAG, "perform update data, result:" + count);
    
    notesAdapter.refresh(loadNotesFromDatabase());
}
```

#### 在NoteActivity界面添加了三个RadioButton，默认选Regular

<img src="https://github.com/LIZHUO99/MobileInternetCh6/blob/master/snapshots/Screenshot4.jpg" width="300" />



## 整体效果如下：

<img src="https://github.com/LIZHUO99/MobileInternetCh6/blob/master/snapshots/Screenshot2.jpg" width="300" />

<img src="https://github.com/LIZHUO99/MobileInternetCh6/blob/master/snapshots/Screenshot3.jpg" width="300" />
