# Android图片压缩总结
#### 图片的存在形式
 - 文件形式(即以二进制形式存在于硬盘上)
 - 流的形式(即以二进制形式存在于内存中)
 - Bitmap形式

> 这三种形式的区别: 文件形式和流的形式对图片体积大小并没有影响,也就是说,如果你手机SD卡上的如果是100K,那么通过流的形式读到内存中,也一定是占100K的内存,注意是流的形式,不是Bitmap的形式,当图片以Bitmap的形式存在时,其占用的内存会瞬间变大, 我试过500K文件形式的图片加载到内存,以Bitmap形式存在时,占用内存将近10M,当然这个增大的倍数并不是固定的

#### 常见的压缩方式
 - 将图片保存到本地时进行压缩, 即将图片从Bitmap形式变为File形式时进行压缩

 **特点**:  File形式的图片确实被压缩了, 但是当你重新读取压缩后的file为 Bitmap是,它占用的内存并没有改变  

    public static void compressBmpToFile(Bitmap bmp,File file){
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        int options = 80;//个人喜欢从80开始,
        bmp.compress(Bitmap.CompressFormat.JPEG, options, baos);
        while (baos.toByteArray().length / 1024 > 100) {
            baos.reset();
            options -= 10;
            bmp.compress(Bitmap.CompressFormat.JPEG, options, baos);
        }
        try {
            FileOutputStream fos = new FileOutputStream(file);
            fos.write(baos.toByteArray());
            fos.flush();
            fos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

**方法说明: **该方法是压缩图片的质量, 注意它不会减少图片的像素,比方说, 你的图片是300K的, 1280*700像素的, 经过该方法压缩后, File形式的图片是在100以下, 以方便上传服务器, 但是你BitmapFactory.decodeFile到内存中,变成Bitmap时,它的像素仍然是1280*700, 计算图片像素的方法是 bitmap.getWidth()和bitmap.getHeight(), 图片是由像素组成的, 每个像素又包含什么呢? 熟悉PS的人知道, 图片是有色相,明度和饱和度构成的.

既然它是改变了图片的显示质量, 达到了对File形式的图片进行压缩, 图片的像素没有改变的话, 那重新读取经过压缩的file为Bitmap时, 它占用的内存并不会少;
因为: bitmap.getByteCount() 是计算它的像素所占用的内存, 


- 将图片从本地读到内存时,进行压缩 ,即图片从File形式变为Bitmap形式 
  **特点: **通过设置采样率, 减少图片的像素, 达到对内存中的Bitmap进行压缩

先看一个方法：

    private Bitmap compressBmpFromBmp(Bitmap image) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        int options = 100;
        image.compress(Bitmap.CompressFormat.JPEG, 100, baos);
        while (baos.toByteArray().length / 1024 > 100) {
            baos.reset();
            options -= 10;
            image.compress(Bitmap.CompressFormat.JPEG, options, baos);
        }
        ByteArrayInputStream isBm = newByteArrayInputStream(baos.toByteArray());
        Bitmap bitmap = BitmapFactory.decodeStream(isBm, null,null);
        return bitmap;
    }

该方法是对内存中的Bitmap进行质量上的压缩, 由上面的理论可以得出该方法是无效的, 而且也是没有必要的,因为你已经将它读到内存中了,再压缩多此一举, 尽管在获取系统相册图片时,某些手机会直接返回一个Bitmap,但是这种情况下, 返回的Bitmap都是经过压缩的, 它不可能直接返回一个原声的Bitmap形式的图片, 后果可想而知

再看另一个方法：

    private Bitmap compressImageFromFile(String srcPath) {
        BitmapFactory.Options newOpts = newBitmapFactory.Options();
        newOpts.inJustDecodeBounds = true;//只读边,不读内容
        Bitmap bitmap = BitmapFactory.decodeFile(srcPath, newOpts);
  
        newOpts.inJustDecodeBounds = false;
        int w = newOpts.outWidth;
        int h = newOpts.outHeight;
        float hh = 800f;//
        float ww = 480f;//
        int be = 1;
        if (w > h && w > ww) {
            be = (int) (newOpts.outWidth / ww);
        } else if (w < h && h > hh) {
            be = (int) (newOpts.outHeight / hh);
        }
        if (be <= 0)
            be = 1;
        newOpts.inSampleSize = be;//设置采样率
          
        newOpts.inPreferredConfig = Config.ARGB_8888;//该模式是默认的,可不设
        newOpts.inPurgeable = true;// 同时设置才会有效
        newOpts.inInputShareable = true;//。当系统内存不够时候图片自动被回收
          
        bitmap = BitmapFactory.decodeFile(srcPath, newOpts);
		//return compressBmpFromBmp(bitmap);//原来的方法调用了这个方法企图进行二次压缩，其实是无效的,大家尽管尝试
        return bitmap;
    }
该方法就是对Bitmap形式的图片进行压缩, 也就是通过设置采样率, 减少Bitmap的像素, 从而减少了它所占用的内存


#### 常见的几种压缩方法

- 质量压缩方法：

    private Bitmap compressImage(Bitmap image) {  
  
        ByteArrayOutputStream baos = new ByteArrayOutputStream();  
        image.compress(Bitmap.CompressFormat.JPEG, 100, baos);//质量压缩方法，这里100表示不压缩，把压缩后的数据存放到baos中  
        int options = 100;  
        while ( baos.toByteArray().length / 1024>100) {  //循环判断如果压缩后图片是否大于100kb,大于继续压缩         
            baos.reset();//重置baos即清空baos  
            image.compress(Bitmap.CompressFormat.JPEG, options, baos);//这里压缩options%，把压缩后的数据存放到baos中  
            options -= 10;//每次都减少10  
        }  
        ByteArrayInputStream isBm = new ByteArrayInputStream(baos.toByteArray());//把压缩后的数据baos存放到ByteArrayInputStream中  
        Bitmap bitmap = BitmapFactory.decodeStream(isBm, null, null);//把ByteArrayInputStream数据生成图片  
        return bitmap;  
    }  

- 图片按比例大小压缩方法（根据路径获取图片并压缩）：

    private Bitmap getimage(String srcPath) {  
        BitmapFactory.Options newOpts = new BitmapFactory.Options();  
        //开始读入图片，此时把options.inJustDecodeBounds 设回true了  
        newOpts.inJustDecodeBounds = true;  
        Bitmap bitmap = BitmapFactory.decodeFile(srcPath,newOpts);//此时返回bm为空  
          
        newOpts.inJustDecodeBounds = false;  
        int w = newOpts.outWidth;  
        int h = newOpts.outHeight;  
        //现在主流手机比较多是800*480分辨率，所以高和宽我们设置为  
        float hh = 800f;//这里设置高度为800f  
        float ww = 480f;//这里设置宽度为480f  
        //缩放比。由于是固定比例缩放，只用高或者宽其中一个数据进行计算即可  
        int be = 1;//be=1表示不缩放  
        if (w > h && w > ww) {//如果宽度大的话根据宽度固定大小缩放  
            be = (int) (newOpts.outWidth / ww);  
        } else if (w < h && h > hh) {//如果高度高的话根据宽度固定大小缩放  
            be = (int) (newOpts.outHeight / hh);  
        }  
        if (be <= 0)  
            be = 1;  
        newOpts.inSampleSize = be;//设置缩放比例  
        //重新读入图片，注意此时已经把options.inJustDecodeBounds 设回false了  
        bitmap = BitmapFactory.decodeFile(srcPath, newOpts);  
        return compressImage(bitmap);//压缩好比例大小后再进行质量压缩  
    }  
- 图片按比例大小压缩方法（根据Bitmap图片压缩）：

    private Bitmap comp(Bitmap image) {  
      
    ByteArrayOutputStream baos = new ByteArrayOutputStream();         
    image.compress(Bitmap.CompressFormat.JPEG, 100, baos);  
    if( baos.toByteArray().length / 1024>1024) {//判断如果图片大于1M,进行压缩避免在生成图片（BitmapFactory.decodeStream）时溢出    
        baos.reset();//重置baos即清空baos  
        image.compress(Bitmap.CompressFormat.JPEG, 50, baos);//这里压缩50%，把压缩后的数据存放到baos中  
    }  
    ByteArrayInputStream isBm = new ByteArrayInputStream(baos.toByteArray());  
    BitmapFactory.Options newOpts = new BitmapFactory.Options();  
    //开始读入图片，此时把options.inJustDecodeBounds 设回true了  
    newOpts.inJustDecodeBounds = true;  
    Bitmap bitmap = BitmapFactory.decodeStream(isBm, null, newOpts);  
    newOpts.inJustDecodeBounds = false;  
    int w = newOpts.outWidth;  
    int h = newOpts.outHeight;  
    //现在主流手机比较多是800*480分辨率，所以高和宽我们设置为  
    float hh = 800f;//这里设置高度为800f  
    float ww = 480f;//这里设置宽度为480f  
    //缩放比。由于是固定比例缩放，只用高或者宽其中一个数据进行计算即可  
    int be = 1;//be=1表示不缩放  
    if (w > h && w > ww) {//如果宽度大的话根据宽度固定大小缩放  
        be = (int) (newOpts.outWidth / ww);  
    } else if (w < h && h > hh) {//如果高度高的话根据宽度固定大小缩放  
        be = (int) (newOpts.outHeight / hh);  
    }  
    if (be <= 0)  
        be = 1;  
    newOpts.inSampleSize = be;//设置缩放比例  
    //重新读入图片，注意此时已经把options.inJustDecodeBounds 设回false了  
    isBm = new ByteArrayInputStream(baos.toByteArray());  
    bitmap = BitmapFactory.decodeStream(isBm, null, newOpts);  
    return compressImage(bitmap);//压缩好比例大小后再进行质量压缩  
	}  



