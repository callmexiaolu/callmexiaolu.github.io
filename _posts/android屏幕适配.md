## 屏幕适配



#### 代码适配

* Where(适用场景)：
  自定义view，使用代码进行单位转换，动态添加view。

* How(如何写)：

  ```java
  import android.content.Context;
  
  /**
   * Created by hosition on 2018/3/31.
   * 工具类
   * 将dp转换为px来使用，以便适配不同的屏幕显示效果
   */
  
  public class DensityUtil {
      /**
       * 根据手机的分辨率从 dp 的单位 转成为 px(像素)
       * @param context   上下文
       * @param dpValue   dp值
       * @return  px值
       */
      public static int dip2px(Context context, float dpValue) {
          final float scale = context.getResources().getDisplayMetrics().density;
          return (int) (dpValue * scale + 0.5f);
      }
  
      /**
       * 根据手机的分辨率从 px(像素) 的单位 转成为 dp
       * @param context   上下文
       * @param pxValue   px值
       * @return  dp值
       */
      public static int px2dip(Context context, float pxValue) {
          final float scale = context.getResources().getDisplayMetrics().density;
          return (int) (pxValue / scale + 0.5f);
      }
  
      /**
       * 将px值转换为sp值，保证文字大小不变
       *
       * @param context 上下文对象
       * @param pxValue 字体大小像素
       * @return 字体大小sp
       */
      public static int px2sp(Context context, float pxValue) {
          final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
          return (int) (pxValue / fontScale + 0.5f);
      }
  
      /**
       * 将sp值转换为px值，保证文字大小不变
       *
       * @param context 上下文对象
       * @param spValue 字体大小sp
       * @return 字体大小像素
       */
      public static int sp2px(Context context, float spValue) {
          final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
          return (int) (spValue * fontScale + 0.5f);
      }
  }
  
  ```

  

#### 文件适配

* Where(适用场景)：

编写固定xml布局，在布局中使用

* How(如何生成)：

通过代码来生成适配文件，然后复制到values文件夹中，直接在xml布局中引用即可动态根据屏幕进行适配。

生成屏幕适配文件代码如下：

备注：由于mac linux windows路径不一致，在mac上可以把想存储的路径文件夹拖到终端上，就能看到对应的路径了。

```java
package t1;
 
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
 
public class sw {
 
	static int[] i = {1,2,3,4,5,6,7,8,9,10,12,14,15,16,18,20,22,24,25,26,28,30,32,35,36,38,
			40,42,45,48,50,55,58,60,65,68,70,72,75,78,80,82,85,86,90,95,98,100,110,120,125,130,
			135,136,140,142,145,146,150,156,160,170,180,190,200,210,220,230,240,250,256,260,
			270,280,290,300,320,340,350,360,370,380,390,400,420,450,460,480,500,510,520,550,
			580,600,610,620,630,640,650,660,670,680,700,800,820,850,880,900,920,950,960,980,
			1000,1024,1050,1080,1100,1200,1250,1300,1400,1500,1600,1800,1920};
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub
 
//		300,360,420,480,500,520,560,600,640,680,720,760,780,800,820,860,
//		900,960,1000,1080,1200,1400,1600,1920,2000
		
		int[] j = {300,360,420,480,500,520,560,600,640,680,720,760,780,800,820,860,
				900,960,1000,1080,1200,1400,1600,1920,2000};
		
		for(int m =0;m<j.length;m++){
			System.out.println(" \n适配大小"+j[m]+"\n");
			createFile(j[m]);
		}
	}
		private static void createFile(int n) {
			File file=new File("F:\\Android适配\\values-sw"+n+"dp");
			if(!file.exists()){//如果文件夹不存在
			file.mkdir();//创建文件夹
				}
			
			try{//异常处理
			
			BufferedWriter bw=new BufferedWriter(new FileWriter("F:\\Android适配\\values-sw"+n+"dp"+"\\dimens.xml"));
			bw.write("<resources>\n");
		
			for(int k =0;k<i.length;k++){
				System.out.println();
				bw.write("  <dimen name=\"dp"+i[k]+"\">"+Math.round(i[k]*(n/300.0))+"dp</dimen>\n");
			}
			bw.write("</resources>\n");
			bw.close();//一定要关闭文件
			}catch(IOException e){
				e.printStackTrace();
			}
				
		}	
}
 
```

相关blog链接：[🔗](https://blog.csdn.net/ink_s/article/details/81197667)