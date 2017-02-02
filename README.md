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
