# ViewFilpper:滑动翻页

--- 

### 简单使用
1. FlipperGallery:画廊

	```Java
	public class FlipperGallery {
	    public final static int TOUCH_MOVING = 20;
	
		private ViewFlipper mViewFlipper;
	    private float fromYPosition;
	    private Context mContext;
	    private int count;
	    private GalleryAdapter adapter;
	    private List<Item> items;
	
	    public FlipperGallery(Context context, ViewFlipper viewFlipper, List<Item> items) {
	        this.items = items;
	        adapter = new GalleryAdapter(context, items);
	        this.mViewFlipper = viewFlipper;
	        this.mContext = context;
	        count = 0;
	        viewFlipper.addView(adapter.getView(count, null, null));
	        initOnTouch();
	    }
	
	    /**
	     * 设置触摸事件
	     */
	    private void initOnTouch() {
	        this.mViewFlipper.setOnTouchListener(new OnTouchListener() {
	            @Override
	            public boolean onTouch(View v, MotionEvent event) {
	                switch (event.getAction()) {
	                    case MotionEvent.ACTION_DOWN:
	                        fromXPosition = event.getX();
	                        break;
	                    case MotionEvent.ACTION_UP:
	                        float toPosition = event.getX();
	                        if (fromXPosition > toPosition + TOUCH_MOVING) {
	                            next();
	                            return true;
	                        } else if (fromXPosition < toPosition - TOUCH_MOVING) {
	                            previous();
	                            return true;
	                        }
	                    default:
	                        break;
	                }
	                return true;
	            }
	        });
	    }
	
	    /**
	     * 前一页
	     */
	    protected void previous() {
	        if (count > 0) {
	            count--;
	            this.mViewFlipper.setInAnimation(AnimationUtils.loadAnimation(mContext,
	                    R.anim.in_from_left));
	            this.mViewFlipper.setOutAnimation(AnimationUtils.loadAnimation(mContext,
	                    R.anim.out_to_right));
	            mViewFlipper.addView(adapter.getView(count, null, null), 0);
	            mViewFlipper.showPrevious();
	            //只缓存一页
	            mViewFlipper.removeViewAt(mViewFlipper.getChildCount() - 1);
	        }
	
	    }
	
	    /**
	     * 下一页
	     */
	    protected void next() {
	        if (count < items.size() - 1) {
	            count++;
	            this.mViewFlipper.setInAnimation(AnimationUtils.loadAnimation(mContext,
	                    R.anim.in_from_right));
	            this.mViewFlipper.setOutAnimation(AnimationUtils.loadAnimation(mContext,
	                    R.anim.out_to_left));
	            mViewFlipper.addView(adapter.getView(count, null, null));
	            mViewFlipper.showNext();
	            //只缓存一页
	            mViewFlipper.removeViewAt(0);
	        }
	    }
	}
	
	```
2. GalleryAdapter:画廊适配器

	```Java 
	public class GalleryAdapter extends BaseAdapter {
	    private List<Item> items;
	    private LayoutInflater layoutInflater = null;
	
	    public GalleryAdapter(Context context, List<Item> items) {
	        this.items = items;
	        layoutInflater = (LayoutInflater) context
	                .getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	    }
	
	    @Override
	    public int getCount() {
	        return items.size();
	    }
	
	    @Override
	    public Object getItem(int position) {
	        return items.get(position);
	    }
	
	    @Override
	    public long getItemId(int position) {
	        return items.get(position).getId();
	    }
	
	    @Override
	    public View getView(int position, View convertView, ViewGroup parent) {
	        View view = layoutInflater.inflate(R.layout.gallery_item, null);
	        TextView txt = ((TextView) view.findViewById(R.id.textView));
	        if (items.get(position).isClicked()) {
	            txt.setText(items.get(position).getValue() + " clicked");
	            txt.setClickable(false);
	            txt.setTag(items.get(position).getId());
	        } else {
	            txt.setText(items.get(position).getValue());
	            txt.setTag(items.get(position).getId());
	            txt.setOnClickListener(new View.OnClickListener() {
	                @Override
	                public void onClick(View v) {
	                    Item item = items.get(items.indexOf(new Item((Integer) v.getTag())));
	                    item.setClicked(true);
	                    ((TextView) v).setText(item.getValue() + " clicked");
	                    v.setClickable(false);
	                }
	            });
	
	        }
	
	        return view;
	    }
	}
	```
