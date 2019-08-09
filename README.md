####先看一下效果图



![如图.gif](https://upload-images.jianshu.io/upload_images/3404974-c78005e903d379fb.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

**介绍一下：就是长按圆圈部位，进度条出现，当你长按的时间到达规定时间（时间可自己设置），完成当前操作。**
### 共有两个难点
>1、按压开始，进度条开始运动
>2、进度条的渐变

#### 一、界面绘制
1、自定义View，LHButton
2、LHButton上添加长按手势
3、添加timer,每次timer结束绘制一次界面
4、添加CAShapeLayer 加入path路径 根据它的**strokeEnd**属性来绘制

`@property (nonatomic, assign) CGFloat progress;`
`@property (nonatomic, strong) NSTimer *displayline;`
/** 进度条图层 */
`@property (nonatomic, strong) CAShapeLayer *progressLayer;`
```
-(CAShapeLayer *)progressLayer{
    if (_progressLayer == nil) {
        _progressLayer = [CAShapeLayer layer];
        _progressLayer.fillColor = [UIColor clearColor].CGColor;
        _progressLayer.strokeColor = KWhiteColor.CGColor;
        _progressLayer.lineWidth = 0;
        _progressLayer.strokeEnd = 0;
        _progressLayer.lineCap = kCALineCapRound;
        CGRect progressFrame = CGRectMake(3, 3, self.bounds.size.width-6, self.bounds.size.width-6);
        UIBezierPath *progressPath = [UIBezierPath bezierPathWithRoundedRect:progressFrame cornerRadius:progressFrame.size.height/2];
        _progressLayer.path = progressPath.CGPath;
    }
    return _progressLayer;
}
```

```
-(NSTimer *)displayline{
    if (_displayline == nil) {
        _displayline = [NSTimer timerWithTimeInterval:0.1 target:self selector:@selector(lineRun) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop]addTimer:_displayline forMode:NSRunLoopCommonModes];
        [self.displayline setFireDate:[NSDate distantFuture]];
    }
    return  _displayline;
}
```
```
-(void)lineRun{
    self.temInterval = self.temInterval+1/10.0;
    self.progress = self.temInterval/self.interval;
//    NSLog(@"%.2f",self.progress);
    if (self.temInterval >= self.interval) {
        [self stop];
        self.actionState(LHProgressButtonStateEnd);
    }
    [self setNeedsDisplay];
}
```
#### 二、进度条渐变的问题
*刚开始是想通过layer的strokeColor来设置，但是发现strokeColor只能设置一次，无法达到渐变效果，然后就改变思路，通过蒙版来实现*
```
-(UIView *)gradientBackgroundView{
    if (_gradientBackgroundView == nil) {
        
        // 渐变背景视图（不包含坐标轴）
       _gradientBackgroundView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, self.bounds.size.width, self.bounds.size.height)];
        /** 创建并设置渐变背景图层 */
        //初始化CAGradientlayer对象，使它的大小为渐变背景视图的大小
        self.gradientLayer = [CAGradientLayer layer];
        self.gradientLayer.frame = self.gradientBackgroundView.bounds;
        //设置渐变区域的起始和终止位置（范围为0-1），即渐变路径
        self.gradientLayer.startPoint = CGPointMake(0, 0.0);
        self.gradientLayer.endPoint = CGPointMake(0.0, 1.0);
        //设置颜色的渐变过程
        self.gradientLayerColors = [NSMutableArray arrayWithArray:@[(__bridge id)[UIColor colorWithRed:253 / 255.0 green:164 / 255.0 blue:8 / 255.0 alpha:1.0].CGColor, (__bridge id)[UIColor colorWithRed:251 / 255.0 green:37 / 255.0 blue:45 / 255.0 alpha:1.0].CGColor]];
        self.gradientLayer.colors = self.gradientLayerColors;
        //将CAGradientlayer对象添加在我们要设置背景色的视图的layer层
        [_gradientBackgroundView.layer addSublayer:self.gradientLayer];
    }
    return _gradientBackgroundView;
}
```
然后上蒙版
```
 // 设置折线图层为渐变图层的mask
    self.gradientBackgroundView.layer.mask = self.progressLayer;
```
###三、到这就基本差不多了
![如图.gif](https://upload-images.jianshu.io/upload_images/4881197-1edd5b2c5b0da811.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)
>最后奉上热乎的demo

[文件Demo]()
