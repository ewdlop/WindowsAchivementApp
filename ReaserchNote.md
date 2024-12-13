```csharp
using System;
using System.Drawing;
using System.Windows.Forms;

namespace TrayAppExample
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            // Create a NotifyIcon to appear in the system tray (right corner)
            NotifyIcon trayIcon = new NotifyIcon
            {
                Icon = SystemIcons.Application, // Replace with your own icon
                Text = "My Background Tray App",
                Visible = true
            };

            // Create a context menu for the tray icon
            ContextMenuStrip trayMenu = new ContextMenuStrip();
            trayMenu.Items.Add("Show Message", null, (sender, e) =>
            {
                MessageBox.Show("This is my background app running from the system tray!");
            });
            trayMenu.Items.Add("Exit", null, (sender, e) => Application.Exit());

            trayIcon.ContextMenuStrip = trayMenu;

            // Keep the application running without showing a form
            Application.Run();

            // When the application ends, ensure the tray icon is removed
            trayIcon.Visible = false;
        }
    }
}
```

```csharp
using System;
using System.Windows.Forms;

namespace BackgroundTrayApp
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            // Create the NotifyIcon (tray icon)
            NotifyIcon trayIcon = new NotifyIcon
            {
                Icon = Properties.Resources.AppIcon, // Ensure you have an icon in your Resources
                Text = "My Background App",
                Visible = true
            };

            // Create a context menu for the tray icon
            ContextMenuStrip trayMenu = new ContextMenuStrip();
            ToolStripMenuItem exitItem = new ToolStripMenuItem("Exit", null, (s, e) => Application.Exit());
            trayMenu.Items.Add(exitItem);
            trayIcon.ContextMenuStrip = trayMenu;

            // Hide the main window (if you have a form, set it not to show)
            Form hiddenForm = new Form
            {
                ShowInTaskbar = false,
                WindowState = FormWindowState.Minimized
            };
            hiddenForm.Load += (s, e) => hiddenForm.Hide();

            // Run an "invisible" application loop
            Application.Run();
            
            // Cleanup when application exits
            trayIcon.Visible = false;
        }
    }
}
```

```csharp
using System;
using System.Runtime.InteropServices;
using System.Windows.Forms;

public class BackgroundKeyListener : Form
{
    private static LowLevelKeyboardProc _hookCallback = HookCallback;
    private static IntPtr _hookID = IntPtr.Zero;

    [STAThread]
    public static void Main()
    {
        _hookID = SetHook(_hookCallback);
        Application.Run(new BackgroundKeyListener());
        UnhookWindowsHookEx(_hookID);
    }

    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        UnhookWindowsHookEx(_hookID);
        base.OnFormClosing(e);
    }

    private static IntPtr SetHook(LowLevelKeyboardProc proc)
    {
        using (var curProcess = System.Diagnostics.Process.GetCurrentProcess())
        using (var curModule = curProcess.MainModule)
        {
            return SetWindowsHookEx(WH_KEYBOARD_LL, proc,
                GetModuleHandle(curModule.ModuleName), 0);
        }
    }

    private delegate IntPtr LowLevelKeyboardProc(
        int nCode, IntPtr wParam, IntPtr lParam);

    private static IntPtr HookCallback(
        int nCode, IntPtr wParam, IntPtr lParam)
    {
        if (nCode >= 0)
        {
            int vkCode = Marshal.ReadInt32(lParam);
            Console.WriteLine("Key Pressed: " + (Keys)vkCode);
        }
        return CallNextHookEx(_hookID, nCode, wParam, lParam);
    }

    private const int WH_KEYBOARD_LL = 13;

    [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    private static extern IntPtr SetWindowsHookEx(int idHook,
        LowLevelKeyboardProc lpfn, IntPtr hMod, uint dwThreadId);

    [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    [return: MarshalAs(UnmanagedType.Bool)]
    private static extern bool UnhookWindowsHookEx(IntPtr hhk);

    [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    private static extern IntPtr CallNextHookEx(IntPtr hhk, int nCode,
        IntPtr wParam, IntPtr lParam);

    [DllImport("kernel32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    private static extern IntPtr GetModuleHandle(string lpModuleName);
}
```