3. Item: JavaBean对象

	```Java
	public class Item {
		private boolean clicked;
		private String value;
		private int id;
	
		public Item(boolean clicked, String value, int id) {
			this.setClicked(clicked);
			this.setId(id);
			this.setValue(value);
		}
	
		public Item(int id) {
			this.setId(id);
		}
	
		public boolean isClicked() {
			return clicked;
		}
	
		public void setClicked(boolean clicked) {
			this.clicked = clicked;
		}
	
		public String getValue() {
			return value;
		}
	
		public void setValue(String value) {
			this.value = value;
		}
	
		public int getId() {
			return id;
		}
	
		public void setId(int id) {
			this.id = id;
		}
	
		@Override
		public boolean equals(Object o) {
			if (o instanceof Item) {
				return (((Item) o).getId() == this.getId());
			}
			return super.equals(o);
		}
	}
	```
4. 主界面

	```Java
	public class MainActivity extends Activity {
		FlipperGallery gallery;
	
		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			gallery = new FlipperGallery(this,
					(ViewFlipper) findViewById(R.id.viewflipper), generateData());
		}
	
		private List<Item> generateData() {
			List<Item> result = new ArrayList<Item>();
			for (int i = 0; i < 5; i++) {
				result.add(new Item(false, "Text" + String.valueOf(i), i));
			}
			return result;
		}
	
	}
	```

5.一些资源文件

	activity_main.xml：主界面
	```
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:paddingBottom="@dimen/activity_vertical_margin"
	    android:paddingLeft="@dimen/activity_horizontal_margin"
	    android:paddingRight="@dimen/activity_horizontal_margin"
	    android:paddingTop="@dimen/activity_vertical_margin"
	    tools:context="fr.vincenti.jm.fviewflipper_gallery.MainActivity" >
	
	    <ViewFlipper
	        android:id="@+id/viewflipper"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:background="@android:color/darker_gray" />
	
	</RelativeLayout>
	```
	gallery_item.xml: 画廊子项目
	```
	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:gravity="center" >
	
	    <TextView
	        android:id="@+id/textView"
	        android:layout_width="wrap_content"
	        android:layout_height="50dp"
	        android:background="@android:color/black"
	        android:padding="5dp"
	        android:text="Large Text"
	        android:textAppearance="?android:attr/textAppearanceLarge"
	        android:textColor="@android:color/white" />
	
	</RelativeLayout>
	```
	
	in_from_left.xml: 从左边进入动画
	```
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	    <translate
	    	android:fromXDelta="-100%p"
	        android:toXDelta="0"
	        android:duration="400"/>
	    <alpha 
	    	android:fromAlpha="1.0"
	        android:toAlpha="1.0"
	        android:duration="400" />
	</set>
	```
	
	in_from_right.xml:从右边进入动画
	```
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	<translate
	        android:fromXDelta="100%p"
	        android:toXDelta="0"
	        android:duration="400"/>
	<alpha
	        android:fromAlpha="1.0"
	        android:toAlpha="1.0"
	        android:duration="400" />
	</set>
	```
	
	out_to_left.xml:从左边退出动画
	```
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	    <translate
	        android:fromXDelta="0"
	        android:toXDelta="-100%p"
	        android:duration="400"/>
	    <alpha    
	        android:fromAlpha="1.0"
	        android:toAlpha="1.0"
	        android:duration="400" />
	</set>
	```
	
	out_to_right.xml:从右边退出动画
	```
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	    <translate
	    	android:fromXDelta="0"
	        android:toXDelta="100%p"
	        android:duration="400"/>
	    <alpha 
	        android:fromAlpha="1.0"
	        android:toAlpha="1.0"
	        android:duration="400" />
	```
