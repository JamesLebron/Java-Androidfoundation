1.自定义recyclerview 的 布局管理器,因为recyclerview 设置wrap_content 也会填满屏幕,可以自定义的布局管理器,让高度根据item 的高度来

 //自定义的 网格布局  recyrview 的高宽度随着item 来变化
    private class MyLayoutManager extends GridLayoutManager {
        public MyLayoutManager(Context context, int spanCount) {
            super(context, spanCount);
        }


   
    }

2.替换一个fragment 为另外一个fragment 
 public void updateToViewPhotoFragment( Fragment fragment) {
        mFragmentManager.beginTransaction().remove((mFragmentManager.findFragmentByTag("android:switcher:" + R.id.mainLayout + ":" + 1))).commit();
        mFragments[1] = fragment;
        mMainFragmentPagerAdapter.notifyDataSetChanged();
    }

3.再按一次退出

private long mExitTime = 0;
 @Override
    public boolean onKeyDown(int keyCode, KeyEvent event)//主要是对这个函数的复写
    {
        if ((keyCode == KeyEvent.KEYCODE_BACK) && (event.getAction() == KeyEvent.ACTION_DOWN)) {
            if (System.currentTimeMillis() - mExitTime > 2000) // 2s内再次选择back键有效
            {
                Toast.makeText(this, "请在按一次返回退出", Toast.LENGTH_LONG).show();
                mExitTime = System.currentTimeMillis();
            } else {
                finish();
                TApplication.app.exit();
            }
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
4. fresco 设置图片加载失败,点击重新加载
  controller = Fresco.newDraweeControllerBuilder()
                .setUri(Uri.parse(mList.get(position).getUserPhoto()))
                .setTapToRetryEnabled(true)
                .build();

5.所有异常的处理类

/**
 * 所有异常处理类
 * 张钦
 * 2015年8月24日
 */
public class ExceptionUtil {
    private static boolean state = false;

    public static void handleException(Exception e) {
        String str = "";
        StringWriter stringWriter = new StringWriter();
        PrintWriter printWriter = new PrintWriter(stringWriter);
        e.printStackTrace(printWriter);
        str = stringWriter.toString();
        if (state) {

            //联网发送
        } else {
            MyLog.i(str);
        }
    }
}
6.volley post 参数设置(http://blog.csdn.net/hpb21/article/details/12163371)
这里以post请求说明，get请求相似设置请求头及超时。
1.自定义request，继承com.android.volley.Request
2.构造方法实现（basecallback,为自定义的监听，实现Response.Listener,ErrorListener接口）--post请求
public BaseRequest(String url,String params, BaseCallback<T> callback)  
   {  
super(Method.POST, url, callback);  
this.callback = callback;  
this.params = params;  
Log.e(TAG, "request:" + params);  
setShouldCache(false);  
   }  
3.请求头设置：重写getHeaders方法
@Override  
   public Map<String, String> getHeaders() throws AuthFailureError  
   {  
Map<String, String> headers = new HashMap<String, String>();  
headers.put("Charset", "UTF-8");  
headers.put("Content-Type", "application/x-javascript");  
headers.put("Accept-Encoding", "gzip,deflate");  
return headers;  
   }  
7.UltimateRecyclerView 的一个bug
在初始化之后,设置监听器,设置init(),在init 里面判断有没有网络,没有则什么也不做.
如果有的话,则donetWork ,在设置监听器里面我设置了UltimateRecyclerView的下拉刷新,
这时候无论有没有网络都会去doNetWork 造成一系列的bug
如果用户打开了 UltimateRecyclerView,这时候断开网络再去刷新,一样会有bug 
解决办法如下.为了更保险,我们在刷新的时候,再去判断网络.
  private void setLeaveAdapter() {
        mAdapter = new MyLeaveAdapter(mList);
        LinearLayoutManager mLinearLayoutManager = new LinearLayoutManager(this);
        //给recyclerview 设置布局管理器
        mRecylerview.setLayoutManager(mLinearLayoutManager);
        //设置adapter
        if (mAdapter != null) {
            mRecylerview.setAdapter(mAdapter);
        }
        //设置监听器
        mRecylerview.setDefaultOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                doNetWork();
            }
        });
    }

8.命名规则
小驼峰  myStudent 变量
大驼峰   MyStudent 类名
另外两种 
TextView tv_name
TextView mTvName

初始化:
initView();  初始化界面
initData();  初始化数据

布局代码命名规则
tv_name

方法命名: 小写开头
private  void getTips(TextView name){

}
9.UltimateRecyclerView 的一个bug
在页面加载之后,如果需要页面不显示recyrviiew 里面的数据,但是又要显示recyclerView 的话.这时候如果不设置布局管理器,如果手动去滑动recyleview ,这时候会报空指针. 什么setHore........

解决方案,初始化的时候,可以先设置recylerView 的布局管理器,得到数据之后再去设置 adapter 即可解决

10.Java内存管理：深入Java内存区域(http://www.cnblogs.com/gw811/archive/2012/10/18/2730117.html)

11.Android studio 查看类的继承结构 选中某个来 按F4, 如果需要查看类的内部结构 按CTRL+O

12.Android开发之如何保证Service不被杀掉（broadcast+system/app）(http://blog.csdn.net/mad1989/article/details/22492519)(http://www.android100.org/html/201406/05/19111.html)

13UitimateRecyclerView 的下拉加载必须要加上一句话
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        if (position < getItemCount() && (customHeaderView != null ? position <= mList.size() : position < mList.size()) && (customHeaderView != null ? position > 0 : true)) {
            mController = Fresco.newDraweeControllerBuilder()
                    .setUri(Uri.parse(mList.get(position).getImageUrl()))
                    .setTapToRetryEnabled(true)
                    .build();
            holder.ivImage.setController(mController);
            holder.ivImage.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //// TODO: 2015/9/1 点击图片跳转到购买页面
                }
            });
        }
    }

14.Android Stduio 发生 Process 'command 'somePath:java.exe'' finished with non-zero exit value 2 异常的解决办法
http://www.cnblogs.com/atwind/p/4560746.html

15.桌面出现2个相同APP 图标的时候,而manifest 里面只有一个引导页的时候,可以这样检查,看看是不是有个启动页面.

 

16.判断activity 是否被finished


17. Gradle finished with non-zero exit value 1 ic_launcher.png: Original is here. The version qualifie 的错误
解决办法:http://blog.csdn.net/zsc357448181/article/details/46650953(经过验证无果,需要去掉存在相同资源的依赖)


