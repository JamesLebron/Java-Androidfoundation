## Android DPI DP PX Bitmap Drawable相关知识点

### 1.相关术语
	1.屏幕尺寸 红米4.7寸 ZUK Z2 PRO 5.2存
	2.屏幕比 16:9 4:3 等
	3.分辨率 主流手机 1920*1080
	4.px 像素 在Android里面 px=dp*density
	4.dpi (dot per inch) 每英寸像素点 dpi=ppi
		dpi计算
		长的平方+宽的平方得到A ,用A 开根号 得到B 最后用B除以尺寸
		java 公式:根 Math.sqrt(Math.pow(1280,2)+Math.pow(720, 2))/4.7 = 312
	5.density 密度  dpi/160=density
	6.dip (Density-independent pixel) dip=dp Android 开发使用单位,代表在dpi 为160的手机上,1dp = 1px
	
	eg:一个手机5寸 分辨率 1920*1080 求出dpi density 
		dpi=√ (1920^2 + 1080^2) / 5 = 440
		density=440/160=2.75
		不计算缩放的条件下,1dp ,在此手机上=2.75px
		在此手机上显示一张原始尺寸为100px*100px的图片,显示出来的尺寸是275px*275px

### 2.一张图片在手机里面到底占用了多少内存?
	手机条件:三星S6 分辨率( 2560x1440 ) 尺寸:5.1 存 
	图片条件:一张png 格式的图片,原始分辨率为 522x686
	编码条件:将此图片放置于 drawable-xxhdpi 
	我们先计算出S6的 dpi(ppi) 575.92344316 四舍五入为576
	那我们计算一张图片的公式是:高px*宽px*图片格式(888,565...所占用的字节数)
	那这张图片占用的内存内存就为:522*686*4吗?
	no,no,no
	因为我们根本没考虑缩放问题?
	深入源码去看bitmap的占用
		公式是这样的,缩放之后的px=int(图片原始px*当前设备densityDpi/图片放置目录densityDpi+0.5)
		eg:
		scaledWidth = int( 522 * 640 / 480f + 0.5) = int(696.5) = 696
		scaledHeight = int( 686 * 640 / 480f + 0.5) = int(915.16666…) = 915   
		公式解析:
			当前densityDpi,设备的densityDpi,这个值很坑,并不是我们去计算出来的dpi,这个值Android 默认内置了,因为当前设备的分辨率是2K屏幕,是Android 的默认值之一,对应xxhdpi densityDpi 为640(详见:http://blog.qiji.tech/archives/2581)
			图片放置的densityDpi,放置在drawable-xxhdpi(对应1080P),对应的densityDpi为480
			最后scaledWidth* scaledHeight*4 才是正确的结果
	默认的drawable 目录对应的densityDpi
		mdpi是160,    density是1,   320×480分辨率
		hdpi是240,    density是1.5, 480×800分辨率
		xhdpi是320,   density是2,   720屏幕
		xxhdpi是480,  density是3,   1080P屏幕
		xxxhdpi是640, density是4,   2K屏幕
```java 
源码解析:
public static Bitmap decodeResourceStream(Resources res, TypedValue value,InputStream is, Rect pad, Options opts) {
        if (opts == null) {
            opts = new Options();
        }
        //这里是取出图片放置目录的densityDpi
        if (opts.inDensity == 0 && value != null) {
       	  //如果把图片直接放在drawable目录,这里density=0,如果放在指定的xhdpi目录,会按照放置目录的densityDpi来赋值
            final int density = value.density;
            //如果把图片直接放在drawable目录会走向这里,opts.inDensity=160
            if (density == TypedValue.DENSITY_DEFAULT) {
                opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
            } else if (density != TypedValue.DENSITY_NONE) {
                opts.inDensity = density;
            }
        }
        //这里是取出设备的densityDpi
        if (opts.inTargetDensity == 0 && res != null) {
            opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
        }
        return decodeStream(is, pad, opts);
    }
```

### 3.如何正确的压缩一张图片?
```java
private Bitmap decodeCaculateBitmap(Resources resources, int resId, int requestWidth, int requestHeight) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        //inJustDecodeBounds,只会获取图片的信息等,不会真正的加载图片
        options.inJustDecodeBounds = true;
        //更改图片的编码格式
        options.inPreferredConfig = Bitmap.Config.RGB_565;
        BitmapFactory.decodeResource(resources, resId, options);
        //计算图片的缩放比例 缩放比例只能是2的倍数,如果为基数,
        options.inSampleSize = calculateInSampleSize(options, requestWidth, requestHeight);
        options.inJustDecodeBounds = false;
        return BitmapFactory.decodeResource(resources, resId, options);
    }

    /**
     * inSampleSize 只能是2的整数倍
     * 这里如果inSampleSize 是7 ,那将会往下找2的整数倍 就是:6
     * 调试亲测
     */
private int calculateInSampleSize(BitmapFactory.Options options, int requestWidth, int requestHeight) {
        //获取图片的原始高宽
        int rawWidth = options.outWidth;
        int rawHeight = options.outHeight;
        int inSampleSize = 1;
        //条件是原始图片高宽需要大于压缩的图片的高宽
        if (rawHeight > requestHeight || rawWidth > requestWidth) {
            int scaleHeight = Math.round((float) rawHeight / (float) requestHeight);
            int scaleWidth = Math.round((float) rawWidth / (float) requestWidth);
            //取最小值
            inSampleSize = scaleHeight < scaleWidth ? scaleHeight : scaleWidth;
        }
        return inSampleSize;
    }
```

### 4.decodeResource()和decodeFile()的区别
	decodeFile()用于读取SD卡上的图,得到的是图片的原始尺寸
	decodeResource()用于读取res,raw等资源,得到的图片的原始尺寸*缩放系数
	
### 5.bitmap的存放,是否需要调用recyle()方法
	3.0之前,bitmap像素数据和bitmap对象都放在native内存中,官方强烈建议调用recyle()方法
	3.0之后,bitmap存在Dalvik堆中.
	但是在官方文档中,并未出现不用recyle()的说明,所以当使用完之后,还是建议调用
	
### 6.bitmap和drawable的区别
	drawable是抽象类,从官方注释上,可以理解为一种可以绘制到canvas的一种东西,是一种抽象概念
	drawable注释下说明了一些drawable存在实体
	1.bitmap
	2.nine
	3.shape
	4.layers
	5.states
	6.levels
	7.scale
	由此可以看出,bitmap 是drawable存在的实体之一
	两者之间可以通过BitmapDrawable联系起来
### 7. ThumbnailUtils 是一个可以生成图片文件和视频文件的缩略图的工具类


		
	