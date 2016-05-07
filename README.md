# injor
一个换肤框架，出自[android-change-skin]（https://github.com/hack2ware/android-change-skin.git）框架,这里发布到jcenter上了，使用compile "com.sien:injor:0.1.0"引入。


使用说明

【换肤类型】：

------------------------------框架提供类型-----------------------------

typeface 字体 （Typeface类型） 注：TextView limit
textSize 字体大小 (dimen类型) 注：TextView limit
text 文字（string类型） 注：TextView limit
textColor 字体颜色(ColorStateList类型) 注：TextView limit
src 图片文件 (drawable类型) 注：ImageView limit
background 背景（drawable类型|color）
divider 分割线(drawable类型) 注：ListView limit

-------------------------------自定义扩展类型--------------------------

paddingdrawable textview绘制图片（Drawable类型） 注：TextView limit

【使用方式】：

1、build.gradle依赖中添加引用：
	compile "com.sien:injor:0.1.0"

2、Application中初始化换肤框架
	SkinManager sm = SkinManager.getInstance().init(this);
	
3、注册自定义换肤类型（可选）
	sm.addProcessor(new PaddingDrawableProcessor());
	
4、需要换肤的页面中注册/取消注册
	SkinManager.getInstance().register(this);
	SkinManager.getInstance().unregister(this);
	
5、换肤
	SkinManager.getInstance().loadInternalSkin("", null);//切换默认风格
	SkinManager.getInstance().loadInternalSkin("red", null);//切换内置红色主题风格
	SkinManager.getInstance().loadPluginSkin(Environment.getExternalStorageDirectory() + File.separator + "ysyskin.apk", "com.suneee.ysyskin", null);//切换插件apk中的风格（注：插件中皮肤的名称是用默认的资源名称）

6、对于需要换肤的控件需要添加tag标签，格式如下：
	android:tag = “skin:资源名称:资源类型”
对同一个组件需要替换多个属性使用“|”间隔
	android:tag = “skin:资源名称:资源类型|skin:资源名称:资源类型”

例：
	skin:tab_main_selector:textColor
	skin:tab_main_selector:textColor|skin:maintab_1_selector:paddingdrawable
	
7、对于动态添加的组件，需要做特殊处理
	组件应用皮肤资源：
		。动态添加tag属性
			textView.setTag("skin:tab_main_selector:textColor|skin:maintab_1_selector:paddingdrawable");
		。应用皮肤资源
			SkinManager.getInstance().injectSkinAsync(textView);
		
	页面级重新应用皮肤资源：
		SkinManager.getInstance().apply(this);
		
8、自定义换肤类型，必须继承自ISkinProcessor类，需重写getName方法，返回换肤类型的名称，还需要重写process(Activity activity, View v, String resName, ResourceManager resourceManager, boolean isInUiThread)方法，实现具体的替换操作。

例如：
/**
 * 切换textview的paddingDrawable
 * Created by suneee012 on 2016/3/29.
 */
public class PaddingDrawableProcessor extends ISkinProcessor{
    public static final String PROCESSOR_PADDINGDRAWABLE = "paddingdrawable";
    private static final String DEFTYPE_DRAWABLE = "drawable";

    public static final int PADDING_LEFT = 1;
    public static final int PADDING_RIGHT = 2;
    public static final int PADDING_TOP = 3;
    public static final int PADDING_BOTTOM = 4;

    private int paddingPosition = PADDING_TOP;

    public PaddingDrawableProcessor(){}
    
    public PaddingDrawableProcessor(int postion){
        if(postion > 0 && postion < 5){
            paddingPosition = postion;
        }
    }

    @Override
    public String getName() {
        return PROCESSOR_PADDINGDRAWABLE;
    }

    @Override
    public void process(Activity activity, View v, String resName, ResourceManager resourceManager, boolean isInUiThread) {
        if(v instanceof TextView) {

            final TextView view = (TextView) v;

            final Drawable drawable = resourceManager.getDrawable(resName);
            if (drawable != null){
                if (isInUiThread) {
                    apply(view,drawable);
                } else {
                    UITaskManager.getInstance().post(new Runnable() {
                        @Override
                        public void run() {
                            apply(view,drawable);
                        }
                    });
                }
            }
        }
    }

    private void apply(TextView view,Drawable drawable){
        drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight());//必须设置图片大小，否则不显示
        if(paddingPosition == PADDING_LEFT){
            view.setCompoundDrawables(drawable, null, null, null);
        }else if(paddingPosition == PADDING_TOP){
            view.setCompoundDrawables(null, drawable, null, null);
        }else if(paddingPosition == PADDING_RIGHT){
            view.setCompoundDrawables(null, null, drawable, null);
        }else if(paddingPosition == PADDING_BOTTOM){
            view.setCompoundDrawables(null, null, null, drawable);
        }
    }
}
	
【注意事项】：

1、不管是内置主题、插件主题，资源最好都是完整的一套（缺失资源使用应用apk中默认风格）

2、（内置主题）需要一套默认皮肤，正常的命名方式，其他风格的皮肤，需要在正常命名后加上相应的后缀，格式如下：
	默认风格：资源名称.xml 
	红色风格：资源名称_red.xml
	
例：
	maintab_1_selector.xml
	maintab_1_selector_red.xml
	
3、（插件主题）apk中皮肤名称与应用默认皮肤名称一致：
	应用apk：资源名称.jpg 
	皮肤插件apk：资源名称.jpg 
例：
	maintab_1_selector.jpg
	maintab_1_selector.jpg	