18.在初始化内置的viewholder 的时候,不要gettag设置数据,此时mlist是为空的



19.设置textview 单行,超过屏幕,显示省略号
    android:ellipsize="end"
            android:singleLine="true"

20.在fragment中放置 PullToRefreshListView 但是无论如何页面都不显示,也不展示.有数据,但是呈现没有数据的空界面.
导致以为数据或代码的错误................................
需要在fragment的oncreate 方法中调用这个带有三个参数的构造函数
 View view = inflater.inflate(R.layout.dzmy_second_fragment, container, false);
不能应用此构造函数
View view = inflater.inflate(R.layout.dzmy_second_fragment, null);

同理其次在PullToRefreshListView 的item 中 初始化布局的时候,必须调用
   convertView = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_list_dzmy_wantsale, parent, false);
这个构造函数,不然容易出现页面布局错乱,大小不一的样子.


20.字体设置不同的颜色
 mTvState.setText(Html.fromHtml("<font color='gray'>已有报价</font><font color='#45b035'>(" + mBean.getOffer_number() + ")</font>"));

21.NetworkDispatcher.run: Unhandled exception java.lang.IllegalStateException: Adapter is detached. java.lang.IllegalStateException: Adapter is detached.   错误解决方案


22.volley 注册时需要使用cookie ?

                    DefaultHttpClient httpclient = new DefaultHttpClient();
                    // PreferencesCookieStore(this);
                    CookieStore cookieStore = new BasicCookieStore(); // new
                    httpclient.setCookieStore(cookieStore);
                    HttpStack httpStack = new HttpClientStack(httpclient);

        mQueue= Volley.newRequestQueue(getApplicationContext(),httpStack);

23. 使用 小写变大写的快捷键
ctrl +shift +x 

24.Android内存管理详细介绍(解析native 内存和你 dalvik 内存区域  和fresco 管理图片的方式有关系)  详细解析了bitmap 是存放在哪里 
http://www.tuicool.com/articles/Jz26zm

15.DDMS Dump 出的文件要经过转换才能被 MAT识别，Android SDK提供了这个工具 hprof-conv (位于 sdk/tools下)
    hprof-conv 1.hprof 2.hprof  
如果还是提示报错,就从platform-tools 复制 hprof.exe 到tools 目录才可以

16.性能优化系列总篇
本文为性能优化系列的总纲，主要介绍性能调优专题计划、何为性能问题、性能调优方式及前面介绍的数据库优化、布局优化、Java(Android)代码优化、网络优化具体对应的调优方式。
http://www.trinea.cn/android/performance/

17.以后 toast 的时候,不要传入  activity.this    会造成内存泄露    当打开activity 打开之后, 如果这时候 toast ,传入的是activity.this.我们马上退出这个activity.按理说,activity 应该销毁..但是toast 还没toast 出来. toast 的方法还持有context 对象 ,此时activity 不会销毁,以后toast.尽量使用getApplicationContext()   或者使用Tapplication.app 

18.Android  推荐使用 静态内部类的原因
http://www.2cto.com/kf/201502/378500.html

static class MyHandler extends Handler {

    WeakReference<BaseActivity> weakReference;

    public MyHandler(BaseActivity baseActivity) {
weakReference = new WeakReference<BaseActivity>(baseActivity);
}

@Override
public void handleMessage(Message msg) {
        BaseActivity bac = weakReference.get();
        if (bac == null)
return;

        if (msg.what == SHOW_MESSAGE || !bac.isFinishing()) {
            bac.mProDiag.show();
}

    }
}


19.Android 国际化工具
https://webtranslateit.com/en

20.翻墙利器
Lantern

21.Gradle 无法下载依赖的时候,可以尝试用2个仓库
mavenCentral()
 jcenter()

22.各种网络库的对比 volley asychttpclient retrofit
http://yanmingming.sinaapp.com/?cat=55

23.比较清晰的MVP 模式的理解
http://blog.csdn.net/lmj623565791/article/details/46596109

24.5个Android开发中比较常见的内存泄漏问题及解决办法
http://www.devstore.cn/essay/essayInfo/4187.html

25.动态更改Fragment 
/**
     * 接收网络修改的广播,用于动态更改fragment
     */
    private class FragmentNetWorkReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            Log.i("onReceive", new Date().toString());
            boolean netWorkState = intent.getBooleanExtra(ConstKey.netWorkState, true);
            //改变成有网
            if (netWorkState) {
                if (mFragments[0] instanceof NoNetWorkFragment) {
                    Main_One_Fragment fragment = new Main_One_Fragment();
                    updateFragment(fragment, 0);
                }
                if (mFragments[1] instanceof NoNetWorkFragment) {
                    Main_Two_Fragment fragment = new Main_Two_Fragment();
                    updateFragment(fragment, 1);
                }
                if (mFragments[2] instanceof NoNetWorkFragment) {
                    Main_Three_Fragment fragment = new Main_Three_Fragment();
                    updateFragment(fragment, 2);
                }
                if (mFragments[3] instanceof NoNetWorkFragment) {
                    Main_Four_Fragment fragment = new Main_Four_Fragment();
                    updateFragment(fragment, 3);
                }
                mMainFragmentPagerAdapter.notifyDataSetChanged();
            }
            //改变成没有网络了
            else {
                if (mFragments[0] instanceof Main_One_Fragment) {
                    NoNetWorkFragment fragment = new NoNetWorkFragment();
                    updateFragment(fragment, 0);
                }
                if (mFragments[1] instanceof Main_Two_Fragment) {
                    NoNetWorkFragment fragment = new NoNetWorkFragment();
                    updateFragment(fragment, 1);
                }
                if (mFragments[2] instanceof Main_Three_Fragment) {
                    NoNetWorkFragment fragment = new NoNetWorkFragment();
                    updateFragment(fragment, 2);
                }
                if (mFragments[3] instanceof Main_Four_Fragment) {
                    NoNetWorkFragment fragment = new NoNetWorkFragment();
                    updateFragment(fragment, 3);
                }
                mMainFragmentPagerAdapter.notifyDataSetChanged();
            }
        }
    }

 /**
     * 注册广播
     */
    private void registerReceiver() {
        receiver = new FragmentNetWorkReceiver();
        IntentFilter intentFilter = new IntentFilter(ConstKey.Main_TWO_Fragment_Receiver);
        registerReceiver(receiver, intentFilter);
    }


26. recyclerview 添加分隔线
 mListView.addItemDecoration(new HorizontalDividerItemDecoration.Builder(this).colorResId(R.color.W).size(24).build());
