# MIXTURE
## 杂七杂八  
### 动态获取一段文字的高度  
需要获取文字的@1字体大小@2每一行文字的限定宽度  
cell高度缓存 用来缓存cell的高度,key:模型 value：cell的高度 ------->key不可以使用“行号”，因为一旦刷新出现新数据就会出现错误  
@property(nonatomic, strong) NSMutableDictionary *cellHeightDicM;  
`- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath`  
调用次数频繁。--->当前有多少条数据调用多少次+屏幕有多少条数据调用多少次  
