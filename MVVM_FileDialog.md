# MainWindow.axaml

```
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="using:GetStartedApp.ViewModels"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="400" d:DesignHeight="450"
        x:Class="GetStartedApp.Views.MainWindow"
        x:DataType="vm:MainWindowViewModel"
        Icon="/Assets/avalonia-logo.ico"
        Title="GetStartedApp"
        Width="400" Height="450">

  <Design.DataContext>
    <!-- This only sets the DataContext for the previewer in an IDE,
             to set the actual DataContext for runtime, set the DataContext property in code (look at App.axaml.cs) -->
    <vm:MainWindowViewModel/>
  </Design.DataContext>

  <StackPanel>
    <Grid Margin="5"
          ColumnDefinitions="120, *, 100"
          RowDefinitions="Auto, Auto, Auto">
      <Label Grid.Row="0" Grid.Column="0" Margin="10">Source DB</Label>
      <TextBox Grid.Row="0" Grid.Column="1" Margin="0 5" Text="{Binding SourceDB}" />
      <Button Grid.Row="0" Grid.Column="2" Margin="0 5" Command="{Binding SourceDBClickCommand}">Select...</Button>
      <Label Grid.Row="1" Grid.Column="0" Margin="10">Target DB</Label>
      <TextBox Grid.Row="1" Grid.Column="1" Margin="0 5" Text="{Binding TargetDB}" />
      <Button Grid.Row="1" Grid.Column="2" Margin="0 5" Command="{Binding TargetDBClickCommand}">Select...</Button>
    </Grid>
  </StackPanel>
</Window>
```

# MainWindow.axaml.cs

```
using Avalonia.Controls;

namespace GetStartedApp.Views;

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
    }
}
```

# MainWindowViewModel.cs

```
using System.Linq;
using System.Threading.Tasks;
using Avalonia.Controls;
using Avalonia.Platform.Storage;
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

namespace GetStartedApp.ViewModels;

public partial class MainWindowViewModel : ViewModelBase
{
    private readonly Window _window;

    [ObservableProperty]
    private string _sourceDB = string.Empty;

    [ObservableProperty]
    private string _targetDB = string.Empty;

    public MainWindowViewModel(Window window)
    {
        _window = window;
    }

    [RelayCommand]
    private async Task SourceDBClick()
    {
        var topLevel = TopLevel.GetTopLevel(_window);
        if (topLevel is null) return;

        var files = await topLevel.StorageProvider.OpenFilePickerAsync(new FilePickerOpenOptions
        {
            Title = "Select Source Database",
            AllowMultiple = false,
            FileTypeFilter = [new FilePickerFileType("SQLite") { Patterns = ["*.db"] }]
        });

        if (files.Any())
        {
            SourceDB = files[0].Path.LocalPath;
        }
    }

    [RelayCommand]
    private async Task TargetDBClick()
    {
        var topLevel = TopLevel.GetTopLevel(_window);
        if (topLevel is null) return;

        var file = await topLevel.StorageProvider.SaveFilePickerAsync(new FilePickerSaveOptions
        {
            Title = "Select Target Database",
            ShowOverwritePrompt = true,
            FileTypeChoices = [new FilePickerFileType("SQLite") { Patterns = ["*.db"] }],
            DefaultExtension = "db"
        });

        if (file is not null)
        {
            TargetDB = file.Path.LocalPath;
        }
    }
}

```