27.关于Android中popupwindow中的listview的onItemClick方法无效的解决方法
  mPopupWindow.setFocusable(true);
开始的时候,只有红米不能响应点击事件.以为是兼容性问题.没想到是这个问题.
28.使用glide 加载 CircleImageView ,第一次不显示图片,第二次再显示图片的问题.或者显示成方的图片 
 解决办法,在XML 中CircleImageView   设置默认需要显示的图片
在glide 显示图片的时候,一定不要加上placeholder来显示默认图片,因为已经有默认图片了.
直接into(imageview)
     Glide.with(this).load(mJsonShop.getLogo()).into(mIvShop);
29.关于正则表达式 经过不确定测试
   在JavaScript和Java 中使用正则表达式可能存在差异,JavaScript的两端存在//
   在JavaScript 中:/^((?!0)\\d+(.\\d{1,2})?)$/
   在Java中:^((?!0)\\d+(.\\d{1,2})?)$
30.关于fragment和fragmentadapter 问题
  在fragment base  的viewPager 里面放置了2个fragment A,B   A fragment里面放置了一个recyclerview ,但是无论怎么设置数据一直不出来.一直检查布局和
数据和recyclerview的adapter,可是都没有问题.弄了好久. 检查fragment base 的 viewpager 的adapter 发现多了这下面一个东西
 @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;//官方推荐写法
        }

去掉就可以了,fuck
附上完整的adapter 

    class MyPagerAdapter extends FragmentPagerAdapter {


        public MyPagerAdapter(FragmentManager fm) {
            super(fm);
        }

        @Override
        public int getCount() {
            return mList.size();//页卡数
        }

        @Override
        public android.support.v4.app.Fragment getItem(int position) {
            return mList.get(position);
        }


        @Override
        public CharSequence getPageTitle(int position) {
            return mTitleList[position];//页卡标题
        }

    }

31.动态设置3张图的大小,自适应屏幕的大小,难点在不要获取原来的layoutpara 来设置高宽度,然后再把这个layoutpara设置给imageview
这是不可以的.需要new 一个layoutparam 给imageview .这个new 的layout para的类型是父布局的类型
@Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        if (position < getItemCount() && (customHeaderView != null ? position <= mList.size() : position < mList.size()) && (customHeaderView != null ? position > 0 : true)) {
            int imageWidth = (G.WIDTH - G.getRealPix(46)) / 3;
            List<ImageView> imageViewList = new ArrayList<>();
            imageViewList.add(holder.mIvPhoto1);
            imageViewList.add(holder.mIvPhoto2);
            imageViewList.add(holder.mIvPhoto3);
            for (ImageView image : imageViewList) {
                image.setLayoutParams(new LinearLayout.LayoutParams(imageWidth, imageWidth));
                Glide.with(mFragment).load("http://b.hiphotos.baidu.com/image/pic/item/b8389b504fc2d562acd3bf97e41190ef77c66cf4.jpg").into(image);
            }
        }
    }
31.利用JavaScript 调用原来的支付宝支付,在支付宝的的回调中,重载页面报错
必须使用webview.post(runnable) 来执行.
  @JavascriptInterface
        public void goAlibabaToPay(String orderNo, String totalFee, String productName, String productDescp, String notifyUrl, final String returnUrl) {
            AlPayUtil payUtil = new AlPayUtil(ActivityLoveCard.this, new AlPayUtil.PayResultLisenter() {
                @Override
                public void payResult(int result) {
                    //支付成功
                    if (result == 9000 || result == 8000) {
                        //支付成功之后,肯定是实名认证成功了的,所以这里设置当前的值为1
                        G.getmUser().setAuthentication(1);
                        mWebView.post(new Runnable() {
                            @Override
                            public void run() {
                                mWebView.loadUrl(mUrls);
                            }
                        });
                    } else {
                        //支付失败
                        mWebView.post(new Runnable() {
                            @Override
                            public void run() {
                                mWebView.loadUrl("javascript:showPayError()");
                            }
                        });
                    }
                }
            });
            payUtil.goAliPay(orderNo, productDescp, productName, totalFee, notifyUrl);
        }

32.webview清除任务栈 ,技术要点
webview 打开A ,A点击了页面跳转到B,B页面点击了跳转到C,C页面点击跳转到了D ,这时候D是可以goback到C ---B---A 的,如果这时候我想D打开了E,并且让清空任务栈,
不能跳回到D,只能留在E ,解决方案如下

33. UltimateRecyclerView 设置stikyheader 的监听器
  private void initData() {
        mList = new ArrayList<>();
        mRvMain.setLayoutManager(new LinearLayoutManager(mActivity));
        mAdapter = new TestAdapter(mList, this);
        StickyRecyclerHeadersDecoration mHeadersDecor = new StickyRecyclerHeadersDecoration(mAdapter);
        mRvMain.addItemDecoration(mHeadersDecor);
        //        mRvMain.setItemAnimator(new jp.wasabeef.recyclerview.animators.LandingAnimator());
        //        mRvMain.getItemAnimator().setAddDuration(300);
        //        mRvMain.getItemAnimator().setRemoveDuration(300);
        StickyRecyclerHeadersTouchListener HeadersTouchListener = new StickyRecyclerHeadersTouchListener(mRvMain.mRecyclerView, mHeadersDecor);
        mRvMain.mRecyclerView.addOnItemTouchListener(HeadersTouchListener);
        HeadersTouchListener.setOnHeaderClickListener(new StickyRecyclerHeadersTouchListener.OnHeaderClickListener() { 
            @Override
            public void onHeaderClick(View header, int position, long headerId) {
                View headerView = header;
                final Button mBtnType = (Button) headerView.findViewById(R.id.chk_banner_act_main);
                if (System.currentTimeMillis() - mLastShowType < 400) {
                    return;
                }
                MobclickAgent.onEvent(mActivity, "homepage_category");
                showPopWindow(mBtnType, mBtnType);
            }
        });
        mRvMain.setAdapter(mAdapter);
        initLocation();
    }
34.关于定位的问题,在重构每获项目的时候,一直不能定位,才发现,之前在是在onstart才调用的 
  mLocClient.start();
  mLocClient.requestLocation();
  而我是在oncreated做的操作,并未调用此方法,导致一直没有定位.......



35.ListView添加header 之后onItemClick position 错位的问题(parent.getAdapter().getItem)
 private AdapterView.OnItemClickListener mAdItemClickLisenter = new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            AdInfo ad = (AdInfo) parent.getAdapter().getItem(position);
        }
    };
