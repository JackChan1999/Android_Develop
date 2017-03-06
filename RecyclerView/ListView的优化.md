1. 复用convertview
2. 使用ViewHolder封装控件，减少findViewById()的调用
3. 如果只涉及到一个Item的某一个控件的更改,不应该去刷新整个ListView(notifyDataSetChanged())而是给这个控件使用setTag(Position)的方式设置不同的tag,然后使用ListView.findViewByTag,找到这个对应的控件进行更改(什么时候需要使用notifyDataSetChanged,当有一个条目添加或者删除,这个时候就必须刷新)
4. 分页加载
5. 分批加载

# ViewHolder类
```java
static class ViewHolder {
TextView text;
ImageView icon;
}
```

```java
 public View getView(int position, View convertView, ViewGroup parent) {
 ViewHolder holder;

 if (convertView == null) {
 convertView = mInflater.inflate(R.layout.list_item_icon_text, null);

 holder = new ViewHolder();
 holder.text = (TextView) convertView.findViewById(R.id.text);
 holder.icon = (ImageView) convertView.findViewById(R.id.icon);

 convertView.setTag(holder);
 } else {
 holder = (ViewHolder) convertView.getTag();
 }

 holder.text.setText(DATA[position]);
 holder.icon.setImageBitmap((position & 1) == 1 ? mIcon1 : mIcon2);

 return convertView;
 }
```

# 分页加载
![ListView](http://img.blog.csdn.net/20160131133620690)

```java
/**
	 * 分页查询数据库的记录
	 * @param pagenumber 第几页，页码 从第0页开始
	 * @param pagesize   每一个页面的大小
	 */
	public List<BlackNumberInfo> findPart(int pagenumber,int pagesize){
		// 得到可读的数据库
		SQLiteDatabase db = helper.getReadableDatabase();
		Cursor cursor = db.rawQuery("select number,mode from blackinfo limit ? offset ?", new String[]{String.valueOf(pagesize),
				String.valueOf(pagesize*pagenumber)
		});
		List<BlackNumberInfo> blackNumberInfos = new ArrayList<BlackNumberInfo>();
		while(cursor.moveToNext()){
			BlackNumberInfo info = new BlackNumberInfo();
			String number = cursor.getString(0);
			String mode = cursor.getString(1);
			info.setMode(mode);
			info.setNumber(number);
			blackNumberInfos.add(info);
		}
		cursor.close();
		db.close();
		SystemClock.sleep(30);
		return blackNumberInfos;
	}
```
# 分批加载

```java
/**
	 * 分批加载数据
	 * @param startIndex 从哪个位置开始加载数据
	 * @param maxCount   最多加载几条数据
	 */
	public List<BlackNumberInfo> findPart2(int startIndex,int maxCount){
		// 得到可读的数据库
		SQLiteDatabase db = helper.getReadableDatabase();
		Cursor cursor = db.rawQuery("select number,mode from blackinfo order by _id desc limit ? offset ?", new String[]{String.valueOf(maxCount),
				String.valueOf(startIndex)
		});
		List<BlackNumberInfo> blackNumberInfos = new ArrayList<BlackNumberInfo>();
		while(cursor.moveToNext()){
			BlackNumberInfo info = new BlackNumberInfo();
			String number = cursor.getString(0);
			String mode = cursor.getString(1);
			info.setMode(mode);
			info.setNumber(number);
			blackNumberInfos.add(info);
		}
		cursor.close();
		db.close();
		SystemClock.sleep(30);
		return blackNumberInfos;
	}
```
