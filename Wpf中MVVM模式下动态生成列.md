# Wpf中MVVM模式下动态生成列

MVVM模式下渲染列表，通常情况下我们会通过已知的属性字段名来指定列名，但如果列名是动态生成的呢？下面提供一种思路。

**MainWindow.xaml**

```c#
<Window x:Class="WpfApplication1.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
        xmlns:WpfApplication1="clr-namespace:WpfApplication1" 
        mc:Ignorable="d" xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
        d:DesignHeight="189" d:DesignWidth="312" Width="300" Height="300">
    <Window.Resources>
        <WpfApplication1:ConfigToDynamicGridViewConverter x:Key="ConfigToDynamicGridViewConverter" />
    </Window.Resources>
    <ListView ItemsSource="{Binding Products}" View="{Binding ColumnConfig, Converter={StaticResource ConfigToDynamicGridViewConverter}}"/>    
</Window>
```

**MainWindow.xaml.cs**

```c#
using System.Collections.Generic;
using System.Windows;

namespace WpfApplication1
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            DataContext = new ViewModel();
        }
    }

    public class ViewModel
    {
        public ColumnConfig ColumnConfig { get; set; }
        public IEnumerable<Product> Products { get; set; }

        public ViewModel()
        {
            Products = new List<Product> { new Product { Name = "Some product", Attributes = "Very cool product" }, new Product { Name = "Other product", Attributes = "Not so cool one" } };
            ColumnConfig = new ColumnConfig { Columns = new List<Column> { new Column { Header = "Name", DataField = "Name" }, new Column { Header = "Attributes", DataField = "Attributes" } } };
        }
    }

    public class ColumnConfig
    {
        public IEnumerable<Column> Columns { get; set; }
    }

    public class Column
    {
        public string Header { get; set; }
        public string DataField { get; set; }
    }

    public class Product
    {
        public string Name { get; set; }
        public string Attributes { get; set; }
    }
}
```

**ConfigToDynamicGridViewConverter.cs**

```c#
using System;
using System.Globalization;
using System.Windows.Controls;
using System.Windows.Data;

namespace WpfApplication1
{
    public class ConfigToDynamicGridViewConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            var config = value as ColumnConfig;
            if (config != null)
            {
                var grdiView = new GridView();
                foreach (var column in config.Columns)
                {
                    var binding = new Binding(column.DataField);
                    grdiView.Columns.Add(new GridViewColumn {Header = column.Header, DisplayMemberBinding = binding});
                }
                return grdiView;
            }
            return Binding.DoNothing;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotSupportedException();
        }
    }
}
```

