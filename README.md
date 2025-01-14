
前言：我在迁移旧项目代码的时候发现别人写很多界面都涉及到一个DataGrid的全选，但是每个都写的很混乱，现在刚好空闲下来，写一个博客，


给部分可能不太会写这个的同学讲一下，怎么实现全选功能，并且可以在任何项目里面复用这个功能。


先准备一个Datagrid，我们给这个DataGrid取名为 dg1。




```
        "False" 
                  CanUserAddRows="False"
                  ItemsSource="{Binding Path=.}"
                  x:Name="dg1" Height="200">
        
```


再准备一个实体类，并且给这个类添加属性变更通知，也就是实现 INotifyPropertyChanged：




```
        public class People : NotifyPropertyChangedBase
        {
            private bool _isChecked;

            public bool IsChecked
            {
                get { return _isChecked; }
                set { _isChecked = value; RaisePropertyChanged(); }
            }

            public string Name { get; set; }
        }

        public class NotifyPropertyChangedBase : INotifyPropertyChanged
        {
            public void RaisePropertyChanged([CallerMemberName] string PropertyName = null)
            {
                PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(PropertyName));
            }

            public event PropertyChangedEventHandler PropertyChanged;
        }
```


上面的 NotifyPropertyChangedBase 是一个基类，也就是说我们的其他类，只要继承了这个类，我们的属性按照上面的添加 RaisePropertyChanged 方法，就可以实现和UI界面的交互。


把如图所示的Peoples的List赋值给dg1（这里分两种情况，因为DataGrid的数据源一般是List或者DataTable，我们先讲List）


