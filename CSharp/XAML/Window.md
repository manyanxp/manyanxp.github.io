[UP](../index.md)

# Windos

## Window.ResourcesでViewModelを呼び出す方法

``` xml
<Window x:Class="MyMessengerSample.MainWindow"
      xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
      xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
      xmlns:viewmodel="clr-namespace:ModelSample"
      DataContext="{DynamicResource viewModel}"
      Title="MainWindow" Height="350" Width="525">
    <Window.Resources>
        <viewmodel:ViewModel x:Key="viewModel"/>
    </Window.Resources>
    <Grid>
        <Button Content="Button" Height="23" Name="button" Width="75" Command="{Binding OK}"/>
    </Grid>
</Window>
```