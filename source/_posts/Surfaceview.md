---
title: Surfaceview
toc: true
date: 2016-05-30 22:56:56
tags: View
categories: View
---
# Surfaceview 
- SurfaceHolder

```
  SurfaceHolder.Callback主要是当底层的Surface被创建、销毁或者改变时提供回调通知，由于绘制必须在Surface被创建后才能进行，因此SurfaceHolder.Callback中的surfaceCreated 和surfaceDestroyed 就成了绘图处理代码的边界。
 
        SurfaceHolder.Callback中定义了三个接口方法：
 
        1、abstract void surfaceChanged(SurfaceHolder holder, int format, int width, int height)：当surface发生任何结构性的变化时（格式或者大小），该方法就会被立即调用。
 
        2、abstract void surfaceCreated(SurfaceHolder holder)：当surface对象创建后，该方法就会被立即调用。
 
        3、abstract void  surfaceDestroyed(SurfaceHolder holder)：当surface对象在将要销毁前，该方法会被立即调

```

<!-- more-->
- setFixSize

```
setFixSize 不是设置SurfaceView的大小，而是告诉系统真     实的 video Size的大小。
改变SurfaceView大小，就是改变SurfaceView layout的大小surfaceView里面有两个大小。一个是surface的大小，在sur  facechanged里面输出width和height的值来查看；还有一个就    是surfaceView本身的大小，重写onMeasure函数可以得到。具    体看下面的代码。而且，视频播放只与surfaceView的大小有关。
 ```


- Surfaceview 简单应用

