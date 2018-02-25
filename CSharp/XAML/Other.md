[UP](../index.md)

# Other

- **<font color="#006e54">キーの同時押し</font>**
```csharp
private void MainWindow_PreviewKeyDown(object sender, KeyEventArgs e)
{
    if (Keyboard.Modifiers == ModifierKeys.Control && e.Key == Key.Tab)
    {
        // 何か処理を描く
    }
}
```

- **<font color="#006e54">RelativeSource</font>**

Reference:[RelativeSource](http://d.hatena.ne.jp/hilapon/20130405/1365143758)

```xml
<TabControl x:Name="test" Margin="10,76,10,10" Grid.Row="2" ItemsSource="{Binding TabItems}" AutomationProperties.AutomationId="CustomerGridView" SelectedIndex="0">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Height="21" Width="100">
                <TextBlock Width="80" Text="{Binding Header}"/>
                <Button Content="X" Command="{Binding DataContext.Close, ElementName=test}" CommandParameter="{Binding ElementName=test, Path=SelectedItem}" Background="Transparent" BorderThickness="0"/>
            </StackPanel>
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <Grid Background="#FFE5E5E5">
                <Grid.RowDefinitions>
                    <RowDefinition Height="auto" />
                    <RowDefinition Height="auto" />
                </Grid.RowDefinitions>
                <WrapPanel>
                    <Button Content="Add Data" HorizontalAlignment="Left" VerticalAlignment="Top" Width="75" 
                            Command="{Binding DataContext.AddData, RelativeSource={RelativeSource AncestorType=Window}}"/>
                    <Button Content="Delete Data" HorizontalAlignment="Left" VerticalAlignment="Top" Width="75" 
                            Command="{Binding DataContext.DelData, RelativeSource={RelativeSource AncestorType=Window}}" 
                            CommandParameter="{Binding ElementName=dgDataList, Path=SelectedItem}"/>
                </WrapPanel>
                <DataGrid Grid.Row="1" x:Name="dgDataList" 
                        ItemsSource="{Binding DataContext.Items, RelativeSource={RelativeSource AncestorType=Window}}" 
                        SelectionMode="Single" 
                        AutoGenerateColumns="False" 
                        HeadersVisibility="All"
                        CanUserAddRows="False"
                        SelectionUnit="FullRow" >
                    <DataGrid.Columns>
                        <DataGridTextColumn Header="Colum1" Binding="{Binding Colum1}" IsReadOnly="True" Width="*"/>
                    </DataGrid.Columns>
                    <DataGrid.InputBindings>
                        <MouseBinding MouseAction="LeftDoubleClick" 
                                        Command="{Binding DataContext.DoubleClickDataSelected, RelativeSource={RelativeSource AncestorType=Window}}" 
                                        CommandParameter="{Binding ElementName=dgDataList, Path=SelectedItem}" />
                    </DataGrid.InputBindings>
                </DataGrid>
            </Grid>
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```