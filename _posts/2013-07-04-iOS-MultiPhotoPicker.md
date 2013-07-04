---
layout: blog
excerpt: iphone中的一个多图选取组件
---

5月份开始转岗做iOS手机App开发，第一个工作便是做一个相册+拍照的多图选取工具。  
淘宝主客户端需要加入退款的功能，而退款很重要的一个需求点便是需要拍照上传凭证。  

接手这个需求，我们希望是把它做成组件化，凡是需要选取照片的地方，都可以通过这个组件来选取。   
把组件化作为目标，在开发的过程中，我也尽量保证这个组件的独立性与封装性。  

下面放两张截图照片吧：  

<img src="http://xuhengfei.com/assets/images/multiphotopicker/snapshot1.jpg" width="320px" height="480px"/>
&nbsp;&nbsp;
<img src="http://xuhengfei.com/assets/images/multiphotopicker/snapshot2.jpg" width="320px" height="480px"/>

分别是拍照界面和相册选取界面。  

这个组件的使用方式也非常简单：

```objective-c

-(void)viewDidLoad{
  //设置临时文件夹，用来保存用户选择的图片(也可以不设置，有默认值)
  [XHFMultiPhotoPicker setLocalCacheFolder:@"path"];
  //弹出照片选择组件，有4个参数，分别是：
  //Type:USER_SELECT 表示用户自己来选择拍照还是相册 (也可以使用其他2个参数：ALBUM  CAMERA)
  //InitPhotos:self.photos 用户已经选中的照片(传入nil表示还没有选中的图片)
  //ViewController: 当前的UIViewController
  //ResultBlock: 用户选择完成后，返回选中的照片数组
  [XHFMultiPhotoPicker pickWithType:USER_SELECT InitPhotos:nil ViewController:self ResultBlock:^(NSArray *photos){
    //返回用户选中的照片Array
    for(XHFSelectPhoto *p in photos){
      NSLog(@"photo local path:%@",p.localPath);
    }
    //do something
    //清除临时照片文件
    [XHFMultiPhotoPicker clearLocalImages];
  }];
  
}

```

坦白说，这不是一个完善的组件。  
有些贴图还没有处理，视觉与交互也有不少待完善的地方。  

淘宝App中的这个组件已经上线，不少地方做了改进。我这边没有将淘宝App中最新的代码拿过来，主要是出于2个原因：  
1.淘宝App中的多图选取组件，毕竟是属于淘宝的，拿出来不太合适  
2.淘宝App中的多图选取组件，其实也是有很多交互的地方需要改进的，包含有些地方，我自己也不认同，这更加促使我不想把那些代码拿过来了  

但是我倒也不完全觉得目前这个组件完全没用处吧。  
如果有人也有同样的需求，欢迎来使用这个组件，当然那些待完善的地方，你可以自己补齐。  
无论如何，总比自己从头开始来的轻松的多把。  
如果你有更大的善意，完善之后愿意对这个组件贡献一点代码。那我不胜感激了。  

最后，附上源代码地址：[https://github.com/xuhengfei/iOS-MultiPhotoPicker](https://github.com/xuhengfei/iOS-MultiPhotoPicker)

