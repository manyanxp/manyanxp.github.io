[UP](../index.md)

# **<font color="#006e54">DataGrid</font>**
---
- **<font color="#006e54">AutoGenerateColumns</font>**
<p>自動でカラムを作成するか、しないか設定できる。</p>
</br>

- **<font color="#006e54">SelectionMode</font>**
 <p>DataGridの選択モード。SingleとExtendがある。</p>
</br>

- **<font color="#006e54">DataGridTextColum</font>**
 <p>
テキストベースのColumnを表示することができる。</br>
Width="*"を設定するとColum幅が最大まで広がった状態に設定できる。</br>
</p>
</br>

- **<font color="#006e54">HeadersVisibility</font>**
 <p>ColumHeaderの表示/非表示ができる</p>
</br>

- **<font color="#006e54">SelectionUnit</font>**
<p>
選択の単位を選べる</br>
・DataGridSelectionUnit.Cell・・・セル単位</br>
・DataGridSelectionUnit.CellOrRowHeader・・・セル単位だが、行で選択も可能</br>
・DataGridSelectionUnit.FullRow・・・行単位</br>
</p>

- **<font color="#006e54">DataGrid.InputBindings</font>**
<p>
・MouseBinding・・・マウスの操作でアクションを発生させることが出来る。</br>
・KeyBinding・・・キーボード操作でアクションを発生させることができる。</br>
</p>

```xml
<DataGrid.InputBindings>
    <MouseBinding MouseAction="LeftDoubleClick" Command="{Binding DoubleClickDataSelected}" CommandParameter="{Binding ElementName=dgDataList, Path=SelectedItem}" />
</DataGrid.InputBindings>
```

- **<font color="#006e54">CanUserAddRows</font>**
<p>
新行を追加する、しないの制御ができる。
</p>

## **<font color="#006e54">Sample Code 1</font>**
```xml
<DataGrid x:Name="dgDataList" 
            ItemsSource="{Binding Items}" 
            SelectionMode="Single" 
            AutoGenerateColumns="False" 
            HeadersVisibility="All"
            CanUserAddRows="False"
            SelectionUnit="FullRow">
    <DataGrid.Columns>
        <DataGridTextColumn Header="Colum1" Binding="{Binding Colum1}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum2" Binding="{Binding Colum2}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum3" Binding="{Binding Colum3}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum4" Binding="{Binding Colum4}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum5" Binding="{Binding Colum5}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum6" Binding="{Binding Colum6}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum7" Binding="{Binding Colum8}" IsReadOnly="True" Width="*"/>
        <DataGridTextColumn Header="Colum8" Binding="{Binding Colum7}" IsReadOnly="True" Width="*"/>
    </DataGrid.Columns>
    <DataGrid.InputBindings>
        <MouseBinding MouseAction="LeftDoubleClick" Command="{Binding DoubleClickDataSelected}" CommandParameter="{Binding ElementName=dgDataList, Path=SelectedItem}" />
    </DataGrid.InputBindings>
</DataGrid>
```
- **<font color="#006e54">Sample Code 2</font>**

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
                        <DataGridTextColumn Header="Colum2" Binding="{Binding Colum2}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum3" Binding="{Binding Colum3}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum4" Binding="{Binding Colum4}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum5" Binding="{Binding Colum5}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum6" Binding="{Binding Colum6}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum7" Binding="{Binding Colum8}" IsReadOnly="True" Width="*"/>
                        <DataGridTextColumn Header="Colum8" Binding="{Binding Colum7}" IsReadOnly="True" Width="*"/>
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