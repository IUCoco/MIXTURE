# MIXTURE
## 杂七杂八  
### 动态获取一段文字的高度  
需要获取文字的@1字体大小@2每一行文字的限定宽度  
### cell高度缓存  
用来缓存cell的高度,key:模型 value：cell的高度 ------->key不可以使用“行号”，因为一旦刷新出现新数据就会出现错误  
@property(nonatomic, strong) NSMutableDictionary *cellHeightDicM;  
`- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath`  
调用次数频繁。--->当前有多少条数据调用多少次+屏幕有多少条数据调用多少次  
@1 使用模型的内存地址作为key，因为key必须转手NSCopying协议，因为每个模型没有准守NSCopying协议，所以使用模型内存地址拼接成字符串（字符串遵守了NSCopying协议）  
```
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    CZQTopic *topic = self.topicArrM[indexPath.row];
    //使用模型的内存地址作为key，因为key必须转手NSCopying协议，因为每个模型没有准守NSCopying协议，所以使用模型内存地址拼接成字符串（字符串遵守了NSCopying协议）
    NSString *cellHeightKey = [NSString stringWithFormat:@"%p", topic];
    CGFloat cellHeight = [self.cellHeightDicM[cellHeightKey] doubleValue];
    if (cellHeight == 0) {
        //文字的Y值
        cellHeight += 55;
        //文字的高度
        CGSize textMaxSize = CGSizeMake(CZQScreenWith - 20, MAXFLOAT);
        //    cellHeight += [topic.text sizeWithFont:[UIFont systemFontOfSize:15] constrainedToSize:textMaxSize].height + 10;
        NSMutableDictionary *attributesDic = [NSMutableDictionary dictionary];
        attributesDic[NSFontAttributeName] = [UIFont systemFontOfSize:15];
        cellHeight += [topic.text boundingRectWithSize:textMaxSize options:NSStringDrawingUsesLineFragmentOrigin attributes:attributesDic context:nil].size.height + 10;
        //工具条
        cellHeight += 35 + 10;
        
        //存储高度
        self.cellHeightDicM[cellHeightKey] = @(cellHeight);
    }
    
    return cellHeight;
}
```
当然在下拉刷新数据的时候应该移除字典中所有缓存的cell高度  
//移除字典中缓存的cell高度  
`[self.cellHeightDicM removeAllObjects];`  
@2 在模型中存储cell的高度  
@3前面说过--->当前有多少条数据调用多少次+屏幕有多少条数据调用多少次因为开始要精确算出滚动条的大小，如果使用估算  
`- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath`或者
`@property (nonatomic) CGFloat estimatedRowHeight`属性  
`- (CGFloat)tableView:(UITableView *)tableView estimatedHeightForRowAtIndexPath:(NSIndexPath *)indexPath`还可以再次优化  


### AFN网络监控  
一般在`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`代理方法里面添加网络监控，manager只会创建一次，这样性能也会好些。  
```

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    //1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    //2.设置窗口根控制器
//    CZQTabBarController *winRootVC = [[CZQTabBarController alloc] init];
//    self.window.rootViewController = winRootVC;
    //2.设置窗口的根控制器为广告控制器
    CZQADViewController *adVC = [[CZQADViewController alloc] init];
    //init底层调用 initWithNib
    self.window.rootViewController = adVC;
    //3.设置为application的主窗口并且显示
    [self.window makeKeyAndVisible];
    
    //监控网络状态
    [[AFNetworkReachabilityManager sharedManager] startMonitoring];
    
    return YES;
}

```
//沙盒会先查看内存再查看沙盒  
```
- (nullable UIImage *)imageFromDiskCacheForKey:(nullable NSString *)key {
    UIImage *diskImage = [self diskImageForKey:key];
    if (diskImage && self.config.shouldCacheImagesInMemory) {
        NSUInteger cost = SDCacheCostForImage(diskImage);
        [self.memCache setObject:diskImage forKey:key cost:cost];
    }

    return diskImage;
}
```
```
//根据网络状况加载不同质量的图片
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager sharedManager];
    //查看沙盒里面是否有原始高质量图片
    //SDWebImage----->一个URL（key）对应一个图片
    UIImage *originalImage = [[SDImageCache sharedImageCache] imageFromDiskCacheForKey:topic.image1];//沙盒会先查看内存再查看沙盒
    if (originalImage) {//以前已经下载过了
        self.imageView.image = originalImage;
    }else{//以前没有下载过
        //根据网络情况判断下载哪种质量图片
        if (manager.isReachableViaWWAN) {//移动蜂窝网络
          [self.imageView sd_setImageWithURL:[NSURL URLWithString:topic.image2]];
        }else if (manager.isReachableViaWiFi){//WiFi
          [self.imageView sd_setImageWithURL:[NSURL URLWithString:topic.image1]];
        }
    }

```



初始版本0.0.1

修改bug阶段—>0.0.2

重大变更阶段—>0.1.0



Xcode中Version与Build区别

- Version（应用程序发布版本号）
- Build（应用程序内部标示）



- 正确选择图片加载方式能够对内存优化起到很大的作用，常见的图片加载方式有下面三种：

1. //方法1  
2. UIImage *imag1 = [UIImage imageNamed:@"image.png"];  
3. //方法2  
4. UIImage *image2 = [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image.png" ofType:nil]];  
5. //方法3  
6. NSData *imageData = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image.png" ofType:nil]];  
7. UIImage *image3 = [UIImage imageWithData:imageData];  

- 第一种方法:imageNamed:
  
  为什么有两种方法完成同样的事情呢？imageNamed的优点在于可以缓存已经加载的图片。苹果的文档中有如下说法：
  This method looks in the system caches for an image object with the specified name and returns that object if it exists. If a matching image object is not already in the cache, this method loads the image data from the specified file, caches it, and then returns the resulting object.
  这种方法会首先在系统缓存中根据指定的名字寻找图片，如果找到了就返回。如果没有在缓存中找到图片，该方法会从指定的文件中加载图片数据，并将其缓存起来，然后再把结果返回。对于同一个图像，系统只会把它Cache到内存一次，这对于图像的重复利用是非常有优势的。例如：你需要在 一个TableView里重复加载同样一个图标，那么用imageNamed加载图像，系统会把那个图标Cache到内存，在Table里每次利用那个图 像的时候，只会把图片指针指向同一块内存。这种情况使用imageNamed加载图像就会变得非常有效。

- 第二种方法和第三种方法本质是一样的:imageWithContentsOfFile:和imageWithData:

而imageWithContentsOfFile方法只是简单的加载图片，并不会将图片缓存起来，图像会被系统以数据方式加载到程序。当你不需要重用该图像，或者你需要将图像以数据方式存储到数据库，又或者你要通过网络下载一个很大的图像时，可以使用这种方式。

- 如何选择

1. //方法1 cach  
2. UIImage *imag1 = [UIImage imageNamed:@"image.png"];  
3. //方法2 no cach  
4. UIImage *image2 = [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"image.png" ofType:nil]];  

如果加载一张很大的图片，并且只使用一次，那么就不需要缓存这个图片。这种情况imageWithContentsOfFile比较合适,系统不会浪费内存来缓存图片。

然而，如果在程序中经常需要重用的图片，那么最好是选择imageNamed方法。这种方法可以节省出每次都从磁盘加载图片的时间。