36.java 代码设置EditText 的输入类型和长度
editText.setInputType(InputType.TYPE_CLASS_NUMBER); //输入类型  
editText.setFilters(new InputFilter[]{new InputFilter.LengthFilter(6)}); //最大输入长度  
editText.setTransformationMethod(PasswordTransformationMethod.getInstance()); //设置为密码输入框

37.dialog 圆角的问题
这个问题困扰了很久,用了很多方法都不能圆角
1.开始用的方式是想到的用的shape 到dialog view 的根布局就可以圆角了.可是圆角是圆角了,背景还是正方形的
2.后来想到使用.9图片 ,但是由于图片不规则还是什么,期间更换了.9图片,还是不行,给根布局设置 .9图片,会强行把根布局拉大,实在是麻烦,各种无解
3.后来经过东哥的指导,写了一个CommonDialog,继承于Dialog,设置了了dialog 的style ,不过发现整个界面都缩小了.才发现了比较重要的原因,设置style 之后style 会冲掉xml view 根布局的任何设置的属性.所以,你无论你根据还是math_parent 还是什么的,都没用,只需要在根布局内部布局一层布局,再放置子view 就可以了.
4.后来发现CommonDialog 根本没什么用,只是载入了一个style 而已,还是想做成通用的dialog 写在dialog 工具类就可以了
5.写成工具类之后,发现圆角是圆角了,可是背景多了一层黑的............................
6.才发现使用的AlertDilaog 不是Dialog ,使用Dialog 就好了................

圆角shape的代码
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/W"/>
    <corners
        android:bottomLeftRadius="20dp"
        android:bottomRightRadius="20dp"
        android:topLeftRadius="20dp"
        android:topRightRadius="20dp"/>
</shape>

工具类
 public static Dialog showView(View view, Context context, int style) {
        Dialog dialog = new Dialog(context, style);
        dialog.setContentView(view);
        return dialog;
    }
Dialog XML 布局 需要设置内部的大小和内部的background ,外层是没有任何作用的


38.设置textview 的行间距
1、android:lineSpacingExtra
设置行间距，如”3dp”。

2、android:lineSpacingMultiplier
设置行间距的倍数，如”1.2″。

39.java 代码里面设置edittext 的input.type  当我想一个自定义view 内部的edittext 的输入类型,想设置为double 小数的类型
mCiMoney.setContentInputType( InputType.TYPE_NUMBER_FLAG_DECIMAL);这样设置的话,打开居然也有小写字母.
这样设置就可以了.mCiMoney.setContentInputType(InputType.TYPE_CLASS_NUMBER | InputType.TYPE_NUMBER_FLAG_DECIMAL);

40.getcolor 方法过时的解决办法
ContextCompat.getColor(this, R.color.V);

41. gradle 编译一直卡不过的问题.不知道为什么在家里gradle一直编译不过,一直开在debug_comblie ,卡了N小时,开始以为是网络的问题,开了翻墙和蓝灯也不行.  然后设置了代理,启动抓包工具,看Android studio 访问的连接.
看到访问的依赖都是http://192.168.203.241:8081/nexus/content/repositories/
为什么会访问到这里的,这样肯定是访问不了的呢.然后发现在根目录有这个
然后我就把jcenter 和mavenCentral 放到前面,还是不行,还是访问192那个,直到我把192这个注释,立马就开始下载依赖了,看来gradle 编译,设置了本地的maven 库,会默认从本地的maven库开始下载. 必须要注释才行.

42.字符串替换{id} 例如www.baidu.com/api/{id} 替换成 www.baidu.com/api/123
   //左大括号的转义：{ ==> u007B
 replaceStr.replaceAll("u007Bid}", "dafds");  不可行
replaceStr.replaceAll("\\{id}", "dafds");         不可行
 url.replaceAll("\\{id\\}", replaceStr);  可行

43.ScrollView 滚到到底部
 new Handler().post(new Runnable() {
                @Override
                public void run() {
                    mSvMain.fullScroll(ScrollView.FOCUS_DOWN);
                }
            });
44.查看依赖的大小的网站 http://www.methodscount.com/

45.viewpager +fragment 的时候,
在第二个fragment 上面有个按钮点击了之后,会有个popwindow ,
这时候,当我切换viewpager的时候,我想掩藏这个popwindow


46.使用Android studio 的时候,添加自定义控件之后,使用快捷键加入 自定义控件的命名控件,
会如图所示显示第二个,这样是错误的,不能出现自定义View ,但是又不会报错,以后注意

47.Error parsing XML: prefix must not be bound to one of the reserved namespace names 
一直报这个错,弄了2小时,翻遍了stackoverflow也不行,后来慢慢排除是Stirng 里面加了 自定义属性,如图.做外包的一群SB,妈的



48.提示XML 布局问题....但是看了自定义View 的包名也是正确完整的啊.   因为是复制的源码,一步一步看,发现源码在inflate一个XML,这个XML有个自定义控件,包名是原来的包名,不是拷贝过来的包名

49.当recylerView 嵌套 recyclerView滚动不流畅的问题






50.  RefreshLayout下拉 和 GridView (ListView)下滑事件冲突
 //RefreshLayout下拉 和 GridView 下滑事件冲突
        //重写子View  GridView 的onScrollLisenter事件
        //在onScroll 判断GridView的第一个可见的条目是不是位置0,并且判断第一个可见的条目的顶部是不是0(在父元素的顶端)
        mGridView.setOnScrollListener(new AbsListView.OnScrollListener() {
            @Override
            public void onScrollStateChanged(AbsListView view, int scrollState) {

            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                boolean enable = false;
                if (mGridView != null && mGridView.getChildCount() > 0) {
                    // check if the first item of the list is visible
                    boolean firstItemVisible = mGridView.getFirstVisiblePosition() == 0;
                    // check if the top of the first item is visible
                    boolean topOfFirstItemVisible = mGridView.getChildAt(0).getTop() == 0;
                    // enabling or disabling the refresh layout
                    enable = firstItemVisible && topOfFirstItemVisible;
                }
                mRefreshLayout.setEnabled(enable);
            }
        });

51.SS翻墙之后,如果Android studio要翻墙下载依赖,必须把Android studio的端口指向SS 翻墙的端口,这样AS 才能翻墙下载依赖.

同样,下载SDK 的时候,一样的设置

52.mac 下Android studio 配置 SVN  时候,不知道SVN的路径
whereis svn -> /usr/bin/svn

