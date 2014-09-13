Stop-Watch
==========

I have created Stop Watch Windows Phone App

MainPage.xaml

<phone:PhoneApplicationPage
    x:Class="StopWatchExample.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d" d:DesignWidth="480" d:DesignHeight="696"
    FontFamily="{StaticResource PhoneFontFamilyNormal}"
    FontSize="{StaticResource PhoneFontSizeNormal}"
    Foreground="{StaticResource PhoneForegroundBrush}"
    SupportedOrientations="Portrait" Orientation="Portrait"
    shell:SystemTray.IsVisible="True">
 
    <!--LayoutRoot is the root grid where all page content is placed-->
    <Grid x:Name="LayoutRoot" Background="Transparent">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>
 
        <!--TitlePanel contains the name of the application and page title-->
        <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
            <TextBlock x:Name="ApplicationTitle" Text="StopWatchExample" Style="{StaticResource PhoneTextNormalStyle}"/>
        </StackPanel>
 
        <Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
 
            <!-- Stopwatch display -->
            <Grid VerticalAlignment="Center"
                  Margin="25 0">
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto" />
                    <RowDefinition Height="Auto" />
                </Grid.RowDefinitions>
 
                <TextBlock Name="elapsedText"
                           Text="0"
                           Grid.Row="0"
                           FontFamily="Arial"
                           FontSize="{StaticResource PhoneFontSizeExtraLarge}"
                           TextAlignment="Center"
                           Margin="0 0 0 50"/>
 
                <ToggleButton Name="startStopToggle"
                              Content="Start"
                              Grid.Row="1"
                              Checked="OnToggleButtonChecked"
                              Unchecked="OnToggleButtonChecked" />
            </Grid>
 
            <!-- Rectangle to simulate disabling -->
            <Rectangle Name="disableRect"
                       Fill="#80000000"
                       Visibility="Collapsed" />
 
            <!-- "Dialog Box" to select TimeSpan formatting -->
            <Border Name="formatDialog"
                    Background="{StaticResource PhoneChromeBrush}"
                    BorderBrush="{StaticResource PhoneForegroundBrush}"
                    BorderThickness="3"
                    HorizontalAlignment="Center"
                    VerticalAlignment="Center"
                    Visibility="Collapsed">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto" />
                        <RowDefinition Height="Auto" />
                    </Grid.RowDefinitions>
 
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="*" />
                        <ColumnDefinition Width="*" />
                    </Grid.ColumnDefinitions>
 
                    <StackPanel Name="radioButtonPanel"
                                Grid.Row="0"
                                Grid.Column="0"
                                Grid.ColumnSpan="2"
                                HorizontalAlignment="Center">
 
                        <RadioButton Content="Hour/Minute/Seconds"
                                     Tag="HourMinuteSecond" />
 
                        <RadioButton Content="Seconds"
                                     Tag="Seconds" />
 
                        <RadioButton Content="Milliseconds"
                                     Tag="Milliseconds" />
                    </StackPanel>
 
                    <Button Grid.Row="1" Grid.Column="0"
                            Content="ok"
                            Click="OnOkButtonClick" />
 
                    <Button Grid.Row="1" Grid.Column="1"
                            Content="cancel"
                            Click="OnCancelButtonClick" />
                </Grid>
            </Border>
        </Grid>
    </Grid>
 
    <phone:PhoneApplicationPage.ApplicationBar>
        <shell:ApplicationBar>
            <shell:ApplicationBarIconButton IconUri="/Images/appbar.feature.settings.rest.png"
                                            Text="format"
                                            Click="OnAppbarFormatClick" />
 
            <shell:ApplicationBarIconButton IconUri="/Images/appbar.refresh.rest.png"
                                            Text="reset"
                                            Click="OnAppbarResetClick" />
        </shell:ApplicationBar>
    </phone:PhoneApplicationPage.ApplicationBar>
</phone:PhoneApplicationPage>

ElapsedTimeFormat.cs

namespace StopWatchExample
{
    public enum ElapsedTimeFormat
    {
        HourMinuteSecond,
        Seconds,
        Milliseconds
    }
}

App.xaml.cs:

using System;
using System.Collections.Generic;
using System.IO.IsolatedStorage;
using System.Linq;
using System.Net;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Animation;
using System.Windows.Navigation;
using System.Windows.Shapes;
using Microsoft.Phone.Controls;
using Microsoft.Phone.Shell;
 
