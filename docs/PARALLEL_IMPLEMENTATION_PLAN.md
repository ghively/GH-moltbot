# Parallel Implementation Plan: Windows Desktop App & Linux Server Support

**Document Version:** 1.0
**Last Updated:** 2026-01-28
**Status:** Planning Phase

---

## Part 1: Executive Summary

### Project Scope and Objectives

This document outlines the implementation plan for two major parallel development tracks:

1. **Windows Desktop Application Port:** Migrate the existing macOS desktop companion app to Windows, providing feature parity with the macOS menu bar application.
2. **Linux Server Support:** Implement full Linux server capabilities with systemd integration, native Bonjour/mDNS discovery via Avahi, and comprehensive packaging.

### Project Goals

| Goal | Description | Priority |
|------|-------------|----------|
| **Feature Parity** | Windows app provides equivalent functionality to macOS app | P0 |
| **Linux First-Class** | Linux becomes a primary deployment target for server operations | P0 |
| **Code Reuse** | Maximize shared code between desktop platforms | P1 |
| **Native Experience** | Each platform feels like a native application | P1 |
| **Automated Delivery** | CI/CD pipelines for all platforms | P2 |

### Success Criteria

- Windows desktop app passes all existing macOS test equivalents
- Linux server runs as a systemd service with auto-restart
- Gateway discovery (Bonjour/mDNS) works across all three platforms
- Automated builds produce installable packages for all platforms
- Documentation covers installation and configuration for each platform
- End-to-end tests validate cross-platform compatibility

---

## Part 2: Windows Desktop App Port

### 2.1 Feature Comparison: macOS App to Windows

