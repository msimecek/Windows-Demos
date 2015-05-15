# Windows 10 Geolocation Demo

## Zjišťujeme lokaci
Vytvoříme ve Visual Studiu 2015 nový projekt a do **MainPage.xaml** vložíme dvě tlačítka a tři TextBlocky:

```xml
                <StackPanel Margin="10">
                    <Button x:Name="StartTracking" Content="Start" Click="StartTracking_Click" />
                    <Button x:Name="StopTracking" Content="Stop" Click="StopTracking_Click" IsEnabled="False" />
                    <TextBlock x:Name="txtLatitude" />
                    <TextBlock x:Name="txtLongitude" />
                    <TextBlock x:Name="txtAccuracy" />
                </StackPanel>
```

V **MainPage.xaml.cs** doplníme do třídy MainPage privátní člen typu Geolocator:

```csharp
    public sealed partial class MainPage : Page
    {

        private Geolocator geolocator = null;
		
		...
	}
```

Doplníme implementaci kliknutí na tlačítko Start:
```csharp
        private async void StartTracking_Click(object sender, RoutedEventArgs e)
        {
            var accessStatus = await Geolocator.RequestAccessAsync();
            
            if (accessStatus == GeolocationAccessStatus.Allowed)
            {
                geolocator = new Geolocator() { ReportInterval = 2000 };
                geolocator.PositionChanged += Geolocator_PositionChanged;
            } 
        }
```

Geolocator_PositionChanged zatím zůstane neimplementované, nejprve ošetříme tlačítko Stop:
```csharp
        private void StopTracking_Click(object sender, RoutedEventArgs e)
        {
            geolocator.PositionChanged -= Geolocator_PositionChanged;
            geolocator = null;

            StopTracking.IsEnabled = false;
            StartTracking.IsEnabled = true;
        }
```

A potom doplníme **PositionChanged**:
```csharp
        private async void Geolocator_PositionChanged(Geolocator sender, PositionChangedEventArgs args)
        {
            await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
            {
                UpdateLocationData(args.Position);
            });
        }
		
        private void UpdateLocationData(Geoposition position)
        {
            if (position == null)
            {
                txtLatitude.Text = "Nemáme data";
                txtLongitude.Text = "Nemáme data";
                txtAccuracy.Text = "Nemáme data";
            }
            else
            {
                txtLatitude.Text = position.Coordinate.Point.Position.Latitude.ToString();
                txtLongitude.Text = position.Coordinate.Point.Position.Longitude.ToString();
                txtAccuracy.Text = position.Coordinate.Accuracy.ToString();
            }
        }
```

Když teď aplikaci spustíme (**F5**), po kliknutí na tlačítko se nic nestane. Nemáme totiž v manifestu vyžádanou patřičnou Capability. Otevřeme proto ještě soubor **Package.appxmanifest** a doplníme do něj:
```xml
<Package ...>
	...

  <Capabilities>
    <DeviceCapability Name="location" />
  </Capabilities>
</Package>
```

Teď už si aplikace po spuštění vyžádá povolení přístupu k lokaci uživatele.

## Přidáváme adaptivní layout
Dál začneme modifikovat soubor MainPage.xaml.

Nejprve přidáme **SplitView**:
```xml
        <SplitView x:Name="splitView" DisplayMode="Inline" IsPaneOpen="True">
            
        </SplitView>
```

A pak ho začneme postupně plnit:
```xml
        <SplitView x:Name="splitView" DisplayMode="Inline" IsPaneOpen="True">
            <SplitView.Pane>
                
            </SplitView.Pane>
            <SplitView.Content>
                
            </SplitView.Content>
        </SplitView>
```

Přidáme ovládací prvky. StackPanel a jedno tlačítko do části SplitView.Panel a celý StackPanel z dřívějška:
```xml
        <SplitView x:Name="splitView" DisplayMode="Inline" IsPaneOpen="True">
            <SplitView.Pane>
                <StackPanel>
                    <Button Content="Menu1" />
                </StackPanel>
            </SplitView.Pane>
            <SplitView.Content>
                <StackPanel Margin="10">
                    <Button x:Name="StartTracking" Content="Start" Click="StartTracking_Click" />
                    <Button x:Name="StopTracking" Content="Stop" Click="StopTracking_Click" IsEnabled="False" />
                    <TextBlock x:Name="txtLatitude" />
                    <TextBlock x:Name="txtLongitude" />
                    <TextBlock x:Name="txtAccuracy" />
                </StackPanel>
            </SplitView.Content>
        </SplitView>
```

Aby byl view dynamický a reagoval na velikost okna, je potřeba ještě doplnit do hostujícího Gridu VisualStateManager:
```xml
        <VisualStateManager.VisualStateGroups>
            <VisualStateGroup>
                <VisualState x:Name="wide">
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowWidth="600" />
                    </VisualState.StateTriggers>
                    <VisualState.Setters>
                        <Setter Target="splitView.DisplayMode" Value="Inline" />
                        <Setter Target="splitView.IsPaneOpen" Value="True" />
                    </VisualState.Setters>
                </VisualState>
                <VisualState x:Name="narrow">
                    <VisualState.StateTriggers>
                        <AdaptiveTrigger MinWindowWidth="0" />
                    </VisualState.StateTriggers>
                    <VisualState.Setters>
                        <Setter Target="splitView.DisplayMode" Value="Overlay" />
                    </VisualState.Setters>
                </VisualState>
            </VisualStateGroup>
        </VisualStateManager.VisualStateGroups>
```