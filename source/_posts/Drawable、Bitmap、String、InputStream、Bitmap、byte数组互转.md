---
title: Drawable、Bitmap、String、InputStream、Bitmap、byte数组互转
toc: true
date: 2016-05-30 22:53:04
tags: 
- Bitmap
- Drawable
categories: Utils
---
```
对于Drawable、Bitmap、Canvas和Paint它们之间的概念不是很清楚，其实它们除了Drawable外早在Sun的J2ME中就已经出现了，但是在Android平台中，Bitmap、Canvas相关的都有所变化。首先让我们理解下Android平台中的显示类是View，但是还提供了底层图形类android.graphics，今天所说的这些均为graphics底层图形接口。 Bitmap - 称作位图，一般位图的文件格式后缀为bmp，当然编码器也有很多如RGB565、RGB888。作为一种逐像素的显示对象执行效率高，但是缺点也很明显存储效率低。我们理解为一种存储对象比较好。 Drawable - 作为Android平下通用的图形对象，它可以装载常用格式的图像，比如GIF、PNG、JPG，当然也支持BMP，当然还提供一些高级的可视化对象，比如渐变、图形等。 Canvas - 名为画布，我们可以看作是一种处理过程，使用各种方法来管理Bitmap、GL或者Path路径，同时它可以配合Matrix矩阵类给图像做旋转、缩放等操作，同时Canvas类还提供了裁剪、选取等操作。 Paint - 我们可以把它看做一个画图工具，比如画笔、画刷。他管理了每个画图工具的字体、颜色、样式。如果涉及一些Android游戏开发、显示特效可以通过这些底层图形类来高效实现自己的应用

```
<!-- more-->

Drawable转Bitmap

```
Resources res = getResources();
Drawable drawable = res.getDrawable(R.drawable.myimage);
BitmapDrawable bd = (BitmapDrawable) d;
Bitmap bm = bd.getBitmap();


public static Bitmap drawableToBitmap(Drawable drawable) {
Bitmap bitmap = Bitmap.createBitmap(
drawable.getIntrinsicWidth(),
drawable.getIntrinsicHeight(),
drawable.getOpacity() != PixelFormat.OPAQUE ?Bitmap.Config.ARGB_8888
: Bitmap.Config.RGB_565);
Canvas canvas = new Canvas(bitmap);
//canvas.setBitmap(bitmap);
drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
drawable.draw(canvas);
return bitmap;
}

```
- Bitmap转Drawable

```

Bitmap bm=xxx; //xxx根据你的情况获取
BitmapDrawable bd=BitmapDrawable(bm);
BtimapDrawable是Drawable的子类，最终直接使用bd对象即可。

mPicPath//本地图片路径转成Bitmap格式
Bitmap pic = BitmapFactory.decodeFile(this.mPicPath);
image.setImageBitmap(pic);
转成Bitmap格式

```

- String to InputStream

```

String str = "String与InputStream相互转换";
InputStream   in_nocode   =   new   ByteArrayInputStream(str.getBytes());   
InputStream   in_withcode   =   new   ByteArrayInputStream(str.getBytes("UTF-8"));

```

- InputStream to String

```
public String convertStreamToString(InputStream is) {
BufferedReader reader = new BufferedReader(new InputStreamReader(is));
StringBuilder sb = new StringBuilder();

String line = null;
try {
while ((line = reader.readLine()) != null) {
sb.append(line + "/n");
}
} catch (IOException e) {
e.printStackTrace();
} finally {
try {
is.close();
} catch (IOException e) {
e.printStackTrace();
}
}

return sb.toString();
}

---

public String inputStream2String (InputStream in) throws IOException {
StringBuffer out = new StringBuffer();
byte[] b = new byte[4096];
for (int n; (n = in.read(b)) != -1;) {
out.append(new String(b, 0, n));
}
return out.toString();
}

---

public static String inputStream2String(InputStream is) throws IOException{
ByteArrayOutputStream baos = new ByteArrayOutputStream();
int i=-1;
while((i=is.read())!=-1){
baos.write(i);
}
return baos.toString();
}

```

- Bitmap → byte[]

```

private byte[] Bitmap2Bytes(Bitmap bm){
ByteArrayOutputStream baos = new ByteArrayOutputStream();
bm.compress(Bitmap.CompressFormat.PNG, 100, baos);
return baos.toByteArray(); }

```

- byte[] → Bitmap

```
private Bitmap Bytes2Bimap(byte[] b){
if(b.length!=0){
return BitmapFactory.decodeByteArray(b, 0, b.length);
}
else {
return null;
}
}

```