| macOS Feature | Windows Equivalent | Implementation Priority | API/Technology |
|---------------|-------------------|------------------------|-----------------|
| **Menu Bar App (LSUIElement)** | System Tray Icon | P0 | Windows Notify Icon API |
| **MenuBarExtra (SwiftUI)** | System Tray Context Menu | P0 | Win32 Shell / WPF NotifyIcon |
| **LaunchAgent Autostart** | Registry Run Key / Task Scheduler | P0 | Registry / Task Scheduler |
| **NSStatusItem with Button** | NotifyIcon with Icon | P0 | Shell_NotifyIcon |
| **Popover Panels** | Custom WPF/Avalonia Windows | P0 | WPF / Avalonia UI |
| **WKWebView (Canvas)** | WebView2 (Edge Chromium) | P0 | Microsoft.Web.WebView2 |
| **Custom URL Scheme (moltbot://)** | Registry URL Protocol Handler | P0 | Windows Registry |
| **File Watcher (dispatch_source_t)** | ReadDirectoryChangesW | P0 | Win32 API |
| **WebSocket Gateway** | WebSocket (same Node.js) | P0 | Native WebSocket |
| **Deep Links** | App URI Activation | P0 | Windows.ApplicationModel.Activation |
| **Bonjour Discovery (NWBrowser)** | WS-Discovery / mDNS | P1 | Bonjour for Windows / native mDNS |
| **Camera Capture (AVFoundation)** | Media Foundation / MFPlay | P1 | Windows.Media.Capture |
| **Screen Recording (CGPreflightScreenCaptureAccess)** | Graphics Capture API | P1 | Windows.Graphics.Capture |
| **Notification Permissions** | Toast Notifications | P1 | Windows.UI.Notifications |
| **Microphone Access** | Audio Capture | P1 | Windows.Media.Capture |
| **Location (CoreLocation)** | Windows.Devices.Geolocation | P2 | Windows Geolocation API |
| **Speech Recognition (SFSpeechRecognizer)** | Windows.Media.SpeechRecognition | P2 | UWP Speech APIs |
| **AppleScript Permission** | N/A (not applicable) | N/A | Alternative: PowerShell remoting |
| **Accessibility (AXIsProcessTrusted)** | UI Automation | P2 | Windows UI Automation |
| **Tailscale Service Integration** | Tailscale IPC | P2 | Tailscale Windows API |
| **Code Signing** | Authenticode Signing | P1 | SignTool.exe |
| **Sparkle Updates** | WinSparkle / native | P2 | WinSparkle or custom |
| **Dock Icon (NSApp)** | Taskbar Icon | P0 | Windows Taskbar |

### 2.2 Technology Stack Evaluation

| Option | Pros | Cons | Recommendation |
|--------|-------|-------|----------------|
| **Electron** | + Cross-platform<br>+ Rich ecosystem<br>+ Web tech stack | - Large bundle size<br>- Higher memory usage<br>- Different from Swift codebase | **Backup option** |
| **Tauri** | + Rust backend (fast)<br>+ Small bundle<br>+ WebView2 on Windows | - Different language (Rust vs Swift)<br>- UI rewrite required | Consider for v2 |
| **Avalonia UI** | + Cross-platform XAML-like<br>+ .NET ecosystem | - C# instead of Swift<br>- UI framework differences | Not recommended |
| **WinUI 3 / Windows App SDK** | + Native Windows<br>+ Modern APIs | - Windows-only<br>- Steep learning curve | For Windows-specific features |
| **Flutter** | + Cross-platform<br>+ Good performance | - Dart language<br>- Different paradigm | Not recommended |
| **MAUI (.NET)** | + Microsoft backing<br>+ XAML | - .NET focus<br>- Different from Swift | Not recommended |
| **Swift on Windows (Swift 6)** | + Code reuse<br>+ Same language | - Still evolving<br>- Limited Windows APIs | **Recommended for shared logic** |
| **Hybrid: Swift + Platform Native** | + Maximum code reuse<br>+ Native UI each platform | - More complex build<br>- Platform conditionals | **Recommended approach** |

### 2.3 Recommended Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Moltbot Cross-Platform Architecture            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌──────────────────┐      ┌──────────────────┐                    │
│  │   macOS App      │      │  Windows App     │                    │
│  │  (Swift/SwiftUI) │      │  (C#/WinUI 3)    │                    │
│  └────────┬─────────┘      └────────┬─────────┘                    │
│           │                          │                              │
│           └──────────┬───────────────┘                              │
│                      │                                             │
│           ┌──────────▼─────────────────────┐                        │
│           │   Shared Business Logic        │                        │
│           │   (Swift Package / Module)     │                        │
│           │   - Gateway Protocol           │                        │
│           │   - WebSocket Client           │                        │
│           │   - State Management           │                        │
│           │   - Configuration              │                        │
│           └──────────┬─────────────────────┘                        │
│                      │                                             │
│           ┌──────────▼─────────────────────┐                        │
│           │   Platform Abstraction Layer   │                        │
│           │   (Protocol-based interfaces)  │                        │
│           └───────────────────────────────┘                        │
│                      │                                             │
│           ┌──────────▼─────────────────────┐                        │
│           │   Native Platform Modules      │                        │
│           │   - MacOSPlatformModule        │                        │
│           │   - WindowsPlatformModule      │                        │
│           │   - LinuxPlatformModule        │                        │
│           └───────────────────────────────┘                        │
│                                                                       │
├─────────────────────────────────────────────────────────────────────┤
│                       Gateway (Node.js - Shared)                    │
│  Already platform-agnostic; runs on macOS, Linux, Windows           │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.4 Detailed Feature Porting Plan

#### 2.4.1 Menu Bar UI to System Tray

**macOS Implementation:** `MenuBar.swift` using `MenuBarExtra` (SwiftUI)

**Windows Implementation:**

```csharp
// Windows System Tray Implementation
namespace Moltbot.Windows
{
    public class SystemTrayIcon : IDisposable
    {
        private NotifyIcon _notifyIcon;
        private ContextMenuStrip _contextMenu;

        public SystemTrayIcon(AppState state)
        {
            _notifyIcon = new NotifyIcon()
            {
                Icon = Resources.MoltbotIcon,
                Text = "Moltbot",
                Visible = true
            };

            BuildContextMenu(state);
            _notifyIcon.Click += OnClick;
        }

        private void BuildContextMenu(AppState state)
        {
            _contextMenu = new ContextMenuStrip();

            // Status indicator
            var statusItem = new ToolStripMenuItem($"Status: {GetStatusText(state)}");
            statusItem.Enabled = false;
            _contextMenu.Items.Add(statusItem);

            // Pause/Resume
            var pauseItem = new ToolStripMenuItem(
                state.IsPaused ? "Resume Moltbot" : "Pause Moltbot");
            pauseItem.Click += (s, e) => TogglePause();
            _contextMenu.Items.Add(pauseItem);

            // Separator
            _contextMenu.Items.Add(new ToolStripSeparator());

            // Settings
            _contextMenu.Items.Add(new ToolStripMenuItem("Settings...", null, OpenSettings));

            // Quit
            _contextMenu.Items.Add(new ToolStripMenuItem("Quit", null, Quit));

            _notifyIcon.ContextMenuStrip = _contextMenu;
        }

        private void OnClick(object sender, EventArgs e)
        {
            // Handle left-click to show chat panel
            if (((MouseEventArgs)e).Button == MouseButtons.Left)
            {
                ShowChatPanel();
            }
        }

        public void UpdateIcon(AppIconState iconState, bool isPaused)
        {
            _notifyIcon.Icon = GetIconForState(iconState, isPaused);
        }

        public void Dispose()
        {
            _notifyIcon?.Dispose();
            _contextMenu?.Dispose();
        }
    }
}
```

**Effort Estimate:** 3-5 days

---

#### 2.4.2 LaunchAgent to Windows Service/Task

**macOS Implementation:** `LaunchAgentManager.swift`

**Windows Implementation - Task Scheduler Approach:**

```powershell
# Installation script for Windows Scheduled Task
$TaskName = "MoltbotGateway"
$ScriptPath = "$env:LOCALAPPDATA\Moltbot\gateway-start.cmd"
$WorkingDir = "$env:USERPROFILE"

# Create startup script
@"
@echo off
cd /d "$WorkingDir"
node "$env:LOCALAPPDATA\Moltbot\gateway.mjs"
"@ | Out-File -FilePath $ScriptPath -Encoding ASCII

# Register scheduled task
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c `"$ScriptPath`""
$trigger = New-ScheduledTaskTrigger -AtLogon
$principal = New-ScheduledTaskPrincipal -UserId $env:USERNAME -LogonType Interactive -RunLevel Highest
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable

Register-ScheduledTask -TaskName $TaskName -Action $action -Trigger $trigger -Principal $principal -Settings $settings
```

**Alternative: Windows Service (for headless operation)**

```csharp
// Windows Service implementation
public class MoltbotWindowsService : ServiceBase
{
    private Process _gatewayProcess;
    private readonly string _nodePath;
    private readonly string _gatewayScript;

    public MoltbotWindowsService()
    {
        _nodePath = LocateNodeExecutable();
        _gatewayScript = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
            "Moltbot", "gateway.mjs");
    }

    protected override void OnStart(string[] args)
    {
        var startInfo = new ProcessStartInfo
        {
            FileName = _nodePath,
            Arguments = $"`"{_gatewayScript}\""",
            UseShellExecute = false,
            RedirectStandardOutput = true,
            RedirectStandardError = true,
            CreateNoWindow = true
        };

        _gatewayProcess = Process.Start(startInfo);
    }

    protected override void OnStop()
    {
        _gatewayProcess?.KillAndWait(TimeSpan.FromSeconds(5));
    }

    protected override void OnPause()
    {
        // Send pause signal to gateway
        _gatewayProcess?.StandardInput.WriteLine("{"jsonrpc":"2.0","method":"control.pause"}");
    }
}
```

**Effort Estimate:** 5-7 days

---

#### 2.4.3 Permission Management (TCC to Windows Permissions)

**macOS Implementation:** `PermissionManager.swift`

| macOS Permission | Windows Equivalent | Registry Key / API |
|-----------------|-------------------|-------------------|
| `NSCameraUsageDescription` | Webcam capability | `Capability\{[video device GUID]}\` |
| `NSMicrophoneUsageDescription` | Audio capture | Windows Privacy Settings |
| `NSScreenCaptureDescription` | Screen capture | `GraphicsCapturePicker` |
| `NSAppleEventsUsageDescription` | N/A | Use COM automation instead |
| `NSSpeechRecognitionUsageDescription` | Speech recognition | Windows.Media.SpeechRecognition |
| `NSLocationUsageDescription` | Geolocation | Windows.Devices.Geolocation |
| Notifications | Toast notifications | Windows.UI.Notifications |

**Windows Permission Request Implementation:**

```csharp
public class WindowsPermissionManager
{
    public static async Task<bool> RequestCameraAccess()
    {
        // Windows doesn't have a centralized permission prompt like macOS
        // We need to attempt access and handle denial
        try
        {
            var capture = new MediaCapture();
            await capture.InitializeAsync(new MediaCaptureInitializationSettings
            {
                StreamingCaptureMode = StreamingCaptureMode.Video
            });
            return true;
        }
        catch (UnauthorizedAccessException)
        {
            // Guide user to Settings
            await ShowPermissionDialog("Camera", "ms-settings:privacy-webcam");
            return false;
        }
    }

    public static async Task<bool> RequestMicrophoneAccess()
    {
        try
        {
            var capture = new MediaCapture();
            await capture.InitializeAsync(new MediaCaptureInitializationSettings
            {
                StreamingCaptureMode = StreamingCaptureMode.Audio
            });
            return true;
        }
        catch (UnauthorizedAccessException)
        {
            await ShowPermissionDialog("Microphone", "ms-settings:privacy-microphone");
            return false;
        }
    }

    public static async Task<bool> RequestScreenRecording()
    {
        // Use GraphicsCapturePicker for user-granted permission
        var picker = new GraphicsCapturePicker();
        var item = await picker.PickSingleItemAsync();
        return item != null;
    }

    private static Task ShowPermissionDialog(string permission, string settingsUri)
    {
        return Task.Run(() =>
        {
            MessageBox.Show(
                $"Moltbot needs {permission} permission. Please enable it in Windows Settings.",
                "Permission Required",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information);

            // Open Windows Settings
            Process.Start(new ProcessStartInfo
            {
                FileName = settingsUri,
                UseShellExecute = true
            });
        });
    }
}
```

**Effort Estimate:** 3-4 days

---

#### 2.4.4 Canvas Window Controller

**macOS Implementation:** `CanvasWindowController.swift` using WKWebView

**Windows Implementation using WebView2:**

```csharp
using Microsoft.Web.WebView2.Core;
using Microsoft.Web.WebView2.Wpf;

public class CanvasWindow : Window
{
    private WebView2 _webView;
    private readonly string _sessionKey;
    private readonly string _canvasRoot;

    public CanvasWindow(string sessionKey, string canvasRoot)
    {
        _sessionKey = sessionKey;
        _canvasRoot = canvasRoot;
        InitializeWebView();
    }

    private async void InitializeWebView()
    {
        _webView = new WebView2();
        _webView.WebMessageReceived += OnWebMessageReceived;

        await _webView.EnsureCoreWebView2Async();

        var settings = await _webView.CoreWebView2.Settings;
        settings.AreDefaultContextMenusEnabled = true;
        settings.AreDevToolsEnabled = true;

        // Register custom protocol handler (equivalent to CanvasScheme)
        await _webView.CoreWebView2.AddWebResourceRequestedFilter(
            "moltbot.canvas://*",
            CoreWebView2WebResourceContext.All);

        _webView.CoreWebView2.WebResourceRequested += OnResourceRequested;

        // Inject A2UI bridge script
        await _webView.CoreWebView2.AddScriptToExecuteOnDocumentCreatedAsync(@"
            (function() {
                window.chrome.webview.addEventListener('message', (event) => {
                    const action = event.data;
                    if (action.type === 'a2ui.action') {
                        // Forward to native handler
                        window.chrome.webview.postMessage({
                            type: 'canvas-a2ui',
                            action: action
                        });
                    }
                });
            })();
        ");

        Content = _webView;
    }

    private void OnResourceRequested(object sender, CoreWebView2WebResourceRequestedEventArgs e)
    {
        var uri = new Uri(e.Request.Uri);
        if (uri.Scheme == "moltbot.canvas")
        {
            // Serve local files from canvas root
            var path = Path.Combine(_canvasRoot, uri.AbsolutePath.TrimStart('/'));
            if (File.Exists(path))
            {
                e.Response = _webView.CoreWebView2.Environment.CreateWebResourceResponse(
                    File.OpenRead(path),
                    200,
                    "OK",
                    "Content-Type: " + GetMimeType(path));
            }
        }
    }

    private void OnWebMessageReceived(object sender, CoreWebView2WebMessageReceivedEventArgs e)
    {
        var message = JsonSerializer.Deserialize<CanvasMessage>(e.WebMessageAsJson);
        switch (message.Type)
        {
            case "canvas-a2ui":
                HandleA2UIAction(message.Action);
                break;
        }
    }

    public void LoadCanvas(string path)
    {
        var uri = $"moltbot.canvas://session/{_sessionKey}/{path.TrimStart('/')}";
        _webView.Source = new Uri(uri);
    }
}
```

**Effort Estimate:** 5-7 days

---

#### 2.4.5 Camera Capture Service

**macOS Implementation:** `CameraCaptureService.swift` using AVFoundation

**Windows Implementation:**

```csharp
using MediaCapture = Windows.Media.Capture.MediaCapture;
using VideoEncodingProfile = Windows.Media.MediaProperties.MediaEncodingProfile;

public class WindowsCameraService
{
    private MediaCapture _capture;

    public async Task<List<CameraDevice>> ListDevices()
    {
        var devices = new List<CameraDevice>();

        var allDevices = await DeviceInformation.FindAllAsync(DeviceClass.VideoCapture);
        foreach (var device in allDevices)
        {
            devices.Add(new CameraDevice
            {
                Id = device.Id,
                Name = device.Name,
                EnclosureLocation = device.EnclosureLocation?.Panel ?? CameraLocation.Unknown
            });
        }

        return devices;
    }

    public async Task<byte[]> Snap(CameraFacing facing, int maxWidth, double quality)
    {
        _capture = new MediaCapture();

        var settings = new MediaCaptureInitializationSettings
        {
            StreamingCaptureMode = StreamingCaptureMode.Video
        };

        // Select camera based on facing
        var devices = await ListDevices();
        var selectedDevice = devices.FirstOrDefault(d =>
            facing == CameraFacing.Front ? d.IsFrontFacing : d.IsBackFacing)
            ?? devices.FirstOrDefault();

        if (selectedDevice != null)
        {
            settings.VideoDeviceId = selectedDevice.Id;
        }

        await _capture.InitializeAsync(settings);

        var encodingProfile = VideoEncodingProfile.CreateMp4(VideoEncodingQuality.High);
        encodingProfile.Video.Width = (uint)maxWidth;

        var photoFile = await KnownFolders.PicturesLibrary.CreateFileAsync(
            $"moltbot-{Guid.NewGuid()}.jpg",
            CreationCollisionOption.ReplaceExisting);

        await _capture.CapturePhotoToStorageFileAsync(encodingProfile, photoFile);

        var buffer = await File.ReadAllBytesAsync(photoFile.Path);

        // Cleanup
        await _capture.CaptureManager.StopPreviewAsync();

        return buffer;
    }

    public async Task<string> RecordClip(CameraFacing facing, int durationMs)
    {
        _capture = new MediaCapture();
        await _capture.InitializeAsync(new MediaCaptureInitializationSettings
        {
            StreamingCaptureMode = StreamingCaptureMode.Video
        });

        var file = await KnownFolders.VideosLibrary.CreateFileAsync(
            $"moltbot-{Guid.NewGuid()}.mp4",
            CreationCollisionOption.ReplaceExisting);

        var profile = MediaEncodingProfile.CreateMp4(VideoEncodingQuality.Auto);
        await _capture.StartRecordToStorageFileAsync(profile, file);

        await Task.Delay(durationMs);
        await _capture.StopRecordAsync();

        return file.Path;
    }
}
```

**Effort Estimate:** 4-6 days

---

#### 2.4.6 Exec Approval System

**macOS Implementation:** `ExecApprovals.swift`

The exec approval system is primarily Gateway-based (Node.js), which is platform-agnostic. The desktop app only needs to display approval prompts.

**Windows Implementation:**

```csharp
public class ExecApprovalPromptWindow : Window
{
    public ExecApprovalPromptWindow(ExecApprovalRequest request)
    {
        Title = "Command Execution Approval";
        Width = 600;
        Height = 400;
        WindowStartupLocation = WindowStartupLocation.CenterOwner;

        Content = BuildPromptUI(request);
    }

    private UIElement BuildPromptUI(ExecApprovalRequest request)
    {
        var stack = new StackPanel { Margin = new Thickness(20) };

        // Header
        stack.Children.Add(new TextBlock
        {
            Text = "Moltbot is requesting permission to execute:",
            FontWeight = FontWeights.Bold,
            Margin = new Thickness(0, 0, 0, 10)
        });

        // Command display
        stack.Children.Add(new TextBox
        {
            Text = request.Command,
            IsReadOnly = true,
            Background = Brushes.LightGray,
            Padding = new Thickness(10),
            Margin = new Thickness(0, 0, 0, 10),
            TextWrapping = TextWrapping.Wrap
        });

        // Working directory
        stack.Children.Add(new TextBlock
        {
            Text = $"Working directory: {request.Cwd}",
            Margin = new Thickness(0, 0, 0, 10)
        });

        // Options
        var optionsPanel = new StackPanel { Margin = new Thickness(0, 10) };
        var rememberCheck = new CheckBox
        {
            Content = "Remember this decision for commands matching this pattern"
        };
        optionsPanel.Children.Add(rememberCheck);
        stack.Children.Add(optionsPanel);

        // Buttons
        var buttonPanel = new StackPanel
        {
            Orientation = Orientation.Horizontal,
            HorizontalAlignment = HorizontalAlignment.Right,
            Margin = new Thickness(0, 20, 0, 0)
        };

        var denyBtn = new Button { Content = "Deny", Width = 100, Margin = new Thickness(0, 0, 10, 0) };
        var allowOnceBtn = new Button { Content = "Allow Once", Width = 100, Margin = new Thickness(0, 0, 10, 0) };
        var allowAlwaysBtn = new Button { Content = "Always Allow", Width = 100 };

        denyBtn.Click += (s, e) => Complete(ExecApprovalDecision.Deny, rememberCheck.IsChecked == true);
        allowOnceBtn.Click += (s, e) => Complete(ExecApprovalDecision.AllowOnce, false);
        allowAlwaysBtn.Click += (s, e) => Complete(ExecApprovalDecision.AllowAlways, true);

        buttonPanel.Children.Add(denyBtn);
        buttonPanel.Children.Add(allowOnceBtn);
        buttonPanel.Children.Add(allowAlwaysBtn);
        stack.Children.Add(buttonPanel);

        return stack;
    }

    public Task<ExecApprovalResult> ShowAndWait()
    {
        var tcs = new TaskCompletionSource<ExecApprovalResult>();
        _resultTask = tcs;
        Show();
        return tcs.Task;
    }
}
```

**Effort Estimate:** 2-3 days

---

#### 2.4.7 Gateway Discovery (Bonjour)

**macOS Implementation:** `GatewayDiscoveryModel.swift` using `NWBrowser`

**Windows has two options:**

1. **Bonjour for Windows** (Apple provides a Windows SDK)
2. **Native mDNS implementation** using Windows sockets

**Recommended: Bonjour for Windows SDK**

```csharp
using System.Net;
using System.Net.Sockets;

public class WindowsBonjourDiscovery : IDisposable
{
    private const string BONJOUR_SERVICE_TYPE = "_moltbot-gateway._tcp.";
    private UdpClient _mdnsClient;
    private CancellationTokenSource _cancellationToken;

    public event EventHandler<DiscoveredGateway> GatewayFound;

    public void StartDiscovery()
    {
        _mdnsClient = new UdpClient(new IPEndPoint(IPAddress.Any, 5353));
        _mdnsClient.MulticastLoopback = true;
        _mdnsClient.JoinMulticastGroup(IPAddress.Parse("224.0.0.251"));

        _cancellationToken = new CancellationTokenSource();
        Task.Run(() => DiscoveryLoop(_cancellationToken.Token));
    }

    private async Task DiscoveryLoop(CancellationToken token)
    {
        while (!token.IsCancellationRequested)
        {
            try
            {
                var result = await _mdnsClient.ReceiveAsync();
                var packet = DnsPacket.Parse(result.Buffer);

                foreach (var ptr in packet.Answers.OfType<PointerRecord>())
                {
                    if (ptr.DomainName.EndsWith(BONJOUR_SERVICE_TYPE))
                    {
                        var gateway = ParseGatewayFromRecord(ptr);
                        GatewayFound?.Invoke(this, gateway);
                    }
                }
            }
            catch (SocketException)
            {
                // Timeout, continue
            }
        }
    }

    private DiscoveredGateway ParseGatewayFromRecord(PointerRecord record)
    {
        // Parse TXT records to get gateway info
        // Similar to macOS GatewayDiscoveryModel
        return new DiscoveredGateway
        {
            DisplayName = ExtractDisplayName(record),
            Host = ExtractHost(record),
            Port = ExtractPort(record)
        };
    }

    public void Dispose()
    {
        _cancellationToken?.Cancel();
        _mdnsClient?.Dispose();
    }
}
```

**Alternative: Use Zeroconf NuGet package**

```csharp
using Zeroconf;

public class WindowsZeroconfDiscovery
{
    public async Task<List<DiscoveredGateway>> DiscoverGateways(TimeSpan timeout)
    {
        var domains = new[] { "local.", "tailnet." };
        var results = await ZeroconfResolver.ResolveAsync(
            BONJOUR_SERVICE_TYPE,
            domains,
            scanTime: timeout,
            retryLookup: true);

        return results.Select(service => new DiscoveredGateway
        {
            DisplayName = service.DisplayName,
            Host = service.IPAddress,
            Port = service.Port,
            TxtRecords = service.TxtRecords
        }).ToList();
    }
}
```

**Effort Estimate:** 5-7 days

---

#### 2.4.8 Browser Detection

**macOS Implementation:** Uses `NSWorkspace` and Launch Services

**Windows Implementation:**

```csharp
public class WindowsBrowserDetector
{
    public static string GetDefaultBrowserPath()
    {
        // Method 1: Check UserChoice registry key (Windows 10+)
        var progId = QueryUserChoiceProgId("http");
        if (progId != null)
        {
            var command = QueryCommandForProgId(progId);
            if (command != null)
                return ExtractExecutablePath(command);
        }

        // Method 2: Check file association
        var cmd = QueryCommandForProgId("http");
        if (cmd != null)
            return ExtractExecutablePath(cmd);

        return null;
    }

    private static string QueryUserChoiceProgId(string protocol)
    {
        using var key = Registry.CurrentUser.OpenSubKey(
            $@"Software\Microsoft\Windows\Shell\Associations\UrlAssociations\{protocol}\UserChoice");
        return key?.GetValue("ProgId") as string;
    }

    private static string QueryCommandForProgId(string progId)
    {
        var commandKey = progId.StartsWith("http")
            ? $@"HTTP\shell\open\command"
            : $@"{progId}\shell\open\command";

        using var key = Registry.ClassesRoot.OpenSubKey(commandKey);
        return key?.GetValue(null) as string;
    }

    private static string ExtractExecutablePath(string command)
    {
        // Handle quoted paths: "C:\Path\to\chrome.exe" --args
        var match = Regex.Match(command, @"""([^""]+\.exe)""", RegexOptions.IgnoreCase);
        if (match.Success)
            return match.Groups[1].Value;

        // Handle unquoted
        match = Regex.Match(command, @"([^\s]+\.exe)", RegexOptions.IgnoreCase);
        return match.Success ? match.Groups[1].Value : null;
    }

    public static List<string> GetInstalledBrowsers()
    {
        var browsers = new List<string>();

        // Common browser paths
        var browserPaths = new[]
        {
            @"C:\Program Files\Google\Chrome\Application\chrome.exe",
            @"C:\Program Files\BraveSoftware\Brave-Browser\Application\brave.exe",
            @"C:\Program Files\Microsoft\Edge\Application\msedge.exe",
            @"C:\Program Files\Mozilla Firefox\firefox.exe",
            @"C:\Users\{USER}\AppData\Local\Google\Chrome\Application\chrome.exe",
            @"C:\Users\{USER}\AppData\Local\BraveSoftware\Brave-Browser\Application\brave.exe",
            @"C:\Users\{USER}\AppData\Local\Microsoft\Edge\Application\msedge.exe"
        };

        foreach (var path in browserPaths)
        {
            var expanded = path.Replace("{USER}", Environment.UserName);
            if (File.Exists(expanded))
                browsers.Add(expanded);
        }

        return browsers;
    }
}
```

**Effort Estimate:** 2-3 days

---

#### 2.4.9 Notification Handling

**macOS Implementation:** `UNUserNotificationCenter`

**Windows Implementation:**

```csharp
using Windows.UI.Notifications;
using Windows.Data.Xml.Dom;

public class WindowsNotificationManager
{
    private const string APP_ID = "bot.molt.Moltbot";

    public static void ShowNotification(string title, string message, Dictionary<string, string> data = null)
    {
        var toastXml = ToastNotificationManager.GetTemplateContent(ToastTemplateType.ToastText02);

        var textElements = toastXml.GetElementsByTagName("text");
        textElements[0].AppendChild(toastXml.CreateTextNode(title));
        textElements[1].AppendChild(toastXml.CreateTextNode(message));

        // Add launch args
        var toast = new ToastNotification(toastXml)
        {
            Launch = data != null
                ? $"action=notification&data={JsonSerializer.Serialize(data)}"
                : "action=notification"
        };

        // Add activation handler
        toast.Activated += OnNotificationActivated;

        ToastNotificationManager.CreateToastNotifier(APP_ID).Show(toast);
    }

    private static void OnNotificationActivated(ToastNotification sender, object args)
    {
        // Handle notification click
        var toastArgs = args as ToastActivatedEventArgs;
        if (toastArgs?.Arguments.StartsWith("action=") == true)
        {
            // Parse and handle
            var query = QueryString.Parse toastArgs.Arguments.Substring(7));
            HandleNotificationAction(query);
        }
    }
}
```

**Effort Estimate:** 2-3 days

---

### 2.5 Windows API Equivalents Reference

| macOS API | Windows API | Header/Assembly |
|-----------|-------------|----------------|
| `NSStatusItem` | `Shell_NotifyIcon` / `NotifyIcon` | shell32.dll |
| `NSMenu` | `ContextMenu` / `ContextMenuMenuStrip` | System.Windows.Forms |
| `WKWebView` | `WebView2` | Microsoft.Web.WebView2 |
| `AVFoundation` (Camera) | `MediaCapture` | Windows.Media.Capture |
| `AVSpeechSynthesizer` | `SpeechSynthesizer` | Windows.Media.SpeechSynthesis |
| `SFSpeechRecognizer` | `SpeechRecognizer` | Windows.Media.SpeechRecognition |
| `CGPreflightScreenCaptureAccess()` | `GraphicsCapturePicker` | Windows.Graphics.Capture |
| `CLLocationManager` | `Geolocator` | Windows.Devices.Geolocation |
| `NWBrowser` (Bonjour) | Bonjour SDK / Zeroconf | Apple Bonjour SDK |
| `launchctl` / `LaunchAgent` | Task Scheduler / Service | taskschd.dll / advapi32 |
| `NSWorkspace.open` | `Process.Start` | System.Diagnostics |
| `NSUserNotificationCenter` | `ToastNotificationManager` | Windows.UI.Notifications |
| `NSAppleScript` | PowerShell / COM Automation | System.Management.Automation |
| `SecRandomCopyBytes` | `RandomNumberGenerator` | System.Security.Cryptography |
| `Keychain` | Windows Credential Manager | Windows.Security.Credentials |
| `NSFileManager` | `System.IO.File` / `FileInfo` | System.IO |
| `NSUserDefaults` | Registry / appsettings.json | Microsoft.Win32.Registry |
| `NSProcessInfo` | `Process` / `Environment` | System.Diagnostics |
| `NSEvent` (monitoring) | `KeyboardHook` / `MouseHook` | user32.dll |

---

### 2.6 File Structure and Organization

```
apps/windows/
├── Moltbot.Windows.sln
├── src/
│   ├── Moltbot.Windows/
│   │   ├── App.xaml
│   │   ├── App.xaml.cs
│   │   ├── Models/
│   │   │   ├── AppState.cs
│   │   │   ├── GatewayConnection.cs
│   │   │   └── ConfigStore.cs
│   │   ├── Views/
│   │   │   ├── SystemTray/
│   │   │   │   ├── SystemTrayIcon.cs
│   │   │   │   └── TrayContextMenu.cs
│   │   │   ├── Settings/
│   │   │   │   ├── SettingsWindow.xaml
│   │   │   │   └── SettingsViewModel.cs
│   │   │   ├── Canvas/
│   │   │   │   ├── CanvasWindow.xaml
│   │   │   │   └── CanvasViewModel.cs
│   │   │   └── Notifications/
│   │   │       └── NotificationWindow.xaml
│   │   ├── Services/
│   │   │   ├── Platform/
│   │   │   │   ├── WindowsPlatformService.cs
│   │   │   │   ├── RegistryService.cs
│   │   │   │   └── TaskSchedulerService.cs
│   │   │   ├── Discovery/
│   │   │   │   └── BonjourDiscovery.cs
│   │   │   ├── Media/
│   │   │   │   ├── CameraService.cs
│   │   │   │   └── ScreenCaptureService.cs
│   │   │   └── Permissions/
│   │   │       └── PermissionManager.cs
│   │   ├── Protocols/
│   │   │   └── IPlatformService.cs
│   │   └── Resources/
│   │       ├── Icons/
│   │       └── Strings/
│   ├── Moltbot.Shared/
│   │   └── [Shared business logic - can be Swift or C#]
│   └── Moltbot.Tests/
├── build/
│   ├── Signing/
│   │   └── certificate.pfx
│   └── Installer/
│       └── wix/
│           └── Product.wxs
├── Package.swift
└── README.md
```

---

### 2.7 Development Environment Setup

**Prerequisites:**

1. **Windows 10/11** with latest updates
2. **Visual Studio 2022** (Community Edition or higher)
   - Workloads: .NET desktop development, C++ desktop development
3. **Windows SDK**
   - Windows 10 SDK (10.0.19041.0 or later)
4. **WebView2 Runtime**
   - Included in Windows 10+ or install separately
5. **Node.js** (for Gateway testing)
   - v22.12.0 or later
6. **Git**
7. **WiX Toolset** (for MSI installer creation)

**Setup Commands:**

```powershell
# Clone repository
git clone https://github.com/your-org/moltbot.git
cd moltbot

# Install Node.js dependencies (for Gateway)
pnpm install

# Open Visual Studio solution
start apps\windows\Moltbot.Windows.sln

# Build
msbuild Moltbot.Windows.sln /p:Configuration=Release

# Run (for development)
bin\x64\Debug\net6.0-windows\Moltbot.Windows.exe
```

**Recommended Visual Studio Extensions:**

- WiX Toolset Visual Studio Extension
- ResXManager (for resource management)
- XAML Styler (for code formatting)

---

### 2.8 Build and Packaging Process

**Build Pipeline Steps:**

```yaml
# .github/workflows/build-windows.yml
name: Build Windows Desktop App

on:
  push:
    paths:
      - 'apps/windows/**'
      - 'packages/**'
  pull_request:

jobs:
  build-windows:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install Dependencies
        run: pnpm install

      - name: Build Gateway
        run: pnpm build

      - name: Build Windows App
        run: |
          cd apps/windows
          dotnet build Moltbot.Windows.sln -c Release

      - name: Create Installer
        run: |
          cd apps/windows/build/installer/wix
          candle Product.wxs
          light Product.wixobj -out MoltbotSetup.msi

      - name: Code Signing
        if: github.ref == 'refs/heads/main'
        env:
          CERTIFICATE_BASE64: ${{ secrets.WINDOWS_CERTIFICATE }}
          CERTIFICATE_PASSWORD: ${{ secrets.WINDOWS_CERT_PASSWORD }}
        run: |
          # Convert base64 cert to pfx
          # Sign the MSI
          signtool sign /f certificate.pfx /p $env:CERTIFICATE_PASSWORD MoltbotSetup.msi

      - name: Upload Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: Moltbot-Windows
          path: |
            apps/windows/bin/Release/net8.0-windows/
            apps/windows/build/MoltbotSetup.msi
```

**WiX Installer Configuration:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
  <Product Id="*"
           Name="Moltbot"
           Language="1033"
           Version="1.0.0.0"
           Manufacturer="Moltbot"
           UpgradeCode="YOUR-GUID-HERE">

    <Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />

    <MajorUpgrade DowngradeErrorMessage="A newer version is already installed." />

    <MediaTemplate EmbedCab="yes" />

    <Feature Id="ProductFeature" Title="Moltbot" Level="1">
      <ComponentGroupRef Id="ProductComponents" />
      <ComponentGroupRef Id="GatewayComponents" />
    </Feature>

    <Directory Id="TARGETDIR" Name="SourceDir">
      <Directory Id="ProgramFilesFolder">
        <Directory Id="INSTALLFOLDER" Name="Moltbot" />
      </Directory>

      <Directory Id="ProgramMenuFolder">
        <Directory Id="ApplicationProgramsFolder" Name="Moltbot"/>
      </Directory>

      <Directory Id="StartupFolder" />
    </Directory>

    <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
      <Component Id="MainExecutable" Guid="YOUR-GUID">
        <File Id="MoltbotExe" Source="$(var.Moltbot.ProjectDir)bin\Release\Moltbot.exe" KeyPath="yes">
          <Shortcut Id="DesktopShortcut" Directory="DesktopFolder" Name="Moltbot" Advertise="yes" />
          <Shortcut Id="StartMenuShortcut" Directory="ApplicationProgramsFolder" Name="Moltbot" Advertise="yes" />
          <Shortcut Id="StartupShortcut" Directory="StartupFolder" Name="Moltbot" Advertise="yes" />
        </File>
        <ProgId Id="MoltbotUrlProtocol" Description="Moltbot URL Protocol" Icon="MoltbotExe" IconIndex="0">
          <Extension Id="moltbot" ContentType="application/x-moltbot">
            <Verb Id="open" Command="Open" TargetFile="MoltbotExe" Argument='"%1"' />
          </Extension>
        </ProgId>
      </Component>
    </ComponentGroup>

    <ComponentGroup Id="GatewayComponents" Directory="INSTALLFOLDER">
      <!-- Gateway files (Node.js) -->
      <Component Id="GatewayFiles" Guid="YOUR-GUID">
        <File Source="$(var.Gateway.BuildDir)gateway.mjs" />
        <File Source="$(var.Gateway.BuildDir)node_modules\**\*" />
      </Component>
    </ComponentGroup>
  </Product>
</Wix>
```

---

### 2.9 Testing Strategy

**Unit Tests:**

```csharp
// Tests using xUnit
public class SystemTrayTests
{
    [Fact]
    public void IconState_Paused_ShowsDisabledIcon()
    {
        var state = new AppState { IsPaused = true };
        var icon = new SystemTrayIcon(state);

        Assert.Equal(IconState.Disabled, icon.CurrentState);
    }

    [Fact]
    public void IconState_Working_ShowsAnimatedIcon()
    {
        var state = new AppState { IsWorking = true };
        var icon = new SystemTrayIcon(state);

        Assert.Equal(IconState.Working, icon.CurrentState);
    }
}

public class DiscoveryTests
{
    [Fact]
    public async Task DiscoverLocalGateway_FindsGateway()
    {
        var discovery = new WindowsBonjourDiscovery();
        var gateways = new List<DiscoveredGateway>();

        discovery.GatewayFound += (s, g) => gateways.Add(g);
        discovery.StartDiscovery();

        await Task.Delay(5000); // Wait for discovery

        Assert.NotEmpty(gateways);
    }
}
```

**Integration Tests:**

```csharp
public class GatewayConnectionTests
{
    [Fact]
    public async Task ConnectToLocalGateway_Succeeds()
    {
        var gateway = new GatewayProcessManager();
        await gateway.StartAsync();

        var connection = new GatewayConnection("ws://localhost:18789");
        await connection.ConnectAsync();

        Assert.True(connection.IsConnected);

        await gateway.StopAsync();
    }

    [Fact]
    public async Task SendAgentRequest_ReceivesResponse()
    {
        using var gateway = await TestGateway.StartAsync();
        var connection = new GatewayConnection(gateway.Url);

        var response = await connection.SendAgentAsync("test message");

        Assert.NotNull(response);
        Assert.True(response.Ok);
    }
}
```

**End-to-End Tests:**

```powershell
# PowerShell-based E2E tests
Describe "Moltbot Windows E2E" {
    BeforeAll {
        $ExePath = "bin\Release\net8.0-windows\Moltbot.Windows.exe"
        $Process = Start-Process $ExePath -PassThru
    }

    It "Should launch without crash" {
        $Process.HasExited | Should -Be $false
    }

    It "Should create system tray icon" {
        # Use UI Automation to verify tray icon
        $TrayIcons = Get-Process | Where-Object { $_.MainWindowTitle -eq "Moltbot" }
        $TrayIcons | Should -Not -BeNullOrEmpty
    }

    It "Should connect to local gateway" {
        # Verify WebSocket connection established
        $Config = Get-Content "$env:LOCALAPPDATA\Moltbot\config.json" | ConvertFrom-Json
        $Config.Gateway.Status | Should -Be "connected"
    }

    AfterAll {
        Stop-Process -Id $Process.Id -Force
    }
}
```

---

### 2.10 Effort Estimates Summary

| Component | Estimated Effort | Dependencies | Notes |
|-----------|-----------------|--------------|-------|
| **System Tray Icon** | 3-5 days | None | Core UI element |
| **Task Scheduler / Service** | 5-7 days | None | Autostart functionality |
| **Permission Manager** | 3-4 days | None | Windows security model |
| **Canvas Window (WebView2)** | 5-7 days | WebView2 runtime | Browser integration |
| **Camera Capture** | 4-6 days | MediaCapture API | Hardware integration |
| **Screen Recording** | 3-4 days | GraphicsCapture API | Desktop capture |
| **Exec Approval UI** | 2-3 days | None | Prompt dialogs |
| **Gateway Discovery (Bonjour)** | 5-7 days | Bonjour SDK | Network discovery |
| **Browser Detection** | 2-3 days | None | Registry queries |
| **Notification Handling** | 2-3 days | Toast APIs | User notifications |
| **File Watcher** | 2-3 days | FileSystemWatcher | Canvas file watching |
| **Configuration Storage** | 2-3 days | Registry / JSON | Settings persistence |
| **Code Signing Setup** | 2-3 days | Certificate | Distribution |
| **Installer (WiX)** | 4-5 days | WiX Toolset | Setup.exe creation |
| **Testing & QA** | 10-14 days | All components | Full test suite |
| **Documentation** | 3-5 days | All components | User guides |
| **Total** | **60-92 days** | | ~3-4.5 months |

---

## Part 3: Linux Server Implementation

### 3.1 Current State

The Gateway (`src/gateway/`) already has significant Linux support:
- **Service Management:** `src/daemon/systemd.ts` provides systemd integration
- **Browser Detection:** `src/browser/chrome.executables.ts` has Linux Chrome detection
- **Platform Detection:** `src/daemon/service.ts` resolves platform-specific services

**What's Missing:**
- Full packaging and distribution
- Native Bonjour/mDNS discovery broadcasting (Gateway can discover but not advertise)
- Comprehensive installation scripts
- Systemd linger management for persistent services

### 3.2 systemd Service Management

**Current Implementation:** `src/daemon/systemd.ts`

The systemd support is already implemented. Key files:

```
src/daemon/
├── systemd.ts              # Core systemd integration
├── systemd-unit.ts         # Unit file parsing/rendering
├── systemd-linger.ts       # User linger management
└── service.ts              # Platform service resolution
```

**Sample systemd Unit Template:**

```ini
[Unit]
Description=Moltbot Gateway Service
Documentation=https://github.com/your-org/moltbot
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/node /opt/moltbot/gateway.mjs
WorkingDirectory=%h/.moltbot
Environment=NODE_ENV=production
Environment=CLAWDBOT_CONFIG_PATH=%h/.config/moltbot/config.yaml
Restart=always
RestartSec=5

# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=%h/.moltbot %h/.config/moltbot

[Install]
WantedBy=default.target
```

**Installation Flow:**

```bash
#!/bin/bash
# scripts/install-linux-service.sh

set -e

SERVICE_NAME="moltbot-gateway"
UNIT_PATH="$HOME/.config/systemd/user/$SERVICE_NAME.service"
GATEWAY_BIN="/opt/moltbot/gateway.mjs"
STATE_DIR="$HOME/.moltbot"
CONFIG_DIR="$HOME/.config/moltbot"

# Create directories
mkdir -p "$STATE_DIR" "$CONFIG_DIR"
mkdir -p "$(dirname "$UNIT_PATH")"

# Install gateway
pnpm build
cp dist/gateway.mjs "$GATEWAY_BIN"

# Create systemd unit
cat > "$UNIT_PATH" <<EOF
[Unit]
Description=Moltbot Gateway Service
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=$GATEWAY_BIN
WorkingDirectory=$STATE_DIR
Environment=NODE_ENV=production
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
EOF

# Enable linger for user services
if command -v loginctl &> /dev/null; then
    loginctl enable-linger "$USER"
    echo "Enabled linger for user $USER"
fi

# Reload systemd and start service
systemctl --user daemon-reload
systemctl --user enable "$SERVICE_NAME"
systemctl --user restart "$SERVICE_NAME"

echo "Moltbot Gateway service installed and started!"
echo "Check status with: systemctl --user status $SERVICE_NAME"
```

**Effort Estimate:** 3-5 days (mostly packaging and scripts)

---

### 3.3 Linux Browser Detection

**Current Implementation:** `src/browser/chrome.executables.ts`

Already implements:

```typescript
export function findChromeExecutableLinux(): BrowserExecutable | null {
  const candidates: Array<BrowserExecutable> = [
    { kind: "chrome", path: "/usr/bin/google-chrome" },
    { kind: "chrome", path: "/usr/bin/google-chrome-stable" },
    { kind: "chrome", path: "/usr/bin/chrome" },
    { kind: "brave", path: "/usr/bin/brave-browser" },
    { kind: "brave", path: "/usr/bin/brave" },
    { kind: "brave", path: "/snap/bin/brave" },
    { kind: "edge", path: "/usr/bin/microsoft-edge" },
    { kind: "chromium", path: "/usr/bin/chromium" },
    { kind: "chromium", path: "/snap/bin/chromium" },
  ];
  return findFirstExecutable(candidates);
}

function detectDefaultChromiumExecutableLinux(): BrowserExecutable | null {
  const desktopId =
    execText("xdg-settings", ["get", "default-web-browser"]) ||
    execText("xdg-mime", ["query", "default", "x-scheme-handler/http"]);
  if (!desktopId) return null;

  const desktopPath = findDesktopFilePath(desktopId.trim());
  if (!desktopPath) return null;

  const execLine = readDesktopExecLine(desktopPath);
  if (!execLine) return null;

  const command = extractExecutableFromExecLine(execLine);
  return { kind: inferKindFromExecutableName(command), path: command };
}
```

**Missing Enhancements:**

1. **Flatpak browser detection**
2. **Snap container browser detection**
3. **AppImage browser detection**

**Enhanced Browser Detection:**

```typescript
// src/browser/linux-flatpak.ts
export function findFlatpakBrowsers(): BrowserExecutable[] {
  const results: BrowserExecutable[] = [];

  try {
    const flatpakList = execFileSync("flatpak", ["list", "--columns=application"], {
      encoding: "utf8",
    });

    const browsers = flatpakList.split("\n")
      .filter(line => {
        const id = line.toLowerCase();
        return id.includes("chrome") ||
               id.includes("chromium") ||
               id.includes("brave") ||
               id.includes("edge");
      });

    for (const browser of browsers) {
      const browserPath = `/var/lib/flatpak/app/${browser}/current/active/export/bin/browser`;
      results.push({
        kind: inferKindFromIdentifier(browser),
        path: `flatpak run ${browser}`,
      });
    }
  } catch {
    // Flatpak not available
  }

  return results;
}

// src/browser/linux-snap.ts
export function findSnapBrowsers(): BrowserExecutable[] {
  const results: BrowserExecutable[] = [];

  try {
    const snapList = execFileSync("snap", ["list"], { encoding: "utf8" });
    const lines = snapList.split("\n").slice(1); // Skip header

    for (const line of lines) {
      const parts = line.split(/\s+/);
      if (parts.length < 2) continue;

      const name = parts[0];
      const isBrowser = ["chromium", "brave", "chromium-browser"].includes(name);

      if (isBrowser) {
        results.push({
          kind: inferKindFromExecutableName(name),
          path: `/snap/bin/${name}`,
        });
      }
    }
  } catch {
    // Snap not available
  }

  return results;
}
```

**Effort Estimate:** 3-4 days

---

### 3.4 Avahi Bonjour Discovery

**Current Implementation:** `src/infra/bonjour-discovery.ts`

The Gateway can already discover Bonjour services using `mdns` Node.js package. What's needed is **advertising** the Gateway on Linux.

**Avahi Integration:**

```bash
#!/bin/bash
# scripts/install-avahi-service.sh

AVAHI_SERVICE_DIR="/etc/avahi/services"
SERVICE_FILE="moltbot-gateway.service"

# Avahi service definition
cat > "$SERVICE_FILE" <<EOF
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">Moltbot Gateway on %h</name>
  <service>
    <type>_moltbot-gateway._tcp</type>
    <port>18789</port>
    <txt-record>lanHost=%h</txt-record>
    <txt-record>sshPort=22</txt-record>
  </service>
</service-group>
EOF

# Install system-wide service (requires sudo)
if [ "$EUID" -eq 0 ]; then
  cp "$SERVICE_FILE" "$AVAHI_SERVICE_DIR/"
  systemctl restart avahi-daemon
else
  echo "Installing avahi service requires sudo. Please run:"
  echo "  sudo cp $SERVICE_FILE $AVAHI_SERVICE_DIR/"
  echo "  sudo systemctl restart avahi-daemon"
fi
```

**Node.js mDNS Advertising (Alternative):**

```typescript
// src/gateway/bonjour-advertise.ts
import { createMulticastDNS, MulticastDNS } from "multicast-dns";

export interface GatewayAdvertisementOptions {
  name: string;
  port: number;
  lanHost?: string;
  tailnetDns?: string;
  sshPort?: number;
  cliPath?: string;
  txtRecords?: Record<string, string>;
}

export class GatewayAdvertisement {
  private mdns: MulticastDNS;
  private serviceName: string;

  constructor(private options: GatewayAdvertisementOptions) {
    this.mdns = createMulticastDNS({
      interface: "0.0.0.0",
      multicast: true,
    });
    this.serviceName = `${options.name}._moltbot-gateway._tcp.local`;
  }

  async start(): Promise<void> {
    const txtRecords = this.buildTxtRecords();

    // Respond to mDNS queries
    this.mdns.on("query", (query) => {
      for (const question of query.questions) {
        if (question.name === this.serviceName ||
            question.type === "PTR" && question.name === "_moltbot-gateway._tcp.local") {

          this.mdns.respond({
            answers: [
              {
                name: "_moltbot-gateway._tcp.local",
                type: "PTR",
                data: this.serviceName,
              },
              {
                name: this.serviceName,
                type: "SRV",
                data: {
                  priority: 0,
                  weight: 0,
                  port: this.options.port,
                  target: this.options.lanHost || "localhost",
                },
              },
              {
                name: this.serviceName,
                type: "TXT",
                data: this.encodeTxtRecords(txtRecords),
              },
            ],
          });
        }
      }
    });

    // Announce our presence
    this.mdns.respond({
      answers: [
        {
          name: this.serviceName,
          type: "SRV",
          ttl: 120,
          data: {
            priority: 0,
            weight: 0,
            port: this.options.port,
            target: this.options.lanHost || "localhost",
          },
        },
      ],
    });
  }

  private buildTxtRecords(): Record<string, string> {
    return {
      ...(this.options.lanHost && { lanHost: this.options.lanHost }),
      ...(this.options.tailnetDns && { tailnetDns: this.options.tailnetDns }),
      sshPort: String(this.options.sshPort ?? 22),
      ...(this.options.cliPath && { cliPath: this.options.cliPath }),
      ...this.options.txtRecords,
    };
  }

  private encodeTxtRecords(records: Record<string, string>): Buffer {
    const entries = Object.entries(records)
      .map(([key, value]) => `${key}=${value}`)
      .join("\n");
    return Buffer.from(entries);
  }

  stop(): void {
    this.mdns.destroy();
  }
}
```

**Effort Estimate:** 5-7 days

---

### 3.5 Linux Packaging

**Target Formats:**
- `.deb` (Debian/Ubuntu)
- `.rpm` (Fedora/RHEL/openSUSE)
- `AppImage` (Universal Linux)
- `tar.gz` (Generic)

#### 3.5.1 Debian Package (.deb)

**Directory Structure:**

```
packages/debian/
├── DEBIAN/
│   ├── control          # Package metadata
│   ├── postinst         # Post-install script
│   ├── prerm            # Pre-remove script
│   └── compat           # Debian compatibility version
├── opt/
│   └── moltbot/
│       ├── gateway.mjs  # Gateway executable
│       └── node_modules/ # Dependencies
├── usr/
│   └── share/
│       └── moltbot/
│           └── config.yaml.example
└── etc/
    └── avahi/
        └── services/
            └── moltbot-gateway.service
```

**control file:**

```
Package: moltbot-gateway
Version: 1.0.0
Section: net
Priority: optional
Architecture: all
Depends: nodejs (>= 22.0), avahi-daemon
Maintainer: Moltbot Developers <dev@moltbot.dev>
Description: Moltbot AI Gateway Service
 Moltbot is a modular AI agent platform that integrates with
 various communication channels and provides intelligent automation.
 .
 This package contains the gateway service that runs on Linux systems.

Package: moltbot-desktop
Version: 1.0.0
Section: net
Priority: optional
Architecture: amd64
Depends: moltbot-gateway, libwebkit2gtk-4.1-0, gtk3
Maintainer: Moltbot Developers <dev@moltbot.dev>
Description: Moltbot Desktop Application (Linux)
 Moltbot desktop companion application for Linux systems.
 Provides system tray integration and native gateway management.
```

**postinst script:**

```bash
#!/bin/bash
set -e

case "$1" in
    configure)
        # Enable systemd user service linger
        if command -v loginctl >/dev/null 2>&1; then
            for user in $(ls /home/); do
                if id "$user" >/dev/null 2>&1; then
                    loginctl enable-linger "$user" 2>/dev/null || true
                fi
            done
        fi

        # Restart avahi to pick up our service
        if systemctl is-active avahi-daemon >/dev/null 2>&1; then
            systemctl restart avahi-daemon
        fi
        ;;

    abort-upgrade|abort-remove|abort-deconfigure)
        ;;
esac

exit 0
```

**Build script:**

```bash
#!/bin/bash
# packages/debian/build.sh

set -e

VERSION=${1:-"1.0.0"}
BUILD_DIR="build/moltbot-${VERSION}"
PACKAGE_NAME="moltbot-gateway_${VERSION}_all.deb"

# Clean and create build directory
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR"

# Copy Gateway files
pnpm build
mkdir -p "$BUILD_DIR/opt/moltbot"
cp -r dist/* "$BUILD_DIR/opt/moltbot/"
cp -r node_modules "$BUILD_DIR/opt/moltbot/"

# Create Avahi service file
mkdir -p "$BUILD_DIR/etc/avahi/services"
cat > "$BUILD_DIR/etc/avahi/services/moltbot-gateway.service" <<'EOF'
<?xml version="1.0" standalone='no'?>
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">
<service-group>
  <name replace-wildcards="yes">Moltbot Gateway on %h</name>
  <service>
    <type>_moltbot-gateway._tcp</type>
    <port>18789</port>
  </service>
</service-group>
EOF

# Calculate installed size
INSTALLED_SIZE=$(du -sk "$BUILD_DIR" | cut -f1)

# Create control file
mkdir -p "$BUILD_DIR/DEBIAN"
cat > "$BUILD_DIR/DEBIAN/control" <<EOF
Package: moltbot-gateway
Version: ${VERSION}
Section: net
Priority: optional
Architecture: all
Depends: nodejs (>= 22.0), avahi-daemon
Installed-Size: ${INSTALLED_SIZE}
Maintainer: Moltbot Developers <dev@moltbot.dev>
Description: Moltbot AI Gateway Service
 Moltbot is a modular AI agent platform.
 This package contains the gateway service.
EOF

# Copy control scripts
cp packages/debian/DEBIAN/* "$BUILD_DIR/DEBIAN/"
chmod 755 "$BUILD_DIR/DEBIAN"/*

# Build the package
dpkg-deb --build "$BUILD_DIR" "$PACKAGE_NAME"

echo "Built: $PACKAGE_NAME"
```

#### 3.5.2 RPM Package

**Spec file:**

```
# packages/rpm/moltbot-gateway.spec
Name:           moltbot-gateway
Version:        1.0.0
Release:        1%{?dist}
Summary:        Moltbot AI Gateway Service
License:        MIT
URL:            https://github.com/your-org/moltbot
Source0:        %{name}-%{version}.tar.gz

BuildArch:      noarch
Requires:       nodejs >= 22.0
Requires:       avahi

%description
Moltbot is a modular AI agent platform that integrates with
various communication channels and provides intelligent automation.
This package contains the gateway service that runs on Linux systems.

%prep
%setup -q

%build
# No build needed, pre-compiled

%install
rm -rf %{buildroot}
mkdir -p %{buildroot}/opt/moltbot
mkdir -p %{buildroot}/etc/avahi/services
mkdir -p %{buildroot}/usr/lib/systemd/user

cp -r * %{buildroot}/opt/moltbot/
cp %{SOURCE1} %{buildroot}/etc/avahi/services/moltbot-gateway.service
cp %{SOURCE2} %{buildroot}/usr/lib/systemd/user/moltbot-gateway.service

%post
# Enable systemd linger for all users
getent passwd | cut -d: -f1 | while read user; do
    loginctl enable-linger "$user" 2>/dev/null || true
done

# Restart avahi
if systemctl is-active avahi-daemon >/dev/null 2>&1; then
    systemctl restart avahi-daemon
fi

%files
/opt/moltbot
/etc/avahi/services/moltbot-gateway.service
/usr/lib/systemd/user/moltbot-gateway.service

%changelog
* $(date +'%a %b %d %Y') Moltbot Developers <dev@moltbot.dev> - 1.0.0-1
- Initial package release
```

**Build script:**

```bash
#!/bin/bash
# packages/rpm/build.sh

set -e

VERSION=${1:-"1.0.0"}

# Setup rpmbuild directory
mkdir -p rpmbuild/{SOURCES,SPECS,SRPMS,RPMS,noarch}

# Create source tarball
pnpm build
tar czf moltbot-gateway-${VERSION}.tar.gz \
    -C dist \
    --transform "s,^,moltbot-gateway-${VERSION}/," \
    *

# Copy to SOURCES
cp moltbot-gateway-${VERSION}.tar.gz rpmbuild/SOURCES/

# Build
rpmbuild -ba packages/rpm/moltbot-gateway.spec \
    --define "_topdir $(pwd)/rpmbuild" \
    --define "version ${VERSION}"

echo "Built: rpmbuild/RPMS/noarch/moltbot-gateway-${VERSION}-1.*.rpm"
```

#### 3.5.3 AppImage

**AppImage build script:**

```bash
#!/bin/bash
# packages/appimage/build.sh

set -e

VERSION=${1:-"1.0.0"}
APPDIR="Moltbot.AppDir"

# Clean
rm -rf "$APPDIR" Moltbot*.AppImage

# Create AppDir structure
mkdir -p "$APPDIR"/{usr/bin,usr/lib,usr/share/applications,usr/share/icons/hicolor/256x256/apps}

# Copy Gateway files
pnpm build
cp -r dist/* "$APPDIR/usr/bin/"
cp -r node_modules "$APPDIR/usr/lib/"

# Create AppRun
cat > "$APPDIR/AppRun" <<'EOF'
#!/bin/bash
SELF=$(readlink -f "$0")
HERE=${SELF%/*}
export LD_LIBRARY_PATH="${HERE}/usr/lib:${LD_LIBRARY_PATH}"
export PATH="${HERE}/usr/bin:${PATH}"
exec "${HERE}/usr/bin/gateway.mjs" "$@"
EOF
chmod +x "$APPDIR/AppRun"

# Copy icon
cp resources/moltbot-icon.png "$APPDIR/usr/share/icons/hicolor/256x256/apps/moltbot.png"

# Create desktop file
cat > "$APPDIR/moltbot.desktop" <<EOF
[Desktop Entry]
Name=Moltbot Gateway
Comment=Moltbot AI Gateway Service
Exec=moltbot-gateway
Icon=moltbot
Type=Application
Categories=Network;
EOF

# Download and run appimagetool
wget -c "https://github.com/AppImage/AppImageKit/releases/download/continuous/appimagetool-x86_64.AppImage"
chmod +x appimagetool-x86_64.AppImage

./appimagetool-x86_64.AppImage "$APPDIR" "Moltbot-Gateway-${VERSION}-x86_64.AppImage"

echo "Built: Moltbot-Gateway-${VERSION}-x86_64.AppImage"
```

**Effort Estimate:** 8-10 days

---

### 3.6 Linux Testing Strategy

**Unit Tests:**

```typescript
// src/daemon/systemd.test.ts (existing)
describe("Linux systemd integration", () => {
  describe("resolveSystemdServiceName", () => {
    it("should resolve default service name", () => {
      const name = resolveGatewaySystemdServiceName(undefined);
      expect(name).toBe("moltbot-gateway");
    });

    it("should resolve custom profile service name", () => {
      const name = resolveGatewaySystemdServiceName("production");
      expect(name).toBe("moltbot-gateway@production");
    });
  });

  describe("buildSystemdUnit", () => {
    it("should generate valid unit file", () => {
      const unit = buildSystemdUnit({
        description: "Test Service",
        programArguments: ["/usr/bin/node", "/opt/moltbot/gateway.mjs"],
        workingDirectory: "/home/user/.moltbot",
        environment: { NODE_ENV: "production" },
      });

      expect(unit).toContain("[Unit]");
      expect(unit).toContain("Description=Test Service");
      expect(unit).toContain("ExecStart=/usr/bin/node /opt/moltbot/gateway.mjs");
      expect(unit).toContain("WorkingDirectory=/home/user/.moltbot");
      expect(unit).toContain("Environment=NODE_ENV=production");
    });
  });
});
```

**Integration Tests (Docker):**

```yaml
# tests/linux/docker-compose.test.yml
version: '3.8'

services:
  # Test on Ubuntu
  ubuntu-test:
    image: ubuntu:22.04
    container_name: moltbot-ubuntu-test
    volumes:
      - ../../:/workspace
      - node-modules:/workspace/node_modules
    working_dir: /workspace
    command: bash -c "
      apt-get update &&
      apt-get install -y nodejs npm avahi-daemon systemd &&
      pnpm install &&
      pnpm build &&
      pnpm test
    "

  # Test on Fedora
  fedora-test:
    image: fedora:38
    container_name: moltbot-fedora-test
    volumes:
      - ../../:/workspace
      - node-modules:/workspace/node_modules
    working_dir: /workspace
    command: bash -c "
      dnf install -y nodejs npm avahi systemd &&
      pnpm install &&
      pnpm build &&
      pnpm test
    "

  # Test on Debian
  debian-test:
    image: debian:bookworm
    container_name: moltbot-debian-test
    volumes:
      - ../../:/workspace
      - node-modules:/workspace/node_modules
    working_dir: /workspace
    command: bash -c "
      apt-get update &&
      apt-get install -y nodejs npm avahi-daemon systemd &&
      pnpm install &&
      pnpm build &&
      pnpm test
    "

volumes:
  node-modules:
```

**Package Installation Tests:**

```bash
#!/bin/bash
# tests/linux/test-package-install.sh

set -e

CONTAINER_NAME="moltbot-package-test-$$"

cleanup() {
    docker rm -f "$CONTAINER_NAME" 2>/dev/null || true
}
trap cleanup EXIT

test_deb_package() {
    local package=$1
    local image=$2

    echo "Testing $package on $image"

    docker run --name "$CONTAINER_NAME" \
        -v "$(pwd)/packages/debian/build:/packages" \
        -d "$image" sleep infinity

    # Install dependencies
    docker exec "$CONTAINER_NAME" apt-get update
    docker exec "$CONTAINER_NAME" apt-get install -y curl

    # Install package
    docker exec "$CONTAINER_NAME" dpkg -i "/packages/$package" || \
        docker exec "$CONTAINER_NAME" apt-get install -f -y

    # Verify installation
    docker exec "$CONTAINER_NAME" test -f /opt/moltbot/gateway.mjs
    docker exec "$CONTAINER_NAME" test -f /etc/avahi/services/moltbot-gateway.service

    # Test service
    docker exec "$CONTAINER_NAME" systemctl --user status moltbot-gateway || true

    echo "PASSED: $package on $image"
}

# Run tests
test_deb_package "moltbot-gateway_1.0.0_all.deb" "ubuntu:22.04"
test_deb_package "moltbot-gateway_1.0.0_all.deb" "debian:bookworm"
```

**Effort Estimate:** 5-7 days

---

### 3.7 Linux Effort Estimates Summary

| Component | Estimated Effort | Dependencies | Notes |
|-----------|-----------------|--------------|-------|
| **systemd Service Scripts** | 3-5 days | None | Mostly complete, needs polish |
| **Browser Detection Enhancement** | 3-4 days | None | Flatpak/Snap detection |
| **Avahi Integration** | 5-7 days | avahi-daemon | Service advertising |
| **.deb Packaging** | 3-4 days | dpkg-deb | Debian/Ubuntu support |
| **.rpm Packaging** | 3-4 days | rpmbuild | Fedora/RHEL support |
| **AppImage Packaging** | 2-3 days | appimagetool | Universal binary |
| **Installation Scripts** | 2-3 days | None | User-facing installers |
| **Testing & CI** | 5-7 days | All components | Docker matrix testing |
| **Documentation** | 2-3 days | All components | Linux-specific guides |
| **Total** | **28-40 days** | | ~1.5-2 months |

---

## Part 4: Cross-Platform Shared Code

### 4.1 Architecture for Code Sharing

```
packages/
├── moltbot-protocol/          # Platform-agnostic protocol definitions
│   ├── src/
│   │   ├── gateway.ts        # Gateway WebSocket protocol
│   │   ├── events.ts         # Event definitions
│   │   ├── config.ts         # Configuration schemas
│   │   └── types.ts          # Shared types
│   ├── package.json
│   └── tsconfig.json
│
├── moltbot-gateway/           # Platform-agnostic Gateway (Node.js)
│   ├── src/
│   │   ├── gateway/          # Already cross-platform
│   │   ├── agents/
│   │   ├── channels/
│   │   └── tools/
│   ├── package.json
│   └── tsconfig.json
│
├── moltbot-platform-api/      # Platform abstraction interface
│   ├── src/
│   │   ├── interfaces.ts     # IPlatformService, etc.
│   │   ├── types.ts
│   │   └── protocol.ts
│   ├── package.json
│   └── tsconfig.json
│
└── moltbot-ui/                # Shared UI components (React)
    ├── src/
    │   ├── components/       # Reusable React components
    │   ├── hooks/
    │   └── styles/
    ├── package.json
    └── tsconfig.json
```

### 4.2 Protocol Definitions (Already Shared)

The Gateway protocol is already platform-agnostic:

**WebSocket Protocol:**
```typescript
// packages/moltbot-protocol/src/gateway.ts
export interface GatewayMessage {
  jsonrpc: "2.0";
  id: string | number;
  method: string;
  params?: Record<string, unknown>;
}

export interface GatewayPush {
  type: "snapshot" | "event" | "error";
  data?: unknown;
}

// All platform apps use the same protocol
export class GatewayConnection {
  async send(message: GatewayMessage): Promise<GatewayResponse>;
  onPush(callback: (push: GatewayPush) => void): void;
}
```

### 4.3 Platform Abstraction Layer

Define interfaces that each platform implements:

```typescript
// packages/moltbot-platform-api/src/interfaces.ts

export interface IPlatformService {
  readonly name: "macOS" | "Windows" | "Linux";

  // Service management
  installAutostart(executable: string, args: string[]): Promise<void>;
  uninstallAutostart(): Promise<void>;
  isAutostartEnabled(): Promise<boolean>;

  // Discovery
  startDiscovery(serviceType: string): Observable<DiscoveredService>;

  // Permissions
  requestPermission(permission: PlatformPermission): Promise<PermissionStatus>;
  checkPermission(permission: PlatformPermission): PermissionStatus;

  // Native UI
  showTrayIcon(): void;
  hideTrayIcon(): void;
  updateTrayIcon(state: IconState): void;

  // Native dialogs
  showApprovalPrompt(request: ApprovalRequest): Promise<ApprovalResponse>;

  // Platform-specific
  openSettings(settingsPath: string): void;
}

export enum PlatformPermission {
  Camera = "camera",
  Microphone = "microphone",
  ScreenRecording = "screenRecording",
  Notifications = "notifications",
  Accessibility = "accessibility",
  SpeechRecognition = "speechRecognition",
  Location = "location",
}

export interface DiscoveredService {
  name: string;
  host: string;
  port: number;
  txtRecords: Map<string, string>;
}

export interface ApprovalRequest {
  command: string;
  workingDirectory: string;
  reason?: string;
}

export interface ApprovalResponse {
  decision: "deny" | "allowOnce" | "allowAlways";
  remember?: boolean;
}
```

### 4.4 Shared UI Components

**React components usable across all desktop apps:**

```typescript
// packages/moltbot-ui/src/components/ChatPanel.tsx
export interface ChatPanelProps {
  gatewayUrl: string;
  sessionKey: string;
  onDisconnect?: () => void;
}

export const ChatPanel: React.FC<ChatPanelProps> = (props) => {
  // Same component works in:
  // - macOS (WKWebView injection)
  // - Windows (WebView2 injection)
  // - Linux (WebView2 / WebKitGTK)

  return (
    <div className="moltbot-chat-panel">
      <ChatMessages />
      <ChatInput />
    </div>
  );
};

// Platform-agnostic Canvas API
export interface CanvasAPI {
  load(path: string): void;
  eval(script: string): Promise<string>;
  snapshot(path?: string): Promise<string>;
  onAction(callback: (action: A2UIAction) => void): void;
}
```

### 4.5 Configuration Management

Shared configuration schema (YAML/JSON):

```yaml
# Shared config schema
gateway:
  mode: local | remote
  port: 18789
  bindAddress: "127.0.0.1"

  remote:
    url: "wss://gateway.example.ts.net"
    sshTarget: "user@gateway.example.ts.net"
    sshIdentity: "~/.ssh/id_ed25519"

channels:
  whatsapp:
    enabled: true
  telegram:
    enabled: false

agents:
  default: "pi"

  pi:
    model: "claude-sonnet-4-20250514"
    tools: ["bash", "browser", "filesystem"]

tools:
  bash:
    security: "allowlist"
    ask: "on-miss"

ui:
  accentColor: "#6366f1"
  assistantName: "Moltbot"
  assistantAvatar: "critter"
```

Platform-specific paths:

```typescript
// packages/moltbot-platform-api/src/paths.ts
export const PlatformPaths = {
  macOS: {
    configDir: () => path.join(os.homedir(), ".config", "moltbot"),
    stateDir: () => path.join(os.homedir(), ".local", "state", "moltbot"),
    logsDir: () => path.join(os.homedir(), "Library", "Logs", "moltbot"),
  },
  Windows: {
    configDir: () => path.join(process.env.LOCALAPPDATA || "", "Moltbot"),
    stateDir: () => path.join(process.env.LOCALAPPDATA || "", "Moltbot", "state"),
    logsDir: () => path.join(process.env.LOCALAPPDATA || "", "Moltbot", "logs"),
  },
  Linux: {
    configDir: () => path.join(os.homedir(), ".config", "moltbot"),
    stateDir: () => path.join(os.homedir(), ".local", "state", "moltbot"),
    logsDir: () => path.join(os.homedir(), ".local", "state", "moltbot", "logs"),
  },
};
```

---

## Part 5: Parallel Development Timeline

### 5.1 Phased Approach

```
Phase 1: Foundation (Weeks 1-4)
├── Week 1-2: Project Setup & Platform Abstraction
│   ├── Create Windows project structure
│   ├── Define shared interfaces
│   ├── Set up CI/CD pipeline
│   └── Documentation structure
│
├── Week 3-4: Core Windows Components
│   ├── System tray icon
│   ├── WebSocket gateway client
│   ├── Configuration storage
│   └── Task scheduler autostart

Phase 2: Windows Feature Parity (Weeks 5-12)
├── Week 5-6: Canvas & WebView2
│   ├── Canvas window implementation
│   ├── A2UI bridge
│   └── File watching
│
├── Week 7-8: Permissions & Security
│   ├── Permission manager
│   ├── Camera access
│   ├── Screen recording
│   └── Code signing setup
│
├── Week 9-10: Discovery & Integration
│   ├── Bonjour discovery
│   ├── Browser detection
│   ├── Exec approval UI
│   └── Notifications
│
├── Week 11-12: Polish & Testing
│   ├── End-to-end testing
│   ├── Bug fixes
│   └── Documentation

Phase 3: Linux Server Enhancement (Weeks 13-16)
├── Week 13-14: Service Management
│   ├── systemd polish
│   ├── Installation scripts
│   └── Linger management
│
├── Week 15: Packaging
│   ├── .deb package
│   ├── .rpm package
│   └── AppImage
│
├── Week 16: Discovery & Testing
│   ├── Avahi integration
│   ├── Package testing
│   └── Documentation

Phase 4: Cross-Platform Integration (Weeks 17-20)
├── Week 17-18: Shared Code
│   ├── Protocol standardization
│   ├── UI component sharing
│   └── Configuration sync
│
├── Week 19-20: Final Testing & Release
│   ├── Cross-platform E2E tests
│   ├── Release candidate builds
│   └── Launch preparation
```

### 5.2 Dependencies and Critical Path

```
Critical Path (must be sequential):
1. Platform Abstraction Layer → 2 weeks
2. Windows System Tray → 1 week (depends on #1)
3. Windows Canvas/WebView2 → 2 weeks (depends on #2)
4. Gateway Discovery (all platforms) → 1 week (parallel with #3)
5. Code Signing & Distribution → 1 week (after all features)
6. Final Testing & Release → 2 weeks (after all above)

Parallelizable Tracks:
Track A (Windows UI): ────────────────────────
Track B (Windows Native APIs): ────────────────────────
Track C (Linux Packaging): ───────────────────
Track D (Testing): ───────────────────────────
Track E (Documentation): ───────────────────
```

### 5.3 Milestones

| Milestone | Target | Deliverables |
|-----------|--------|--------------|
| **M1: Foundation** | Week 4 | Project structure, CI/CD, core abstractions |
| **M2: Windows Alpha** | Week 8 | Basic Windows app with system tray and gateway connection |
| **M3: Windows Beta** | Week 12 | Feature parity with macOS, testing |
| **M4: Linux RC** | Week 16 | Complete Linux packaging and installation |
| **M5: Multi-Platform GA** | Week 20 | All platforms tested and documented |

---

## Part 6: Resource Requirements

### 6.1 Team Structure

**Minimum Team (6-8 people):**

| Role | Count | Responsibilities |
|------|-------|------------------|
| **Windows Developer** | 2 | C#/WinUI 3 implementation, native APIs |
| **Linux/Sysadmin** | 1 | systemd, packaging, Avahi integration |
| **macOS Developer** | 1 (existing) | Code sharing, knowledge transfer |
| **Backend Developer** | 1 | Gateway (already cross-platform) |
| **QA Engineer** | 1 | Cross-platform testing |
| **Technical Writer** | 1 | Documentation |
| **DevOps Engineer** | 0.5 | CI/CD, code signing |
| **Tech Lead** | 0.5 | Architecture, coordination |

**Optimal Team (10-12 people):**
- Add 1 dedicated Windows UI/UX designer
- Add 1 automation test engineer
- Full-time DevOps engineer

### 6.2 Skills Needed

**For Windows Development:**
- C# 10+ / .NET 8
- WinUI 3 or WPF
- WebView2
- Windows Registry
- Task Scheduler API
- Windows Security model
- MSI/WiX packaging

**For Linux Development:**
- systemd service management
- Package management (deb/rpm)
- Shell scripting
- Avahi/mDNS
- Container testing (Docker)

**Shared:**
- TypeScript/Node.js
- WebSocket protocol
- Git/GitHub workflows
- CI/CD (GitHub Actions)

### 6.3 Tools and Infrastructure

**Development Tools:**
- Visual Studio 2022 (Windows)
- Xcode 15+ (macOS)
- VS Code (all platforms)
- Node.js 22+
- pnpm

**CI/CD:**
- GitHub Actions
- Code signing certificates (Apple, Microsoft, optionally Linux)
- Notarization services

**Testing:**
- Windows Test VMs (Win 10, Win 11)
- Linux containers (Ubuntu 22.04, Debian 12, Fedora 38)
- macOS Test VMs (Monterey, Ventura, Sonoma)

**Distribution:**
- GitHub Releases
- Chocolatey (Windows)
- Homebrew (macOS/Linux)
- Snap Store (Linux)
- Flathub (Linux)

---

## Part 7: Risk Assessment

### 7.1 Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **WebView2 availability on older Windows** | Medium | Low | Bundle WebView2 runtime with installer; provide fallback |
| **Bonjour discovery inconsistencies** | High | Medium | Implement multiple discovery methods; validate across networks |
| **Code signing certificate delays** | High | Low | Start certificate acquisition early; have backup code signing strategy |
| **systemd differences across distros** | Medium | High | Test on major distros; use container matrix testing |
| **macOS Swift 6 on Windows limitations** | Medium | Medium | Use C# for Windows native with Swift for shared logic |
| **Permission model differences** | High | Medium | Document platform differences; provide clear user guidance |
| **AppCompat issues (antivirus, security software)** | High | Medium | Test with common AV software; provide exclusions guide |

### 7.2 Project Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **Team availability** | High | Medium | Cross-train team members; document extensively |
| **Scope creep** | Medium | High | Strict feature freeze at M2; defer v2 features |
| **Platform-specific bugs** | High | High | Dedicated QA per platform; automated testing |
| **Third-party dependency issues** | Medium | Low | Pin dependency versions; monitor upstream changes |
| **Timeline overruns** | Medium | Medium | Buffer time in estimates; prioritize P0 features |

### 7.3 Mitigation Strategies

**Technical:**
1. **Incremental Releases:** Ship alpha releases early to gather feedback
2. **Feature Flags:** Allow disabling unstable features
3. **Fallback Mechanisms:** Provide degraded functionality when APIs unavailable
4. **Extensive Logging:** Platform-specific telemetry for debugging

**Process:**
1. **Weekly Demos:** Ensure cross-team visibility
2. **Code Reviews:** All platform code reviewed by platform expert
3. **Testing Guild:** Regular testing knowledge-sharing sessions
4. **Documentation First:** Document before implementing

---

## Appendix A: File Reference

### macOS App Key Files

```
apps/macos/Sources/Moltbot/
├── MenuBar.swift                    # System tray entry point
├── AppState.swift                   # Application state management
├── LaunchAgentManager.swift          # Autostart management
├── PermissionManager.swift           # TCC permission handling
├── CameraCaptureService.swift        # Camera/microphone access
├── CanvasWindowController.swift     # Canvas (WebView) host
├── ExecApprovals.swift              # Command approval UI
├── GatewayConnection.swift          # WebSocket gateway client
├── GatewayDiscoveryHelpers.swift    # Bonjour discovery helpers
├── ScreenRecordService.swift        # Screen capture
└── Resources/
    └── Info.plist                   # macOS app metadata
```

### Gateway Platform Files

```
src/daemon/
├── service.ts                       # Platform service resolver
├── launchd.ts                       # macOS LaunchAgent implementation
├── systemd.ts                       # Linux systemd implementation
├── schtasks.ts                      # Windows Task Scheduler implementation
├── systemd-linger.ts               # Linux linger management
└── systemd-unit.ts                 # Unit file parsing

src/browser/
├── chrome.executables.ts            # Cross-platform browser detection
└── chrome.ts                        # Cross-platform Chrome launching
```

---

## Appendix B: API Reference

### Platform Detection

```typescript
// Runtime platform detection
const platform = process.platform; // 'darwin' | 'linux' | 'win32'

// Service resolution (already implemented)
const service = resolveGatewayService(); // Returns platform-specific implementation

// Browser detection (already implemented)
const browser = resolveBrowserExecutableForPlatform(config, process.platform);
```

### WebSocket Gateway Protocol

**Connect:**
```javascript
const ws = new WebSocket('ws://localhost:18789');

// Initial handshake
ws.send(JSON.stringify({
  jsonrpc: "2.0",
  id: 1,
  method: "hello",
  params: {
    token: "your-token",
    version: "1.0"
  }
}));
```

**Send Agent Message:**
```javascript
ws.send(JSON.stringify({
  jsonrpc: "2.0",
  id: 2,
  method: "agent",
  params: {
    message: "Hello, Moltbot!",
    sessionKey: "main",
    thinking: "default"
  }
}));
```

---

## Appendix C: Configuration Reference

### Platform-Specific Paths

| Config Item | macOS | Windows | Linux |
|-------------|-------|---------|-------|
| **User Config** | `~/.config/moltbot/config.yaml` | `%LOCALAPPDATA%\Moltbot\config.yaml` | `~/.config/moltbot/config.yaml` |
| **User State** | `~/.local/state/moltbot/` | `%LOCALAPPDATA%\Moltbot\state\` | `~/.local/state/moltbot/` |
| **Logs** | `~/Library/Logs/moltbot/` | `%LOCALAPPDATA%\Moltbot\logs\` | `~/.local/state/moltbot/logs/` |
| **Cache** | `~/Library/Caches/moltbot/` | `%TEMP%\Moltbot\` | `~/.cache/moltbot/` |
| **Autostart** | `~/Library/LaunchAgents/bot.molt.mac.plist` | `Registry\HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | `~/.config/systemd/user/moltbot-gateway.service` |

---

## Appendix D: Testing Checklist

### Windows Testing Checklist

- [ ] App launches without crash on Windows 10 and 11
- [ ] System tray icon appears and is clickable
- [ ] Left-click opens chat panel
- [ ] Right-click shows context menu
- [ ] Settings window opens correctly
- [ ] Canvas loads (WebView2)
- [ ] Camera capture works (with permission)
- [ ] Screen recording works (with permission)
- [ ] Gateway discovery finds local gateways
- [ ] Exec approval prompts appear
- [ ] Notifications display correctly
- [ ] Autostart works after reboot
- [ ] Uninstaller removes all files and registry entries
- [ ] Code signature validates

### Linux Testing Checklist

- [ ] Gateway runs as systemd user service
- [ ] `systemctl --user start moltbot-gateway` works
- [ ] Gateway starts automatically on login (linger enabled)
- [ ] Avahi advertises gateway on network
- [ ] Gateway can be discovered by macOS app
- [ ] .deb package installs correctly on Debian/Ubuntu
- [ ] .rpm package installs correctly on Fedora/RHEL
- [ ] AppImage runs on multiple distributions
- [ ] Service survives system restart
- [ ] Logs rotate correctly
- [ ] Configuration hot-reloads

---

## Appendix E: Glossary

| Term | Definition |
|------|------------|
| **Gateway** | The central Node.js server that handles WebSocket connections, agent execution, and channel management |
| **Canvas** | The WebView-based UI for displaying agent responses and A2UI components |
| **A2UI** | Agent-to-UI actions - protocol for agents to control the UI |
| **TCC** | macOS Transparency, Consent, and Control system for privacy permissions |
| **WebView2** | Microsoft Edge WebView2 runtime for rendering web content in native apps |
| **systemd** | Linux init system and service manager |
| **Avahi** | Linux implementation of mDNS/Bonjour |
| **Exec Approval** | Security feature requiring user approval before executing commands |
| **Bonjour** | Apple's implementation of Zeroconf/mDNS |
| **NWBrowser** | macOS Network framework API for service discovery |
| **LaunchAgent** | macOS mechanism for launching user agents at login |
| **Task Scheduler** | Windows component for scheduled task execution |
| **WiX** | Windows Installer XML toolset for creating MSI installers |

---

## Document Change History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-28 | Planning | Initial comprehensive plan |

---

**Status:** Ready for Review
**Next Steps:**
1. Review and approve plan
2. Assign team members
3. Set up milestone tracking
4. Begin Phase 1: Foundation