![](https://img2024.cnblogs.com/blog/2064545/202501/2064545-20250110171814863-1974661235.png)


 当然现在DataGrid我们还没有添加列，所以他还什么都显示不出来，所以我们还要给我们的DataGrid添加列




```
        "False" 
                  CanUserAddRows="False"
                  ItemsSource="{Binding Path=.}"
                  x:Name="dg1" Height="200">
            
                
                
            
        
```


如上面代码所示，添加了两列，一个是选择列，一个是姓名列。这个时候运行项目，会发现这个选择列特别奇怪，要点两次，里面的checkbox才会被选中，所以我们要改造一下这个选择列，我们自己写一个选择列出来。




```
"False" 
  CanUserAddRows="False"
  ItemsSource="{Binding Path=.}"
  x:Name="dg1" Height="200">
    
        
            
                
                    
                
            
        
        "{Binding Name}" Header="Name"/>
    

```


如上图所示，我们把DataGridCheckBoxColumn替换成上面红色的代码部分，也就是重新写一个模板，这个时候运行项目，和原本采用DataGridCheckBoxColumn的效果一样，但是我们现在只需要点击一下按钮就可以选中行了，


为了演示，我们可以自己添加一个TextBlock来清晰的显示我们是选中了哪一行数据


我们在dg1的上面添加一个TextBlock，代码如下




```
Text="{Binding ElementName=dg1, Path=SelectedItem.Name}" VerticalAlignment="Top"/>
```


如图所示，我们通过红色部分的代码进行绑定（如果你清晰的知道自己绑定的对象是个什么类型，我们都可以通过像红色部分代码一样来快捷的绑定，剩下的事情就交给WPF去帮我们做就行了）


运行一下我们的代码，然后切换一下选中行，textblock就会跟着选中行一起变化文字


![](https://img2024.cnblogs.com/blog/2064545/202501/2064545-20250110173426604-1480239972.png)


 然后现在选择功能有了，还需要添加一个全选的功能，我们把选择列的列头改造一下，代码如下：




```
 "False" 
           CanUserAddRows="False"
           ItemsSource="{Binding Path=.}"
           x:Name="dg1" Height="200">
     
         
             
                 
             
             
                 
                     "{Binding IsChecked,Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" HorizontalAlignment="Center"/>
                 
             
         
         "{Binding Name}" Header="Name"/>
     
 
```


上面红色部分的代码是我们新增的代码，添加完以后，运行一下项目，就会发现列头变成了一个选择框，我们就是通过点击这个选择框来实现全选和全不选的功能


关键部分来了，如果我们只是想实现功能的话，就很简单，给这个checkbox添加checked事件和unchecked事件就行了，代码如下：




```
 "Center" VerticalAlignment="Center" Margin="0,0,1,0"
           Checked="CheckBox_Checked"
           Unchecked="CheckBox_Unchecked"/>
```




```
        private void CheckBox_Checked(object sender, RoutedEventArgs e)
        {
            foreach(var people in Peoples)
            {
                people.IsChecked = true;
            }
        }

        private void CheckBox_Unchecked(object sender, RoutedEventArgs e)
        {
            foreach (var people in Peoples)
            {
                people.IsChecked = false;
            }
        }
```


添加完上面代码，我们运行项目，点击全选框，就可以实现全选和全不选，但是这样子一点也不优雅，不能复用，所以我们要改一下。


改这个就需要用到behavior这个东西，这个如果没用过的话，会觉得很不好理解，但是它不是很难，多用就知道怎么用了。


先去nuget上面安装一下依赖包


![](https://img2024.cnblogs.com/blog/2064545/202501/2064545-20250110174548935-340062413.png)


 找到上面这个  Behaviors.WPF,安装一下，然后添加如下代码：




```
    public class DataGridSelectedAllBehavior : Behavior
    {
        protected override void OnAttached()
        {
            AssociatedObject.Checked += AssociatedObject_Checked;
            AssociatedObject.Unchecked += AssociatedObject_Unchecked;
        }

        protected override void OnDetaching()
        {
            AssociatedObject.Checked -= AssociatedObject_Checked;
            AssociatedObject.Unchecked -= AssociatedObject_Unchecked;
        }

        private void AssociatedObject_Unchecked(object sender, RoutedEventArgs e)
        {
            MessageBox.Show("hello unchecked");
        }

        private void AssociatedObject_Checked(object sender, RoutedEventArgs e)
        {
            MessageBox.Show("hello checked");
        }
    }
```




```
"Center" VerticalAlignment="Center" Margin="0,0,1,0">
    
        
    

```


把之前的checked事件和unchecked事件给删掉，改成红色代码部分，添加命名空间：xmlns:i\="http://schemas.microsoft.com/xaml/behaviors"，local是你项目的命名空间，根据项目添加，我的项目是叫wpfapp1，所以我的是：xmlns:local\="clr\-namespace:WpfApp1"


我们再运行一下项目，点击一下选择框，就会出现下面的提示


![](https://img2024.cnblogs.com/blog/2064545/202501/2064545-20250110175321824-2036908354.png)


 如果成功的弹出了上面的提示，就说明到现在，代码都没有问题了，然后就是接着调整代码了。


 我们的 DataGridSelectedAllBehavior是继承的Behavior，这里的Behavior括号里面是一个泛型，因为我们是把这个Behavior附加到CheckBox上的，所以我们就选择CheckBox，如果你想附加别的，比如Button，你就填写Button。


 然后它下面就会有AssociatedObject这个对象，我们附加的是什么东西，这个AssociatedObject就是个什么东西


 然后我们把代码改成如下所示：




```
        private void AssociatedObject_Unchecked(object sender, RoutedEventArgs e)
        {
            var peoples = AssociatedObject.DataContext as List;
            if (peoples != null)
            {
                foreach(var people in peoples)
                {
                    people.IsChecked = false;
                }
            }
        }

        private void AssociatedObject_Checked(object sender, RoutedEventArgs e)
        {
            var peoples = AssociatedObject.DataContext as List;
            if (peoples != null)
            {
                foreach (var people in peoples)
                {
                    people.IsChecked = true;
                }
            }
        }
```


到这里还没完，你运行会发现，我们的AssociatedObject的DataContext是一个null值，所以我们还要修改一下xaml里面的代码




```
"Center" VerticalAlignment="Center" Margin="0,0,1,0"
          DataContext="{Binding RelativeSource={RelativeSource AncestorType=DataGrid}, Path=DataContext}">
    
        
    

```


通过上面红色部分的代码，我们就可以把我的这个Peoples的list传递给我们的AssociatedObject，我们再运行项目，就实现了全选和全不选的功能。


但是到这里还没完，因为People这个对象肯定不能用到项目里面去啊，这个只是一个测试类，项目里面又不是每个类都有  选择  这个属性的，那怎么办


我们通过接口来实现




```
    public class People : NotifyPropertyChangedBase, IModelIsChecked
    {
        private bool _isChecked;

        public bool IsChecked
        {
            get { return _isChecked; }
            set { _isChecked = value; RaisePropertyChanged(); }
        }

        public string Name { get; set; }
    }

    public interface IModelIsChecked
    {
        bool IsChecked { get; set; }
    }
```


添加一个  IModelIsChecked  的interface，然后让我们的People继承它


再修改一下我们behavior的代码




```
        private void AssociatedObject_Unchecked(object sender, RoutedEventArgs e)
        {
            var peoples = AssociatedObject.DataContext as IList;
            if (peoples != null)
            {
                foreach(var people in peoples)
                {
                    var item = people as IModelIsChecked;
                    if (item != null)
                    {
                        item.IsChecked = false;
                    }
                }
            }
        }

        private void AssociatedObject_Checked(object sender, RoutedEventArgs e)
        {
            var peoples = AssociatedObject.DataContext as IList;
            if (peoples != null)
            {
                foreach (var people in peoples)
                {
                    var item = people as IModelIsChecked;
                    if (item != null)
                    {
                        item.IsChecked = true;
                    }
                }
            }
        }
```


这样我们的类就和我们的Behavior解耦了，我们只要后面的类实现了这个IModelIsChecked 的接口，就都能实现全选功能了。


现在还有一种情况，就是如果我们的数据源不是一个List，而是一个DataTable的情况，一个是可以采用把DataTable转化成List的形式然后走上面的逻辑，还有一个就是同样可以修改我们的behavior来实现




```
private void AssociatedObject_Unchecked(object sender, RoutedEventArgs e)
{
    var peoples = AssociatedObject.DataContext as IList;
    if (peoples != null)
    {
        foreach(var people in peoples)
        {
            var item = people as IModelIsChecked;
            if (item != null)
            {
                item.IsChecked = false;
            }
        }
    }

    var dataTable = AssociatedObject.DataContext as DataTable;
    if(dataTable != null)
    {
        foreach(DataRow row in dataTable.Rows)
        {
            row["IsChecked"] = false;
        }
    }
}

private void AssociatedObject_Checked(object sender, RoutedEventArgs e)
{
    var peoples = AssociatedObject.DataContext as IList;
    if (peoples != null)
    {
        foreach (var people in peoples)
        {
            var item = people as IModelIsChecked;
            if (item != null)
            {
                item.IsChecked = true;
            }
        }
    }

    var dataTable = AssociatedObject.DataContext as DataTable;
    if (dataTable != null)
    {
        foreach (DataRow row in dataTable.Rows)
        {
            row["IsChecked"] = true;
        }
    }
}
```




```
            DataTable dt = new DataTable();
            dt.Columns.Add("IsChecked",typeof(bool));
            dt.Columns.Add("Name", typeof(string));
            dt.Rows.Add(false, "Tom");
            dt.Rows.Add(false, "Jerry");
            dg1.DataContext = dt;
```


只要我们的DataTable有名为IsChecked的列就好了


如果没有IsChecked怎么办


![](https://img2024.cnblogs.com/blog/2064545/202501/2064545-20250110182317048-622905500.png)


你知道我要说什么的


这里推荐大家加一下QQ群：332035933   （这里面平时基本上没什么人说话，但是如果有人问问题，都会很积极的回答，java，.net，vue的大佬都有）。


 


 本博客参考[西部世界梯子官网](https://lalami.org)。转载请注明出处！
