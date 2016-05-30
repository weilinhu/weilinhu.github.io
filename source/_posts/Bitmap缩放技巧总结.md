---
title: Bitmap缩放技巧总结
toc: true
date: 2016-05-30 22:33:03
tags: Bitmap
categories: 性能优化
---
1.使用BitmapFactory.Options来控制缩放比例，降低图片色彩模式
```
BitmapFactory.Options options = new BitmapFactory.Options();
  options.inJustDecodeBounds = true;
  // 获取这个图片的宽和高
  Bitmap bitmap = BitmapFactory.decodeFile(path, options); // 此时返回bm为空
  options.inJustDecodeBounds = false;
  // 计算缩放比
  int be = (int) (options.outHeight / (float) 200);
  if (be <= 0)
   be = 1;
  options.inSampleSize = 2; // 图片长宽各缩小二分之一
  //每个像素占据2个字节
  options.inPreferredConfig = Bitmap.Config.RGB_565;
  // 重新读入图片，注意这次要把options.inJustDecodeBounds 设为 false哦
  bitmap = BitmapFactory.decodeFile(path, options);
由于只是对bitmap加载到内存一次，所以效率比较高。解析速度快
```
<!-- more-->
2.Bitmap+Matrix
```
public static Bitmap zoomBitmap(Bitmap bitmap, int width, int height) {
        if (bitmap == null) {
            return null;
        }
    int w = bitmap.getWidth();
        int h = bitmap.getHeight();
        Matrix matrix = new Matrix();
        float scaleWidth = ((float) width / w);
        float scaleHeight = ((float) height / h);
        matrix.postScale(scaleWidth, scaleHeight);
        Bitmap newbmp = Bitmap.createBitmap(bitmap, 0, 0, w, h, matrix, true);
        return newbmp;
    }
 ```
 
> 对于Drawable、Bitmap、Canvas和Paint它们之间的概念不是很清楚，其实它们除了Drawable外早在Sun的J2ME中就已经出现了，但是在Android平台中，Bitmap、Canvas相关的都有所变化。
  首先让我们理解下Android平台中的显示类是View，但是还提供了底层图形类android.graphics，今天所说的这些均为graphics底层图形接口。
  Bitmap - 称作位图，一般位图的文件格式后缀为bmp，当然编码器也有很多如RGB565、RGB888。作为一种逐像素的显示对象执行效率高，但是缺点也很明显存储效率低。我们理解为一种存储对象比较好。
  Drawable - 作为Android平下通用的图形对象，它可以装载常用格式的图像，比如GIF、PNG、JPG，当然也支持BMP，当然还提供一些高级的可视化对象，比如渐变、图形等。
  Canvas - 名为画布，我们可以看作是一种处理过程，使用各种方法来管理Bitmap、GL或者Path路径，同时它可以配合Matrix矩阵类给图像做旋转、缩放等操作，同时Canvas类还提供了裁剪、选取等操作。
   Paint - 我们可以把它看做一个画图工具，比如画笔、画刷。他管理了每个画图工具的字体、颜色、样式。
  如果涉及一些Android游戏开发、显示特效可以通过这些底层图形类来高效实现自己的应用。
  
  


