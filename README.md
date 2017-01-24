# TableViewMutableSelectDemo
UITableView多选Demo
好久没写博客了，写些平常用的到的东西吧。一些列表经常需要编辑多选的功能，而UITableview自带多选删除的功能，使用起来方便，不需要自己去做数据存储和选中状态转换，可以减少不少开发时间。下面就来介绍下UITableview多选的使用。
效果 ： 

![123.gif](http://upload-images.jianshu.io/upload_images/2106071-9f5eae75dc6cd0a8.gif?imageMogr2/auto-orient/strip)

#####UITableViewCellEditingStyle
编辑状态UITableViewCellEditingStyle有三种模式：
* UITableViewCellEditingStyleDelete
* UITableViewCellEditingStyleInsert
* UITableViewCellEditingStyleNone

多选框的风格, 只需要风格同时包含UITableViewCellEditingStyleDelete和UITableViewCellEditingStyleInsert就可以了

````
- (UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath{
return UITableViewCellEditingStyleDelete | UITableViewCellEditingStyleInsert;
}
````
然后设置UITableview的``Editing``状态，就可开启tableview的多选功能。
####使用自带的编辑状态去做多选功能的好处
* 不用去记录数据源中选中的数组，通过``indexPathsForSelectedRows``方法就可以获取到选中的cell,做删除操作时也会方便很多.
* 进入退出编辑状态方便，还有自带的动画。ps:自定义cell时，子视图应加载在cell的contentView上。
* 在要求不高的情况下，自带的图标就能满足需求。


![11111.png](http://upload-images.jianshu.io/upload_images/2106071-300d98087e1bbe1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####接下来先说说全选和全不选的操作,以下都是在单section情况下
* 做全选也简单，遍历一遍数据源 然后选中每一条。
````
[self.listData enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
[self.tableView selectRowAtIndexPath:[NSIndexPath indexPathForRow:idx inSection:0] animated:NO scrollPosition:UITableViewScrollPositionNone];
}];
````
* 全不选，reload就可以变为全不选状态了，如果不想reload这样简单粗暴的，也可以取出当前选中的indexpath数组，遍历反选也可以。
````
[self.tableView reloadData];
/** 遍历反选
[[self.tableView indexPathsForSelectedRows] enumerateObjectsUsingBlock:^(NSIndexPath * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
[self.tableView deselectRowAtIndexPath:obj animated:NO];
}];
*/
````

* 多section情况类似，在删除操作的时候注意点，删除到section的cell到最后一个时 需要删除整个section。


#####编辑操作,下面以删除为例

````
NSMutableIndexSet *insets = [[NSMutableIndexSet alloc] init];
[[self.tableView indexPathsForSelectedRows] enumerateObjectsUsingBlock:^(NSIndexPath * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
[insets addIndex:obj.row];
}];
[self.listData removeObjectsAtIndexes:insets];
[self.tableView deleteRowsAtIndexPaths:[self.tableView indexPathsForSelectedRows] withRowAnimation:UITableViewRowAnimationFade];

/** 数据清空情况下取消编辑状态*/
if (self.listData.count == 0) {
self.navigationItem.rightBarButtonItem.title = @"编辑";
[self.tableView setEditing:NO animated:YES];
[self showEitingView:NO];
/** 带MJ刷新控件重置状态
[self.tableView.footer resetNoMoreData];
[self.tableView reloadData];
*/
}
````


#####Cell点击效果的问题，有的人会觉得那个蓝色的背景很难看，想去掉,可以在自定义的cell里面：
````
self.multipleSelectionBackgroundView = [UIView new];
````

还可以修改有点击选中图标的颜色,其他默认图片的颜色也会因此而修改，
````
self.tintColor = [UIColor redColor];
````

效果:
![12222.png](http://upload-images.jianshu.io/upload_images/2106071-eec0f036c0786ae6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果不想使用默认图标的话，也可以在自定义：
````
-(void)layoutSubviews
{
for (UIControl *control in self.subviews){
if ([control isMemberOfClass:NSClassFromString(@"UITableViewCellEditControl")]){
for (UIView *v in control.subviews)
{
if ([v isKindOfClass: [UIImageView class]]) {
UIImageView *img=(UIImageView *)v;
if (self.selected) {
img.image=[UIImage imageNamed:@"xuanzhong_icon"];
}else
{
img.image=[UIImage imageNamed:@"weixuanzhong_icon"];
}
}
}
}
}
[super layoutSubviews];
}


//适配第一次图片为空的情况
- (void)setEditing:(BOOL)editing animated:(BOOL)animated
{
[super setEditing:editing animated:animated];
for (UIControl *control in self.subviews){
if ([control isMemberOfClass:NSClassFromString(@"UITableViewCellEditControl")]){
for (UIView *v in control.subviews)
{
if ([v isKindOfClass: [UIImageView class]]) {
UIImageView *img=(UIImageView *)v;
if (!self.selected) {
img.image=[UIImage imageNamed:@"weixuanzhong_icon"];
}
}
}
}
}

}
````

效果：
![13333.png](http://upload-images.jianshu.io/upload_images/2106071-a50c3c9de8885b47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

demo地址：[https://github.com/yxsufaniOS/TableViewMutableSelectDemo](https://github.com/yxsufaniOS/TableViewMutableSelectDemo)