```
 1、在Activity的OnCreate函数中设置好SurfaceView，包括设置SurfaceHolder.Callback对象和SurfaceHolder对象的类型，具体如下
SurfaceView mpreview = (SurfaceView) this.findViewById(R.id.camera_preview);
SurfaceHolder mSurfaceHolder = mpreview.getHolder();
mSurfaceHolder.addCallback(this);

	1. //为了实现照片预览功能，需要将SurfaceHolder的类型设置为PUSH  
	2.         //这样，画图缓存就由Camera类来管理，画图缓存是独立于Surface的

mSurfaceHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);

2、在SurfaceHolder.Callback的surfaceCreated函数中，使用Camera的Open函数开机摄像头硬件，这个API在SDK 2.3之前，是没有参数的，2.3以后支持多摄像头，所以开启前可以通过getNumberOfCameras先获取摄像头数目，再通过getCameraInfo得到需要开启的摄像头id，然后传入Open函数开启摄像头，假如摄像头开启成功则返回一个Camera对象，否则就抛出异常；

3、开启成功的情况下，在SurfaceHolder.Callback的surfaceChanged函数中调用getParameters函数得到已打开的摄像头的配置参数Parameters对象，如果有需要就修改对象的参数，然后调用setParameters函数设置进去（SDK2.2以后，还可以通过Camera：：setDisplayOrientation设置方向）

4、同样在surfaceChanged函数中，通过Camera：：setPreviewDisplay为摄像头设置SurfaceHolder对象，设置成功后调用Camera::startPreview函数开启预览功能，上面3,4两步的代码可以如下所示
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h)
{
//已经获得Surface的width和height，设置Camera的参数
Camera.Parameters parameters = camera.getParameters();
parameters.setPreviewSize(w, h);
List<Size> vSizeList = .getSupportedPictureSizes();
for(int num = 0; num < vSizeList.size(); num++)
{
Size vSize = vSizeList.get(num);
}
if(this.getResources().getConfiguration().orientation != Configuration.ORIENTATION_LANDSCAPE)
{
//如果是竖屏
parameters.set("orientation", "portrait");
//在2.2以上可以使用
//camera.setDisplayOrientation(90);
}
else
{
parameters.set("orientation", "landscape");
//在2.2以上可以使用
//camera.setDisplayOrientation(0);
}
camera.setParameters(parameters);
try {
//设置显示
camera.setPreviewDisplay(holder);
} catch (IOException exception) {
camera.release();
camera = null;
}
//开始预览
camera.startPreview();
}

---


	 // 当Surface被创建的时候，该方法被调用，可以在这里实例化Camera对象  
	         //同时可以对Camera进行定制  
	         camera = Camera.open(); //获取Camera实例  


	  /** 
	          * Camera对象中含有一个内部类Camera.Parameters.该类可以对Camera的特性进行定制 
	          * 在Parameters中设置完成后，需要调用Camera.setParameters()方法，相应的设置才会生效 
	          * 由于不同的设备，Camera的特性是不同的，所以在设置时，需要首先判断设备对应的特性，再加以设置 
	          * 比如在调用setEffects之前最好先调用getSupportedColorEffects。如果设备不支持颜色特性，那么该方法将 
	          * 返回一个null 
	          */  
	         try {  
	               
	             Camera.Parameters param = camera.getParameters();  
	             if(this.getResources().getConfiguration().orientation != Configuration.ORIENTATION_LANDSCAPE){  
	                 //如果是竖屏  
	                 param.set("orientation", "portrait");  
	                 //在2.2以上可以使用  
	                 //camera.setDisplayOrientation(90);  
	             }else{  
	                 param.set("orientation", "landscape");  
	                //在2.2以上可以使用  
	                 //camera.setDisplayOrientation(0);                
	             }  
	             //首先获取系统设备支持的所有颜色特效，有复合我们的，则设置；否则不设置  
	            List<String> colorEffects = param.getSupportedColorEffects();  
	             Iterator<String> colorItor = colorEffects.iterator();  
	             while(colorItor.hasNext()){  
	                 String currColor = colorItor.next();  
	                 if(currColor.equals(Camera.Parameters.EFFECT_SOLARIZE)){  
	                     param.setColorEffect(Camera.Parameters.EFFECT_SOLARIZE);  
	                     break;  
	                }  
	             }  
	             //设置完成需要再次调用setParameter方法才能生效  
	             camera.setParameters(param);  
	               
	            camera.setPreviewDisplay(holder);  
	               
            /** 
	              * 在显示了预览后，我们有时候希望限制预览的Size 
	             * 我们并不是自己指定一个SIze而是指定一个Size，然后 
	              * 获取系统支持的SIZE，然后选择一个比指定SIZE小且最接近所指定SIZE的一个 
	              * Camera.Size对象就是该SIZE。 
	             *  
	            */  
	             int bestWidth = 0;  
	             int bestHeight = 0;  
	               
	             List<Camera.Size> sizeList = param.getSupportedPreviewSizes();  
	             //如果sizeList只有一个我们也没有必要做什么了，因为就他一个别无选择  
	             if(sizeList.size() > 1){  
	                 Iterator<Camera.Size> itor = sizeList.iterator();  
	                 while(itor.hasNext()){  
	                     Camera.Size cur = itor.next();  
	                     if(cur.width > bestWidth && cur.height>bestHeight && cur.width <MAX_WIDTH && cur.height < MAX_HEIGHT){  
	                         bestWidth = cur.width;  
	                         bestHeight = cur.height;  
	                    }  
	                 }  
	                 if(bestWidth != 0 && bestHeight != 0){  
	                     param.setPreviewSize(bestWidth, bestHeight);  
	                    //这里改变了SIze后，我们还要告诉SurfaceView，否则，Surface将不会改变大小，进入Camera的图像将质量很差  
	                    surfaceView.setLayoutParams(new LinearLayout.LayoutParams(bestWidth, bestHeight));  
	                }  
	             }  
	             camera.setParameters(param);  
	         } catch (Exception e) {  
	             // 如果出现异常，则释放Camera对象  
	             camera.release();  
	         }  
	           
	        //启动预览功能  
	         camera.startPreview();  


	1.  // 当Surface被销毁的时候，该方法被调用  
	2.         //在这里需要释放Camera资源  
	3.         camera.stopPreview();  
	4.         camera.release();  

---

5、假设要支持自动对焦功能，则在需要的情况下，或者在上述surfaceChanged调用完startPreview函数后，可以调用Camera::autoFocus函数来设置自动对焦回调函数，该步是可选操作，有些设备可能不支持，可以通过Camera::getFocusMode函数查询。代码可以参考如下：
// 自动对焦
camera.autoFocus(new AutoFocusCallback()
{
@Override
public void onAutoFocus(boolean success, Camera camera)
{
if (success)
{
// success为true表示对焦成功，改变对焦状态图像
ivFocus.setImageResource(R.drawable.focus2);
}
}
});


6、在需要拍照的时候，调用takePicture(Camera.ShutterCallback, Camera.PictureCallback, Camera.PictureCallback, Camera.PictureCallback)函数来完成拍照，这个函数中可以四个回调接口，ShutterCallback是快门按下的回调，在这里我们可以设置播放“咔嚓”声之类的操作，后面有三个PictureCallback接口，分别对应三份图像数据，分别是原始图像、缩放和压缩图像和JPG图像，图像数据可以在PictureCallback接口的void onPictureTaken(byte[] data, Camera camera)中获得，三份数据相应的三个回调正好按照参数顺序调用，通常我们只关心JPG图像数据，此时前面两个PictureCallback接口参数可以直接传null；

7、每次调用takePicture获取图像后，摄像头会停止预览，假如需要继续拍照，则我们需要在上面的PictureCallback的onPictureTaken函数末尾，再次掉哟更Camera::startPreview函数；

8、在不需要拍照的时候，我们需要主动调用Camera::stopPreview函数停止预览功能，并且调用Camera::release函数释放Camera，以便其他应用程序调用。SDK中建议放在Activity的Pause函数中，但是我觉得放在surfaceDestroyed函数中更好，示例代码如下
// 停止拍照时调用该方法
public void surfaceDestroyed(SurfaceHolder holder)
{
// 释放手机摄像头
camera.release();
}
以上就是自己实现拍照程序的的流程，一般还可以还可以获取预览帧的图像数据，可以分别通过Camera::setPreviewCallback和Camera::setOneShotPreviewCallback来设置每帧或下一帧图像数据的回调，这里就不做展开了。

---

/** A safe way to get an instance of the Camera object. */
public static Camera getCameraInstance(){
    Camera c = null;
    try {
        c = Camera.open(); // attempt to get a Camera instance
    }
    catch (Exception e){
        // Camera is not available (in use or does not exist)
    }
    return c; // returns null if camera is unavailable
}

---

```