namespace StopWatchExample
{
    public partial class App : Application
    {
        // Application Setting
        public ElapsedTimeFormat ElapsedTimeFormat { set; get; }
 
        // Easy access to the root frame
        public PhoneApplicationFrame RootFrame { get; private set; }
 
        // Constructor
        public App()
        {
            // Global handler for uncaught exceptions. 
            // Note that exceptions thrown by ApplicationBarItem.Click will not get caught here.
            UnhandledException += Application_UnhandledException;
 
            // Standard Silverlight initialization
            InitializeComponent();
 
            // Phone-specific initialization
            InitializePhoneApplication();
        }
 
        // Code to execute when the application is launching (eg, from Start)
        // This code will not execute when the application is reactivated
        private void Application_Launching(object sender, LaunchingEventArgs e)
        {
            LoadSettings();
        }
 
        // Code to execute when the application is activated (brought to foreground)
        // This code will not execute when the application is first launched
        private void Application_Activated(object sender, ActivatedEventArgs e)
        {
            LoadSettings();
        }
 
        // Code to execute when the application is deactivated (sent to background)
        // This code will not execute when the application is closing
        private void Application_Deactivated(object sender, DeactivatedEventArgs e)
        {
            SaveSettings();
        }
 
        // Code to execute when the application is closing (eg, user hit Back)
        // This code will not execute when the application is deactivated
        private void Application_Closing(object sender, ClosingEventArgs e)
        {
            SaveSettings();
        }
 
        void LoadSettings()
        {
            IsolatedStorageSettings settings = IsolatedStorageSettings.ApplicationSettings;
 
            if (settings.Contains("elapsedTimeFormat"))
                ElapsedTimeFormat = (ElapsedTimeFormat)settings["elapsedTimeFormat"];
            else
                ElapsedTimeFormat = ElapsedTimeFormat.HourMinuteSecond;
        }
 
        void SaveSettings()
        {
            IsolatedStorageSettings settings = IsolatedStorageSettings.ApplicationSettings;
            settings["elapsedTimeFormat"] = ElapsedTimeFormat;
            settings.Save();
        }
 
        // Code to execute if a navigation fails
        void RootFrame_NavigationFailed(object sender, NavigationFailedEventArgs e)
        {
            if (System.Diagnostics.Debugger.IsAttached)
            {
                // A navigation has failed; break into the debugger
                System.Diagnostics.Debugger.Break();
            }
        }
 
        // Code to execute on Unhandled Exceptions
        private void Application_UnhandledException(object sender, ApplicationUnhandledExceptionEventArgs e)
        {
            if (System.Diagnostics.Debugger.IsAttached)
            {
                // An unhandled exception has occurred; break into the debugger
                System.Diagnostics.Debugger.Break();
            }
        }
 
        #region Phone application initialization
 
        // Avoid double-initialization
        private bool phoneApplicationInitialized = false;
 
        // Do not add any additional code to this method
        private void InitializePhoneApplication()
        {
            if (phoneApplicationInitialized)
                return;
 
            // Create the frame but don't set it as RootVisual yet; this allows the splash
            // screen to remain active until the application is ready to render.
            RootFrame = new PhoneApplicationFrame();
            RootFrame.Navigated += CompleteInitializePhoneApplication;
 
            // Handle navigation failures
            RootFrame.NavigationFailed += RootFrame_NavigationFailed;
 
            // Ensure we don't initialize again
            phoneApplicationInitialized = true;
        }
 
        // Do not add any additional code to this method
        private void CompleteInitializePhoneApplication(object sender, NavigationEventArgs e)
        {
            // Set the root visual to allow the application to render
            if (RootVisual != RootFrame)
                RootVisual = RootFrame;
 
            // Remove this handler since it is no longer needed
            RootFrame.Navigated -= CompleteInitializePhoneApplication;
        }
 
        #endregion
    }
}

MainPage.xaml.cs 

using System;
using System.Diagnostics;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Navigation;
using Microsoft.Phone.Controls;
using Microsoft.Phone.Shell;
using System.Globalization;
 
namespace StopWatchExample
{
    public partial class MainPage : PhoneApplicationPage
    {
        Stopwatch stopwatch = new Stopwatch();
        TimeSpan suspensionAdjustment = new TimeSpan();
        string decimalSeparator = NumberFormatInfo.CurrentInfo.NumberDecimalSeparator;
 