53.react native 每次新建项目的时候必须要联网,所以想到复制项目,改项目名
然后,运行react-native upgrade就重建android/ios这两个目录了，工程名字都会改过来。手工再改下index.ios.js和index.android.js里面的组件名就行了。
这时候一直报application  BUILD.CONFIG 的一个错.....
找了好久,原来是要修改APP目录下面的manifest 里面的包名,修改来和项目一模一样就可以了...

54.React 使用ListView 一直报错.React Native evaluating 'dataSource.rowIdentities
不知道是什么错,然后打日志,打开调试,才看到说需要dataSource,但是已经给了dataSource 啊.才发现dataSoruce 是写在ListView之间得.所以没显示.
正确示范:
 <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderRow}
            >
            </ListView>

55.react native 点击listview  的item ,响应超级慢,至少3秒才会响应.百思不得姐,然后发现是在dubugger.....
56.react native ListView  contentContainerStyle  设置flexWrap不换行的原因
 解释原因：由于在rn 0.28之后的版本上官方已经修改了flexWrap:'wrap'的工作方式了，之前版本的是flexWrap:'wrap'和默认的alignItems: 'stretch'是一起工作的；如果是0.28之后的版本，你需要加上alignItems: 'flex-start' 
57.butterknife 绑定View 一直为空的原因.之前想把butterknife 放到library 里面,不是app  里面
然后绑定始终为空.找了好久也不知道什么原因.原来是butterknife  不能放在library 里面
然后把butterknife  这三句 放到App 的的build 文件里就好了
    compile 'com.jakewharton:butterknife:8.4.0'
    apt 'com.jakewharton:butterknife-compiler:8.4.0'
    apply plugin: 'android-apt'
58.LeakCannay也要放在app下面的gradle 才能安装内存泄露检测程序

59.在Android Android studio的时候,默认会在安装目录有个gradle 的目录,但是莫名其妙的,在升级新版之后,项目一直在同步

http://blog.csdn.net/chrisyuu/article/details/52711025

肯定是gradle的配置出现问题,在Android studio 目录下的gradle 目录又生成了一个gradle 目录,且一直在下载gradle..........
这时候可以改项目中的的gradle 的路径,这样的话每次打开项目或者新建项目都会去这样弄一次


解决办法,就是配置默认的gradle 路径,在根目录有个bash_profile 在里面配置好默认的gradle路径
这里我们把默认的路径指向根目录的gradle 路径,而不是Android studio 下面的路径.配置好了之后重启Android studio
就可以了.

60.Android 前台service 和后台service的区别,以及各种后台服务优势对比
  http://blog.csdn.net/u014449096/article/details/51151142
http://mp.weixin.qq.com/s/y99YvYrjEc03zK8kMeBraw

61.gradle 编译跳过 test task
    tasks.withType(Test) {
 enabled=false
    }


allprojects {
    repositories {
        jcenter()
    }

    //skip Test tasks
    gradle.taskGraph.whenReady {
        tasks.each { task ->
            if (task.name.contains("Test"))
            {
                task.enabled = false
            }
        }
    }

}

或者命令行编译用 gradle assembelDebug 也是不执行test tasks的

62.android插件化、组件化、热补丁傻傻分不清
http://blog.csdn.net/edisonliao666/article/details/53914600

63.fresco 各种配置
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="20dp"
    android:layout_height="20dp"
    fresco:fadeDuration="300"
    fresco:actualImageScaleType="focusCrop"
    fresco:placeholderImage="@color/wait_color"
    fresco:placeholderImageScaleType="fitCenter"
    fresco:failureImage="@drawable/error"
    fresco:failureImageScaleType="centerInside"
    fresco:retryImage="@drawable/retrying"
    fresco:retryImageScaleType="centerCrop"
    fresco:progressBarImage="@drawable/progress_bar"
    fresco:progressBarImageScaleType="centerInside"
    fresco:progressBarAutoRotateInterval="1000"
    fresco:backgroundImage="@color/blue"
    fresco:overlayImage="@drawable/watermark"
    fresco:pressedStateOverlayImage="@color/red"
    fresco:roundAsCircle="false"
    fresco:roundedCornerRadius="1dp"
    fresco:roundTopLeft="true"
    fresco:roundTopRight="false"
    fresco:roundBottomLeft="false"
    fresco:roundBottomRight="true"
    fresco:roundWithOverlayColor="@color/corner_color"
    fresco:roundingBorderWidth="2dp"
    fresco:roundingBorderColor="@color/border_color"
  />

fresco 载入本地图片
  viewHolder.mIvImage.setImageURI(Uri.parse("res://x/" + R.drawable.tianshenghaomi));


类型	描述
center	居中，无缩放
centerCrop	保持宽高比缩小或放大，使得两边都大于或等于显示边界。居中显示。
focusCrop	同centerCrop, 但居中点不是中点，而是指定的某个点
centerInside	使两边都在显示边界内，居中显示。
如果图尺寸大于显示边界，则保持长宽比缩小图片。
fitCenter	保持宽高比，缩小或者放大，使得图片完全显示在显示边界内。居中显示
fitStart	同上。但不居中，和显示边界左上对齐
fitEnd	同fitCenter， 但不居中，和显示边界右下对齐
fitXY	不保存宽高比，填充满显示边界
none	如要使用tile mode显示, 需要设置为none
这些缩放类型和Android ImageView 支持的缩放类型几乎一样.


package com.facebook.fresco.sample.configs;
 
import android.content.Context;
import android.content.res.Resources;
import android.graphics.drawable.Drawable;
import android.net.Uri;
import android.os.Environment;
 
import com.facebook.cache.disk.DiskCacheConfig;
import com.facebook.common.internal.Supplier;
import com.facebook.common.util.ByteConstants;
import com.facebook.drawee.backends.pipeline.Fresco;
import com.facebook.drawee.drawable.ProgressBarDrawable;
import com.facebook.drawee.generic.GenericDraweeHierarchy;
import com.facebook.drawee.generic.GenericDraweeHierarchyBuilder;
import com.facebook.drawee.generic.RoundingParams;
import com.facebook.drawee.interfaces.DraweeController;
import com.facebook.drawee.interfaces.SimpleDraweeControllerBuilder;
import com.facebook.fresco.sample.R;
import com.facebook.fresco.sample.instrumentation.InstrumentedDraweeView;
import com.facebook.imagepipeline.cache.MemoryCacheParams;
import com.facebook.imagepipeline.common.ImageDecodeOptions;
import com.facebook.imagepipeline.common.ResizeOptions;
import com.facebook.imagepipeline.core.ImagePipelineConfig;
import com.facebook.imagepipeline.request.ImageRequest;
import com.facebook.imagepipeline.request.ImageRequest.RequestLevel;
import com.facebook.imagepipeline.request.ImageRequestBuilder;
/**
 * 配置才是关键～～～细细看来确实是不错的图片缓存框架，
 * @author notreami
 *
 */
