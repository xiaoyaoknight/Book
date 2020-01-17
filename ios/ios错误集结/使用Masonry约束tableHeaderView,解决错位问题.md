# 使用Masonry约束tableHeaderView,解决错位问题

```
[self.tableView beginUpdates];
[self.tableView setTableHeaderView:myTableHeaderView];
[self.tableView endUpdates];
 
//下面这部分很关键,重新布局获取最新的frame,然后赋值给myTableHeaderView
[myTableHeaderView setNeedsLayout];
[myTableHeaderView layoutIfNeeded];
CGSize size = [myTableHeaderView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize];
CGRect headerFrame = myTableHeaderView.frame;
headerFrame.size.height = size.height;
myTableHeaderView.frame = headerFrame;
 
self.tableView.tableHeaderView = myTableHeaderView;
```
网上基本全是这一个回答，来回转载。
但是我自己的代码试了不行。


##解决方案：
```
- (void)setHeaderView {
    self.tableView.tableHeaderView = self.headerView;
    [self.headerView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.mas_equalTo(self.tableView);
    }];
}
```
或者如参考[iOS开发 masonry 设置tableHeadView](http://www.bubuko.com/infodetail-1326470.html)中所述：包一层view效果一样。

```
  UIView *headView = [[UIView alloc] initWithFrame:CGRectMake(0, 0, 320, 1)];
    
  ContainerView *con = [[ContainerView alloc] init];
  con.backgroundColor = [UIColor blackColor];
  [headView addSubview:con];
  [con mas_makeConstraints:^(MASConstraintMaker *make) {
         make.edges.equalTo(headView);
  }];

    CGFloat height = [headView systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height;
    CGRect frame = headView.frame;
    frame.size.height = height;
    headView.frame = frame;
    tableView.tableHeaderView = headView;
```