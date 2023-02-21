## WPF中应用华为鸿蒙字体

伴随着华为鸿蒙系统的发布，系统内置的字体开放免费商用，最近注意到某些网站如B站已经应用该字体了，着实有被惊艳到。下面介绍如何在WPF程序中应用该字体。

### 关于 HarmonyOS Sans

**华为鸿蒙字体** (HarmonyOS Sans) 是华为和汉仪字库合作定制，专门为鸿蒙操作系统设计打造，设计上聚焦于功能性、普适性，字形和谷歌思源黑体、阿里巴巴普惠体以及 OPPO 手机公司的 OPPO SANS等免费商用字体有点类似，是一款适合阅读的多字重中性字体。字体文件可通过以下链接获取[HarmonyOS 设计资源下载](https://developer.harmonyos.com/cn/design/resource)

### 引入字体文件

通过以上链接下载到字体压缩包HarmonyOS-Sans.zip后，解压出我们所需要的字体文件，这里以HarmonyOS_Sans_SC_Regular.ttf为例。

在WPF项目中创建Resources目录，在Resources目录下进一步创建Font目录，将上一步解压出的字体文件添加至Font目录下，并将该文件属性的Build Action设置为Resource。

编辑**App.xaml**文件，添加该字体资源：

```c#
  <Application.Resources>
    <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary>
                    <FontFamily x:Key="Harmony">pack://application:,,,/Resources/Font/HarmonyOS_Sans_SC_Regular.ttf#HarmonyOS Sans SC Regular</FontFamily>
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
```

至此，鸿蒙字体资源即配置完成，后面在任何控件上设置字体属性为**Harmony**静态资源即可，例如：

```c#
<TextBlock FontFamily="{StaticResource Harmony}" Text="构建万物互联的智能世界" />
```

### 设置默认字体

如果不想逐个窗口逐个控件地设置该字体，而是想在全局应用该字体，则可以在Resources目录下添加资源字典文件：**DefaultSettingsDictionary.xaml**

```c#
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style TargetType="{x:Type TextBlock}">
        <Setter Property="FontFamily" Value="{StaticResource Harmony}" />
        <Setter Property="FontSize" Value="14" />
    </Style>
    <Style TargetType="{x:Type TextElement}">
        <Setter Property="FontFamily" Value="{StaticResource Harmony}" />
        <Setter Property="FontSize" Value="14" />
    </Style>
    <Style TargetType="{x:Type Window}">
        <Setter Property="FontFamily" Value="{StaticResource Harmony}"/>
        <Setter Property="FontSize" Value="14"/>
    </Style>
</ResourceDictionary>
```

然后，再次编辑**App.xaml**文件，声明该资源文件：

```c#
  <Application.Resources>
    <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary>
                    <FontFamily x:Key="Harmony">pack://application:,,,/Resources/Font/HarmonyOS_Sans_SC_Regular.ttf#HarmonyOS Sans SC Regular</FontFamily>
                </ResourceDictionary>
                <ResourceDictionary Source="Resources/DefaultSettingsDictionary.xaml">
                </ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
  </Application.Resources>
```

此时，该WPF应用默认即为鸿蒙字体了，快来体验吧。