public class ConfigConstants {
  private static final int MAX_HEAP_SIZE = (int) Runtime.getRuntime().maxMemory();//分配的可用内存
  public static final int MAX_MEMORY_CACHE_SIZE = MAX_HEAP_SIZE / 4;//使用的缓存数量
   
  public static final int MAX_SMALL_DISK_VERYLOW_CACHE_SIZE = 5 * ByteConstants.MB;//小图极低磁盘空间缓存的最大值（特性：可将大量的小图放到额外放在另一个磁盘空间防止大图占用磁盘空间而删除了大量的小图）
  public static final int MAX_SMALL_DISK_LOW_CACHE_SIZE = 10 * ByteConstants.MB;//小图低磁盘空间缓存的最大值（特性：可将大量的小图放到额外放在另一个磁盘空间防止大图占用磁盘空间而删除了大量的小图）
  public static final int MAX_SMALL_DISK_CACHE_SIZE = 20 * ByteConstants.MB;//小图磁盘缓存的最大值（特性：可将大量的小图放到额外放在另一个磁盘空间防止大图占用磁盘空间而删除了大量的小图）
   
  public static final int MAX_DISK_CACHE_VERYLOW_SIZE = 10 * ByteConstants.MB;//默认图极低磁盘空间缓存的最大值
  public static final int MAX_DISK_CACHE_LOW_SIZE = 30 * ByteConstants.MB;//默认图低磁盘空间缓存的最大值
  public static final int MAX_DISK_CACHE_SIZE = 50 * ByteConstants.MB;//默认图磁盘缓存的最大值
   
   
  private static final String IMAGE_PIPELINE_SMALL_CACHE_DIR = "imagepipeline_cache";//小图所放路径的文件夹名
  private static final String IMAGE_PIPELINE_CACHE_DIR = "imagepipeline_cache";//默认图所放路径的文件夹名
 
  private static ImagePipelineConfig sImagePipelineConfig;
 
  private ConfigConstants(){
       
  }
  /**
   * 初始化配置，单例
   */
  public static ImagePipelineConfig getImagePipelineConfig(Context context) {
    if (sImagePipelineConfig == null) {
      sImagePipelineConfig = configureCaches(context);
    }
    return sImagePipelineConfig;
  }
 
 
 