        public MainPage()
        {
            InitializeComponent();
            DisplayTime();
        }
 
        void OnToggleButtonChecked(object sender, RoutedEventArgs e)
        {
            if ((bool)startStopToggle.IsChecked)
            {
                stopwatch.Start();
                startStopToggle.Content = "Stop";
                CompositionTarget.Rendering += OnCompositionTargetRendering;
            }
            else
            {
                stopwatch.Stop();
                startStopToggle.Content = "Start";
                CompositionTarget.Rendering -= OnCompositionTargetRendering;
            }
        }
 
        void OnCompositionTargetRendering(object sender, EventArgs args)
        {
            DisplayTime();
        }
 
        void OnAppbarResetClick(object sender, EventArgs args)
        {
            stopwatch.Reset();
            startStopToggle.IsChecked = false;
            suspensionAdjustment = new TimeSpan();
            DisplayTime();
        }
 
        void OnAppbarFormatClick(object sender, EventArgs args)
        {
            disableRect.Visibility = Visibility.Visible;
            formatDialog.Visibility = Visibility.Visible;
 
            // Initialize radio buttons
            ElapsedTimeFormat currentFormat = (Application.Current as App).ElapsedTimeFormat;
 
            foreach (UIElement child in radioButtonPanel.Children)
            {
                RadioButton radio = child as RadioButton;
                ElapsedTimeFormat radioFormat =
                    (ElapsedTimeFormat)Enum.Parse(typeof(ElapsedTimeFormat),
                                                  radio.Tag as string, true);
                radio.IsChecked = currentFormat == radioFormat;
            }
        }
 
        void OnOkButtonClick(object sender, RoutedEventArgs args)
        {
            foreach (UIElement child in radioButtonPanel.Children)
            {
                RadioButton radio = child as RadioButton;
                if ((bool)radio.IsChecked)
                    (Application.Current as App).ElapsedTimeFormat =
                        (ElapsedTimeFormat)Enum.Parse(typeof(ElapsedTimeFormat),
                                                      radio.Tag as string, true);
            }
 
            OnCancelButtonClick(sender, args);
        }
 
        void OnCancelButtonClick(object sender, RoutedEventArgs args)
        {
            disableRect.Visibility = Visibility.Collapsed;
            formatDialog.Visibility = Visibility.Collapsed;
            DisplayTime();
        }
 
        void DisplayTime()
        {
            TimeSpan elapsedTime = stopwatch.Elapsed + suspensionAdjustment;
            string str = null;
 
            switch ((Application.Current as App).ElapsedTimeFormat)
            {
                case ElapsedTimeFormat.HourMinuteSecond:
                    str = String.Format("{0:D2} {1:D2} {2:D2}{3}{4:D2}",
                                        elapsedTime.Hours, elapsedTime.Minutes,
                                        elapsedTime.Seconds, decimalSeparator,
                                        elapsedTime.Milliseconds / 10);
                    break;
 
                case ElapsedTimeFormat.Seconds:
                    str = String.Format("{0:F2} sec", elapsedTime.TotalSeconds);
                    break;
 
                case ElapsedTimeFormat.Milliseconds:
                    str = String.Format("{0:F0} msec", elapsedTime.TotalMilliseconds);
                    break;
            }
 
            elapsedText.Text = str;
        }
 
        protected override void OnNavigatedFrom(NavigationEventArgs args)
        {
            PhoneApplicationService service = PhoneApplicationService.Current;
            service.State["stopWatchRunning"] = (bool)startStopToggle.IsChecked;
            service.State["suspensionAdjustment"] = suspensionAdjustment + stopwatch.Elapsed;
            service.State["tombstoneBeginTime"] = DateTime.Now;
 
            base.OnNavigatedFrom(args);
        }
 
        protected override void OnNavigatedTo(NavigationEventArgs args)
        {
            PhoneApplicationService service = PhoneApplicationService.Current;
 
            if (service.State.ContainsKey("stopWatchRunning"))
            {
                suspensionAdjustment = (TimeSpan)service.State["suspensionAdjustment"];
 
                if ((bool)service.State["stopWatchRunning"])
                {
                    suspensionAdjustment += DateTime.Now -
                                                (DateTime)service.State["tombstoneBeginTime"];
                    startStopToggle.IsChecked = true;
                }
                else
                {
                    DisplayTime();
                }
            }
            base.OnNavigatedTo(args);
        }
    }
}