  /**
   * 初始化配置
   */
  private static ImagePipelineConfig configureCaches(Context context) {
      //内存配置
      final MemoryCacheParams bitmapCacheParams = new MemoryCacheParams(
                ConfigConstants.MAX_MEMORY_CACHE_SIZE, // 内存缓存中总图片的最大大小,以字节为单位。
                Integer.MAX_VALUE,                     // 内存缓存中图片的最大数量。
                ConfigConstants.MAX_MEMORY_CACHE_SIZE, // 内存缓存中准备清除但尚未被删除的总图片的最大大小,以字节为单位。
                Integer.MAX_VALUE,                     // 内存缓存中准备清除的总图片的最大数量。
                Integer.MAX_VALUE);                    // 内存缓存中单个图片的最大大小。
       
      //修改内存图片缓存数量，空间策略（这个方式有点恶心）
      Supplier<MemoryCacheParams>  mSupplierMemoryCacheParams= new Supplier<MemoryCacheParams>() {
        @Override
         public MemoryCacheParams get() {
            return bitmapCacheParams;
        }
    };
     
    //小图片的磁盘配置
      DiskCacheConfig diskSmallCacheConfig =  DiskCacheConfig.newBuilder()
              .setBaseDirectoryPath(context.getApplicationContext().getCacheDir())//缓存图片基路径
              .setBaseDirectoryName(IMAGE_PIPELINE_SMALL_CACHE_DIR)//文件夹名
//            .setCacheErrorLogger(cacheErrorLogger)//日志记录器用于日志错误的缓存。
//            .setCacheEventListener(cacheEventListener)//缓存事件侦听器。
//            .setDiskTrimmableRegistry(diskTrimmableRegistry)//类将包含一个注册表的缓存减少磁盘空间的环境。
              .setMaxCacheSize(ConfigConstants.MAX_DISK_CACHE_SIZE)//默认缓存的最大大小。
              .setMaxCacheSizeOnLowDiskSpace(MAX_SMALL_DISK_LOW_CACHE_SIZE)//缓存的最大大小,使用设备时低磁盘空间。 
              .setMaxCacheSizeOnVeryLowDiskSpace(MAX_SMALL_DISK_VERYLOW_CACHE_SIZE)//缓存的最大大小,当设备极低磁盘空间
//            .setVersion(version)
              .build();
       
    //默认图片的磁盘配置
      DiskCacheConfig diskCacheConfig =  DiskCacheConfig.newBuilder()
              .setBaseDirectoryPath(Environment.getExternalStorageDirectory().getAbsoluteFile())//缓存图片基路径
              .setBaseDirectoryName(IMAGE_PIPELINE_CACHE_DIR)//文件夹名
//            .setCacheErrorLogger(cacheErrorLogger)//日志记录器用于日志错误的缓存。
//            .setCacheEventListener(cacheEventListener)//缓存事件侦听器。
//            .setDiskTrimmableRegistry(diskTrimmableRegistry)//类将包含一个注册表的缓存减少磁盘空间的环境。
              .setMaxCacheSize(ConfigConstants.MAX_DISK_CACHE_SIZE)//默认缓存的最大大小。
              .setMaxCacheSizeOnLowDiskSpace(MAX_DISK_CACHE_LOW_SIZE)//缓存的最大大小,使用设备时低磁盘空间。
              .setMaxCacheSizeOnVeryLowDiskSpace(MAX_DISK_CACHE_VERYLOW_SIZE)//缓存的最大大小,当设备极低磁盘空间
//            .setVersion(version)
              .build();
       
      //缓存图片配置
      ImagePipelineConfig.Builder configBuilder = ImagePipelineConfig.newBuilder(context)
//            .setAnimatedImageFactory(AnimatedImageFactory animatedImageFactory)//图片加载动画
              .setBitmapMemoryCacheParamsSupplier(mSupplierMemoryCacheParams)//内存缓存配置（一级缓存，已解码的图片）
//            .setCacheKeyFactory(cacheKeyFactory)//缓存Key工厂
//            .setEncodedMemoryCacheParamsSupplier(encodedCacheParamsSupplier)//内存缓存和未解码的内存缓存的配置（二级缓存）
//            .setExecutorSupplier(executorSupplier)//线程池配置
//            .setImageCacheStatsTracker(imageCacheStatsTracker)//统计缓存的命中率
//            .setImageDecoder(ImageDecoder imageDecoder) //图片解码器配置
//            .setIsPrefetchEnabledSupplier(Supplier<Boolean> isPrefetchEnabledSupplier)//图片预览（缩略图，预加载图等）预加载到文件缓存
              .setMainDiskCacheConfig(diskCacheConfig)//磁盘缓存配置（总，三级缓存）
//            .setMemoryTrimmableRegistry(memoryTrimmableRegistry) //内存用量的缩减,有时我们可能会想缩小内存用量。比如应用中有其他数据需要占用内存，不得不把图片缓存清除或者减小 或者我们想检查看看手机是否已经内存不够了。
//            .setNetworkFetchProducer(networkFetchProducer)//自定的网络层配置：如OkHttp，Volley
//            .setPoolFactory(poolFactory)//线程池工厂配置
//            .setProgressiveJpegConfig(progressiveJpegConfig)//渐进式JPEG图
//            .setRequestListeners(requestListeners)//图片请求监听
//            .setResizeAndRotateEnabledForNetwork(boolean resizeAndRotateEnabledForNetwork)//调整和旋转是否支持网络图片
              .setSmallImageDiskCacheConfig(diskSmallCacheConfig)//磁盘缓存配置（小图片，可选～三级缓存的小图优化缓存）
              ;   
    return configBuilder.build();       
  }
  //圆形，圆角切图，对动图无效
  public static RoundingParams getRoundingParams(){
      RoundingParams roundingParams = RoundingParams.fromCornersRadius(7f);
//    roundingParams.asCircle();//圆形
//    roundingParams.setBorder(color, width);//fresco:roundingBorderWidth="2dp"边框  fresco:roundingBorderColor="@color/border_color"
//    roundingParams.setCornersRadii(radii);//半径
//    roundingParams.setCornersRadii(topLeft, topRight, bottomRight, bottomLeft)//fresco:roundTopLeft="true" fresco:roundTopRight="false" fresco:roundBottomLeft="false" fresco:roundBottomRight="true"
//    roundingParams. setCornersRadius(radius);//fresco:roundedCornerRadius="1dp"圆角
//    roundingParams.setOverlayColor(overlayColor);//fresco:roundWithOverlayColor="@color/corner_color"
//    roundingParams.setRoundAsCircle(roundAsCircle);//圆
//    roundingParams.setRoundingMethod(roundingMethod);
//    fresco:progressBarAutoRotateInterval="1000"自动旋转间隔
      // 或用 fromCornersRadii 以及 asCircle 方法
      return roundingParams;
  }
   
//Drawees   DraweeHierarchy  组织
  public static GenericDraweeHierarchy getGenericDraweeHierarchy(Context context){
      GenericDraweeHierarchy gdh = new GenericDraweeHierarchyBuilder(context.getResources())
//            .reset()//重置
//            .setActualImageColorFilter(colorFilter)//颜色过滤
//            .setActualImageFocusPoint(focusPoint)//focusCrop, 需要指定一个居中点
//            .setActualImageMatrix(actualImageMatrix)
//            .setActualImageScaleType(actualImageScaleType)//fresco:actualImageScaleType="focusCrop"缩放类型
//            .setBackground(background)//fresco:backgroundImage="@color/blue"背景图片
//            .setBackgrounds(backgrounds)
//            .setFadeDuration(fadeDuration)//fresco:fadeDuration="300"加载图片动画时间
              .setFailureImage(ConfigConstants.sErrorDrawable)//fresco:failureImage="@drawable/error"失败图
//            .setFailureImage(failureDrawable, failureImageScaleType)//fresco:failureImageScaleType="centerInside"失败图缩放类型
//            .setOverlay(overlay)//fresco:overlayImage="@drawable/watermark"叠加图
//            .setOverlays(overlays)
              .setPlaceholderImage(ConfigConstants.sPlaceholderDrawable)//fresco:placeholderImage="@color/wait_color"占位图
//            .setPlaceholderImage(placeholderDrawable, placeholderImageScaleType)//fresco:placeholderImageScaleType="fitCenter"占位图缩放类型
//            .setPressedStateOverlay(drawable)//fresco:pressedStateOverlayImage="@color/red"按压状态下的叠加图
              .setProgressBarImage(new ProgressBarDrawable())//进度条fresco:progressBarImage="@drawable/progress_bar"进度条
//            .setProgressBarImage(progressBarImage, progressBarImageScaleType)//fresco:progressBarImageScaleType="centerInside"进度条类型
//            .setRetryImage(retryDrawable)//fresco:retryImage="@drawable/retrying"点击重新加载
//            .setRetryImage(retryDrawable, retryImageScaleType)//fresco:retryImageScaleType="centerCrop"点击重新加载缩放类型
              .setRoundingParams(RoundingParams.asCircle())//圆形/圆角fresco:roundAsCircle="true"圆形
              .build();
      return gdh;
  }
   
   
//DraweeView～～～SimpleDraweeView——UI组件
//  public static SimpleDraweeView getSimpleDraweeView(Context context,Uri uri){
//    SimpleDraweeView simpleDraweeView=new SimpleDraweeView(context);
//    simpleDraweeView.setImageURI(uri);
//    simpleDraweeView.setAspectRatio(1.33f);//宽高缩放比
//    return simpleDraweeView;
//  }
   
  //SimpleDraweeControllerBuilder
  public static SimpleDraweeControllerBuilder getSimpleDraweeControllerBuilder(SimpleDraweeControllerBuilder sdcb,Uri uri,  Object callerContext,DraweeController draweeController){
      SimpleDraweeControllerBuilder controllerBuilder = sdcb
                .setUri(uri)
                .setCallerContext(callerContext)
//              .setAspectRatio(1.33f);//宽高缩放比
                .setOldController(draweeController);
      return controllerBuilder;
  }
   
  //图片解码
  public static ImageDecodeOptions getImageDecodeOptions(){
      ImageDecodeOptions decodeOptions = ImageDecodeOptions.newBuilder()
//            .setBackgroundColor(Color.TRANSPARENT)//图片的背景颜色
//            .setDecodeAllFrames(decodeAllFrames)//解码所有帧
//            .setDecodePreviewFrame(decodePreviewFrame)//解码预览框
//            .setForceOldAnimationCode(forceOldAnimationCode)//使用以前动画
//            .setFrom(options)//使用已经存在的图像解码
//            .setMinDecodeIntervalMs(intervalMs)//最小解码间隔（分位单位）
              .setUseLastFrameForPreview(true)//使用最后一帧进行预览
              .build();
      return decodeOptions;
  }
   
  //图片显示
  public static ImageRequest getImageRequest(InstrumentedDraweeView view,String uri){
      ImageRequest imageRequest = ImageRequestBuilder.newBuilderWithSource(Uri.parse(uri))
//            .setAutoRotateEnabled(true)//自动旋转图片方向
//            .setImageDecodeOptions(getImageDecodeOptions())//  图片解码库
//            .setImageType(ImageType.SMALL)//图片类型，设置后可调整图片放入小图磁盘空间还是默认图片磁盘空间
//            .setLocalThumbnailPreviewsEnabled(true)//缩略图预览，影响图片显示速度（轻微）
              .setLowestPermittedRequestLevel(RequestLevel.FULL_FETCH)//请求经过缓存级别  BITMAP_MEMORY_CACHE，ENCODED_MEMORY_CACHE，DISK_CACHE，FULL_FETCH
//            .setPostprocessor(postprocessor)//修改图片
//            .setProgressiveRenderingEnabled(true)//渐进加载，主要用于渐进式的JPEG图，影响图片显示速度（普通）
              .setResizeOptions(new ResizeOptions(view.getLayoutParams().width, view.getLayoutParams().height))//调整大小
//            .setSource(Uri uri)//设置图片地址
              .build();
      return imageRequest;
  }
   
//DraweeController 控制 DraweeControllerBuilder
  public static DraweeController getDraweeController(ImageRequest imageRequest,InstrumentedDraweeView view){
      DraweeController draweeController = Fresco.newDraweeControllerBuilder()
//            .reset()//重置
              .setAutoPlayAnimations(true)//自动播放图片动画
//            .setCallerContext(callerContext)//回调
              .setControllerListener(view.getListener())//监听图片下载完毕等
//            .setDataSourceSupplier(dataSourceSupplier)//数据源
//            .setFirstAvailableImageRequests(firstAvailableImageRequests)//本地图片复用，可加入ImageRequest数组
              .setImageRequest(imageRequest)//设置单个图片请求～～～不可与setFirstAvailableImageRequests共用，配合setLowResImageRequest为高分辨率的图
//            .setLowResImageRequest(ImageRequest.fromUri(lowResUri))//先下载显示低分辨率的图
              .setOldController(view.getController())//DraweeController复用
              .setTapToRetryEnabled(true)//点击重新加载图
              .build();
      return draweeController;
  }
   
  //默认加载图片和失败图片
  public static Drawable sPlaceholderDrawable;
  public static Drawable sErrorDrawable;
   
  @SuppressWarnings("deprecation")
public static void init(final Resources resources) {
        if (sPlaceholderDrawable == null) {
          sPlaceholderDrawable = resources.getDrawable(R.color.placeholder);
        }
        if (sErrorDrawable == null) {
          sErrorDrawable = resources.getDrawable(R.color.error);
        }
      }
}

东哥fresco 配置
    private void initFresco() {
        DiskCacheConfig diskCacheConfig = DiskCacheConfig.newBuilder()
                .setBaseDirectoryPath(getCacheDir())
                .setBaseDirectoryName("FrescoCache")
                .setMaxCacheSize(100 * ByteConstants.MB)
                .build();
        ImagePipelineConfig config = ImagePipelineConfig.newBuilder(this)
                .setMainDiskCacheConfig(diskCacheConfig)
                        // Bitmap缓存存储Bitmap对象，这些Bitmap对象可以立刻用来显示或者用于后处理；5.0以下位于ashmem，5.0及以上位于JavaHeap
                .setBitmapMemoryCacheParamsSupplier(mBitmapMemoryCacheParamsSupplier)
                        // 未解码图片的内存缓存，这个缓存存储的是原始压缩格式的图片
                .setEncodedMemoryCacheParamsSupplier(mEncodedMemoryCacheParamsSupplier)
                .setMemoryTrimmableRegistry(mMemoryTrimmableRegistry)
                .build();
        Fresco.initialize(this, config);
    }

 private Supplier<MemoryCacheParams> mEncodedMemoryCacheParamsSupplier = new Supplier<MemoryCacheParams>() {
        // We want memory cache to be bound only by its memory consumption
        private static final int MAX_CACHE_ENTRIES = Integer.MAX_VALUE;
        private static final int MAX_EVICTION_QUEUE_ENTRIES = Integer.MAX_VALUE;

        @Override
        public MemoryCacheParams get() {
            final int maxCacheSize = getMaxCacheSize();
            Logger.e("mEncodedMemoryCacheParamsSupplier: ", "maxCacheSize: " + maxCacheSize / ByteConstants.MB);
            final int maxCacheEntrySize = maxCacheSize / 8;
            return new MemoryCacheParams(
                    maxCacheSize,
                    MAX_CACHE_ENTRIES,
                    maxCacheSize,
                    MAX_EVICTION_QUEUE_ENTRIES,
                    maxCacheEntrySize);
        }

        private int getMaxCacheSize() {
            final int maxMemory = (int) Math.min(Runtime.getRuntime().maxMemory(), Integer.MAX_VALUE);
            if (maxMemory < 16 * ByteConstants.MB) {
                return ByteConstants.MB;
            } else if (maxMemory < 32 * ByteConstants.MB) {
                return 2 * ByteConstants.MB;
            } else {
                return 4 * ByteConstants.MB;
            }
        }
    };

64.jrebel  for Android 加速Android编译的软件,秒杀instant run

65.四种Post提交数据的方式
协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。
 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。
1.application/x-www-form-urlencoded 原生表单提交,不设置enctype属性
Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码

2.multipart/form-data 原生表单提交的enctype设置为multipart/form-data
首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 --boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 --boundary-- 标示结束 ,这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

3.application/json
 
4.text/xml 使用较少

65.https 的通信步骤
http://www.cnblogs.com/bjlhx/p/6560870.html

66openssl、x509、crt、cer、key、csr、ssl、tls 这些都是什么鬼?
http://www.cnblogs.com/yjmyzz/p/openssl-tutorial.html

67.我一般编译卡死的时候就在命令行里运行 gradlew clean --debug 看看是因为什么而卡死

67.adb 查看cpu 占用
连接手机,开启USB 调试
输入 adb shell
 输入 top -m 10 -s cpu  就可以每隔10S 刷新 CPU 使用率

68.Android跳到方法块的最后一行和最前面一行
MOVE to code block start
MOVE to code block end